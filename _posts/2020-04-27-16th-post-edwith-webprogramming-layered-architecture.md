---
title: "edwith 웹프로그래밍: 레이어드 아키텍처"
date: 2020-04-27 23:06:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

[노션(Notion)에서 포스팅 보기](https://www.notion.so/2020-04-27-layered-architecture-568d4fc31fb24d2f94bcac6dbe634d67)

# 레이어드 아키텍처

---

---

## 1. 컨트롤러와 서비스

---

비지니스 메소드를 별도의 서비스 객체에서 구현하도록 하고 컨트롤러는 Service 객체를 사용하도록 한다.

![1](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/2020-04-27-1.png?raw=true)

## 2. 서비스 객체란?

---

비지니스 로직을 수행하는 메소드를 가지고 있는 개체를 서비스 객체라고 한다.

보통 하나의 비지니스 로직은 하나의 트랜잭션으로 동작한다.

## 3. 트랜잭션이란?

---

트랜잭션은 하나의 논리적인 작업을 의미한다.

트랜잭션의 특징은 크게 4가지로 구분된다.

1. **원자성**
    - 전체가 성공 거나 전체가 실패하는 것. 마지막 단계까지 성공해야 성공한 것. 중간에 실패하면 전체가 실패한다.
    - 전체 5단계 중 4단계에서 오류가 발생했다면, 앞의 작업들을 모두 원래대로 복원을 시켜야 한다. 이를 rollback 이라고 한다. 5단계까지 모두 성공했을 때 정보를 반영하는 것을 commit이라고 한다. rollback 또는 commit이 일어나는 것이 하나의 트랜잭션 처리이다.
2. **일관성**
    - 트랜잭션의 작업 처리 결과가 항상 일관성이 있어야 한다.
    - 트랜잭션이 진행되는 동안에 데이터가 변경되더라도 업데이트된 데이터로 트랜잭션이 진행되는 것이 아니라, 처음에 트랜잭션을 진행 하기 위해 참조한 데이터로 진행된다.
3. **독립성**
    - 둘 이상의 트랜잭션이 병행 실행되고 있을 경우에 어느 하나의 트랜잭션이라도 다른 트랜잭션의 연산에 끼어들 수 없다. 하나의 특정 트랜잭션이 완료될 때까지, 다른 트랜잭션이 특정 트랜잭션의 결과를 참조할 수 없다.
4. **지속성**
    - 트랜잭션이 성공적으로 완료되었을 경우, 결과는 영구적으로 반영되어야 한다.

## 4. JDBC 프로그래밍에서 트랜잭션 처리 방법

---

1. DB에 연결된 후 Connection 객체의 `setAutoCommit` 메소드에 false를 파라미터로 지정한다.
2. 입력, 수정, 삭제 SQL이 실행한 후 모두 성공했을 경우 Connection이 가지고 있는 `commit()`메소드를 호출한다.

## 5. Spring MVC에서 트랜잭션 처리 방법

---

1. `@EnableTransactionManagement`  어노테이션을 사용하여 Spring Java Config 파일에서 트랜잭션을 활성화.
2. Java Config를 사용하게 되면 `PlatformTransactionManager` 구현체를 모두 찾아서 그중에 하나를 매핑에 사용한다.
3. 특정 트랜잭션 매니저를 사용하고자 한다면, `TransactionManagementConfigurer` 를 Java Config 파일에서 구현하고 원하는 트랜잭션 매니저를 리턴하도록 한다.
4. 그렇지 않다면, 특정 트랜잭션 매니저 객체를 생성 시 `@Primary`어노테이션을 지정한다.

## 6. 서비스 객체에서 중복으로 호출되는 코드의 처리

---

- 데이터 엑세스 메소드를 별도의 Repository(Dao) 객체에서 구현하도록 하고 서비스는 Repository 객체를 사용하도록 한다.

## 7. 레이어드 아키텍처

---

![2](https://github.com/sounmind/sounmind.github.io/blob/master/_posts/img/2020-04-27-2.png?raw=true)

### Layers

1. **Presentation**
    - 컨트롤러 객체가 동작하게 하면 된다.
    - 현재 수업(Web)에서는 Presentation Layer을 화면에 보여지게 만들 것이지만, 이 레이어를 플랫폼에 맞게 바꾸어서 사용할 수도 있다. → 재사용, 유지보수 측면에서 Presentation과 Service, Repository의 설정을 분리하여 관리하는 것이 좋다.
2. **Service**
    - 비즈니스 메서드를 가지고 있는 서비스 객체가 동작하게 하면 된다.
    - 해당 Repository의 DAO 사용
3. **Repository**
    - 데이터베이스 작업 수행

## 8. 설정의 분리

---

1. Spring 설정 파일을 프레젠테이션 레이어쪽과 나머지를 분리할 수 있다.
2. `web.xml`파일에서 프레젠테이션 레이어에 대한 스프링 설정은 `DispatcherServlet`이 읽도록 하고, 그 외의 설정은 `ContextLoaderListener`를 통해서 읽도록 한다.
3. `DispatcherServlet`을 경우에 따라서 2개 이상 설정할 수 있는데 이 경우에는 각각의 `DispatcherServlet`의 `ApplicationContext`가 각각 독립적이기 때문에 각각의 설정 파일에서 생성한 빈을 서로 사용할 수 없다.
4. 위의 경우와 같이 동시에 필요한 빈은 `ContextLoaderListener`를 사용함으로써 공통으로 사용하게 할 수 있다.
5. `ContextLoaderListner`와 `DispatcherServlet`은 각각 `ApplicationContext`를 생성한다.
    1. `ContextLoaderListener`가 생성하는 `ApplicationContext`가 root 컨텍스트가 된다.
    2. `DispatcherServlet`이 생성한 인스턴스는 root 컨텍스트를 부모로 하는 자식 컨텍스트가 된다. 자식 컨텍스트들은 root 컨텍스트의 설정 빈을 사용할 수 있다.

---

---

---

## 2020-04-27 배운 것과 느낀 점

---

1. 15분 정도의 짧은 이론 강의였기에 무난하게 들을 수 있었고, 개념에 집중했다. 아직까지는 어려운 줄 모르겠고(너무 모르기에) 직접 실습해보고 문제에 부딪쳐 봐야 그 난이도를 체감할 수 있을 것 같다.
2. 사실 본 강의보다 노션(Notion)과 깃허브를 어떻게 연동할지 고민이었다. 본질에 집중해야 하건만 안타깝다.
