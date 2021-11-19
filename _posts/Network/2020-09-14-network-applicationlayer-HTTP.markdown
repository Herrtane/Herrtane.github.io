---
layout: post
title: <Network> 08. Application layer - HTTP (완결)
date: 2020-09-14 17:00:23 +0900
category: Network
comments: true
---
처음으로 직접 내가 만든 github 블로그에서 하나의 카테고리를 완성해서 참 뿌듯하다. 이제 컴퓨터 네트워크의 마지막 포스팅으로, Application layer의 HTTP에 대해서 간단히 살펴보고 이 카테고리를 마무리지으려고 한다.  

## HTTP

World Wide Web의 대표적인 통신 프로토콜인 HTTP는 현재 대부분의 서버와 클라이언트에서 사용될 정도로 기본적이면서도 중요하다. HTTP의 동작 원리는 단순하다. 한 쪽이 반대쪽에게 **HTTP request**를 보내면, 반대쪽에서 이에 **HTTP response**를 하는 구조이다. 모든 HTTP는 이 방식으로 동작한다.

<br/>

HTTP에서 주로 사용되는 method는 다음과 같다.
1. GET
2. PUT
3. DELETE
4. POST
5. HEAD

<br/>

또한 HTTP에서 사용되는 status code는 **three-digit numeric code**이며, 자주 사용되는 코드로는 다음과 같다.
1. 200 : OK.
2. 302 : Redirect.
3. 404 : Not Found.

<br/>

**Proxy**는 HTTP에서 client와 server 사이에서 중개하는 역할을 하는데, 보안, 추가기능 등의 다양한 응용 분야에 사용된다.

<br/>
<br/>

## 마치며

사실 HTTP를 포함한 세부적인 Application의 기능들은 컴퓨터 네트워크 과목 자체에서 다루기보다는, 별도의 카테고리에서 다루는 것이 적절하다고 생각한다. 지금까지 Network 카테고리에서 포스팅한 내용은 학기 중에 배웠던 전반적인 컴퓨터 네트워크 이론이고, 반드시 앞으로도 기억해야겠다고 판단하여 포스팅으로 남겼다.

<br/>

이 외적인 심화적인 내용은 앞으로 졸업까지 남은 기간동안 내가 프로젝트나 개인적인 공부를 진행하면서 천천히 추가해나갈 예정이다. 어쨌든, 이렇게 하나의 전공 내용을 카테고리로 포스팅해서 의미 있는 시간이었다! 스스로에게 박수를 보내고 싶다!
