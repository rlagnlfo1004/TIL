## 🔍 **Dispatcher-Servlet 이란?**

디스패처 서블릿의 dispatch는 "보내다"라는 뜻을 가지고 있다. 그리고 이러한 단어를 포함하는 디스패처 서블릿은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임해주는 **프론트 컨트롤러(Front Controller)**라고 정의할 수 있다.

이것을 보다 자세히 설명하자면, 클라이언트로부터 어떠한 요청이 오면 Tomcat(톰캣)과 같은 서블릿 컨테이너가 요청을 받게 된다. 그리고 이 모든 요청을 프론트 컨트롤러인 **디스패처 서블릿**이 가장 먼저 받게 된다. 그러면 디스패처 서블릿은 공통적인 작업을 먼저 처리한 후에 해당 요청을 처리해야 하는 컨트롤러를 찾아서 작업을 위임합니다.
<br></br>
> 🔍 Front Controller(프론트 컨트롤러) 란?
>
>Front Controller는 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러로, MVC 구조에서 함께 사용되는 디자인 패턴이다.
> 
<br></br>
## 🔍 **Dispatcher-Servlet 의 장점**

과거에는 모든 서블릿을 URL 매핑을 위해 web.xml에 모두 등록해주어야 했지만, dispatcher-servlet이 해당 어플리케이션으로 들어오는 모든 요청을 핸들링해주고 공통 작업을 처리면서 상당히 편리하게 이용할 수 있게 되었다. 컨트롤러를 구현해두기만 하면 디스패처 서블릿가 알아서 적합한 컨트롤러로 위임을 해주는 구조가 되었다.
<br></br><br></br>
## ⚡️ **정적 자원(Static Resources)의 처리**

Dispatcher Servlet이 요청을 Controller로 넘겨주는 방식은 효율적으로 보인다. 하지만, Dispatcher Servlet이 모든 요청을 처리하다보니 이미지나 HTML/CSS/JavaScript 등과 같은 정적 파일에 대한 요청마저 모두 가로채는 까닭에 정적자원(Static Resources)을 불러오지 못하는 상황도 발생하곤 한다. 이러한 문제를 해결하기 위해 개발자들은 2가지 방법을 고안해냈다.

1. 정적 자원 요청과 애플리케이션 요청을 분리
2. 애플리케이션 요청을 탐색하고 없으면 정적 자원 요청으로 처리

각각의 방식에 대해 자세히 알아보자

### 1. 정적 자원 요청과 애플리케이션 요청을 분리

이에 대한 해결책은 두가지가 있는데, 첫번째는 클라이언트의 요청을 2가지로 분리하여 구분하는 것이다.

- /apps 의 URL로 접근하면 Dispatcher Servlet이 담당한다.
- /resources 의 URL로 접근하면 Dispatcher Servlet이 컨트롤할 수 없으므로 담당하지 않는다.

이러한 방식은 괜찮지만 상당히 코드가 지저분해지며, 모든 요청에 대해서 저런 URL을 붙여주어야 하므로 직관적인 설계가 될 수 없다. 그래서 이러한 방법의 한계를 느끼고 다음의 방법으로 처리를 하게 되었다.
### 2. 애플리케이션 요청을 탐색하고 없으면 정적 자원 요청으로 처리

두번째 방법은 Dispatcher Servlet이 요청을 처리할 컨트롤러를 먼저 찾고, 요청에 대한 컨트롤러를 찾을 수 없는 경우에, 2차적으로 설정된 자원(Resource) 경로를 탐색하여 자원을 탐색하는 것이다. 이렇게 영역을 분리하면 효율적인 리소스 관리를 지원할 뿐 아니라 추후에 확장을 용이하게 해준다는 장점이 있다.
<br></br><br></br>
## ⚡️ Dispatcher-Servlet(디스패처 서블릿)의 동작 과정

앞서 설명한대로 디스패처 서블릿은 적합한 컨트롤러와 메소드를 찾아 요청을 위임해야 한다. Dispatcher Servlet의 처리 과정을 살펴보면 다음과 같다.

