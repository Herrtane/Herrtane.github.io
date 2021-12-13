---
layout: post
title: <Database> 05. 관계 데이터 모델의 개념
date: 2021-12-13 20:20:23 +0900
category: Database
comments: true
---

생활코딩에서 배웠었던 sql의 개략적인 문법을 포스팅한지 어연 1년이 지났다. 드디어 데이터베이스 전공 내용을 독학했던 것들을 정리해서 포스팅 시작하겠다! 미리 정리했던 내용들을 한꺼번에 포스팅하도록 하겠다.

## DBMS 언어

1. DDL : Data Definition Language
ex) CREATE, DROP, ALTER
2. DML : Data Manipulation Language
ex) SELECT, UPDATE, DELETE, INSERT
3. DCL : Data Control Language
ex) GRANT, REVOKE 

## 관계 데이터 모델의 개념

- domain : 한 애트리뷰트에 나타날 수 있는 값들의 집합.
- cardinality : 릴레이션의 투플 수
- degree : 릴레이션의 애트리뷰트 수
- null : 0이 아니고, 공백 문자도 아니고, 공백 문자열도 아닌, '무'를 나타내는 개념
- relation schema : 릴레이션의 이름과 애트리뷰트들의 집합 (= intension)
ex) EMPLOYEE(EMPNO(밑줄), EMPNAME, ..)
- relation instance : 릴레이션의 특정 시점에 들어 있는 투플들의 집합 (= extension)
- relational database schema : 하나 이상의 relation schema들로 이루어짐 

## 릴레이션의 특성 

- 동일한 투플이 두개 이상 존재하지 않는다.
- 애트리뷰트들의 순서는 중요하지 않다.
- 한 투플의 각 애트리뷰트는 원잣값을 갖는다.
- 투플들의 순서는 중요하지 않다. 

## 키 

- super key : 한 릴레이션 내의 특정 투플을 고유하게 식별하는 애트리뷰트들의 집합. 투플이 중복되지만 않으면 super key가 됨.
- candidate key : super key면서 최소한의 애트리뷰트들의 모임으로 이루어진 key. 따라서 candidate key의 애트리뷰트는 하나라도 빼면 고유성을 상실한다.
- composite key : candidate key 중에 둘 이상의 애트리뷰트들로 이루어진 key.
- primary key : candidate key 중에 대표적으로 선정한 key.
- surrogate key : 자연스러운 primary key를 찾을 수 없는 경우, 인위적인 숫자를 부여해서 애트리뷰트에 추가한 key.
- alternate key : primary key를 제외한 candidate key.
- foreign key : 어떤 릴레이션의 primary key를 참조하는 key. 릴레이션간의 관계를 나타내기 위해서 사용됨. 참조되는 릴레이션의 primary key와 동일한 domain을 가져야함. 

## 무결성 제약조건 (integrity constraint) 

- 데이터의 정확성 또는 유효성을 지키기 위해 데이터베이스 상태가 만족시켜야 하는 조건. 사용자에 의한 갱신이 데이터베이스의 일관성을 깨지 않도록 보장하는 수단의 역할을 한다. 

- integrity constraint도 relation schema의 한 부분이다. 

1. domain constraint : 각 애트리뷰트 값에 제한을 거는 것.
ex) AGE는 0~100, SALARY > 900000
2. key constraint : 키 애트리뷰트에 중복된 값이 존재해서는 안된다는 조건.
3. entity integrity constraint : primary key를 구성하는 어떤 애트리뷰트도 null값을 가질 수 없다는 조건.
4. referential integrity constraint : 릴레이션 R2의 foreign key가 R1의 primary key를 참조할 때, foreign key의 값은 R1의 어떤 투플의 primary key의 값과 같거나, null이다. 즉, 릴레이션은 참조할 수 없는 foreign key 값을 가질 수 없다. 

- integrity constraint를 유지하기 위해, 삽입, 삭제, 수정 등의 갱신이 일어날 때마다 적절한 조치를 취한다. 예시는 다루지 않겠다. 

## 마치며 

본격적으로 데이터베이스 이론에 대해서 시작한다.