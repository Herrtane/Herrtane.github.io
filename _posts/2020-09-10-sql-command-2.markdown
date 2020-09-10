---
layout: post
title: <SQL> 02. MySQL 기본 명령어 (2) - Table
date: 2020-09-10 15:10:23 +0900
category: SQL
comments: true
---
이번 포스팅에서는 schema속으로 들어가서, schema를 이루고 있는 요소인 table에 대해서 다루어보려고 한다. 

### table의 요소

공학수학, 선형대수 등에서 이미 행렬이나 표의 구성요소를 배웠다. 가로축 성분들을 row(행), 세로축 성분들을 column(열)이라고 한다. SQL에서는 특히, **row는 각각의 데이터 하나를 의미하고, column은 데이터의 타입 혹은 종류를 의미한다**고 보면 된다.

### table의 생성

엑셀 등의 spread sheet도 database와 형태가 유사하다. database가 엑셀과 차별되는 특성은 column의 데이터 타입을 강제할 수 있다는 것이다. 예를 들면, ID column은 int형으로, Title column은 char형으로 강제할 수 있다. 아무튼, 본격적으로 table을 만드는 명령어의 예시는 아래와 같다.

```
CREATE TABLE my_table (
    id INT NOT NULL AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    date DATETIME NOT NULL,
    ...
    PRIMARY KEY(id)
);
```

### table 보기

이렇게 table을 생성했다면, 현재 내가 선택한 schema에 어떤 table이 존재하는지 보고싶을 수 있다. 그 때, 이 명령어를 사용한다.

```
SHOW TABLES;
```

보다 더 자세한, 특정 table의 정보를 보고 싶다면, Describe의 약자인 DESC 명령어를 사용한다.

```
DESC my_table;
```

### CRUD

대부분의 프로그램이 띠는 기본적이고 공통적인 데이터의 처리 기능을 CRUD라고 하는데, 아래와 같다. 

- C : Create
- R : Read
- U : Update
- D : Delete

<br/>
<br/>

## 마치며

다음 포스팅은 이어서 CRUD와 관련된 명령어에 대해서 포스팅할 예정이다.
