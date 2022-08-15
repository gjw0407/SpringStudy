# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 - 3강 서블릿, JSP, MVC 패턴

# 요구사항

## 회원 도메인 모델

```java
@Getter @Setter
public class Member {
	private Long id;
	private String username;
	private int age;

	public Member() {
	}

	public Member(String username, int age) {
		this.username = username;
		this.age = age;
	}
}
```

## 회원 저장소

```java
/**
 * 동시성 문제가 고려되어 있지 않음, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
 * 동시성 문제 따로 정리
 */
public class MemberRepository {
	private static Map<Long, Member> store = new HashMap<>(); //static 사용
	private static long sequence = 0L; //static 사용
	private static final MemberRepository instance = new MemberRepository();
	
	public static MemberRepository getInstance() {
		return instance;
	}
	
	private MemberRepository() {
	}
	
	public Member save(Member member) {
		member.setId(++sequence);
		store.put(member.getId(), member);
		return member;
	}
	public Member findById(Long id) {
		return store.get(id);
	}
	public List<Member> findAll() {
		return new ArrayList<>(store.values());
	}
	public void clearStore() {
		store.clear();
	}
}
```

## 싱글톤 패턴

전역 변수를 사용하지 않고 객체를 하나만 생성 하도록 하며, 생성된 객체를 어디에서든지 참조할 수 있도록 하는 패턴

### 장점

- 메모리 관리
    - 한번의 new 연산자로 고정된 메모리 영역 사용
- 데이터 공유
    - 여러 클래스에서 접근 가능
    - **BUT 동시성 문제 발생 가능**

### 단점

- 동시성 문제
    - syncronized 키워드 사용
- 테스트가 어려움
    - 자원을 공유하기 때문에 격리된 환경에서는 매번 인스턴스의 상태를 초기화 필요
- 의존 관계상 클라이언트가 구체 클래스에 의존
    - DIP, OCP 위반
- 자식클래스를 만들 수 없음
- 내부 상태 변경 어려움

→ 스프링 컨테이너가 싱글톤의 문제점 해결

# 서블릿 활용

싱글톤으로 선언 시

- private으로 막아야함

```java
// 불가
private MemberRepository memberRepository = new MemberRepository();

// 가능
private MemberRepository memberRepository = MemberRepository.getInstance();
```

# JSP 활용

```java
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
// request, response 사용 가능
 MemberRepository memberRepository = MemberRepository.getInstance();
 System.out.println("save.jsp");
 String username = request.getParameter("username");
 int age = Integer.parseInt(request.getParameter("age"));
 Member member = new Member(username, age);
 System.out.println("member = " + member);
 memberRepository.save(member);
%>
<html>
	<head>
	 <meta charset="UTF-8">
	</head>
		<body>
		성공
		<ul>
		 <li>id=<%=member.getId()%></li>
		 <li>username=<%=member.getUsername()%></li>
		 <li>age=<%=member.getAge()%></li>
		</ul>
		<a href="/index.html">메인</a>
	</body>
</html>
```

```java
// JAVA import와 같음
<%@ page import="hello.servlet.domain.member.MemberRepository" %>

// JAVA 코드 입력
<% ~~ %>

// JAVA 코드 출력
<%= ~~ %>
```

- JSP = JAVA + HTML (역할 분담이 안되어 있음)
- 유지보수에 안 좋음

# MVC 패턴

## 탄생 이유

- 라이프 사이클이 다름
    - UI 수정 & 비즈니스 로직 수정은 서로에게 영향을 주면 안됨

![Untitled](https://user-images.githubusercontent.com/61227459/184577714-c42f2f95-d5ab-4586-a583-f9be2b9c689d.png)

- 컨트롤러는 비즈니스 로직 X → 서비스에 비즈니스 로직
    - 서비스 호출만 함

## MVC 패턴 적용

> /WEB-INF
프로젝트 폴더의 경로로 url 입력해도 안 페이지 안나옴 → 콘트롤러를 무조건 거쳐야 함
> 

### redirect vs forward

- redirect
    - 클라이언트에 응답이 나갔다가 redirect 경로로 다시 요청 → 클라이언트가 인지
    - 주소창에 잠깐 보여짐
    - 새로운 요청
    - 시스템에 변화가 생기는 요청 (회원가입, 글쓰기 등)
- forward
    - 서버 내부에서 일어남 → 클라이언트가 인지 못함
    - 요청 정보 그대로 전달
    - 시스템에 변화가 생기지 않는 요청 (글 목록 보기, 검색)

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/
new-form")
public class MvcMemberFormServlet extends HttpServlet {
	 @Override
	 protected void service(HttpServletRequest request, HttpServletResponse 
	response) throws ServletException, IOException {
		 String viewPath = "/WEB-INF/views/new-form.jsp";
		 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
		 dispatcher.forward(request, response);
	 }
}
```

![Untitled 1](https://user-images.githubusercontent.com/61227459/184577716-01ef1e89-9a41-4cd3-9563-1c79cb3d899c.png)

![Untitled 2](https://user-images.githubusercontent.com/61227459/184577711-0fac755a-d12e-4350-86e7-9bf4fae5304d.png)

## 한계

### 포워드 중복

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
		 dispatcher.forward(request, response);
```

### ViewPath 중복

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

### 공통 처리 어려움

- 반복되는 코드 처리 안됨

→ 수문장이 필요함: 프론트 컨트롤러 패턴