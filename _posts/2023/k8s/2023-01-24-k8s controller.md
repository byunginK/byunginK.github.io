---
layout: post
title: "kubernetes 기초 2 (controller)"
date: 2023-01-24
categories: [k8s]
---

# ReplicationController 란?

요구하는 Pod의 개수를 보장하며 파드 집합의 실행을 항상 안정적으로 유지하도록 함.

#### 기본구성

- selector (key:value로 구성)
- relicas
- template (Pod 컨테이너 템플릿)

##### ReplicationController yaml 예제

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
```

위 예제에서 template아래의 내용은 기존 Pod 구성 yaml의 내용이다. selector의 key value는 pod 템플릿의 metadata의 labels에 있는 값으로 사용된다. (labels 필수로 있어야한다.)<br/>

`kubectl edit rc rc-nginx`을 하게되면 vi가 실행되면서 yaml 템플릿이나온다. 해당 yaml을 수정할 수 있다.
scale관련 명령어를 통해서도 replicas를 변경할 수 있다. `kubectl scale rc rc-nginx --replicas=2`<br/>

만약 pod의 컨테이너 이미지의 버전을 변경하게 되면, 바로 가동중인 pod는 업데이트되지않는다. 다만 pod를 지우거나 새로 생성되는 pod는 변경된 버전의 컨테이너가 된다. rolling-update 속성을 가지며, 무중단 배포가 가능하게 된다. (자동 update는 아니다.)

# ReplicaSet

ReplicationController과 성격은 동일. 차이점은 풍부한 selector를 지원.
matchLabels, matchExpressions 연산자를 사용

### matchExpressions operator

- In : Key와 values를 지정하여 key, value가 일치하는 pod만 연결
- NotIn : Key는 일치하고 value는 일치하지 않는 pod에 연결
- Exists : Key에 맞는 label의 pod를 연결
- DoesNotExist : key와 다른 label의 pod를 연결

##### ReplicaSet yaml 예제

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLables:
      app: webui
    matchExpressions:
      - { key: ver, operator: In, value: ["1.14", "1.15"] }
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
```

`kubectl delete rs rs-nginx --cascade=false` 명령어 에서 cascade를 false로 하게되면 ReplicaSet 컨트롤은 삭제되지만 pod는 유지된다.

# Deployment

rolling update & rolling back을 목적으로 만들어졌다. ReplicaSet의 부모 컨트롤러이며, rolling update, rolling back 기능을 제외하고 동일하다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
  selector:
    matchLables:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
```

### rolling update란?

pod의 버전 업데이트를 무중단 배포를 하는 것을 의미한다. 지속성 가능.

### rolling update

```
kubectl set image deployment <deploy_name> <container_name>=<new_version_image> -- record
```

rolling update를 할 수 있다. (record는 update내용을 기록)<br/>

```
kubectl set image deployment deploy-nginx nginx-container=nginx:1.15 --record
```

record 명령어를 했기 때문에 `kubectl rollout status deployment deploy-nginx`을 통해 update 기록을 볼 수 있다.
`kubectl rollout [pause/resume] deployment deploy-nginx` puase 또는 resume을 통해 업데이트를 일시정지 재개를 할 수 있다.

### Deployment rolling update 커스텀

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  progressDeadlineSeconds: 600
  revisionHistoryLimit: 10 ---- 업데이트 record기록을 얼마만큼 보관하고 있을지 설정
  strategy:
    rollingUpdate:
      maxSurge: 25% ----- 최대 업데이트도중 운영할 수 있는 pod 설정. replicas개수에서 퍼센트로 나온 갯수 더함
      maxUnavailable: 25%
    type: RollingUpdate
  replicas: 3
  selector:
    matchLables:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
```

### rolling back

- `kubectl rollout undo deployment deploy-nginx` 바로 이전의 업데이트하였던, 버전으로 돌아간다.
- `kubectl rollout undo deployment deploy-nginx --to-revision=3` 히스토리에서 3번 revision에 해당하는 버전으로 롤백된다.

### Deployment 어노테이션

어노테이션을 통해 history에서 좀 더 간결하게 표현도 가능

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
  annotations:
    kubernetes.io/change-cause: version 1.14
spec:
  progressDeadlineSeconds: 600
  revisionHistoryLimit: 10
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  replicas: 3
  selector:
    matchLables:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
```

yaml파일로 rolling update 하는 방법이다. <br/>
위의 annotations 부분의 버전을 올리고, 아래 spec의 image에서 버전을 올리고 `kubectl apply -f <yaml 파일명>` 실행
