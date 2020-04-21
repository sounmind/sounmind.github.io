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
> Spring MVC에서 DispatcherServlet이 FrontController의 역할을 한다. 따라서 **DispatcherServlet을 FrontController로 설정**해야 한다. 그 방법은 3가지가 있다.

- **```web.xml``` 파일에 설정:**
이전에 서블릿 하나 생성하면, servlet과 servlet mapping이라는 xml 태그에다 각각의 값들을 넣어줘서 설정했었다.
  - servlet-name은 servlet mapping이 갖고 있는 servlet-name과 일치하기만 하면 된다.
  - servelt-class가 실제로 사용자가 동작시킬 클래스를 의미한다. 이때 Spring이 제공하고 있는 DispatcherServlet을 이용할 것이다. 따라서 반드시 패키지 명을 포함해서 Spring이 제공하고 있는 클래스 명을 제대로 넣어줘야 한다.
  - init-param에서, DispatcherServlet의 경우 Spring에서 제공하고 있기 때문에 실제로 사용자가 무슨 일을 하고 싶은지에 대한 정보는 알 수가 없다. 따라서 init-param에는 그런 정보를 입력한다.


2. ```javax.servlet.ServletContainerInitializer``` 사용
3. ```org.springframework.web.WebApplicationInitializer``` 인터페이스를 구현해서 사용


### 3.2 Controller 작성 실습 (2/3)
### 3.3 Controller 작성 실습 (3/3)
