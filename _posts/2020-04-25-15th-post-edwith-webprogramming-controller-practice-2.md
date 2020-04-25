---
title: "edwith 웹프로그래밍:  Spring MVC를 이용한 웹페이지 작성 실습 (2)"
date: 2020-04-25 22:40:00 +09:00
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
#### 3. Spring MVC를 이용한 웹 어플리케이션 작성 실습
##### 3.1 Controller 작성 실습 (1/3)

---
---
---
### TODAY 2020-04-25
# 3. Spring MVC를 이용한 웹 어플리케이션 작성 실습
## 3.2 Controller 작성 실습 (2/3) - 값 하나하나 전달? No! DTO를 만들 수 있어요!
### 실습 목표
1. http://localhost:8080/mvcexam/userform 으로 요청을 보내면 이름, email, 나이를 물어보는 폼이 보여진다.
2. 폼에서 값을 입력하고 확인을 누르면 post 방식으로 http://localhost:8080/mvcexam/regist에 정보를 전달하게 된다.
3. regist에서는 입력받은 결과를 콘솔 화면에 출력한다.

> #### /mvcexam/src/main/webapp/WEB-INF/views/userform.jsp
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
<form method="post" action="regist">
name: <input type="text" name="name"><br>
email: <input type="text" name="email"><br>
age: <input type="text" name="age"><br>
<input type="submit" value="확인">
</form>
</body>
</html>
```

---

> #### /mvcexam/src/main/java/kr/or/connect/mvcexam/controller/UserController.java
```java
package kr.or.connect.mvcexam.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import kr.or.connect.mvcexam.dto.User;

@Controller
public class UserController {
	@RequestMapping(path="/userform", method=RequestMethod.GET)
	public String userform() {
		return "userform"; // view name
	}

	@RequestMapping(path="/regist", method=RequestMethod.POST)
	public String regist(@ModelAttribute User user) {
		// @ModelAttribute를 사용해 DTO 처럼 데이터 뭉치를 만들자!
		// 스프링 MVC가 알아서 일치하는 이름의 값들을 주르르 받아와서 User 객체를 생성하고 그 값들을 넣어준다.

		System.out.println("사용자가 입력한 user 정보입니다. 해당 정보를 이용하면 코드가 와야 합니다.");
		System.out.println(user);
		return "regist";
	}
}
```

---

> #### /mvcexam/src/main/java/kr/or/connect/mvcexam/dto/User.java
```java
package kr.or.connect.mvcexam.dto;

public class User {
	private String name;
	private String email;
	private int age;


	public void setName(String name) {
		this.name = name;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public void setAge(int age) {
		this.age = age;
	}

	public String getName() {
		return name;
	}
	public String getEmail() {
		return email;
	}
	public int getAge() {
		return age;
	}

	@Override
	public String toString() {
		return "User [name=" + name + ", email=" + email + ", age=" + age + "]";
	}
}
```

---

> #### /mvcexam/src/main/webapp/WEB-INF/views/regist.jsp
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
<h2>성공적으로 등록되었습니다.</h2>
</body>
</html>
```


## 3.2 Controller 작성 실습 (3/3) - PathVariable로 들어왔을 때 요청 처리
### 실습 목표
1. http://localhost:8080/mvcexam/goods/{id} 으로 요청을 보낸다.
2. 서버는 id를 콘솔에 출력하고, 사용자의 브라우저 정보를 콘솔에 출력한다.
3. 서버는 HttpServletRequest를 이용해서 사용자가 요청한 PATH 정보를 콘솔에 출력한다.

> #### /mvcexam/src/main/webapp/WEB-INF/views/goodsById.jsp
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
id: ${id } <br>
user_agent: ${userAgent } <br>
path: ${path } <br>
</body>
</html>
```

---

> #### /mvcexam/src/main/java/kr/or/connect/mvcexam/controller/GoodsController.java
```java
package kr.or.connect.mvcexam.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestHeader;

@Controller
public class GoodsController {
	// path variable로 들어왔을 떄 요청 처리
	@GetMapping("/goods/{id}")
	public String getGoodsById(@PathVariable(name="id") int id, @RequestHeader(value="User-Agent", defaultValue="myBrowser") String userAgent, HttpServletRequest request, ModelMap model) {
		String path = request.getServletPath();

		System.out.println("id: "+ id);
		System.out.println("user_agent: "+ userAgent);
		System.out.println("path: "+ path);

		model.addAttribute("id", id);
		model.addAttribute("userAgent", userAgent);
		model.addAttribute("path", path);
		return "goodsById";
	}

}
```

---
---
---

### 2020-04-25 배운 것과 느낀 점
- alt+shift+y를 누르면 이클립스 코드가 자동 줄바뀜이 된다.

실제 투자 시간: 1시간, 그저 코드를 따라 치기만 했다. 실제로 이해한 것은 얼마나 될까? 다만, 어제 배운 것이 어렴풋이 기억나고 흐름은 전혀 어색하지 않았다. 그래도 ... 아마 머릿속에 제대로 정리가 안 되어 있을 것이다.  
절대적인 시간이 너무 적다. 학교 과제를 미리미리 했어야 했는데 안타깝다. 비전공자인 만큼 시간 관리가 더 철저해야 한다.

---
---
---
