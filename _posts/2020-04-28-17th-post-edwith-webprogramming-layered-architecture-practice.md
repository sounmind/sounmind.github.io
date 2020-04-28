---
title: "edwith 웹프로그래밍: 레이어드 아키텍처 실습 - 방명록 만들기"
date: 2020-04-28 20:43:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

# 레이어드 아키텍처 실습 - 방명록 만들기

---

---

## 1.  활용하는 개념과 기술

---

1.  Spring JDBC를 이용한 Dao 작성
2. Controller + Service + Dao
3. 트랜잭션 처리
4. Spring MVC에서 폼 값 입력 받기
5. Spring MVC에서 redirect 하기
6. Controller에서 jsp에게 전달한 값을 JSTL과 EL을 이용해 출력하기

## 2. 요구 사항

---

1. 데이터베이스
    1. 방명록 정보는 guestbook 테이블에 저장된다.
    2. id는 자동으로 입력된다.
    3. id, 이름, 내용, 등록일을 저장한다.
    4. 방명록에 글을 쓰거나, 방명록의 글을 삭제할 때는 log 테이블에 클라이언트의 ip 주소, 등록(삭제) 시간,  등록/삭제(method 칼럼) 정보를 데이터베이스에 저장한다.
        1. 사용하는 테이블 이름은 log 이다.
        2. id는 자동으로 입력되도록 한다.
2. 화면
    1. `http://localhost:8080/guestbook/`을 요청하면 자동으로  `/guestbook/list`로 리다이렉트 된다.
        1. 방명록이 없으면 건 수는 0이 나오고 아래에 방명록을 입력하는 폼이 보여진다.
        2. 이름과 내용을 입력하고, 등록버튼을 누르면 `/guestbook/write URL`로 입력한 값을 전달하여 저장한다.
        3. 값이 저장된 이후에는 `/guestbook/list`로 리다이렉트 된다.
        4. 입력한 한 건의 정보가 보여진다.
    2. 방명록 내용과 폼 사이에 방명록 페이지 링크를 가진 페이지 숫자를 표시한다. 방명록 다섯 건 당 한 페이지로 설정한다.
        1. 방명록 6건이 입력되면 아래 페이지 수가 1, 2 로 두 개가 된다.
        2. `/guestbook/list?start=0`을 요청하고, 2 페이지를 누르면 `/guestbook/list?start=5`를 요청하게 된다.
        3. `/guestbook/list`는 `/guestbook/list?start=0`과 결과가 같다.  

![3](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/20-04-28-3.png?raw=true)

![4](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/20-04-28-4.png?raw=true)

## 3. 방명록 클래스 다이어그램

---

![1](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/20-04-28-1.png?raw=true)

- 웹 레이어 설정 파일: `web.xml` 에서는 두 개의 자바 config 파일(`WebMvcContextConfiguration.java` , `ApplicationConfig.java` )에 대해 설정한다.
    1. DispatcherService가 읽어들이는 **웹 레이어 설정 파일:**  `WebMvcContextConfiguration.java`
    2. `ApplicationContextListener`가 읽어들이는 **비지니스, 레파지토리 레이어 설정 파일:**  `ApplicationConfig.java` 이다.
        - 이 config 파일은 **비지니스, 레파지토리 레이어 설정 파일:** `DbConfig.java` 를 import 한다.

![2](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/20-04-28-2.png?raw=true)

1. URL 요청을 처리하는 핸들러: GuestbookController
    1. index.jsp (리다이렉트 하는 코드만)
    2. list.jsp
    - 이 컨트롤러는 비즈니스 로직을 가지고 있는 서비스 객체를 사용하게 될 것.
        1. 인터페이스: GuestbookService
        2. 실제 구현 클래스: GuestbookServiceImpl
            - 비즈니스 로직 수행 (하나의 트랜잭션 단위)
                1. logDao (저장만 하고 나머지 작업 수행 X → 별도 sql  필요 없음)
                2. GuestBookDao (여러 개의 sql 작업 수행)
                    1. GuestbookDaoSqls (sql 관리 )
2. DTO
    1. DTO: Guestbook (컨트롤러, list.jsp, 인터페이스, GuestbookDao 가 사용)
    2. DTO: Log (GuestbookServiceImpl, LogDao 가 사용)

---

---

---

---

## 2020-04-28 배운 것과 느낀 점

---

1. 배운 것
    1. 프로그램 만드는 일은 엄청나게 복잡하다. 하지만 업계 사람들에게는 이것이 기본 지식이다. 머릿속에 당연히 들어 있다. 그리고 업계 사람들은 나보다 훨씬 더 일찍 시작했고, 많은 노력을 들여 이 지식을 습득하고 연습해서 자기 것으로 만들었다. 나는 초짜다. 지금 머리 아프고 막막한 것은 당연하다.
    2. 머리 아프고 막막함에도 나는 왜 개발자가 되고 싶은가?
        1. 이 일이 나의 길, 내가 세상에 기여할 수 있는 일, 그럼으로써 이 사회에서 돈을 벌고 먹고 살아갈 수 있는 일이라고 믿고 있기 때문이다.
        2. 비록 옛날이고 지금 배우는 것에 비해 쉬운 것이기는 하지만, 프로그램을 짜고 실행할 때 희열을 느꼈다. 최근에도 그랬다. 그런데 지금은 그렇지 않을 때도 있다. 그렇다면 나에게 개발 일이 맞지 않는 것일까? 어떤 분야든 깊게 파고들다보면 지치는 부분이 생긴다고 하는데, 지금인 것일까? 지금까지 와서, 취업할 때가 다 되어서 이런 고민을 한다고? 이미 무난한 인생을 살고 있지 않은데 무슨 소용인가. 이렇게 된 거 여기저기 부딪쳐 보는 수밖에 없다. 그러니까 알아봐야 한다. 그렇다고 내가 개발자가 되기 위한 행동을 멈추지 않을 것이다. 지금은 나는 개발자가 되고 싶다고 생각하고, 개발자가 나에게 맞다고 믿고 있으니까.
2. 느낀 점
    1. 나에게 부족한 것들이 엄청나다!
        - 절대적인 시간 투자 부족: 하루에 프로그래밍을 공부하는 시간이 몇 시간인가? 물었는데 적을 때는 1시간, 많을 때는 끽해야 3시간이다. 나는 내 상황을 정확히 파악하지 못하고 실력도 거의 없는 것임에도 불구하고 현재 행동은 매우 굼뜨다. 나는 나의 문제를 개선하는데 충분한 노력을 하지 않고 있다. 한심하기 짝이 없다. 바뀌어야 한다.
    2. 해야 할 것들이 생겼다.
        1. 개발자는 어떤 사람이 하는 게 맞는지 찾아보고 나에게 어떤 치명적인 문제가 있나? 나는 개발자를 하는 게 맞을까? 확인하기.
        2. Spring 개념 복습
        3. 강의 영상 하나하나 따라가면서 나만의 방식으로 재구성하기
            - 아톰 보다는 노션이 가독성이 좋으므로, 노션으로 빠르게 의사코드로 프로그램의 구조를 분석해보자.
