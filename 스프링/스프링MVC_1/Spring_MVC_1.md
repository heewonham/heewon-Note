## 목차

1. 웹 애플리케이션 이해
2. 서블릿
3. 서블릿, JSP, MVC 패턴
4. MVC 프레임워크 만들기
5. 스프링 MVC - 구조 이해
6. 스프링 MVC - 기본 기능
7. 스프링 MVC - 웹 페이지 만들기



-----

# 01. 웹 애플리케이션 이해

## 웹서버

* HTTP 기반으로 동작

* 정적 리소스 제공, 기타 부가기능
* 정적(파일) HTML, CSS, JS, 이미지, 영상
* 예) NGINX, APACHE



### 웹 애플리케이션 서버 

* HTTP 기반으로 동작
* 웹 서버 기능 포함+ (정적 리소스 제공 가능)
* 프로그램 코드를 실행해서 `애플리케이션 로직` 수행
* 동적 HTML, HTTP API(JSON)
* 서블릿, JSP, 스프링 MVC
* 예) 톰캣(Tomcat) Jetty, Undertow



웹 서버와 웹 애플리케이션 서버의 용어는 모호하다. 왜냐하면 

* 웹서버도 프로그램을 실행하는 기능을 포함하고
* 웹 애플리케이션 서버도 웹 서버의 기능을 제공하기 때문이다.
* WAS는 애플리케이션 코드를 실행하는 데 더 특화되어있다고 생각하면 된다.



웹 시스템의 구성은 WAS, DB만으로도 가능하지만,,,

1. 이럴 경우 WAS가 너무 많은 역할을 담당하게 되어 서버 과부하가 우려 된다. 
2. HTML, CSS, JS 파일 같은 경우 간단하지만 애플리케이션 로직은 복잡하다. 애플리케이션 로직이 정적 리소스 때문에 수행이 어려운 경우가 발생하면 안 된다. 
3. WAS가 (개발자들의 실수 버그 날 가능성이 높음) 장애시 오류 화면도 노출 불가능하다. 



### 그러므로...

### 웹 시스템 구성 - WEB, WAS, DB

<img src = "./img_mvc1/2.png">

 정적 리소스는 웹 서버가 처리하고 애플리케이션 로직같은 동적인 처리가 필요하면 WAS에 요청을 위임한다. 즉, WAS는 중요한 애플리케이션 로직 처리 전담한다.

이렇게 나누면 효율적인 리소스 관리가 가능하다. 예를 들어, 정적 리소스가 많이 사용되면 Web 서버 증설하고 애플리케이션 리소스가 많이 사용되면 WAS 증설할 수 있다.

또한 정적 리소스만 제공하는 웹 서버는 잘 죽지 않고 애플리케이션 로직이 동작하는 WAS 서버는 잘 죽기 때문에 WAS, DB 장애시 `WEB 서버가 오류 화면 제공 가능`하다.



## 서블릿

HTML form 데이터를 전송하는 과정에서 서버는

1. 서버 TCP/IP 연결 대기/ 소켓 연결
2. HTTP 요청 메세지 파싱해서 읽기
3. POST 방식, /save 인지
4. Content-Type 확인
5. HTTP 메세지 바디 내용 파싱
6. 저장 프로세스 실행
7. 비즈니스 로직 실행 : 데이터베이스 저장 요청
8. HTTP 응답 메시지 생성 시작 (HTTP 시작 라인, header, body에 HTML 생성)
9. TCP/IP 응답 전달, 소켓 종료

의 과정을 거쳐야한다.



하지만 서블릿을 지원하는 WAS를 사용할 경우

