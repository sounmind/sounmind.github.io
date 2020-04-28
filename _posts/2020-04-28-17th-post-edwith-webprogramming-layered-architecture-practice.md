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

## 4. 실제로 만들어보기, index.jsp에서 리다이렉트까지

### Maven Project: guestbook 생성

1. **pom.xml에 필요한 libraries 추가하고 maven 업데이트 하기**
    - properties
        1. spring version 정보 추가
        2. json 이용하기 위한 jackson library version 정보 추가
    - dependency (artifact ids below)
        1. spring-context
        2. spring-webmvc
        3. javax.servlet-api (Servlet)
        4. javax.servlet.jsp-api
        5. jstl
        6. spring-jdbc
        7. spring-tx (트랜지션)
        8. mysql-connector-java (mysql 드라이버)
        9. commons-dbcp2 (data source 사용)
        10.  jackson-databind (json 사용)
        11. jackson-datatype-jdk8 (json 사용)
    - Build - plugin
        1. maven-compiler-plugin (jdk 1.8 사용)
2. Navigator → guestbook → .settings → ~facet.core.xml → web module 을 version="3.1"로 수정.
3. guestbook → properties → project facets → Dynamic Web Module version 이 3.1로 바뀌어있는지 확인.
4. 설정 파일: src/main/main/java 아래에 kr.or.connect.guestbook.config 패키지 만들기
    1.  Dispatcher Servlet이 읽어들이는 설정 파일 →**웹 레이어 설정 파일:**  `WebMvcContextConfiguration` 생성 (`WebMvcConfigurerAdapter` 상속 → 내가 원하는 설정들을 하기 위해)
        1. `@Configuration` → 설정이라는 것을 알려주기
        2. `@EnableWebMvc` → 기본적인 설정들을 자동으로 해주기 위해
        3. `@ComponentScan` → 컨트롤러를 알아서 읽어오기 위해
        4. 필요한 메서드들 @Override
            1. `addResourceHandlers`
                - URL 요청이 **특정한 방식**으로 들어오는 것들을 **특정한 폴더에서 읽어오도록** 처리
            2. `configureDefaultServletHandling`
                - `configurer.enable();`→ default servlet handler를 사용하게 한다.
                - 매핑 정보가 없는 URL 요청이 들어왔을 때, Spring의 default servlet handler가 처리하도록 한다.
                - 매핑이 없는 URL이 넘어왔을 때 WAS의 default servlet이 static한 자원을 읽어서 보여줄 수 있게끔 해주는 설정
            3. `addViewController`
                - 특정 URL에 대한 처리를 컨트롤러 클래스를 작성하지 않고 매핑할 수 있도록 해준다.
                - "/"라고 요청이 들어오면 "index"라는 이름의 뷰로 보여준다.
            4. `InternalResourceViewResolver`
                - 적절한 view resolver가 실제로 뷰의 이름을 가지고 어떤 뷰인지에 대한 정보를 찾아낼 수 있게 한다.
                - resolver에다 Prefix와 Suffix를 지정하게 함으로써 index가 들어왔을 때 /WEB-INF/views 라는 디렉터리 밑에 있는 index.jsp를 출력한다.
    2. 데이터베이스 관련 설정: `DBConfig` ← `TransactionManagementConfigurer` 구현(Implements) ← 사용자 간의 트랜잭션 처리를 위한 `PlatformTransactionManager`을 설정하기 위한 조건(1)
        1. `@Configuration`
        2. `@EnableTransactionManagement` → 트랜잭션과 관련된 설정을 자동으로 해준다.
        3. 필요한 메서드 @Override
            1.  `annotationDrivenTransactionManager` ← **사용자 간의 트랜잭션 처리를 위한 `PlatformTransactionManager`을 설정하기 위한 조건(2)**: 해당 메서드에서 트랜잭션을 처리할 `PlatformTransactionManager()` 객체를 반환하게 하면 된다.
                - 현재 `transactionManager()`를 호출하고 있음.
        4. `@Bean`
            1. `transactionManager()`  메서드
                - `PlatformTransactionManager()` 를 리턴하고 있다.
            2. 데이터베이스 연결하기 위해 사용했었던 DataSource Bean 등록
    3. **비지니스, 레파지토리 레이어 설정 파일:** `ApplicationConfig` 작성
        1. `@Configuration`
        2. @ComponentScan → basePackages에 각각의 패키지를 지정하여 Dao나 Service에 구현되어 있는 컴포넌트들을 읽어온다.
        3. @Import → DBConfig.class → Dao와 Service를 사용할 때 DBConfig에서 사용되고 있는 것들을 쓰기 위해
    4. web.xml 설정
        1. 웹 모듈 2.3으로 지정되어 있는 것 version="1.0"으로 수정
        2.
        3. \<context-param\>
            1. \<param-name\>contextClass
                - \<param-value\>org.springframework.web.context.support.**AnnotationConfigWebApplicationContext → 리스너가 실행될 때 이런  ApplicationContext 를 이용하겠다는 뜻**
            2. \<param-name\>**contextConfigLocation**
                - \<param-value\>kr.or.connect.guestbook.config.**ApplicationConfig** → **listener가 실행될 때 참고하는 파일 이름이 이 이름과 같아야 한다.**
        4. \<listener\>
            1. \<listener-class\>org.springframework.web.context.**ContextLoaderListener → 서버가 올라갈 때, Context가 로딩되는 이벤트가 일어났을 때 실행된다.**
                - **이 리스너가 실행될 때, context-param에 등록된 이름의 파일을 가져다 사용한다.**
        5. \<servlet\>
            1. \<servlet-class\>org.springframework.web.servlet.DispatcherServlet → 모든 요청을 받을 때, 이 DispatcherServlet 클래스가 받는다. → **프론트 서블릿으로 등록**
        6. \<init-param\>
            1. \<param-name\>**contextClass**
                - \<param-value\> org.springframework.web.context.support.**AnnotationConfigWebApplicationContext** →**DispatcherServlet을 실행할 때 이런 ApplicationContext 를 이용하겠다는 뜻**
            2. \<param-name\>**contextConfigLocation**
                - \<param-value\>kr.or.connect.guestbook.config.**WebMvcContextConfiguration** → **DispatcherServlet이 실행될 때 WebMvcContextConfiguration에 들어있는 설정들을 참고하겠다는 뜻.**
        7. \<servlet-mapping\>
            - **\<url-pattern\>/\<url-pattern\> → 모든 요청을 다 받는다는 뜻**
        8. \<filter\> 요청이 수행되기 전, 응답이 나가기 전 수행된다.
            1. \<filter-name\>encodingFilter
                1. \<filter-class\>org.springframework.web.filter.ChracterEncodingFilter
                2. **\<url-pattern\>/*\</url-pattern\> → 모든 요청에 대하여 필터 적용**
5. webapp → WEB-INF → views 디렉터리 생성, index.jsp 파일 작성
    1. 방명록 요구사항 → "index"라는 요청이 들어왔을 때 "list"라는 요청으로 redirect 한다.
        - `response.sendRedirect("list");`

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
        4. 이제까지 잊고 있었던 궁금한 것들 리스트 만들기
            1. 왜 하필 Maven project를 만드는 것일까?
            2. WAS는 무엇이었지? 웹 애플리케이션 서버. 미들웨어.
                - 웹 애플리케이션과 서버 환경을 만들어 동작시키는 기능을 제공하는 소프트웨어 프레임워크이다. [웹 애플리케이션 서버](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98_%EC%84%9C%EB%B2%84)

            3. 자바에서 **구현(Implements)** 의 의미는 무엇일까? 자바는 어설프게 배워 중요한 개념을 이해하지 못하는 것 같다. 자바의 정석 책을 봐야겠다!
