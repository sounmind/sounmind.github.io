---
title: "edwith 웹프로그래밍:  Spring MVC를 이용한 웹페이지 작성 실습"
date: 2020-04-21 16:28:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

(지난 이야기)
## 1. Spring MVC
### 1.1 MVC란?
### 1.2 Java에서 MVC Model의 두 가지 형태
## 2. Spring MVC 구성요소와 기본 동작 흐름
### 2.1 Spring MVC 기본 동작 흐름
### 2.2 Spring MVC 구성요소

---
---
---

### TODAY 2020-04-21
# 3. Spring MVC를 이용한 웹 어플리케이션 작성 실습
## 3.1 Controller 작성 실습 (1/3)
### Controller 작성 실습 목표
1. 웹 브라우저에서 ```http://localhost:8080/mvcexam/plusform``` 이라고 요청을 보내면 서버는 웹 브라우저에게 2개의 값을 입력 받을 수 있는 입력 창과 버튼이 있는 화면을 출력한다.
2. 웹 브라우저에게 2개의 값을 입력하고 버튼을 클릭하면 ```http://localhost:8080/mvcexam/plus``` URL로 2개의 입력 값이 POST 방식으로 서버에게 전달된다. 서버는 2개의 값을 더한 후, 그 결과 값을 JSP에게 request scope로 전달하여 출력한다.

#### 3.1.1 maven 프로젝트를 만든다.

> archetypes의 Artifact id은 ```maven-archetype-webapp```으로 한다.

#### 3.1.2 ```pom.xml```에 필요한 부분들을 추가한다.

```java    
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
 <groupId>kr.or.connect</groupId>
  <artifactId>mvcexam</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>mvcexam Maven Webapp</name>
  <url>http://maven.apache.org</url>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<spring.version>4.3.14.RELEASE</spring.version>
	</properties>
	<dependencies>

		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<!-- servlet -->
		<dependency>
	        <groupId>javax.servlet</groupId>
	        <artifactId>javax.servlet-api</artifactId>
	        <version>3.1.0</version>
	        <scope>provided</scope>
	        <!-- servlet라이브러리를 컴파일 시에만 사용되고 배포 시에는 사용되지 않는다는 것을 의미합니다. -->
	    </dependency>

	    <!-- jsp -->
	    <dependency>
	        <groupId>javax.servlet.jsp</groupId>
	        <artifactId>javax.servlet.jsp-api</artifactId>
	        <version>2.3.1</version>
	        <scope>provided</scope>    
	    </dependency>

	    <!-- jstl -->
	    <dependency>
	        <groupId>javax.servlet</groupId>
	        <artifactId>jstl</artifactId>
	        <version>1.2</version>
	    </dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<finalName>mvcexam</finalName>
		<plugins>
		<!-- jdk 1.8을 사용하기 위함 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.6.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

#### 3.1.3 Spring MVC를 사용하기 위해 DispatcherServlet을 FrontController로 설정하기
> Spring MVC에서 DispatcherServlet이 FrontController의 역할을 한다. 따라서 **DispatcherServlet을 FrontController로 설정** 해야 한다. 그 방법은 3가지가 있다.

1. **```web.xml``` 파일에 설정:**
2. ```javax.servlet.ServletContainerInitializer``` 사용 --> 수업에서 다루지 않음.
    - 서블릿 3.0 스펙 이상에서 ```web.xml``` 파일을 대신해서 사용할 수 있다.
3. ```org.springframework.web.WebApplicationInitializer``` 인터페이스를 구현해서 사용 --> 수업에서 다루지 않음.
    - Spring MVC는 ServletContainerInitializer를 구현하고 있는 SpringServletContainerInitializer를 제공한다.
    - SpringServletContainerInitializer는 WebApplicationInitializer 구현체를 찾아 인스턴스를 만들고 해당 인스턴스의 onStartup 메소드를 호출하여 초기화한다.
    - 단점: 처음 웹 애플리케이션이 구동되는 시간이 오래 걸릴 수 있다.`

> 첫 번째 방법에 대해 더 자세히 알아보도록 하자!