`비즈니스 로직 실행 : 데이터 베이스 저장 요청` 만 실행하면 된다.

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet{
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response){
        // 애플리케이션 로직
        ...
    }
}
```

* HTTP 요청 정보를 편리하게 사용할 수 있는 `HttpServletRequest` 지원
* HTTP 응답 정보를 편리하게 제공할 수 있는 `HttpServletResponse` 지원



<img src = "./img_mvc1/1.png">

1. WAS는 Request, Response 객체를 새로 만들어서 서블릿 객체 호출
2. 개발자는 Request 객체에서 HTTP 요청 정보를 편리하게 꺼내서 사용 or Response 객체에 HTTP 응답 정보를 편리하게 입력
3. WAS는 Response 객체에 담겨있는 내용으로 HTTP 응답 정보를 생성



### 서블릿 컨테이너 특징

톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 한다. 서블릿 컨테이너는 서블릿 객체를 생성, 초기화, 호출, 종료의 생명주기를 관리한다.

* 서블릿 객체는 고객 요청이 올 때 마다 계속 객체를 생성하는 것은 비효율적이기 때문에 `싱글톤`으로 관리한다.
*  최초 로딩 시점에 서블릿 객체를 미리 만들어두고 재활용한다. 그러므로 모든 고객 요청은 동일한 서블릿 객체 인스턴스에 접근하고 `공유 변수 사용에 주의`해야한다. 
* 서블릿 컨테이너 종료시 함께 종료된다.
* JSP도 서블릿으로 변환 되어서 사용한다.
* 동시 요청을 위한 멀티 쓰레드 처리 지원



## 동시 요청 - 멀티 쓰레드

### 스레드

애플리케이션 코드를 하나씩 순차적으로 실행하는 것이 스레드이고 스레드가 없다면 자바 애플리케이션 실행이 불가능하다. 스레드는 한번에 `하나의 코드라인`만 수행한다. 동시 처리가 필요하면 스레드를 추가로 생성해야 한다.



### 1. HTTP 요청마다 스레드 생성시

#### 장점

하나의 스레드가 지연되어도 나머지 스레드가 정상 동작하고, 동시 요청을 처리할 수 있고, 리소스가 허용할때 까지 처리 가능하다.

#### 단점

고객 요청이 올 때 마다 스레드를 생성하면 응답 속도 늦어진다(스레드 생성비용 비쌈)

스레드는 "context switching" 발생할 때 프로세스만큼은 아니지만 비용이 발생한다.

고객 요청이 너무 많이 올 경우 CPU, 메모리 임계치가 넘어 서버가 죽을 가능성도 있다.



### 2. 스레드 풀로 관리

<img src = "./img_mvc1/3.png">

스레드 풀은 필요한 스레드를 미리 만들어 보관하고 관리한다. 톰캣의 경우 최대 200개 기본 설정이다.

스레드가 미리 생성되어 있어 스레드를 생성하고 종료하는 비용을 절약하고 응답 시간이 빠르다. 그리고 너무 많은 요청이 온다고 해도 스레드 최대치로 인해 최대치까지 안전하게 처리할 수 있다.



### 3. 스레드 풀의 실무 tip

* WAS의 튜닝포인트는 `최대 스레드 (max thread) 수`이다.
* 너무 낮으면, 서버 리소스는 여유롭지만 클라이언트는 금방 응답 지연
* 너무 높으면, 동시요청이 많을 시 리소스 임계점 초과로 서버 다운 가능성
* 스레드 풀의 적정 숫자는 애플리케이션 로직의 복잡도, CPU, 메모리, IO 리소스 상황에 따라 모두 다르다
* 성능 테스트 : 최대한 실제 서비스와 유사하게 성능 테스트 시도
  * 아파치 ab , 제이미터, nGrinder



## WAS 멀티 스레드 지원

WAS는 멀티스레드 부분을 처리해주기 때문에 개발자가 멀티스레드 관련 코드를 신경쓰지 않아도 된다. 마치 싱글스레드 프로그래밍하듯 편리하게 소스 코드를 개발한다. 

`멀티 스레드 환경이므로 싱글톤 객체(서블릿, 스프링 빈) 주의해서 사용`



### 서버사이드 렌더링, 클라이언트 사이드 렌더링

#### SSR - 서버 사이드 렌더링

서버에서 최종 HTML를 생성해서 웹 브라우저에 전달한다.

* 주로 정적인 화면에 사용
* 관련기술: JSP, 타임리프 -> 백엔드 개발자



#### CSR - 클라이언트 사이드 렌더링

HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용한다.

* 주로 동적인 화면에 사용, 웹 환경을 마치 앱 처럼 필요한 부분부분 변경할 수 있음
* 예) 구글 지도, Gmail, 구글 캘린더
* 관련기술: React, Vue.js -> 웹 프론트엔드 개발자

> React, Vue.js를 CSR + SSR 동시에 지원하는 웹 프레임워크도 있음
> SSR을 사용하더라도, 자바스크립트를 사용해서 화면 일부를 동적으로 변경 가능



# 02. 서블릿

## HttpServletRequest

서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 HTTP 요청 메시지를 파싱하여 그 결과를 HttpServletRequest 객체에 담아 제공한다.

* `request.setAttribute(name, value)`
* `request.getAttribute(name)`





<img src = "./img_mvc1/4.png">

##### Start Line

* HTTP 메소드
* URL
* 쿼리 스트링
* 스키마, 프로토콜

<img src = "./img_mvc1/5.png">

##### Header

- Content-Type: 표현 데이터의 형식
  - text/html; charset=utf-8
  - application/json
  - image/png
- Content-Encoding: 표현 데이터의 압축 방식
- Content-Language: 표현 데이터의 자연 언어
- Content-Length: 표현 데이터의 길이

<img src = "./img_mvc1/6.png">

##### Body

* form 파라미터 형식 조회
* message body 데이터 직접 조회



#### 상태코드

- 1xx (Information): 요청이 수신되어 처리중
- 2xx (Successful): 요청 정상 처리
- 3xx (Redirection): 요청을 완료하려면 추가 행동이 필요
- 4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함



## HTTP 요청 데이터 개요

### 1. GET : 쿼리 파라미터

/url`?username=hee&age=20`

메세지 바디 없이, URL 쿼리 파라미터에 데이터를 포함해서 전달한다. 검색, 필터, 페이징 등에 많이 사용하는 방식이다.

* `?` 시작 표시이며, 추가 파라미터는 `&`으로 연결한다.

```java
// 단일 파라미터 조회
// 중복일 경우 첫번째 값을 반환한다.
String username = request.getParameter("username");

