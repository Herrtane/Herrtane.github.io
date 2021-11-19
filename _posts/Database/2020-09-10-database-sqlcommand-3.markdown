---
layout: post
title: <Database> 03. MySQL 기본 명령어 (3) - CRUD(INSERT,SELECT,UPDATE,DELETE)
date: 2020-09-10 16:10:23 +0900
category: Database
comments: true
---
이번 포스팅에서는 CRUD의 성질을 띠는 MySQL의 명령어들에 대해서 다루어보려고 한다. 

### INSERT

현재 테이블에 하나의 내용을 추가하고 싶다면, 즉 하나의 row를 추가하고 싶다면, INSERT 명령어를 사용한다.

```
INSERT INTO my_table (title,desciption,date,author) VALUES('Apple','Apple is red',NOW(),'lee');
```

### SELECT

현재 테이블에 어떤 row가 추가되었는지 자세한 정보를 보고 싶다면, SELECT 명령어를 사용한다. 여기서 *는 와일드카드 명령어로서, '모든'을 나타낸다고 생각하면 된다.

```
SELECT * FROM my_table;
```

만약 my_table 중에 몇몇 column만 보고싶다면, 아래 예시처럼 입력하면 된다.

```
SELECT id,title,author FROM my_table;
```

더 나아가서, 만약 더 자세한 특정 조건에 해당하는 column의 row만 보고싶다면, WHERE 구문을 추가하면 된다.

```
SELECT id,title,author FROM my_table WHERE author='lee';
```

이외에도 ORDER BY 등의 구문이 있지만, 이는 추후 검색을 통해서 알아도 충분하므로 생략하도록 하겠다.

### UPDATE

table을 수정하는 과정은 아래와 같다.

```
UPDATE my_table SET title = 'Banana' WHERE id=1;
```

**주의**할 것은, WHERE 구문을 빠뜨릴경우, table 전체가 저렇게 수정되니까, 굉장히 조심하도록 하자.

### DELETE

원하는 table의 내용을 지우고 싶다면, 다음과 같이 하자.
(**DELETE는 절대로 주의해서 사용하도록 하자**)

```
DELETE FROM my_table WHERE id=1;
```

**주의 of 주의!**할 것은, WHERE 구문을 빠뜨릴경우, table 전체가 삭제되므로, 굉장히 조심하도록 하자. 

생활코딩 이고잉님 왈,
> DELETE문 잘못 사용하시면 큰일납니다..! 인생이 바뀔 수 있어요.

<br/>
<br/>

## 마치며

다음 포스팅은 관계형 데이터베이스에 대해서 포스팅할 예정이다.
