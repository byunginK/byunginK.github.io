---
layout: post
title: "Next JS 도커 빌드 및 최적화"
date: 2023-10-09
categories: [docker]
---
# Next.js 프로젝트 docker 배포 + 이미지 크기 줄이기


이번에 Docker를 이용하여 전체 프로젝트의 배포 설정을 구현하기로 결정했다.

현재 프로젝트의 구성은 server / client 두 개의 레포로 나뉘어져 있어서, 각각 Dockerfile을 구성하기로 했다.

Docker로 배포한다고 해서 특별한 것은 없다. dependency package를 설치하고, build 하고, 실행하는 커맨드를 설정해두면 된다.

이에 맞게 다음과 같은 정말 간단한 수준의 파일을 작성했다.

```docker
FROM node:16-alpine
WORKDIR /usr/src/app
COPY package.json ./
RUN yarn
COPY . .
RUN yarn build
CMD [ "yarn", "start" ]
```

하지만 이렇게 해서 docker build를 했을 때, 이미지의 크기가 무려 2GB를 훌쩍 넘어갔다...

이 상태로는 docker pull/push 하는 데도 부담이 크고, 과금도 과금대로 나갈 거라서 크기를 줄일 필요가 있었다.

알아보니 next.js 공식 레포의 example 중에서 next.js 프로젝트를 docker로 빌드하는 상황에서의 모범적인 Dockerfile을 업로드 해두었다!

[https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)

해당 파일을 조금 수정하여 프로젝트에 적용했다.

줄인 후에는 도커 이미지 크기가 184MB로 1/10 이하로 줄어들었다!

Dockerfile은 패키지 설치, 빌드, 실행의 3단계로 진행되는 것은 동일하지만, 빌드 파일 중에서 필요한 부분만을 가져와서 이미지의 크기를 대폭 줄일 수 있었다.

### 0. 기본 의존성들을 설치

```docker
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /usr/src/app

```

프로젝트 전체에서 사용할 node:18-alpine 이미지를 base로 이름 붙인다.

뒤에서는 FROM base AS ___ 로 사용할 수 있다

alpine의 패키지 매니저를 통해 libc6-compat를 설치한다.

alpine 자체가 경량화를 위해 최소한의 라이브러리를 가지고 있다 보니, process.dlopen 을 수행하기 위해서는 libc6-compat 라이브러리를 추가적으로 설치해야 한다고 한다.

[https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine](https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine)

### 1. 노드 패키지 설치

```docker
# Install dependencies based on the preferred package manager
COPY package.json yarn.lock ./
RUN yarn --frozen-lockfile --production;
RUN rm -rf ./.next/cache
```

패키지 매니저 yarn을 통해 노드 패키지들을 설치한다.

이 때 --frozen-lockfile을 통해 yarn.lock에 작성한 그대로 패키지를 설치하도록 작성했다.

이렇게 함으로써 로컬 실행 환경에서 사용한 라이브러리와 정확히 일치하는 형태로 패키지를 설치할 수 있다 (reproduction of builds)

또한 cache에 저장된 것들을 1차적으로 삭제해서 이미지를 경량화 시키도록 구성했다.

### 2. 프로젝트 빌드

```docker
# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /usr/src/app
COPY --from=deps /usr/src/app/node_modules ./node_modules
COPY . .
RUN yarn build
```

이전 단계에서 수행한 설치 내용을 바탕으로 빌드를 프로젝트를 빌드한다.

### 3. 프로젝트 실행

```docker
# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /usr/src/app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /usr/src/app/public ./public
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

이전 단계에서 빌드한 내용을 바탕으로 프로젝트를 실행한다.

이 때 주목해야 할 점은 standalone 디렉토리의 파일을 사용하여 프로젝트를 실행하고 있다는 것이다.

next.config.js에 다음과 같이 설정을 추가하면 standalone에 파일들이 생성된다.

```
module.exports = {
  output: 'standalone',
}
```

이렇게 할 경우 next.js에서 자동으로 프로덕션 배포에 필요한 파일들만 추출해서 독립 실행 가능한 standalone 폴더를 만들어 준다.

이 때 프로젝트를 실행할 수 있는 server.js 파일을 만들어 주기 때문에 이걸로 프로젝트를 실행하면 된다.

(public 폴더와 static 폴더 하위에 있는 내용은 standalone에 포함되지 않으므로, 따로 가지고 와야 한다)

### 전체 파일

```docker
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /usr/src/app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock ./
RUN yarn --frozen-lockfile --production;
RUN rm -rf ./.next/cache

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /usr/src/app
COPY --from=deps /usr/src/app/node_modules ./node_modules
COPY . .
RUN yarn build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /usr/src/app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /usr/src/app/public ./public
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

◎ npm 버전

```docker
# Install dependencies only when needed
FROM node:alpine AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app
# COPY package.json yarn.lock ./
COPY package.json package-lock.json ./
# RUN yarn install --frozen-lockfile
RUN npm ci

# Rebuild the source code only when needed
FROM node:alpine AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
# RUN yarn build && yarn install --production --ignore-scripts --prefer-offline
RUN npm run build && npm install --production --ignore-scripts --prefer-offline

# Production image, copy all the files and run next
FROM node:alpine AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# You only need to copy next.config.js if you are NOT using the default configuration
# COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

USER nextjs

EXPOSE 3000

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry.
ENV NEXT_TELEMETRY_DISABLED 1

# CMD ["yarn", "start"]
CMD ["npm", "run", "start"]
```