//파라미터 이름들 모두 조회
Enumeration<String> parameterNames = request.getParameterNames();
request.getParameterNames().asIterator()
    .forEachRemaining(paramName -> System.out.println(paramName +
"=" + request.getParameter(paramName)));

//파라미터를 Map으로 조회
Map<String, String[]> parameterMap = request.getParameterMap(); 

//복수 파라미터 조회
String[] usernames = request.getParameterValues("username"); 
```



### 2. POST : HTML Form

content-type:  `application/x-www-form-urlencoded`

메시지 `바디`에 쿼리 파리미터 형식으로 전달한다. username=hello&age=20 회원가입, 상품주문 등 HTML Form 사용한다.

* 쿼리 파라미터 조회 메소드를 그대로 사용하면 된다.
* 클라이언트 입장에서는 두 입장은 차이가 있지만 서버 입장에서는 둘의 형식이 동일하기 때문이다.



### 3. HTTP message body

body에 데이터를 직접 담아서 요청하는 방식으로 HTTP API에서 주로 사용한다. 

JSON, XML, TEXT 가능하고 주로 JSON 많이 사용한다. POST, PUT, PATCH가 가능하다.



#### 메시지 바디 - 단순 텍스트

content-type : `text/plain`

```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns ="/request-bodystring")
public class RequestBodyStringServlet extends HttpServlet {
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        
        System.out.println("messageBody = " + messageBody);
        response.getWriter().write("ok");
    }
}
```

* inputStream은 byte 코드를 반환하기 때문에 우리가 읽을 수 있는 문자(String)로 보려면 문자표(Charset)를 지정해주어야 한다. 여기서는 UTF_8 Charset을 지정해주었다.



#### 메시지 바디 - JSON

content-type : `application/json`

ObjectMapper을 통해 messagebody의 내용을 Data class로 변환해준다.

JSON 결과를 파싱해서 자바 객체로 변환하기 위해서는 Jackson, Gson 같은 JSON 변환 라이브러리가 필요하다.

스프링 MVC는 기본적으로 Jackson 라이브러리(ObjectMapper)을 제공한다.

```java
private ObjectMapper objectMapper = new ObjectMapper();

