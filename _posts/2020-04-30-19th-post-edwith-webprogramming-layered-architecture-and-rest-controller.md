---
title: "edwith 웹프로그래밍: 레이어드 아키텍처 실습 - 방명록 만들기 (3)"
date: 2020-04-30 15:55:00 +09:00
categories: programming
tags: [web, boostcourse, programming]
---

# 레이어드 아키텍처 실습 3번째: 서비스 레이어

---

---

## 1. 서비스 레이어 패키지 생성

---

1. 패키지 `service`: 서비스 인터페이스만 가지고 있음
    1. 클래스 `GuestbookService.java` 인터페이스 GuestbookService
        - 비즈니스
            1. 방명록 정보를 페이지 별로 읽어오기.
            2. 페이징 처리를 위해서 전체 건 수 구하기
            3. 방명록 저장하기 (ip 정보를 인자로 전달, ip 정보는 LogDao를 통해 Log 테이블에 저장)
            4. id에 해당하는 방명록 삭제
2. 패키지 `service.impl`: 실제로 구현체를 가지고 있음. 인터페이스를 구현함.
    1. 클래스 `GuestbookServiceImpl.java` `implements GuestbookService` → GuestbookService를 구현하겠다는 뜻.
    마우스를 GuestbookServiceImpl에 가져다 대면 나오는 `Add unimplemented methods`를 눌러 해당 메서드들을 모두 Implement 한다.
    그리고나서 메서드들을 하나씩 구현하면 된다!
        1. `@Service` 어노테이션 → Service Layer에 해당하는 클래스라는 뜻
            - `@Autowired` 어노테이션 → GuestbookDao와 LogDao의 Bean 자동으로 등록
        2. 메서드 `getGuestbooks(Integer start)` → guestbook 방명록의 시작되는 부분의 목록을 가져온다.
            1. `@Transactional` 어노테이션 → 읽기만 하는 메서드라는 뜻
            2. `guestbookDao.selectAll(start, GuestbookService.LIMIT);` → 두 번째 인자는 한 페이지에 보여주는 방명록의 개수이다.
            3. `return list;`
        3. 메서드 `deleteGuestbook(Long id, String ip)`
            1. `@Transaction(readOnly=false)` 어노테이션 → 읽기 전용이 아니기 때문에 따로 설정해줘야 한다.
            2. `int deleteCount = guestbookDao.deleteById(id);` → guestbookDao에서 삭제 메서드 가져오기.
            3. `Log log = new Log();` → 삭제 했을 때 log에 입력해야 함. 요구사항대로 처리

                `log.setIp(ip);
                log.setMethod("delete");
                log.setRegdate(new Data());` → 요구사항대로 log 테이블에서 요구하는 정보를 채웠다.
                `logDao.insert(log);` → 데이터 삽입 실행

            4. `return deleteCount;`
        4. 메서드 `addGuestbook(Guestbook guestbook, String ip)` → guestbook에는 클라이언트 - 컨트롤러에서 받아온 정보가 담겨있다.
            1. `@Transaction(readOnly=false)` 어노테이션 → 읽기 전용이 아니기 때문에 따로 설정해줘야 한다.
            2. `guestbook.setRegdate(new Date());` → 등록날짜는 받아오지 않았기 때문에 따로 등록
            3. `Long id = guestbookDao.insert(guestbook);` → Dao로 데이터 입력 실행, Long 타입의 id가 리턴됨.
            guestbook.setId(id); → Log에다 id를 남기기 위해 값을 채워준다.
            4. `Log log = new Log();` → 입력 했을 때 log에 입력해야 함. 요구사항대로 처리

                `log.setIp(ip);
                log.setMethod("insert");
                log.setRegdate(new Data());` → 요구사항대로 log 테이블에서 요구하는 정보를 채웠다.
                `logDao.insert(log);` → 데이터 삽입 실행

            5. `return guestbook;`
        5. 메서드 getCount() → 페이징 처리. 전체 몇 건인지, 같이 얻어온다.
            - `return guestbookDao.selectCount();`

3. 테스트

---

