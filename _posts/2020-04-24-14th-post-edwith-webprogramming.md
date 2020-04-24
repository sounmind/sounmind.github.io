---
title: "edwith 웹프로그래밍:  Spring MVC를 이용한 웹페이지 작성 실습 (1)"
date: 2020-04-24 22:48:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

(지난 이야기)
#### 1. Spring MVC
##### 1.1 MVC란?
##### 1.2 Java에서 MVC Model의 두 가지 형태
#### 2. Spring MVC 구성요소와 기본 동작 흐름
##### 2.1 Spring MVC 기본 동작 흐름
##### 2.2 Spring MVC 구성요소

---
---
---

### TODAY 2020-04-24

# 3. Spring MVC를 이용한 웹 어플리케이션 작성 실습
## 3.1 Controller 작성 실습 (1/3)
### Controller 작성 실습 목표

1. 웹 브라우저에서 ```http://localhost:8080/mvcexam/plusform``` 이라고 요청을 보내면 서버는 웹 브라우저에게 2개의 값을 입력 받을 수 있는 입력 창과 버튼이 있는 화면을 출력한다.
2. 웹 브라우저에게 2개의 값을 입력하고 버튼을 클릭하면 ```http://localhost:8080/mvcexam/plus``` URL로 2개의 입력 값이 POST 방식으로 서버에게 전달된다. 서버는 2개의 값을 더한 후, 그 결과 값을 JSP에게 request scope로 전달하여 출력한다.

#### 3.1.1 maven 프로젝트를 만든다.
#### 3.1.2 ```pom.xml```에 필요한 부분들을 추가한다.
#### 3.1.3 Spring MVC를 사용하기 위해 DispatcherServlet을 FrontController로 설정하기
##### DispatcherServlet을 FrontController로 **```web.xml```파일에서 설정하기:**
##### Controller(Handler) 클래스 작성하기
##### ```/mvcexam/src/main/java/kr/or/connect/mvcexam/config/WebMvcContextConfiguration.java```을 작성한다.
##### ```/mvcexam/src/main/webapp/WEB-INF/web.xml```을 작성한다.
##### 테스트 해보기 위해 WEB-INF 폴더 아래 views 폴더를 만들고 main.jsp 파일을 작성한다.

---
---

## 3.1 Controller 작성 실습
### Controller 작성 실습 목표
1. 웹 브라우저에서 **```http://localhost:8080/mvcexam/plusform```** 이라고 **요청** 을 보내면 **서버** 는 **웹 브라우저** 에게 2개의 값을 입력 받을 수 있는 입력 창과 버튼이 있는 화면을 **출력** 한다.
2. 웹 브라우저에게 2개의 값을 입력하고 버튼을 클릭하면 **```http://localhost:8080/mvcexam/plus``` URL로** 2개의 입력 값이 **POST 방식으로 서버에게 전달된다.** **서버는** 2개의 값을 더한 후, **그 결과 값을 JSP에게 request scope로 전달하여 출력한다.**

### 실습
1. 입력 두 개 받아내는 plusForm.jsp 파일 만들기

> #####  /mvcexam/src/main/webapp/WEB-INF/views/plusForm.jsp

```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
<!-- aciton에다가 원하는 url을 지정하면 이 url로 서버에 post 방식으로 요청 -->
<form method="post" action="plus">
value1 : <input type="text" name="value1"><br>
value2 : <input type="text" name="value2"><br>
<input type="submit" value="확인">
</form>
</body>
</html>
```

2. plusform으로부터 요청을 받았을 때 처리해서 출력할 plusResult.jsp 만들기

> ##### /mvcexam/src/main/webapp/WEB-INF/views/plusResult.jsp

```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
${value1} 더하기 ${value2} (은/는) ${result} 입니다.
</body>
</html>
```

3. **핵심!**: (따로 패키지에) 컨트롤러 클래스(PlusController.java) 만들기
    - 3.1. 컨트롤러라는 것을 나타낼 **@Controller 어노테이션** 붙이기

    자, 다시 실습 목표를 살펴보면, 컨트롤러는 요청 2개를 받아서 처리해야 한다.

    - 3.2. 요청1: **plusform url로 요청(get 방식)이 들어오면, 2개의 값 입력 받을 수 있는 입력 창과 버튼이 있는 화면 출력**
    - 3.3. 요청2: **plus url로 요청이 들어오면, 2개의 입력 값이 post 방식으로 서버에 전달되어, 서버는 그것을 더하고 결과 값을 jsp에게 request scope로 전달** 후 출력  

