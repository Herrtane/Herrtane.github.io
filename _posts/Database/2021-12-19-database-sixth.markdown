---
layout: post
title: <Database> 10. 뷰와 시스템 카탈로그
date: 2021-12-19 23:35:23 +0900
category: Database
comments: true
---

## 뷰 

- 다른 릴레이션으로부터 유도된 릴레이션 (derived relation)
- 기존의 기본 릴레이션 (base relation)이나 또 다른 뷰에 대한 SELECT문의 형태로 정의되는 가상 릴레이션 (virtual relation)
- 자체적으로 디스크에 저장된 투플들을 갖고 있지는 않음.
- 릴레이션으로부터 데이터를 검색하거나 갱신할 수 있는 동적인 창 (dynamic window)의 역할을 함.
- 이와 대비되는 단어는 스냅샷 (snapshot)으로, 어느 시점에 SELECT문의 결과를 기본 릴레이션의 형태로 저장해놓은 것을 의미.
- 뷰를 통해 복잡한 질의를 간단하게 할 수 있고, 데이터 무결성과 독립성을 보장함. 또한 데이터 보안 기능을 제공함. (객체지향에서의 캡슐화와 비슷한 느낌) 

```
CREATE VIEW 뷰이름 [(애트리뷰트(들))]
AS SELECT문
[WITH CHECK OPTION];
```
ex) 

```
CREATE VIEW STU_CNO3 (STUNO, STUNAME)
AS SELECT STUNO, STUNAME
FROM STUDENT
WHERE STUNO = 3;
``` 

- 뷰를 제거하는 구문은 간단함. 

```
DROP VIEW 뷰이름;
``` 

## 시스템 카탈로그 

- 메타 데이터 : 데이터에 관한 데이터
- 시스템 카탈로그는 데이터베이스의 객체와 구조들에 관한 모든 데이터를 포함하므로, 시스템 카탈로그도 메타데이터.
- 시스템 카탈로그를 데이터 사전 (data dictionary) 또는 시스템 테이블이라고도 부름.
- 질의 최적화 (query optimization) : DBMS가 질의를 수행하는 여러가지 방법들 중에서 가장 비용이 적게 드는 방법을 찾는 과정.
- 어떤 사용자도 시스템 카탈로그를 직접 갱신할 수 없음.