# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 - 6강 스프링 MVC 기본 기능

# 로깅

SLF4J - Interface

Logback - 구현체

```java
private final Logger log = LoggerFactory.getLogger(getClass());
```

## 레벨

```java
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
#package = hello.springmvc
logging.level.hello.springmvc=info
```

trace > debug > info > warn > error

개발 서버: debug

운영 서버: info

@Slf4j

- Logger 따로 선언 안해도 됨

### 주의

```java
// 의미 없는 연산이 일어남
log.trace("log" + name);
// 이렇게 쓰는걸로
log.trace("log : {}", name);
```

## 장점

- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있음
- 상황에 맞게 로그 조절 (level별로)
- 성능이 뛰어남 (System.out.println 보다)
- 콘솔에만 출력하는 것이 아니라 파일이나 네트워크에도 기록 남김
    - 파일을 분할 가능 (너무 크면)

# 요청 매핑

## @RequestMapping

```java
// HTTP 매서드 없으면 다 허용
@RequestMapping(value = "reqmap", method = RequestMethod.GET)
public String helloBasic() {
	 log.info("helloBasic");
	 return "ok";
}
```

@Controller

- String 반환하면 뷰를 찾아 뷰가 랜더링 됨

@RestController

- 반환 값으로 뷰를 찾지 않고, HTTP 메시지 바디에 바로 입력
    - 위의 예시에서는 ok를 메세지로 받음

## @GetMapping, @PostMapping, @DeleteMapping, @PutMapping, @PatchMapping

> PUT : 리소스의 전체를 변경할 때 사용 (partial modifications), 변경하는 데이터만 요청하면 된다. (멱등성o)
PATCH : 리소스의 부분을 변경할 때 사용, 변경하지 않는 데이터도 함께 전송해야 한다. (멱등성x)
멱등성 : 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질
> 

## @PathVariable()

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data){
}
```

- 변수명이 같으면 생략 가능

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable String userId){
}
```

```java
/**
 * 파라미터로 추가 매핑
 * params="mode",
 * params="!mode"
 * params="mode=debug"
 * params="mode!=debug" (! = )
 * params = {"mode=debug","data=good"}
 */
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
 log.info("mappingParam");
 return "ok";
}
```

## 특정 헤더 조건 매핑

```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug"
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
 log.info("mappingHeader");
 return "ok";
}
```

## 미디어 타입

```java
/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * accept from client (수신)
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 * 415 반환 (Unsupported Media Type)
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
 log.info("mappingConsumes");
 return "ok";
}
```

```java
/**
 * Accept 헤더 기반 Media Type
 * what resource is given to client (출력)
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 * 406 반환 (Not Acceptable)
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
 log.info("mappingProduces");
 return "ok";
}
```

> ****Content-Type 헤더는 현재 전송하는 데이터가 어떤 타입인지에 대한 설명을 하는 개념
 Accept 헤더는 클라이언트가 서버에게 어떤 특정한 데이터 타입을 보낼때 클라이언트가 보낸 특정 데이터 타입으로만 응답을 해야함****
> 

# HTTP 요청 - 기본, 헤더 조회

```java
@Slf4j
@RestController
public class RequestHeaderController {
 @RequestMapping("/headers")
 public String headers(HttpServletRequest request,
 HttpServletResponse response,
 HttpMethod httpMethod,
 Locale locale,
 @RequestHeader MultiValueMap<String, String>
headerMap,
 @RequestHeader("host") String host,
 @CookieValue(value = "myCookie", required = false)
String cookie
 ) {
	 log.info("request={}", request);
	 log.info("response={}", response);
	 log.info("httpMethod={}", httpMethod);
	 log.info("locale={}", locale);
	 log.info("headerMap={}", headerMap);
	 log.info("header host={}", host);
	 log.info("myCookie={}", cookie);
	 return "ok";
	 }
}
```

MultiValueMap

- 하나의 키에 여러 값을 받을 수 있음

```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2")
```

# HTTP 요청

- GET - 쿼리 파라미터
- POST - HTML Form
- HTTP message body
    - HTTP API
    - POST, PUT, PATCH

## @RequestParam

- @RequestParam(required = true) → 필수
- @RequestParam(required = true, defaultValue = ””) → 없으면 default

