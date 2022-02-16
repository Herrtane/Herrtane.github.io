---
layout: post
title: <Database> 07. SQL
date: 2021-12-13 21:20:23 +0900
category: Database
comments: true
---

## SQL 

- SQL (Structured Query Language)
- 비절차적 언어 (선언적 언어)이므로 사용자가 원하는 데이터(what)만 명시 가능. 처리하는 방법(how)은 명시 불가.
- 관계 대수랑은 다르게 관계 연산자들이 수행되는 순서는 명시하지 않음.
- 대화식 SQL(interactive SQL)과 내포된 SQL(embedded SQL)로 나뉨. embedded SQL은 C,C++등의 언어 내에 SQL을 포함하여 사용하는 방식.
- SQL의 Q가 Query지만, 질의 뿐만 아니라 데이터 정의, 수정, 제어 등 여러가지 역할을 할 수 있다. 

## 데이터 정의어 (DDL) 

- CREATE DOMAIN, TABLE, VIEW, INDEX
- ALTER TABLE
- DROP DOMAIN, TABLE, VIEW, INDEX 

CREATE 예시문

```
CREATE TABLE STUDENT
(STUNO NUMBER NOT NULL,
STUNAME CHAR(20) UNIQUE,
GRADE CHAR(10) DEFAULT 'Freshman',
MONEY NUMBER CHECK (MONEY < 1000000),
CLASSNO NUMBER CHECK (CLASSNO IN (1,2,3,4,5,6)) DEFAULT 1,
PRIMARY KEY (STUNO),
FOREIGN KEY (CLASSNO) REFERENCES CLASS (CLASSNO) ON DELETE CASCADE);
``` 

아래는 무결성 제약조건을 추가, 삭제하는 과정이다. STUDENT_PK라는 이름의 제약조건을 추가하고, 삭제한다.
ALTER TABLE문을 통해 언제든지 릴레이션에 제약조건을 추가, 삭제할 수 있음을 기억하자.

```
ALTER TABLE STUDENT ADD CONSTRAINT STUDENT_PK
PRIMARY KEY (STUNO);

ALTER TABLE STUDENT DROP CONSTRAINT STUDENT_PK;
```

### 애트리뷰트의 제약조건 

- NOT NULL : primary key에는 명시하는 것을 권장.
- UNIQUE : 동일한 애트리뷰트 값을 갖는 투플이 두 개 이상 존재하지 않도록 보장.
- DEFAULT : 애트리뷰트에 NULL값 대신에 특정 값을 티폴트 값으로 지정할 수 있음.
- CHECK : 애트리뷰트가 가질 수 있는 값들의 범위를 지정.
- ON DELETE CASCADE : CLASS 릴레이션에서 기본키의 값이 삭제되면 (어떤 투플이 삭제), STUDENT 릴레이션에서도 이 값을 참조하는 모든 투플들이 연쇄적으로 삭제됨.
- ON DELETE SET NULL, ON DELETE NO ACTION, ON DELETE SET DEFAULT 도 있음. 

## 데이터 검색 : SELECT 

- SELECT 문의 형식 

```
SELECT [DISTINCT] 애트리뷰트
FROM 릴레이션
[WHERE 조건]
[GROUP BY 애트리뷰트]
[HAVING 조건]
[ORDER BY 애트리뷰트 [ASC|DESC]];
``` 

- SELECT-FROM-WHERE 블록 : SELECT-FROM-WHERE절로 이루어진 기본적인 문장. 

- 별칭(alias) : 투플 변수를 사용해서 
```
FROM STUDENT AS S, CLASS AS C 
```
처럼 나타내는 것. AS는 생략 가능. 

- LIKE : 문자열 비교
```
WHERE STUNAME LIKE '이%';
WHERE STUNAME LIKE '_성_';
WHERE STUNAME LIKE '%한';
``` 

- NULL 값 주의사항 : 어떤 애트리뷰트에 들어있는 값이 NULL인가 비교하기 위해서 'STUNO=NULL', 'STUNO<>NULL', 'NULL > 300'처럼 나타내면 안됨. IS NULL로 나타내야 함.
```
SELECT *
FROM STUDENT
WHERE MONEY > 30000;
``` 
위의 문장에서 MONEY가 NULL인 투플이 존재한다면, 이 투플들은 검색 결과에 포함되지 않음. 만약 검색하고 싶다면, 뒤에 OR MONEY IS NULL을 포함시켜야함. 

- 집단함수 : COUNT, SUM, AVG, MAX, MIN과 같은 함수. SELECT절과 HAVING절에만 나타낼 수 있음. WHERE절은 각 투플에 적용되기 때문에 나타낼 수 없음. 

- 조인 : 가장 흔한 형태는 아래와 같다. 

```
SELECT ...
FROM R, S
WHERE R.A <비교 연산자> S.B;
```
조인 조건을 생략하거나 틀리게 표현했을 때는 카티션 곱이 생성되므로, 정확하게 명시하는 습관을 들여야함. 

- 자체 조인(self join) : 한 릴레이션에 속하는 투플을 동일한 릴레이션에 속하는 투플들과 조인하는 것. 

- 중첩 질의(nested query) : 부질의(subquery)라고도 함. WHERE절에 나타낼 수 있음. 

```
SELECT ...
FROM ...
WHERE ... (SELECT ... FROM ... WHERE ...);
``` 

## 데이터 조작어 (DML) 

### INSERT 

```
INSERT 
INTO 릴레이션(애트리뷰트1, ..., 애트리뷰트n)
VALUES (값1, ..., 값n); 

INSERT
INTO 릴레이션(애트리뷰트1, ..., 애트리뷰트n)
SELECT 애트리뷰트1', ..., 애트리뷰트n'
FROM ...
WHERE ...;
``` 

### DELETE 

```
DELETE FROM 릴레이션
[WHERE 조건];
``` 

### UPDATE 

```
UPDATE 릴레이션
SET 애트리뷰트 = 값 또는 식[, ...]
[WHERE 조건];
``` 

## Trigger와 Assertion

추후 필요시 다루도록 하겠음.