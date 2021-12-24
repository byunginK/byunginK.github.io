---
layout: post
title:  "Docker install"
date:   2021-12-24
categories: [web]
---
# 도커 설치

## 리눅스에 설치

```cmd
sudo -i
패스워드 입력 후
apt install docker.io
```
이후 필요한 이미지 `pull` 하여 사용 자세한 내용은 추가 post 기재 예정

## window에 설치

1. 도커사용을 위해 wsl2을 사용해야한다. (wsl2의 자세한 내용은 링크 참조)
- [WSL(Windows Subsystem for Linux)](https://docs.microsoft.com/ko-kr/windows/wsl/compare-versions#wsl-2-architecture)

2. "Linux용 Windows 하위 시스템" 옵션 기능 사용

```cmd
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

3. "가상 머신 플랫폼" 옵션 기능 사용

```cmd
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
입력 후 시스템 재부팅

4. WSL2를 기본 버전으로 지정

```cmd
wsl --set-default-version 2
```
- "WSL 2에 커널 구성 요소 업데이트가 필요합니다. 자세한 내용은 https://aka.ms/wsl2kernel을 참조하십시오." 라는 출력과 함께 제대로 명령이 적용되지 않으면 해당 링크에 들어가서 업데이트 패키지를 다운로드 후 실행한다.

5. 사용하고자하는 리눅스 배포판 설치
Microsoft Store에서 원하는 리눅스 배포판을 설치한다. 
나는 Ubuntu를 설치하여 사용 하였다.

6. 설치
우분투 리눅스에서 위에 언급한 방법으로 설치해도 되고 또는 WSL2까지 업데이트하였다면 Docker 공홈에서 Desktop for windows를 다운받아 사용하면 된다.
