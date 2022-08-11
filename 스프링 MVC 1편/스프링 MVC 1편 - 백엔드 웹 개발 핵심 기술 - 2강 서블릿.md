# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 - 2강 서블릿

# @WebServlet

```java
@WebServlet(name = “helloServlet”, urlPatterns=”/hello”)
public class Servlet extends HttpServlet{

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException{
		
	}

}
```

- name: 서블릿 이름
- urlPatterns: URL 매핑

![Untitled](https://user-images.githubusercontent.com/61227459/184145267-7f039ed0-ea20-423a-b391-0e6196b98433.png)

- ContentType → content 형태
- CharacterEncoding → urt-8이  표준

HTTP 정보 로그로 확인

```java
logging.level.org.apache.coyote.http11=debug
```

```java
HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="88", "Google Chrome";v="88", ";Not A Brand";v="99"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 11_2_1) AppleWebKit/537.36 
(KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/
webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost:8080/basic.html
Accept-Encoding: gzip, deflate, br
Accept-Language: ko,en-US;q=0.9,en;q=0.8,ko-KR;q=0.7
]
```

![Untitled 1](https://user-images.githubusercontent.com/61227459/184145255-51f13ae4-731c-473c-aec0-8a9c4ecfc2d5.png)

---

# HttpServletRequest

- HTTP 요청 메세지를 파싱

## HTTP 요청 메세지

```java
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
username=kim&age=20
```

### Start Line

- HTTP 메소드
- URL
- 쿼리 스트링
- 스키마(http), 프로토콜(HTTP/1.1)
- ex) POST /save HTTP/1.1

### Header

- 헤더 조회
- ex) Host, Content-Type

### Body

- form 파라미터 형식 조회
- message body 데이터 직접 조회

### 기능

- 임시 저장소 기능
    - setAttribute (저장)
    - getAtrribute (조회)
- 세션 관리 기능
    - getSession

### Accept-Language

```java
getLocales()
```

- 원하는 언어의 순위

---

# HTTP 요청 데이터

## GET 쿼리 파라미터

이름이 같은 복수 파라미터 조회

```java
getParameterValues()
```

- 파라미터 이름은 하나인데 값은 중복
- getParameter는 첫 번째 값을 반환
- 메시지 바디를 사용하지 X → content-type 없음

## POST HTML Form

- content-type: application/x-www-form-urlencoded
- getParamter로도 조회 가능

## API 베시지 바디 - JSON

```java
// jackson에서 제공하는 JSON 변환 라이브러리
private ObjectMapper objectMapper = new ObjectMapper();

@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException{
	ServletInputStream inputStream = request.getInputStream();
	String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

	System.out.println("messageBody" + messageBody);

	HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

	System.out.println("helloData.username" + helloData.getUsername());
	System.out.println("helloData.age" + helloData.getAge());

	response.getWriter().write("ok");
}
```

---

# HTTPServletResponse

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException{
	// [status-line]	
	// response 상태 (200, 400, ...)
	response.setStatus(HttpServletResponse.SC_OK);

	// [response-headers]
	// Content, Cookie, redirect
	response.setHeader();
  ...

	// [message body]
	PrintWriter writer = response.getWriter();
	writer.println("ok");
}
```

- 응답 코드
    - **1xx(정보) :** 요청을 받았으며 프로세스를 계속 진행
        - 해당 코드는 HTTP 1.0에서 지원되지 않음
    - **2xx(성공) :** 요청을 성공적으로 받았으며 인식했고 수용
        - 200(OK)
            - 정보는 요청에 따른 응답으로 반환
        - 202(Accepted)
            - 요청 수신만. 요청 처리에 대한 결과를 HTTP로 비동기 응답을 보내야하는 것에 대해 명시하지 않음
    - **3xx(리다이렉션) :** 요청 완료를 위해 추가 작업 조치가 필요
        - 302(Found)
            - 요청한 리소스의 URI가 일시적으로 변경됨
    - **4xx(클라이언트 오류) :** 요청의 문법이 잘못되었거나 요청을 처리할 수 없음
        - 400(Bad Request)
            - 잘못된 문법
        - 401(Unauthorized)
            - **비인증(unauthenticated)**, ~~미승인(unauthorized)~~ → 클라이언트는 스스로를 인증
        - 404(Forbidden)
            - 미승인이어서 서버는 거절을 위한 적절한 응답을 보냄
            - 401과 다른 점은 서버가 클라이언트가 누구인지 알고 있음
    - **5xx(서버 오류) :** 서버가 명백히 유효한 요청에 대한 충족을 실패
        - 500(Internal Server Error)
            - 서버에 문제가 있음, 상세한 설명 X
        - 502(Bad Gateway)
            - 서버가 게이트웨이로부터 잘못된 응답을 수신

[https://www.whatap.io/ko/blog/40/](https://www.whatap.io/ko/blog/40/)

## HTTP 응답 데이터 - 단순 텍스트, HTML

Content-Type: text/html;charset=utf-8

```java
response.setContentType("text/html");
response.setCharacterEncoding("utf-8");

PrintWriter writer = response.getWriter();
writer.println("<html>");
...
```

## HTTP 응답 데이터 - API JSON

Content-Type: appliction/json;charset=utf-8

```java
response.setContentType("appliction/json");
response.setCharacterEncoding("utf-8");

HelloData helloData = new HelloData();
...

// jackson library
String result = objectMapper.writeValueAsString(helloData);
response.getWriter().write(result);
```

> application/json 은 스펙상 utf-8 형식을 사용하도록 정의되어 있다. 그래서 스펙에서 charset=utf-8과 같은 추가 파라미터를 지원하지 않는다. 따라서 application/json 이라고만 사용해야지 application/json;charset=utf-8 이라고 전달하는 것은 의미 없는 파라미터를 추가한 것이 된다.
~~application/json/charset=utf-8~~ → application/json
response.getWriter()를 사용하면 추가 파라미터를 자동으로 추가해버린다. 이때는
response.getOutputStream()으로 출력하면 그런 문제가 없다.
>