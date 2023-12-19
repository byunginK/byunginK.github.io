---
layout: post
title: "OAuth 프로토콜 흐름 방식"
date: 2023-11-01
categories: [oauth]
---

### **1. OAuth, OAuth2.0 이란?**

위키백과에 따르면 **'OAuth_(Open Authorization)_'**는 인터넷 사용자들이 비밀번호를 제공하지 않고, 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로써 사용되는, 접근 위임을 위한 개방형 표준인데요.

사용자의 아이디와 비밀번호 없이 접근 권한을 위임받을 수 있다는 것은, 로그인 및 개인정보 관리 책임을 **'Third-Party Application_(google, kakao, naver 등)_'**에 위임할 수 있다는 것으로, 로그인 및 개인정보 관리에 대한 책임을 위임하는 것뿐만 아니라, 부여받은 접근 권한을 통해 Third-Party Application이 가지고 있는 사용자의 리소스 조회 등의 기능을 수행할 수 있습니다.

2010년 IETF에서 **'OAuth 1.0'** 공식 표준안이 RFC 5849로 발표되었으며, OAuth의 세션 고정 공격을 보완한 **'OAuth 1.0a'**를 거쳐, 현재는 OAuth의 구조적인 문제점을 해결하고, 핵심 요소만을 차용한 유사 프로토콜 WRAP_(Web Resource Access Protocol)_을 기반으로 발표한 **'OAuth 2.0_(RFC 6749, RFC 6750)_'**가 많이 사용되고 있습니다.

OAuth 2.0은 다양한 클라이언트 환경에 적합한 인증_(Authentication)_ 및 인가_(Authorization)_의 부여_(위임)_ 방법_(Grant Type)_을 제공하고, 그 결과로 클라이언트에 접근 토큰_(Access Token)_을 발급하는 것에 대한 구조_(framework)_로 볼 수 있습니다.

___

### **2. 구성 요소** 

**- Client**

클라이언트라는 명칭 때문에 헷갈릴 수 있지만, 여기서 Client는 Third-Party Application의 자원을 사용하기 위해 접근을 요청하는 Service 또는 Application을 뜻하며, 보통 우리가 개발하려는 서비스가 됩니다.

**- Resource Owner**

리소스의 소유자 또는 사용자를 뜻합니다. Third-Party Application의 보호된 자원에 접근할 수 있는 자격을 부여해 주는 주체이며, OAuth2 프로토콜 흐름에서는 클라이언트를 인증_(Authorize)_하는 역할을 수행합니다.

인증이 완료되면 권한 획득 자격_(Authorization Grant)_을 Client에게 부여합니다.

개념적으로는 Resource Owner가 자격을 부여하는 것이지만, 일반적으로 권한 서버_(Authorization Server)_가 Resource Owner와 Client 사이에서 중개 역할을 수행하게 됩니다.

**- Authorization Server & Resource Server**

Authorization Server는 단어 뜻 그대로 권한을 가진 서버로 Resource Owner를 인증하며, Client의 접근 자격을 확인하고 Access Token을 발급해 주는 역할을 합니다.

Resource Server는 google, kakao, naver 등, 사용자의 보호된 자원_(리소스)_을 가지고 있는 서버를 말합니다.

공식 문서에 따르면 Authorization Server와 Resource Server는 별개로 구분되어 있지만, 구성에 따라 아키텍처가 달라질 수도 있는데요.

Authorization Server와 Resource Server가 동일한 서버일 수도 있고, 별개의 서버일 수도 있으며, 하나의 Authorization Server가 여러 개의 Resource Server에 액세스 토큰을 발급할 수도 있습니다.

___

### **3. 인증 방식 및 동작 과정**

OAuth 2.0에서 Client는 Authorization Server에게 아래 4가지 인증 방식으로 토큰을 요청할 수 있는데요. 각각의 동작 방식 및 과정에 대해서 살펴보겠습니다.

**3-1. Authorization Code Grant / 권한 부여 코드 승인 방식**

권한 부여 코드 요청 시, 자체 생성한 Authorization Code를 전달하는 방식으로 response_type=code, grant_type=authorization_code의 형식으로 요청됩니다.

OAuth2에서 가장 기본이 되는 방식이며, 간편 로그인 기능에서 사용되는 방식입니다.

_(해당 방식에는 Refresh Token 사용이 가능합니다.)_