- `@Transaction` 어노테이션의 신비로운 기능
    - 이 어노테이션이 붙어 있는 메소드는 실행되다가 중간에 예외가 생기면 그 지점 이전에 정상적으로 실행됐던 코드들도 덩달아 취소된다.
    - delete를 시험해보기 위해 long타입 정수를 넣어야 했는데, 숫자 뒤에 대문자 L을 붙여야 long타입으로 인식된다.

---

---

---

## 2. 컨트롤러 패키지 생성

---

1. 클래스 `GuestbookController` 생성
    1. `@Controller` 어노테이션을 사용해, 컴포넌트 스캔할 때 해당 빈을 찾아 사용할 수 있다.
    2. `@Autowired` 어노테이션
    `GuestbookService` 라고 이용할 서비스 이름을 적어준다.
    해당 서비스에서는 **2개의 메서드**를 처리하도록 한다.
    3. `@GetMapping(path="/list")` → path가 list로 들어왔을 때 처리한다.
    **메서드(1)** `public String list(@RequestParam(name="start", required=false, defaultValue="0") int start, ModelMap model)` → @RequestParam으로 start라는 값을 꺼내서 사용한다. 만약 값이 없다면, 디폴트 값으로 0을 준다.
        1. `List<Guestbook> list = guestbookService.getGuestbooks(start);` 서비스로부터 start를 넣어서 해당 방명록 목록을 얻어온다.
        2. `int count = guestbookService.getCount();` → 서비스가 가지고 있는 getCount()를 이용해 전체 방명록 수 얻어오기.
        `int pageCount = count / GuestbookService.LIMIT;` → 한 페이지당 보여주는 방명록 개수(LIMIT)으로 전체 방명록 개수(count)를 나눠 총 페이지 수(pageCount)를 구한다.
        `if(count % GuestbookService.LIMIT > 0)` → 나눈 값이 떨어지지 않을 경우, 페이지 수를 하나 늘린다.
        `pageCount++;`
        3. 페이지 수만큼 start의 값을 리스트로 저장, 예를 들어 페이지 수가 3이면, 0, 5, 10 으로 저장된다.
        즉, list?start=0, list?start=5, list?start=10 으로 링크가 걸린다.

            ```java
            List<Integer> pageStartList = new ArrayList<>();
            for(int i=0l i<pageCount; i++){
            	pageStartList.add(i * GuestbookService.LIMIT);
            }
            ```

            이 부분을 JSP에서 사용해야 하기 때문에, JSP에서 사용하도록 model에다가 필요한 부분들을 넣어준다.

            ```java
            model.addAttribute("list", list);
            model.addAttribute("count", count);
            model.addAttribute("pageStartList", pageStartList);
            return "list";
            ```

            이렇게 넣어준 부분을 list.jsp가 사용하면 된다. → views에다 list.jsp 생성

    4. `@PostMapping(path="/write")` → 방명록 입력하는 부분이다. 하지만 자세히는 모른다. 연구가 필요하다.
    **메서드(2)** `public String write(@ModelAttribute Guestbook guestbook, HttpServletRequest request)`
        1. `return "redirect:list";` → 입력을 완료하면 리스트를 리다이렉트해서 업데이트한다.

## 3. views 폴더에 `list.jsp` 생성

---

1. **`<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`** → JSTL 사용 위해 추가
2. 실제 화면 구성
    1. html
    title
    body
        1. `<h1>방명록`
        2. `<br>방명록 전체 수: ${count}`
        3. `<c:forEach items="${list}" var="guestbook">` → 컨트롤러에서 얻어온 방명록 리스트를 담고 있는 list에 있는 Guestbook 객체를 하나 꺼내서 var="guestbook"이라는 변수에 저장한다.**그리고 이를 반복한다.**
            1. EL을 이용해서 변수 guestbook에 담긴 리스트 정보를 출력한다.

            ```html
            ${guestbook.id}<br>
            ${guestbook.name}<br>
            ${guestbook.content}<br>
            ${guestbook.regdate}<br>
            ```

        4. 페이지 링크를 만든다. forEach란 도대체 ... ? '&nbsp'는 줄 바꿈 없는 공백이다.

        ```html
        <c:forEach items="${pageStartList}" var="pageIndex" varStatus="status">
        	<a href="list?start=${pageIndex}">${status.index+1}</a>&nbsp; &nbsp;
        </c:forEach>
        ```

        5. 방명록 입력 받을 수 있는 폼 태그

