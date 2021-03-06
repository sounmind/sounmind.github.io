---
title: "자료구조와 함께 배우는 알고리즘 입문 (5) 검색 알고리즘"
date: 2020-06-12 03:49:00 +09:00
categories: programming
tags: [algorithm, datastructure, programming, book, IT]
---


데이터 집합에서 원하는 값을 가진 요소를 찾아내려면 어떻게 해야 할까?

### 검색과 키

주소록에서 원하는 사람을 검색한다고 가정했을 때,  
'키'를 사용해 사람을 찾는다.  
'키(key)'는 형이 여러 개다. 예를 들어 (국가, 나이, 이름 등 ...) 키 값과 일치하거나, 키 값의 구간을 지정하거나, 키 값과 비슷하도록 지정해서 검색할 수 있다.

### 배열에서 검색하기

배열 검색의 경우 다음의 알고리즘을 활용한다.

1.  선형 검색: 무작위로 늘어놓은 데이터 모임에서 검색을 수행한다.
2.  이진 검색: 일정한 규칙으로 늘어놓은 데이터 모임에서 아주 빠른 검색을 수행한다.
3.  해시법: 추가,삭제가 자주 일어나는 데이터 모임에서 아주 빠른 검색을 수행한다.
    -   체인법: 같은 해시 값의 데이터를 선형 리스트로 연결하는 방법
    -   오픈 주소법: 데이터를 위한 해시 값이 충동할 때 재해시하는 방법.

데이터 집합이 있을 때 '검색'만이 목적이라면, 계산 시간이 짧은 것을 선택하면 된다. 하지만 검색 이외에도 데이터의 추가, 삭제 등을 자주하는 경우라면 검색 이외의 작업에 소요되는 비용을 종합적으로 평가하여 알고리즘을 선택하여 한다. 즉, 어떤 목적을 이루기 위해 선택할 수 있는 알고리즘이 여러 가지인 경우에는 용도나 목적, 실행 속도, 자료 구조 등을 고려하여 알고리즘을 선택해야 한다.

### 선형 검색(linear serach, 순차 검색)

#### 보초법

선행 검색은 반복할 떄마다 다음의 종료 조건 1과 2를 모두 판단한다. 단순한 판단이라고 생각할 수 있지만 종료 조건을 검사하는 비용은 결코 무시할 수 없다.

-   종료 조건
    1.  검색할 값을 발견하지 못하고 배열의 끝을 지나간 경우
    2.  검색할 값과 같은 요소를 발견한 경우

이 비용을 반으로 줄이는 방법이 보초법이다. 배열의 요소 마지막에 보초(sentinel) 값을 넣어놔서 종료 조건 1을 생략하는 것이다. 이 때 보초 값은 검색하려는 값과 동일하다.  
즉, 매번 배열 하나하나 검색할 때 종료 조건 2개를 판단하는 것이 아니라, 조건 1개를 판단하고, 값을 리턴할 때에만 찾은 값이 마지막 값(보초 값)인지 아닌지(검색하려 했던 값), 1번만 더 판단하면 된다.

---

#### 출처:

1.  책 <자료구조와 함께 배우는 알고리즘 입문 자바편>