#### DispatcherServlet을 FrontController로 **```web.xml```파일에서 설정하기:**

  1. **spring 설정을 읽어들이도록 DispatcherServlet 설정**
      - 이전에 서블릿 하나 생성하면, servlet과 servlet mapping이라는 xml 태그에다 각각의 값들을 넣어줘서 설정했었다.
      - servlet-name은 servlet mapping이 갖고 있는 servlet-name과 일치하기만 하면 된다.
      - servelt-class가 실제로 사용자가 동작시킬 클래스를 의미한다. 이때 Spring이 제공하고 있는 DispatcherServlet을 이용할 것이다. 따라서 반드시 패키지 명을 포함해서 Spring이 제공하고 있는 클래스 명을 제대로 넣어줘야 한다.
      - init-param에서, DispatcherServlet의 경우 Spring에서 제공하고 있기 때문에 실제로 사용자가 무슨 일을 하고 싶은지에 대한 정보는 알 수가 없다. 따라서 init-param에는 그런 정보를 입력한다.

  2. **Java config spring 설정 읽어들이도록 DispatcherServlet 설정**
      - init-param에 xml 파일이 아니라 자바 클래스 이름을 넣는다. xml이 아니라 javaConfig 파일을 읽어온다.
      - 요청이 들어올 경우, 서블릿이었을 경우 url-pattern에다 사용자가 원하는 URL 주소를 넣는다.   
      - servlet-mapping에서 url pattern 부분이 '/'라고 설정되어 있다. 이 때문에, 어떤 특정한 하나의 요청만 받아들이는 것이 아니라, 모든 요청을 받게 된다.

- **DispatcherServlet이 읽어들여야 될 설정은 별도로 하게 된다.**
  - 이런 설정은 **Java config** 로 한다. DispatcherServlet은 해당 설정 파일을 읽어들여서 내부적으로 Spring 컨테이너인 ApplicationContext를 생성하게 된다.
  - ```kr.or.connect.webmvc.config.WebMvcContextConfiguration```
  - 여기서 사용하고 있는 어노테이션들을 상세히 살펴보자.
      1. **```@Configuration```**
          - ```org.springframework.context.annotion```의 Configuration 애노테이션과 Bean 애노테이션 코드를 이용하여 스프링 컨테이너에 새로운 빈 객체를 제공할 수 있다.

      2. **```@EnableWebMvc```**
          - DispatcherServlet의 RequestMappingHandlerMapping, RequestMappingHandlerAdapter, ExceptionHandlerExceptionResolver, MessageConverter 등 **Web에 필요한 빈들을 대부분 자동으로 설정 해준다.**
          - *xml로 설정할 때의 ```<mvc:annotion-driven/>```와 동일하다.*
          - 기본 설정 이외의 설정이 필요하다면 WebMvcConfigurerAdapter를 상속받도록 Java config class를 작성한 후, 필요한 메소드를 오버라이딩 하도록 한다.
              - 조금 더 깊게 알고 싶다면, EnableWebMvc의 소스코드를 살펴보면 된다. 해당 소스코드를 살펴보면, DelegatingWebMvcConfiguration을 import 하고 있는 것을 알 수 있다. 그 클래스는 WebMvcConfigurationSupport를 상속받고 있는 것을 볼 수 있다. 이 객체에 대한 소스를 찾아보면, 어떤 객체들이 기본으로 설정이 되어 있는지 조금 더 자세히 알 수 있을 것이다.

      3. **```@ComponentScan```**
          - ComponentScan 애노테이션을 이용하면 Controller, Service, Repository, COmponent 애노테이션이 붙은 클래스를 찾아 스프링 컨테이너가 관리하게 된다.
          - DefaultAnnotationHandlerMapping과 RequestMappingHandlerMapping구현체는 다른 핸들러 매핑보다 훨씬 더 정교한 작업을 수행한다. 이 두 개의 구현체는 애노테이션을 사용해 매핑 관계를 찾는 매우 강력한 기능을 가지고 있다. 이들 구현체는 스프링 컨테이너 즉 애플리케이션 컨텍스트에 있는 요청 처리를 빈에서 RequestMapping애노테이션을 클래스나 메소드에서 찾아 HandlerMapping 객체를 생성하게 된다. -> HandlerMapping은 서버로 들어온 요청을 어느 핸들러로 전달할지 결정하는 역할을 수행한다.
          - DefaultAnnotationHandlerMapping은 DispatcherServlet이 기본으로 등록하는 기본 핸들러 맵핑 객체이고, RequestMappingHandlerMapping은 더 강력하고 유연하지만, 사용하려면 명시적으로 설정해야 한다.

      4. **WebMvcConfigurerAdapter 클래스**
          - ```org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter```
          - @EnableWebMvc를 이용하면 기본적인 설정이 모두 자동으로 되지만, 기본 설정 이외의 설정이 필요할 경우 해당 클래스를 상속 받은 후, 메소드를 오버라이딩 하여 구현한다.