#### 그리고 아래 어노테이션을 한 번 살펴보자.
    
#### @RequestMapping
- Http 요청과 이를 다루기 위한 Controller의 메소드를 연결하는 어노테이션
- Http Method와 연결하는 방법은 다양하다
- @RequestMapping("/users", method=RequestMethod.POST)
- Spring 4.3 버전부터 사용 가능
  - @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping
- 그렇다. 이 어노테이션(@RequestMapping)을 사용해야 한다는 것을 알 수 있다.

#### Spring MVC가 지원해주는 것들
또한 Spring MVC가 지원하는 **Controller 메소드 인수 타입**, **메소드 인수 어노테이션**, **메소드 리턴 값** 의 종류가 다양하다. 선언만 해서 사용할 수 있도록 Spring이 만들어주기 때문에 잘 활용해야 한다.

#### 예: @RequestParam
- Mapping된 메소드의 Argument에 붙일 수 있는 어노테이션
- @RequestParam의 name에는 http parameter의 name과 맵핑이 된다.
- 예: html form tag input의 name 속성과 맵핑된다.
- @RequestParam의 required: 필수인지 아닌지 판단.

#### 예(2): @PathVariable    
- url에 '?변수=값' 방식을 사용할 때, 넘겨온 값을 받기 위해 사용된다.
- @RequestMapping의 path에 변수명을 입력받기 위한 place holder가 필요하다.
- place holder의 이름과 PathVariable의 name 값이 같으면 맵핑된다.
- required 속성은 default true이다.

#### 예(3): @RequestHeader
- 요청 정보의 헤더 정보를 읽어 들일 때 사용한다.
- @RequestHeader(name="헤더명") String 변수명

> 변수명을 잘 입력하는 것이 중요하다

> ##### /mvcexam/src/main/java/kr/or/connect/mvcexam/controller/PlusController.java

```java
package kr.or.connect.mvcexam.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class PlusController {
	@GetMapping(path="/plusform")
	// view name 을 넘기기: ModelAndView라는 객체를 리턴하거나 String으로 간단하게 뷰 이름만 넘겨줄 수 있다.
	// 요청처리 참고: https://sounmind.github.io/programming/12th-post/#2213-%EC%9A%94%EC%B2%AD-%EC%B2%98%EB%A6%AC
	public String plusform() {
		// plusform url로 요청(get 방식)이 들어오면,
		// 2개의 값 입력 받을 수 있는 입력 창과 버튼이 있는 화면을 출력하도록 view name 리턴
		return "plusForm";
	} // Config 클래스에 가서, @Bean에 의해 파일 경로에 뷰 이름을 붙여 뷰 파일을 찾아서 출력할 것

	@PostMapping(path="/plus")
	public String plus(@RequestParam(name="value1", required=true) int value1, @RequestParam(name="value2", required=true) int value2, ModelMap modelMap) {
		// plus url로 요청이 들어오면, 2개의 입력 값이 post 방식으로 서버에 전달되어,
		// 서버는 그것을 더하고 결과 값을 해당 jsp에게 request scope로 전달

		int result = value1 + value2;

		// HttpServletRequest을 받아서 직접 전달할 수 있겠지만, 종속되는 것을 피하기 위해 modelMap 사용한다
		// Spring에서 제공하는 modelMap이라는 객체를 사용하면, 알아서 request scope에 맵핑 시켜준다
		modelMap.addAttribute("value1", value1);
		modelMap.addAttribute("value2", value2);
		modelMap.addAttribute("result", result);
		return "plusResult"; // view name

	}

}
```

---
---
---

### 2020-04-24 배운 것과 느낀 점
오늘은 아침에 늦게 일어나서 일과가 아예 저녁으로 늦춰서 이 공부도 밤에 하게 되었다.
29분 분량의 영상을 보며 공부했다.
종속의 개념이 궁금하다. Spring을 쓴다면, 아예 Spring만 쓰는 것이 종속되지 않는 것인가?
책 Clean Code 에서 말하는 좋은 코드를 다시 살펴봐야겠다.
그 이전에, Spring을 왜 쓰게 되었는지, 근본적인 이해도 계속 수시로 머릿속에 집어넣어줘야 한다.

---
---
---

### 3.2 Controller 작성 실습 (2/3)
### 3.3 Controller 작성 실습 (3/3)