> int는 null이면 안됨
Integer는 null이어도 가능 → 객체이기 때문
> 

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
	log.info("username={}, age={}", paramMap.get("username"),
	paramMap.get("age"));
	return "ok";
}
```

- @ResponseBody : View 조회를 무시하고 HTTP message body에 직접 해당 내용 입력
    - @Controller 때문에 view를 조회하게 되는데 @ResponseBody가 이를 바꿈

> MultiValueMap → ex. userIds=id1&userIds=id2
> 

## @ModelAttribute

@Data → @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor

- @EqualsAndHashCode
    - equals : 두 객체의 내용이 같은지, 동등성(equality) 비교
        - 객체의 내용 비교
    - hashcode : 두 객체가 같은 객체인지, 동일성(identity) 비교
        - hashcode() : 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값
        - 해시주소 비교

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
	log.info("username={}, age={}", helloData.getUsername(),
	helloData.getAge());
	return "ok";
}
```

- 객체를 생성하고 객체의 프로퍼티(getter, setter)를 찾아 값을 입력(바인딩)

> Argument Resolver
**Controller로 들어온 파라미터를 가공하거나 수정 기능을 제공하는 객체
ex. HttpSession에서 세션 로드, HttpServletRequest에서 요청 url 및 ip 정보 등**
> 

## 단순 텍스트

> Stream은 바이트 코드여서 encoding 필요
StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
> 

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
	String messageBody = httpEntity.getBody();
	log.info("messageBody={}", messageBody);
	return new HttpEntity<>("ok");
}
```

- HttpEntity: header, boy 편리하게 조회
    - 요청 파라미터를 조회하는 기능과 관계 없음
    - view 조회 X
    - RequestEntity<>, ResponseEntity<>

### @RequestBody

- HttpEntity 대체
- 메세지 바디를 직접 조회 (파라미터를 상관 X)
- HTTP 요청의 바디내용을 통째로 자바객체로 변환해서 매핑된 메소드 파라미터로 전달

## JSON

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
	log.info("username={}, age={}", data.getUsername(), data.getAge());
	return "ok";
}
```

@ResponseBody 요청

- JSON 요청 → HTTP 메시지 컨버터 → 객체

@ResponseBody 응답

- 객체 → HTTP 메시지 컨버터 → JSON 응답

# 응답 - 정적 리소스, 뷰 템플릿

## 정적 리소스

src/main/resources

- /static , /public , /resources , /META-INF/resources

## 뷰 템플릿

주로 동적으로 생성하는 용도

src/main/resources/templates

```java
@Controller
public class ResponseViewController {
	@RequestMapping("/response-view-v1")
	public ModelAndView responseViewV1() {
		ModelAndView mav = new ModelAndView("response/hello")
		.addObject("data", "hello!");
		return mav;
	}
	@RequestMapping("/response-view-v2")
	public String responseViewV2(Model model) {
		model.addAttribute("data", "hello!!");
		return "response/hello";
	}
	// void의 경우 mapping/url 경로를 찾아감 -> response/hello.html을 찾아감
	@RequestMapping("/response/hello")
	public void responseViewV3(Model model) {
		model.addAttribute("data", "hello!!");
	}
}
```

> String을 반환하는 경우 - View or HTTP 메시지
@ResponseBody 가 없으면 response/hello 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
@ResponseBody 가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 response/hello 라는 문자가 입력된다.
> 

# HTTP 응답 - HTTP API, 메시지 바디에 직접 입력

```java
// ResponseEntity<>()에서는 status 설정 가능하지만 객체로는 불가능해 @ResponseStatus() 활용
@ResponseStatus(HttpStatus.OK)
@ResponseBody
@GetMapping("/response-body-json-v2")
public HelloData responseBodyJsonV2() {
	HelloData helloData = new HelloData();
	helloData.setUsername("userA");
	helloData.setAge(20);
	return helloData;
}
```

@RestController = @Controller + @ResponseBody

- 뷰 템플릿을 사용하는 것이 아니라

# HTTP 메시지 컨버터

![Untitled](https://user-images.githubusercontent.com/61227459/185412823-41a12d02-04ca-45a3-8a85-6f6950851e32.png)

> 0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
> 
- ByteArrayHttpMessageConverter : byte[] 데이터를 처리
    - 클래스 타입: byte[] , 미디어타입: */*
    - 요청 예) @RequestBody byte[] data
    - 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream
- StringHttpMessageConverter : String 문자로 데이터를 처리
    - 클래스 타입: String , 미디어타입: */*
    - 요청 예) @RequestBody String data
    - 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
- MappingJackson2HttpMessageConverter : application/json
    - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
    - 요청 예) @RequestBody HelloData data
    - 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json