![출처 : https://mangkyu.tistory.com/18](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbcff5H%2FbtstbdRuSr9%2FpNKnGdMwftSWmiGLHA7yL0%2Fimg.png)

출처 : https://mangkyu.tistory.com/18

1. 클라이언트의 요청을 DispatcherServlet이 받음
    - 모든 HTTP 요청은 DispatcherServlet이 먼저 받음 (Spring MVC의 프론트 컨트롤러)
2. 요청 정보를 통해 처리할 핸들러(=컨트롤러)를 찾음
    - DispatcherServlet은 `HandlerMapping`에게 물어서 어떤 컨트롤러가 이 요청을 처리할지 결정함
3. 핸들러 어댑터(HandlerAdapter)를 찾아서 선택함
    - DispatcherServlet은 해당 컨트롤러를 실행시킬 수 있는 적절한 `HandlerAdapter`를 찾음
4. 핸들러 어댑터가 컨트롤러를 실행시킴
    - `HandlerAdapter`는 요청 객체(HttpServletRequest 등)를 넘기고, 컨트롤러를 실제로 호출함
5. 컨트롤러가 비즈니스 로직을 수행하고 결과를 반환함
    - 예를 들어, DB에서 데이터를 조회하고, 뷰 이름 또는 JSON 응답을 반환함
6. 핸들러 어댑터가 컨트롤러의 반환값을 처리함
    - 반환값은 보통 `ModelAndView`, `ResponseEntity`, 또는 그냥 `@ResponseBody`의 JSON 등일 수 있음
7. 핸들러 어댑터가 반환값을 처리함
8. 서버의 응답을 클라이언트로 반환함
<br></br>
<br></br>
### 1. 클라이언트의 요청을 디스패처 서블릿이 받음

앞서 설명하였듯 디스패처 서블릿은 가장 먼저 요청을 받는 프론트 컨트롤러이다. 서블릿 컨텍스트(웹 컨텍스트)에서 필터들을 지나 스프링 컨텍스트에서 디스패처 서블릿이 가장 먼저 요청을 받게된다. 이를 그림으로 표현하면 다음과 같다. 

![출처 : https://mangkyu.tistory.com/18](https://blog.kakaocdn.net/dn/oN96r/btrw7SYEpgr/lKLp5nqEZUJR32GoPc9bwk/img.png)

출처 : https://mangkyu.tistory.com/18
<br></br>
### 2. 요청 정보를 통해 처리할 핸들러(=컨트롤러)를 찾음

디스패처 서블릿은 요청을 처리할 핸들러(컨트롤러)를 찾고 해당 객체의 메소드를 호출한다. 따라서 가장 먼저 어느 컨트롤러가 요청을 처리할 수 있는지를 식별해야 하는데, 해당 역할을 하는 것이 바로 **HandlerMapping**입니다. 최근에는 @Controller에 @RequestMapping 관련 어노테이션을 사용해 컨트롤러를 작성하는 것이 일반적이다. 하지만 예전 스펙을 따라 Controller 인터페이스를 구현하여 컨트롤러를 작성할 수도 있다. 즉, 컨트롤러를 구현하는 방법이 다양하기 때문에 스프링은 HandlerMapping 인터페이스를 만들어두고, 다양한 구현 방법에 따라 요청을 처리할 대상을 찾도록 되어 있습니다.

오늘날 흔한 @Controller 방식은 RequestMappingHandlerMapping가 처리한다. 이는 @Controller로 작성된 모든 컨트롤러를 찾고 파싱하여 HashMap으로 <요청 정보, 처리할 대상> 관리합니다. 여기서 처리할 대상은 **HandlerMethod** **객체로 컨트롤러, 메소드 등**을 갖고 있는데, 이는 스프링이 **리플렉션을 이용해 요청을 위임**하기 때문이다. 그래서 요청이 오면 (Http Method, URI) 등을 사용해 요청 정보를 만들고, HashMap에서 요청을 처리할 대상(HandlerMethod)를 찾은 후에 HandlerExecutionChain으로 감싸서 반환한다. HandlerExecutionChain으로 감싸는 이유는 컨트롤러로 요청을 넘겨주기 전에 처리해야 하는 **인터셉터 등을 포함**하기 위해서이다.
<br></br>
> 🔍 **리플렉션(Reflection)이란?**
>
>**자바의 리플렉션(Reflection)**은 클래스나 메서드, 필드 등의 정보를 런타임에 동적으로 조회하고 실행할 수 있게 해주는 기능이다. 즉, 코드가 실행되는 도중에 객체의 타입, 메서드, 필드에 접근하거나 호출 가능하다. DispatcherServlet이 컨트롤러 메서드를 직접 컴파일 타임에 알 수 없기 때문에, 요청이 들어올 때 적절한 메서드를 찾아서 실행해야 한다. 이때 리플렉션을 이용해서 @RequestMapping, @GetMapping, @PostMapping 등의 정보를 분석하고, 해당 메서드를 찾아 호출한다.
> 

> 🔍 **HandlerExecutionChain이란?**
> 
> 
> 
> HandlerExecutionChain은 **"컨트롤러(Handler) + 그에 적용될 인터셉터(Interceptor)"**의 묶음 객체이다. DispatcherServlet이 **HandlerMapping**에게 "이 요청은 누가 처리해야 해?"라고 물어보면, HandlerExecutionChain을 반환해준다. 
> 

> 🔍 **인터셉터(Interceptor)이란?**
>
>**컨트롤러가 실행되기 전/후에 공통 작업을 수행하기 위해 사용하는 컴포넌트**로, 필터처럼 요청을 가로채지만, Spring MVC 레벨에서 동작하며, 주로 인증, 권한 확인, 로깅, 시간 측정 등에 사용된다. **DispatcherServlet과 Controller 사이에 위치한다.**
> 
<br></br>
### 3. 핸들러 어댑터(HandlerAdapter)를 찾아서 선택함

이후에 컨트롤러로 요청을 위임해야 하는데, 디스패처 서블릿은 컨트롤러로 요청을 직접 위임하는 것이 아니라 HandlerAdapter를 통해  위임한다. 그 이유는 앞서 설명하였듯 컨트롤러의 구현 방식이 다양하기 때문이다. 과거에는 컨트롤러를 Controller 인터페이스로 구현하였는데, Ruby On Rails가 어노테이션 기반으로 관례를 이용한 프로그래밍을 내세워 혁신을 일으키면서 스프링 역시 이를 도입하게 되었다.그래서 **다양하게 작성되는 컨트롤러에 대응하기 위해 스프링은 HandlerAdapter라는 어댑터 인터페이스를 통해 어댑터 패턴을 적용**함으로써 컨트롤러의 구현 방식에 상관없이 요청을 위임할 수 있도록 하였다.
<br></br>
### 4. 핸들러 어댑터가 컨트롤러를 실행시킴

핸들러 어댑터가 컨트롤러로 요청을 위임한 전/후에 공통적인 전/후처리 과정이 필요하다. 대표적으로 인터셉터들을 포함해 요청 시에 @RequestParam, @RequestBody 등을 처리하기 위한 ArgumentResolver들과 응답 시에 ResponseEntity의 Body를 Json으로 직렬화하는 등의 처리를 하는 ReturnValueHandler 등이 핸들러 어댑터에서 처리된다. **ArgumentResolver 등을 통해 파라미터가 준비 되면 리플렉션을 이용해 컨트롤러로 요청을 위임한다.**
<br></br>
### 5. 컨트롤러가 비즈니스 로직을 수행하고 결과를 반환함

이후에 컨트롤러는 서비스를 호출하고 우리가 작성한 비지니스 로직들이 진행된다.
<br></br>
### 6. 핸들러 어댑터가 컨트롤러의 반환값을 처리함

비지니스 로직이 처리된 후에는 컨트롤러가 반환값을 반환한다. 응답 데이터를 사용하는 경우에는 주로 ResponseEntity를 반환하게 되고, 응답 페이지를 보여주는 경우라면 String으로 View의 이름을 반환할 수도 있다.
<br></br>
### 7. 핸들러 어댑터가 반환값을 처리함

HandlerAdapter는 컨트롤러로부터 받은 응답을 응답 처리기인 **ReturnValueHandler**가 후처리한 후에 디스패처 서블릿으로 돌려준다. 만약 **컨트롤러가 ResponseEntity를 반환하면 HttpEntityMethodProcessor가 MessageConverter를 사용해 응답 객체를 직렬화하고 응답 상태(HttpStatus)를 설정**합니다. 만약 컨트롤러가 View 이름을 반환하면 ViewResolver를 통해 View를 반환합니다.
<br></br>
### 8. 서버의 응답을 클라이언트로 반환함

디스패처 서블릿을 통해 반환되는 응답은 다시 필터들을 거쳐 클라이언트에게 반환된다. 이때 응답이 데이터라면 그대로 반환되지만, 응답이 화면이라면 View의 이름에 맞는 View를 찾아서 반환해주는 ViewResolver가 적절한 화면을 내려준다.
<br></br>

## 📚 출처
https://mangkyu.tistory.com/18