@Override
protected void service(HttpServletRequest request, HttpServletResponse 
response) throws ServletException, IOException {
    
 	ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    
    HelloData helloData = objectMapper.readValue(messageBody, HelloData.class); 

}
```



## HttpServletResponse

#### response 설정

``` java
//[status-line]
response.setStatus(HttpServletResponse.SC_OK); //200

//[response-headers]
response.setHeader("Content-Type", "text/plain;charset=utf-8");
response.setHeader("Cache-Control", "no-cache, no-store, mustrevalidate");
response.setHeader("Pragma", "no-cache");
response.setHeader("my-header","hello");

//[Header 편의 메서드]
content(response);
cookie(response);
redirect(response);

//[message body]
PrintWriter writer = response.getWriter();
writer.println("ok");
```

#### response 편의 메서드

```java
//[response-headers]
response.setContentType("text/plain");
response.setCharacterEncoding("utf-8");

//[Header 편의 메서드]
Cookie cookie = new Cookie("myCookie", "good");
cookie.setMaxAge(600); //600초
response.addCookie(cookie);

response.sendRedirect("/basic/hello-form.html");
```



## HTTP 응답 데이터

### 1. 단순 텍스트

* writer.println("OK");
* 처럼 OK를 그대로 HTML에 찍음



### 2. HTML 응답

* writer을 통해 직접 보내는 방식, content-type을 `text/html` 으로 지정해야 한다.

  ```java
   writer.println("<html>");
   writer.println("<body>");
   writer.println(" <div>안녕?</div>");
   writer.println("</body>");
   writer.println("</html>");
  ```



### 3. HTTP API - message body, JSON 응답

* content-type을 `application/json` 으로 지정해야 한다.

```java
	HelloData data = new HelloData();
	data.setUsername("kim");
	data.setAge(20);
 
	String result = objectMapper.writeValueAsString(data);
	//{"username":"kim","age":20}
 	response.getWriter().write(result);
```

* application/json 은 스펙상 utf-8 형식을 사용하도록 정의되어 있다. 그래서 스펙에서 charset=utf-8 과 같은 추가 파라미터를 지원하지 않는다. 
  * application/json ( o )
  * application/json;charset=utf-8 ( x )
* response.getWriter()를 사용하면 추가 파라미터를 자동으로 추가해버린다. 이때는
  response.getOutputStream()으로 출력하면 그런 문제가 없다.





# 03. 서블릿, JSP, MVC 패턴 

로그

trace -> debug -> info -> warn -> error

로그의 레벨을 정할 수 있다.

점점 등급이 높아짐

trace 로그 다 볼꺼임

운영시스템에서 보는 정보



출력은 전부 다봄 

디폴트가 info



문자열 처럼 + 할 수 있지만 연산을 함으로써 메모리를 사용하게 됨

그렇기 때문에 파라미터를 넘기는 작업을 해야한다.



최근에는 로그가 100메가 넘으면 분할한다든지 혹은 압축해서 백업하는 기능이 추가된다.

성능도 더 좋은데 내부 버퍼링 기능이 있어서

버퍼링이 성능최적화에 맞춰져있어서 한꺼번에 출력이 모여도 성능을 극한으로 해줌 



아규먼트 리졸버는 아규먼트를 찾는것이고

아규먼트 중에서 http body를 그대로 처리하는 것들은 http 컨버터가 처리해줌



프로듀서 = 컨텐트 타입

accept (producer), content type(consume)

 

핵심 엔티티는 data 사용할때 주의 - setter까지 나올 수 있으므로

그러므로 DTO에만 @data를 쓴다.





```
<!--  추가  param 타임리프 자체에서 쿼리 파라미터를 조회할 수 있게 함  -->
<h2 th:if="${param.status}" th:text="저장완료"></h2>
```