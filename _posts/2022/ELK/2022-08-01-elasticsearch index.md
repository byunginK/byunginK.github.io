---
layout: post
title: "Elasticsearch index"
date: 2022-08-25
categories: [JPA]
---

# elasticsearch Index 생성

- 인덱스 (RDB에서 table에 해당)

```cmd
curl -XPUT http://localhost:9200/classes
```

- 인덱스가 생성되었는지 확인
  (pretty는 json형태로 깔끔하게 표기)

```cmd
curl -XGET http://localhost:9200/classes?pretty
```

- 인덱스릴 지우는 방법

```cmd
curl -XDELETE http://localhost:9200/classes
```

# Document 생성

```cmd
curl -XPOST http://localhost:9200/classes/class/1/ -d '{"title" : "Algorithm", "professor", "John"}
```

1. classes : 인덱스 명
2. class : 타입 명
3. id : 1
4. -d 이후에 넣을 json

### document를 파일 형태 (백업, 복구 등)

```cmd
curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json
```

- 파일 명을 넣어주면된다.