#### Controller(Handler) 클래스 작성하기
- ```@Controller``` 애노테이션을 클래스 위에 붙인다. 그렇다면 이것이 ComponentScan이 읽어들여서 스프링 컨테이너가 관리한다. 즉, 이 클래스가 컨트롤러라고 설정된다.
- 맵핑을 위해 ```@RequestMapping``` 애노테이션을 클래스나 메소드에서 사용한다.
  - 요청이 들어왔을 때 어떤 URL로 들어온 요청인지를 알아내서 실제로 처리해야 하는 컨트롤러가 뭔지, 그 컨트롤러에서 구현하고 있는 메서드가 뭔지 알아내기 위해 꼭 해야 한다.

#### ```@RequestMapping```
- Http요청과 이를 다루기 위한 Controller의 메소드를 연결하는 어노테이션
- Http Method와 연결하는 방법
- @RequestMapping("/users", method=RequestMethod.POST)
  - /users 라는 URL로 POST 방식으로 요청이 들어오면 method를 수행하겠다.
- Spring 4.3 버전부터는 아래와 같은 애노테이션 사용 가능
  - @GetMapping
  - @PostMapping
  - @PutMapping
  - @DeleteMapping
  - @PatchMapping

- Http 특정 헤더와 연결하는 방법
  -@
- Http Parameter와 연결하는 방법
  -@
- Content-Type Header와 연결하는 방법
  -@
- Accept Header와 연결하는 방법
  -@

---
---
---


### ```/mvcexam/src/main/java/kr/or/connect/mvcexam/config/WebMvcContextConfiguration.java```을 작성한다.

```java
package kr.or.connect.mvcexam.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages={"kr.or.connect.mvcexam.controller"})
// 중괄호 안에 패키지 안에 넣으면 해당 패키지 하나만 사용하면, 중괄호 없으면 패키지 어려 개 사용
// base package 지정해줘야함. 어느 패키지부터 찾아야할지 몰라서 수행이 잘 안 될 수 있다.

// 이 클래스는 DispatcherServlet이 실행될 때 설정을 읽어들이는 파일
public class WebMvcContextConfiguration extends WebMvcConfigurerAdapter {
	@Override // 이런 URL로 요청이 들어오면, 이렇게 찾아보세요!
	public void addResourceHandlers(ResourceHandlerRegistry registry) { // 들어오는 URL 요청을 애플리케이션 루트 디렉터리 아래에 있는 각각의 디렉토리를 만들어 놓고 사용하게 할 것.
        registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/META-INF/resources/webjars/").setCachePeriod(31556926);
        registry.addResourceHandler("/css/**").addResourceLocations("/css/").setCachePeriod(31556926);
        registry.addResourceHandler("/img/**").addResourceLocations("/img/").setCachePeriod(31556926);
        registry.addResourceHandler("/js/**").addResourceLocations("/js/").setCachePeriod(31556926);
	}

	// default servlet handler를 사용하게 합니다.
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
		// configureDefaultServletHandling
		// DefaultServletHandlerConfigurer 객체의 enable이라는 메서드 호출 -> DefaultServletHandler를 사용하도록 함.
		// DefaultServletHandler은 WAS의 DefaultServlet에게 일을 넘기게 된다.
		// DefaultServlet이 정적(static)인 자원을 읽어서 보여준다.
	}


	@Override
	public void addViewControllers(final ViewControllerRegistry registry) {
		System.out.println("addViewControllers 가 호출됩니다. ");
		registry.addViewController("/").setViewName("main"); // 주소가 들어오면, main을 보여줘요!
		//main정보만으로는 뷰 정보를 찾을 수 없다.
		// 뷰 정보는 getInternalResourceViewResolver라는 메서드에서 설정된 형태로 뷰를 사용하게 된다.
	}

	@Bean
	public InternalResourceViewResolver getInternalResourceViewResolver() {
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/views/"); // 뷰정보 앞에다가 파라미터 정보를 붙여 넣어 주고
		resolver.setSuffix(".jsp"); // 뷰정보 뒤에다가 파라미더 정보를 붙여 넣어달라는 뜻.
		// 결국, 뷰 정보는 WEB-INF/views/main.jsp 라는 파일을 찾아주게 된다.
		return resolver;

	}
}
```