3. 테스트 해보기 → 프로젝트 실행 → 성공적으로 실행 됐다.

---

---

---

# RestController

---

---

## RestController란?

---

- Rest API 또는 Web API를 개발하기 위해서 스프링 MVC는 @RestController를 제공한다.
이전 버전의 @Controller와 @ResponseBody를 포함한다.
- Rest API와 Web API는 무엇인가?
      - [API](https://ko.wikipedia.org/wiki/API)
      - [Web API](https://en.wikipedia.org/wiki/Web_API)

- JSON이란?
      - [JSON](https://ko.wikipedia.org/wiki/JSON)

## MessageConverter

---

RestController를 사용하기 위해 중요한 개념: 아래와 같은 역할을 한다.
자바 객체와 HTTP 요청/응답 바디를 변환하는 역할;
예를 들면 **외부에서 전달 받은** **JSON 메서드**를 **내부에서 사용할 수 있는 개체로** **변환**하거나
**컨트롤러를 리턴한 객체가** **클라이언트에게** **JSON**으로 **변환**해서 **전달**할 수 있도록 하는 역할

@EnableWebMvc 어노테이션을 사용하게 되면 기본으로 제공된다.

### JSON 응답 문제

json으로 변환을 하기 위해 기본 MessageConverter는 Jackson 라이브러리를 사용하고 있다.
만약 Jackson 라이브러리가 제대로 추가되어 있지 않으면, JSON을 처리하는 Convertor가 등장하지 않기 때문에 500번 오류가 발생한다.
사용자가 임의의 메시지 컨버터를 사용하도록 하려면 WebMvcConfigurerAdapter의 configureMessageConverters 메소드를 오버라이딩 해야 한다.

### 질문

1. Web API에서 JSON 메시지를 자주 사용하는 이유는 무엇인가? → 쉽고 플랫폼 독립적이어서!
2. JSON 메시지의 장점
    1. JSON의 장점
        - JSON은 텍스트로 이루어져 있으므로, 사람과 기계 모두 읽고 쓰기 쉽다.
        - [프로그래밍 언어](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%96%B8%EC%96%B4)와 [플랫폼](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%8C%85_%ED%94%8C%EB%9E%AB%ED%8F%BC)에 독립적이므로, 서로 다른 시스템간에 객체를 교환하기에 좋다.
        - [자바스크립트](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8)의 문법을 채용했기 때문에, 자바스크립트에서 `eval` 명령으로 곧바로 사용할 수 있다. 이런 특성은 자바스크립트를 자주 사용하는 웹 환경에서 유리하다. 그러나 실질적으로 `eval` 명령을 사용하면 외부에서 악성 코드가 유입될 수 있다. [모질라 파이어폭스](https://ko.wikipedia.org/wiki/%EB%AA%A8%EC%A7%88%EB%9D%BC_%ED%8C%8C%EC%9D%B4%EC%96%B4%ED%8F%AD%EC%8A%A4) 3.5, [인터넷 익스플로러](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%EC%9D%B5%EC%8A%A4%ED%94%8C%EB%A1%9C%EB%9F%AC) 8, [오페라](https://ko.wikipedia.org/wiki/%EC%98%A4%ED%8E%98%EB%9D%BC_(%EC%9B%B9_%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80)) 10.5, [사파리](https://ko.wikipedia.org/wiki/%EC%82%AC%ED%8C%8C%EB%A6%AC_(%EC%9B%B9_%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80)), [구글 크롬](https://ko.wikipedia.org/wiki/%EA%B5%AC%EA%B8%80_%ED%81%AC%EB%A1%AC) 등 대부분의 최신 [웹 브라우저](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80)는 JSON 전용 파서 기능을 내장하고 있으므로 이런 기능을 사용하는 것이 더 안전할 뿐만 아니라 빠른 방법이다.

        - [JSON](https://ko.wikipedia.org/wiki/JSON#%EC%9E%A5%EC%A0%90)

## 실습 (guestbook 프로젝트에서)

1. jackson 라이브러리 추가
2. Rest Controller 사용을 위해 controller 패키지에 Rest Controller 클래스 GuestbookApiController 만들기
    1. `@RestController
    @RequestMapping(path="/guestbooks")
    public class GuestbookApiController`
        1. `@Autowired
        GuestbookService guestbookService;`
        2. `@GetMapping` → 패스가 따로 없는 이유는 위에 @RequestMapping이 붙어 있기 때문. **path가 /guestbooks라고 시작하면서 GET 방식으로 요청이 들어오면** 이 메서드를 실행한다는 의미.
        `public Map<String, Object> list(@RequestParam(name="start", required=false, default Value="0") int start)` → **결과 값으로 Map 객체를 반환
        application/json 요청이기 때문에 DispatcherServlet은 jsonMessageConverter를 내부적으로 사용해서 해당 Map 객체를 JSON으로 변환**해서 전송을 한다.
        3. `@PostMapping` → **POST 방식의 요청을 받으면 write 메서드 호출**
        `public Guestbook write(@RequestBody Guestbook guestbook, HttpServletRequest request)` → **Guestbook 객체로 반환, JSON으로 변환**되어서 **클라이언트에게 응답**이 간다.
        4. @DeleteMapping("/{id}") → path 정보가 있다. **/guestbooks/에서 id가 붙여진다는 의미**. 이런 **id 정보를 PathVariable** 이라 한다.
        `public Map<String, String> delete(**@PathVariable(name="id")** Long id, HttpServletRequest request)` → 메서드가 성공적으로 실행되면 Map 객체 생성
            1. Map 객체의 키 값은 "success"로 주고 값은 true 또는 false가 리턴이 된다.

                ```java
                String clientIp = request.getRemoteAddr();
                int deleteCOunt = guestbookService.deleteGuestbook(id, clientIp);
                return Collections.singLectonMap("success", deleteCount > 0 ? "true" : "false");
                ```

3. Rest API를 테스트 하기 위한 클라이언트가 필요하다. 다양한 테스트 클라이언트 중 크롬 브라우저의 Restlet Client을 이용해 실습을 해보자.

- Restlet Client라는 이름은 없고 Talend API Tester 라고 이름이 바뀌어져 있었다. UI와 사용법은 같으니 그냥 이걸 사용하면 된다.

  - [Talend API Tester - Free Edition](https://chrome.google.com/webstore/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm?hl=ko)

---

---

---

## 2020-04-30 배운 것과 느낀 점

---

1. 와! 끝나지 않을 것 같았던 강의가 끝났다. 하나 하나 타이핑하면서 듣다보니 시간이 엄청 오래걸렸다. 그럼에도 불구하고 뭔가 제대로 배운 것 같은 느낌은 안 들어서 좀 슬프기도 하다. 이미 체화되어 있는 걸까 아니면 ... 정말로 시간 낭비한 것일까. 어찌됐든 지금까지의 과거가 그랬듯이 앞으로의 미래도 나의 선택과 결정으로 이뤄질 것이다. 부족하면 다시 배우면 된다. 중요한 것은 끝까지 포기하지 않고 나아가는 것이다.
2. 4월까지 프로젝트 제출까지 하려 했건만 강의만 듣고 말았다. 이 공부에 그만큼 열심히 임하지 않았다는 뜻이며 그만큼 내용이 어려웠다는 것이기도 하다. 5월의 목표는 프로젝트 2개 제출이다. 정말 정말 어려울 것 같다. 모든 힘을 다 쏟아부어야 한다. 현재 직접적으로 이 수업과 관련 없는 전공 수업 4개의 공부도 병행하면서 해야 한다. 그럼에도 불구하고 나는 이것을 할 이유가 있다. 자잘한 현실적이고, 내가 부족해서 나오는 이유들은 차치하고, 중요한 것은 개발자가 간절히 되고 싶기 때문이다.
3. 프로젝트를 완성하기 위해 어차피 개념 복습을 하겠지만, 평생 갈 공부라 생각하고 하나하나 차근차근 다시 보고 내 것으로 만들자.
