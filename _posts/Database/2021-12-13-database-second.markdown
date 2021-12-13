---
layout: post
title: <Database> 06. 관계 대수
date: 2021-12-13 20:20:23 +0900
category: Database
comments: true
---

## 관계 해석과 관계 대수

- 관계 해석 (relational calculus) : 원하는 데이터만 명시하는 선언적인 언어. 질의를 어떻게 수행할 것인지는 명시하지 않음.
- 관계 대수 (relational algebra) : 어떻게 질의를 수행할 것인지를 명시하는 절차적인 언어. 

## 관계 대수

- binary operator : 이항연산자
- unary operator : 단항연산자
- 순수 관계 연산자 : selection, projection, join, division
- 일반 집합 연산자 : union, difference, Cartesian product, intersection 

### selection

- 한 릴레이션에서 predicate를 만족하는 투플들의 부분집합을 생성
- 기호 : σ <실렉션조건> (릴레이션) 

### projection

- 한 릴레이션의 애트리뷰트들의 부분집합을 생성
- 기호 : π <애트리뷰트 리스트> (릴레이션)
- 투플의 중복은 제거해야 함. 

### 집합 연산자

- union, difference, intersection, Cartesian product들이 있음. 자세한 설명은 다루지 않음. 

- Cartesian product : cardinality가 i인 R1(A1,A2,...,An)과 cardinality가 j인 R2(B1,B2,...,Bn)의 Cartesian product R1XR2는 degree가 n+m이고, cardinality가 i*j이고, 애트리뷰트가 (A1,A2,...,An,B1,B2,...,Bn)인 투플들의 모든 가능한 조합으로 이루어진 릴레이션이다. 크기가 매우 클 수 있으므로 이 자체로 유용하진 않음.

## 조인

### 세타조인

- Cartesian product 중에서 세타조인 조건을 만족하는 투플들만 골라낸 릴레이션. 
- 기호 : R ▷◁ (R.attribute 조건 S. attribute) S 

### 동등조인 (equijoin)

- 세타조인 중에서 비교연산자가 =인 조인. 

### 자연조인 (natural join)

- 동등조인에서 겹치는 조인 애트리뷰트를 한 개 제외한 조인.
- 기호 : R * R.attribute, S.attribute S 

### 외부조인 (outer join)

- 상대 릴레이션에서 대응되는 투플을 갖지 못하는 투플이나 조인 애트리뷰트에 널값이 들어있는 투플들을 다루기 위해서 조인 연산을 확장한 것.
- 왼쪽 외부 조인, 오른쪽 외부 조인, 완전 외부 조인이 있으며, 각각 상대 릴레이션에 관련된 투플이 없으면 그 결과 릴레이션의 해당 부분은 null로 채움. 기호는 조인 기호에서 해당 방향으로 위아래 삐죽 그은 모양.