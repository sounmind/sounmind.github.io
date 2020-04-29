---
title: "edwith 웹프로그래밍: 레이어드 아키텍처 실습 - 방명록 만들기 (2)"
date: 2020-04-29 22:44:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

# 레이어드 아키텍처 실습(2)

---

---

## 1. 복습

---

1. 오류 해결

    이전에 구조만 정리하고, 실제 코드를 입력하지 않은 상태라 코드를 입력하고 실행해봤는데 문제가 발생했다. url이 /list로 넘어가지 않는 것이다. 아래와 같은 오류가 콘솔에서 나왔다.

    > 클래스 [org.springframework.web.context.ContextLoaderListener]의 애플리케이션 리스너를 설정하는 중 오류 발생 ...

    바로 구글링해서 비슷한 문제를 겪고 해결한 사람의 블로그 포스팅을 찾았고 그대로 했더니 해결됐다. 참으로 신기하다. 문제는 maven update를 진행했을 때, maven 라이브러리 경로가 삭제되는 현상이 발생하는 것이었다. 그래서 따로 경로를 설정해줘야 했다.

    [[문제해결] 심각: Error configuring application listener of class org.springframework.web.context.ContextLoaderListener](https://myblog.opendocs.co.kr/archives/1657)

    자, 4개 중 첫번째 영상을 보고 실습을 했다. 다음 영상으로 넘어가보자.

## 2. 방명록 - DTO, DAO 작성

---

구조를 먼저 영상을 보고 타이핑하며 이해한 다음, 실제로 코드를 입력하자.

1. 실제 테이블을 입력한다.
    1. mysql 실행 후 테이블 생성

        ![1](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/20-04-29-1.PNG?raw=true)

    2. dto 패키지를 만들고 guestbook과 log 2개의 클래스를 만든다.
        1. guestbook
            - 해당 이름의 테이블이 가지고 있는 필드들을 선언해주고 getter setter 메서드와 toString까지 만든다.
        2. log
            - 해당 이름의 테이블이 가지고 있는 필들을 선언해주고 getter setter 메서드와 toString 메서드까지 오버라이딩한다.
    3. dao 패키지를 만들고 guestbookDao, logDao 2개의 클래스를 만든다.
        1. `@Repository`
        2. `.usingGeneratedKeyColumns("id");` → 자동으로 id 값 생성
        3. `insertAction.executeAndReturnKey` 메서드 → insert 문을 내부적으로 생성해서 실행하고, 자동으로 생성된 id 값을 리턴.
    4. GuestbookDaoSqls 클래스 생성해서 쿼리들을 관리한다.
        1. limit → 시작 값, 끝날 때의 값들을 설정해서 특정한 부분만 select 할 수 있다.
    5. GuestbookDao 클래스 생성
        1. @Repository
        2. static으로 GuestbookDaoSqls를 import해서 사용
    6. 테스트하는 클래스 생성

        ---

        ---

        ---

## 2020-04-29 배운 것과 느낀 점

---

1. 가볍게 배운 DTO, DAO 실습
