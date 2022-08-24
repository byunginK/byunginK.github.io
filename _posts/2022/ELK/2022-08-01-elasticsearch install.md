---
layout: post
title: "Elasticsearch 설치 (Ubuntu)"
date: 2022-08-24
categories: [JPA]
---

# 우분투 elasticsearch 설치

### JDK는 필수로 설치되어있어야 한다.

1. DEB 파일 다운로드

```cmd
dpkg -i elasticsearch-5.1.1.deb
```

- 설치 path: /usr/share/elasticsearch
- config file path: /etc/elasticsearch
- init script at: /etc/init.d/elasticsearch

2. 서버를 키고 끄때 자동으로 키고 끄는 기능

```cmd
sudo systemctl enable elasticsearch.service

sudo service elasticsearch start
sudo service elasticsearch stop
```

3. 실행 확인

```cmd
curl -XGET 'localhost:9200'
```
