---
layout: post
title: <SQL> 04. MySQL 기본 명령어 (4) - JOIN
date: 2020-09-10 20:20:23 +0900
category: SQL
comments: true
---
이번 포스팅에서는 관계형 데이터베이스의 성격을 나타내는 JOIN 명령어에 대해서 다루어보려고 한다.

## 관계형 데이터베이스

모든 데이터를 하나의 표에 나타낸다면 직관적인 파악이 가능하다는 장점이 있지만, 만약 중복되는 데이터가 존재할 경우, 그 수가 많아질수록 수정하기가 굉장히 번거로워지는 단점이 존재한다.

<br/>

관계형 데이터베이스의 특징을 이용하면 이러한 단점을 극복할 수 있다. 중복의 '악취'가 나는 부분의 table을 분리함으로서 별도의 참조 데이터 table을 만들게 되면, 눈에 띄게 유지 보수가 용이해지고 중복 또한 제거할 수 있다. 다만, 한번에 직관적으로 자료를 파악하기는 쉽지 않다는 단점이 존재한다.

<br/>

JOIN을 다루기에 앞서, 아래의 링크를 통해서 JOIN의 종류를 벤다이어그램을 통해서 개념적으로 파악할 수 있으므로, 링크를 첨부해놓도록 하겠다.

https://sql-joins.leopard.in.ua/

### LEFT JOIN

**이번 포스팅에서 나오는 코드는 생활코딩에서 진행했던 실습을 바탕으로 적은 코드이므로, 그동안 포스팅에서 언급되지 않은 row, column, schema명이 등장한다. 이 점은 양해하면서 읽어주기를 바란다.**

```
SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.id;
```

JOIN된 table을 깔끔하게 확인하려면,

```
SELECT topic.id AS topic_id, title, description, created, name, profile FROM topic LEFT JOIN author ON topic.author_id = author.id;
```
id가 author table과 topic table에 중복되기 때문에, 어느 id인지 명시해주어야한다.

### INNER JOIN

INNER JOIN은 **양쪽 table 모두에 존재하는 행**만으로 새로운 table을 만드는 것이다. 따라서, JOIN하는 대상의 행에 NULL값이 존재한다면, 이는 table에서 제외된다.

### FULL OUTER JOIN

FULL OUTER JOIN은 LEFT JOIN과 RIGHT JOIN의 합집합이다. 참고로, 이 JOIN은 MySQL에서 지원하지 않는다. 이는 이후에 다룰 UNION (UNION DISTINCT) 명령어를 통해서 해결할 수 있다.

### EXCLUSIVE JOIN

EXCLUSIVE JOIN은 FULL OUTER JOIN에서 INNER JOIN을 제외한 것이다. 집합으로 생각하면 쉽다. 사실, FULL OUTER JOIN이나 EXCLUSIVE JOIN은 실무에서 잘 사용되지 않는다고 한다. 그냥 개념적으로 짚고 넘어가도록 하자.

<br/>
<br/>

## 마치며

JOIN은 사실 다양하게 존재하지만, 이번 포스팅에서는 JOIN의 의미를 다루고싶어서 LEFT JOIN만 자세하게 다루었다. 사실 다른 경우도 거의 내용이 유사하므로, 이는 별도로 포스팅을 하지는 않겠다.

<br/>

다음 포스팅은 생활코딩에서 미처 다루지 않았던, 추가적인 명령어나 기능에 대해서 다루면서 SQL 카테고리를 마무리지으려고 한다.
