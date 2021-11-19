---
layout: post
title: <Database> 01. MySQL 기본 명령어 (1) - Login과 Schema/Database
date: 2020-09-09 23:10:23 +0900
category: Database
comments: true
---
8개월 전, 그러니까 2학년을 마친 겨울방학 때 처음으로 웹 서버라는 것에 흥미가 생겼었다. 그래서 알게된 것이 생활코딩. 내게 있어서는 프로그래밍 공부면에서 큰 전환점이자 큰 도움이 되었던 강의였다. 생활코딩을 통해서 직접 웹 서버를 구축하고 나만의 홈페이지를 제법 그럴싸하게 만들 수 있었기 때문이다. 지금 이 블로그를 운영하는 것도, 생활코딩에서 배웠던 웹에 대한 중요한 지식이 바탕이 되었기에 가능한 것이라고 생각한다.

<br/>

서론이 길었다. 생활코딩에서 배웠던 HTML, CSS, Javascript, PHP, MySQL을 언젠가 한번 복습하자고 마음 먹었었는데, 마침 요즘 수박 겉핥기식으로나마 Web hacking에 대해서 공부하고 있는 과정에서 Web에 대한 지식들이 다시금 필요해졌다. 그래서 MySQL을 시작으로 나에게 필요한 지식들을 생활코딩때 배웠던 내용을 바탕으로 포스팅할 예정이다. 

### MySQL 서버 접속

mysql이 설치된 bin폴더로 접근하거나, cmd나 터미널 상에 bin폴더를 환경변수로 설치한 이후, 아래의 명령어를 실행한다. -uroot는 관리자 권한으로 실행한다는 것이다. 만약 -uherrtane이면 herrtane이라는 계정으로 실행한다는 것이다.

```
mysql -uroot -p
```

### schema 혹은 database 생성 및 삭제

가장 먼저, 모든 table들의 집합인 schema(혹은 database)를 다루어야 한다. 생성과 삭제 명령어는 아래와 같다. 앞으로 schema명은 tutorial로 통일한다.

```
CREATE DATABASE tutorial;
DROP DATABASE tutorial; 
```

### 모든 schema 보기

지금까지 생성된 모든 schema들을 보는 방법이다.

```
SHOW DATABASES;
```

### 사용할 schema 선택하기

여러개의 존재하는 schema들 중에, 내가 명령어를 쓰고 변경하는 등 직접 사용할 schema를 선택하는 방법이다.

``` 
USE tutorial;
```

<br/>
<br/>

## 마치며

다음 포스팅은 이어서 table을 다루는 명령어에 대해서 포스팅할 예정이다.
