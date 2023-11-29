---
layout: post
title: "[MySQL] Query Plan 이란"
date: 2023-08-11
categories: [DB]
---
# [MySQL] Query Plan 보는 법


먼저 쿼리 플랜이란 무엇인지 알아보겠습니다

SQL을 처리하는 최저비용의 경로를 생성해주는 DBMS 내부 핵심엔진인 쿼리 옵티마이저가 쿼리를 수행할 때 생성한 최적의 처리경로를 실행계획(Query Plan)이라고 합니다.

MySQL에서는 실행할 쿼리문 앞에 'EXPLAN' 키워드를 이용해 실행계획에 대한 정보를 살펴 볼 수 있습니다.

이 실행계획을 이용하면 이슈가 발생하는 쿼리에 대한 이해를 도울 뿐만 아니라, 어떻게 최적화할 지에 대한 인사이트를 제공합니다.

어떤 식으로 제공을 하는 지 예제를 통해 확인해보겠습니다.

```
EXPLAIN SELECT * FROM short_url su LEFT OUTER JOIN short_url_stat sus ON su.hash = sus.hash WHERE deleted_date IS NULL;
```

위와 같이 1:N 관계를 가진 테이블을 조인하여 SELECT한 쿼리문 앞에 'EXPLAIN' 키워드를 붙여서 실행을 한 Output입니다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdbbqa9%2FbtqD8PMH5VI%2FzlEXJRccyWVzs9eFeLK9A1%2Fimg.png)

항목을 보면서 설명드리면,

- id : 쿼리 내의 select 문의 실행 순서
- select_type : select 문의 유형입니다.

> 1. SIMPLE: 단순 select ( union이나 서브쿼리를 사용하지 않음 )
> 2. PRIMARY: 가장 외곽에 있는 select문
> 3. UNION: union에서의 두번째 혹은 나중에 따라오는 select문
> 4. DEPENDENT UNION: union에서의 두번째 혹은 나중에 따라오는 select문, 외곽 쿼리에 의존적이다.
> 5. UNION RESULT: union의 결과물
> 6. SUBQUERY: 서브쿼리의 첫번째 select- DEPENDENT SUBQUERY: 서브쿼리의 첫번째 select, 바깥 쪽 쿼리에 의존적이다.
> 7. DERIVED: from절의 서브쿼리

- table : 참조하고 있는 테이블명
- type : 조인타입이며 쿼리 성능과 아주 밀접한 항목입니다. 쉽게 말씀드리면 아래 항목들 중에서 밑으로 갈 수록 안 좋은 쿼리형태입니다.

> 1. system : 테이블에 단 하나의 행만 존재(=시스템 테이블). const 조인의 특별한 형태이다.
> 2. const : 하나의 매치되는 행만 존재하는 경우. 하나의 행이기 때문에 상수로 간주되며, 한번만 읽어들이기 때문에 무척 빠르다.
> 3. eq_ref : 조인수행을 위해 각 테이블에서 하나의 행만이 읽혀지는 형태. const 타입 외에 가장 훌륭한 조인타입이다.
> 4. ref : ref조인에서 키의 가장 왼쪽 접두사 만 사용하거나 키가 a PRIMARY KEY또는 UNIQUE인덱스 가 아닌 경우 (즉, 조인이 키 값을 기반으로 단일 행을 선택할 수없는 경우) 사용된다. 사용되는 키가 몇 개의 행과 만 일치하는 경우 이는 좋은 조인 유형이다.
> 5. fulltext : fulltext 색인을 사용하여 수행된다.
> 6. ref_or_null : 이 조인 유형은 비슷 ref하지만 MySQL이 NULL값 을 포함하는 행을 추가로 검색한다는 점이 다르다. 이 조인 유형 최적화는 하위 쿼리를 해결하는 데 가장 자주 사용된다.
> 7. index_merge : 인덱스 병합 최적화가 적용되는 조인타입. 이 경우, key컬럼은 사용된 인덱스의 리스트를 나타내며 key_len 컬럼은 사용된 인덱스중 가장 긴 key명을 나타낸다.
> 8. range : 인덱스를 사용하여 주어진 범위 내의 행들만 추출된다. key 컬럼은 사용된 인덱스를 나타내고 key_len은 사용된 가장 긴 key부분을 나타낸다. ref 컬럼은 이 타입의 조인에서 NULL 이다. range 타입은 키 컬럼이 상수와 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN 또는 IN 연산에 사용될때 적용된다.
> 9. index : 이 타입은 인덱스가 스캔되는걸 제외하면 ALL과 같다. 보통 인덱스 파일이 데이터 파일보다 작기 때문에 ALL보다 빠르다.
> 10. ALL : 이전 테이블과의 조인을 위해 풀스캔이 된다. 만약 조인에 쓰인 첫 테이블이 고정이 아니라면 비효율적이다. 그리고 대부분의 경우 아주 느리며, 보통 상수값이나 상수인 컬럼값으로 row를 추출하도록 인덱스를 추가하여 ALL 타입을 피할 수 있다.


- possible_keys : MySQL이 해당 테이블의 검색에 사용할 수 있는 인덱스들을 나타냅니다.
- key : MySQL이 실제 사용한 key나 인덱스를 나타냅니다.
- key_len : MySQL이 사용한 인덱스의 길이를 나타낸다. key 컬럼의 값이 NULL이면 이 컬럼의 값도 NULL입니다.
- ref : 행을 추출하는 데 키와 함께 사용 된 컬럼이나 상수값을 나타냅니다.
- rows : 이 값은 쿼리 수행에서 MySQL이 찾아야하는 데이터행 수의 예상값을 나타냅니다. 추정 수치이며 항상 정확하지 않다.
- filtered : filetered열에 나타난 조건에 의해 필터링 될 테이블 행의 예상 비율을 나타낸다. 즉 rows는 검사 된 행 수를 나타내고 rows * filtered / 100은 이전 테이블과 조인 될 행 수를 표시합니다.
- Extra : MySQL이 이 쿼리를 어떻게 해석하는 지에 대한 추가 정보가 들어있습니다.

이번 이슈 같은 경우는 총 22개의 테이블 조인 중 2번의 self 조인이 발생했는데, 이 조인에서 참조되는 컬럼이 인덱스 되지 않은 컬럼을 self 조인하게 되어 풀스캔되는 현상이었습니다. 따라서 Query Plan으로 보면, type의 최저의 성능을 내는 ALL 타입의 조인에서 참조 컬럼을 인덱싱하여 검증한 결과, ref 타입으로 변경되어 성능을 회복할 수 있었습니다.

Query Plan을 사용하여 간략하게라도 Plan을 볼 수 있다면 복잡한 쿼리라도 당황하지 않고 쿼리의 문제를 파악할 수 있는 스킬이 생기지 않을까 생각됩니다.

감사합니다.

> 참고
> 
> 
> [http://iloveulhj.github.io/posts/sql/sql-optimizer-principle.html](http://iloveulhj.github.io/posts/sql/sql-optimizer-principle.html)
> 
> [https://gradle.tistory.com/4](https://gradle.tistory.com/4)