![](https://blog.kakaocdn.net/dn/bJJErS/btr5e6nEZkM/aRli2fYiLrEDc3scZATlA1/img.jpg)

Authorization Code Grant

1.  권한 부여 코드 요청 시, response_type=code로 요청하게 되면 클라이언트는 권한 서버_(Authorization Server)_에서 제공하는 로그인 페이지를 브라우저에 띄워 출력합니다.
2.  해당 페이지를 통해 사용자가 로그인을 하면 Authorization Server는 권한 부여 코드 요청 시 전달받은 redirect_uri로 Authorization Code를 전달합니다.
3.  Client는 전달받은 Authorization Code를 Authorization Server의 API를 통해 Access Token으로 교환하게 됩니다.

*******

**Access Token을 획득하는 과정에서 Authorization Code 발급 과정이 들어간 이유?**

만약 Authorization Code를 발급받는 과정이 생략되고 Access Token을 바로 발급받는다면, Authorization Server는 Access Token을 전달하기 위한 방법으로 권한 부여 코드 요청 시 전달받은 redirect_uri을 사용할 수밖에 없는데요.

Redirect URI을 통해 데이터를 전달하는 방법은 URI 자체에 데이터를 실어 전달하는 방법 밖에 없으며, 이 방법을 사용하면 중요한 데이터인 Access Token이 브라우저를 통해 바로 노출되게 됩니다.

이러한 문제를 보완하기 위해 Authorization Code 발급 과정이 추가된 것이며, redirect uri로 전달된 Authorization Code는 프런트엔드에서 백엔드로 전달되고, 백엔드는 전달받은 Authorization Code를 가지고 Authorization Server의 API에 요청하여 Access Token을 발급받습니다.

이 과정을 통해 백엔드 사이에서 access token이 전달되기 때문에 액세스 토큰의 탈취 위험이 줄어드는 등, 다른 방식에 비해 보안적으로 안전하다는 특징이 있습니다.

**3-2. Implicit Grant / 암묵적 승인 방식**

자격 증명을 안전하게 저장하기 힘든 클라이언트 사이드에서의 OAuth2 인증에 최적화된 방식으로 response_type=token의 형식으로 요청됩니다.

암묵적 승인 방식은 권한 부여 코드_(Authorization Code)_ 발급 과정 없이 바로 액세스 토큰이 발급되는데요.

access token이 uri를 통해 바로 전달되기 때문에 만료 기간을 짧게 설정해야 한다는 특징이 있습니다.

해당 방식은 refresh token의 사용이 불가능한 방식이며, 이 방식에서 Authorization Server는 client_secret을 사용해 클라이언트를 인증하지 않습니다.

access token을 획득하기 위한 절차가 간소화되기 때문에 응답성과 효율성은 높아지지만, access token이 uri로 전달되는 보안적인 측면에서의 단점이 있습니다.

![](https://blog.kakaocdn.net/dn/tgjgS/btr5fNuAYEs/gwf8HLqsOeuvQqAV1vJ000/img.jpg)

Implicit Grant

1.  권한 부여 승인 요청 시 response_type을 token으로 설정하여 요청하면, 클라이언트는 권한 서버에서 제공하는 로그인 페이지를 브라우저에 띄워 출력합니다.
2.  해당 페이지를 통해 사용자가 로그인을 하면 Authorization Server는 권한 부여 승인 요청 시 전달받은 redirect_uri로 Authorization Code가 아닌 access token을 전달하게 됩니다.

**3-3. Resource Owner Password Credentials Grant / 자원 소유자 자격 증명 방식**

username, password로만 access token을 발급받는 방식으로 grant_type=password의 형태로 요청합니다.

이 방식은 권한 서버, 리소스 서버, 클라이언트가 모두 같은 시스템에 속해 있을 때만 사용할 수 있는 방식이며, 해당 방식에서는 refresh token을 사용할 수 있습니다.

![](https://blog.kakaocdn.net/dn/SVT9t/btr5dSDeHRG/nTkgszrvxHv2626a976M6K/img.jpg)

Resource Owner Password Credentials Grant

username, password를 통해 바로 access token을 발급받는 간단한 로직입니다.

**3-4. Client Credentials Grant / 클라이언트 자격 증명 방식**

클라이언트의 자격 증명만으로 access token을 획득하는 방식으로 grant_type=client_credentials 형식으로 요청합니다.

쉽게 말하면 User가 아닌 Client_(서비스 또는 애플리케이션)_에 대한 인가_(Authorization)_가 필요할 때 사용되는 방식인데요.

OAuth2의 권한 부여 방식 중 가장 간단하며, 클라이언트 자신이 관리하는 리소스 혹은 권한 서버에 해당 클라이언트를 위한 제한된 리소스 접근 권한이 설정되어 있는 경우에 사용됩니다.

이 방식은 자격 증명을 안전하게 보관할 수 있는 클라이언트에서만 사용되어야 하며, refresh token은 사용할 수 없습니다.

![](https://blog.kakaocdn.net/dn/8fXqm/btr5e5ChYGD/cWrCm3oiWsjhgDVPTHFSJ1/img.jpg)

Client Credentials Grant

**< 참고 자료 >**

[https://blog.naver.com/mds_datasecurity/222182943542](https://blog.naver.com/mds_datasecurity/222182943542)

[https://hudi.blog/oauth-2.0/](https://hudi.blog/oauth-2.0/)