---

### ```/mvcexam/src/main/webapp/WEB-INF/web.xml```을 작성한다.

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app>

  <display-name>Spring JavaConfig Sample</display-name>

  <servlet>
    <servlet-name>mvc</servlet-name> <!-- 아래 서블릿 맵핑의 인자 때문에 같은 이름의 서블릿이 실행된다. -->
    <!-- 스프링이 제공하고 있는 DispatcherServlet을 FrontController로 지정할 것이다, 라는 내용의 서블릿 클래스가 아래 등록되어 있음 -->
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param><!-- Bean 공장, IoC와 DI를 적용하게 되는 클래스, 블로그 JDBC 실습(1) 참고할 것. -->
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param><!-- DispatcherServlet이 실행될 때 설정들을 읽을 파일을 지정하고 있다. -->
      <param-name>contextConfigLocation</param-name>
      <param-value>kr.or.connect.mvcexam.config.WebMvcContextConfiguration</param-value><!-- 항상 패키지 명을 포함해서 등록해야 한다. -->
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>mvc</servlet-name> <!-- 아래 URL이 들어오면 여기 이름과 같은 이름의 servlet이 실행된다 -->
    <url-pattern>/</url-pattern> <!-- '/'로 들어오면 모든 요청이 될 수 있다. -->
  </servlet-mapping>

</web-app>

<!-- 2020-04-23 원래 있던 코드 전체 주석 처리, edwith 홈페이지에서 복붙한 것이 위에 것. -->
<!-- 2020-04-23
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
version="3.1">
-->

<!-- 2020-04-21 위의 문서 변경 -->

<!-- 구버전
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
 -->
<!-- 2020-04-21 위의 변경 때문에  '<web-app>' 삭제-->

<!-- 2020-04-23
  <display-name>Archetype Created Web Application</display-name>
</web-app>
 -->
```

---

### 테스트 해보기 위해 WEB-INF 폴더 아래 views 폴더를 만들고 main.jsp 파일을 작성한다.

```java
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
<h1>main page입니다...!</h1>
</body>
</html>
```


### 2020-04-23 배운 것과 느낀 점
중간에 컴퓨터 사용자 이름을 바꾸는 바람에 당연한 듯 실행되던 톰캣 서버를 재설정 해줬다. 그 과정에서 내가 서버를 잘 이해하고 있나? 톰캣은 정확히 무엇이지? 물었을 때 제대로 대답할 수 없다는 것을 깨달았다. 졸음 참아가며 진도 나가면 뭐하나, 지나가면 다 까먹고 실질적인 실력은 없는 것이나 마찬가지인데.
1. 내가 왜 이 코드 작성을 왜 하고 있는지, 곰곰이 생각하며 이유를 따지자. 당연한 것을 당연하게 받아들이지 말자.
2. 깊은 학습과 장기 기억을 위해서는 복습이 생명이다. 수시로 복습하자.
3. 오후 3시 ~ 6시에 공부하는 것은 현재 몸의 주기 상 집중이 힘드므로, 아침에 공부하자.



---
---
---
### 3.2 Controller 작성 실습 (2/3)
### 3.3 Controller 작성 실습 (3/3)
