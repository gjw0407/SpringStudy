# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 - 1강 웹 애플리케이션 이해

Web server: Static 응답 (Apache web server) html css js

Web Application server: 동적 응답 application 

Tomcat은 WAS로 구분되긴 하지만 웹 서버 기능도 탑재해있다. 

JSP: HTML 내 자바

Servlet: 자바 내 HTML

둘 다 동적으로 Request를 전달해주는 역할 수행

Servlet은 3개의 주요 메서드 존재

Init Service Destroy

Tomcat 역할

1. Socket 생성 및 Conn Open
2. Thread 할당 
3. Servlet 생애 주기 관리 (Servlet Container)

DispatcherServlet

- HandlerMapping: Controller로 등록된 Bean 조회 (bean 정보)
- ViewResolver: 결과로 얻은 view의 path를 resolve
- HandlerAdaptor: bean 정보 얻은거로 Controller에게 처리 위임 (DispatcherServlet: RequestMappingHandlerAdapter 전략 사용) 전략을 초기에 작성하는데

dispatcherServlet - getHandlerAdapter

![Untitled](https://user-images.githubusercontent.com/61227459/183903401-569ee61b-cadf-4710-887d-313b042be453.png)

HandlerMethod의 인스턴스면 해당 어댑터 선택 —→ Default 전략 사용 시 명시한 Adapter에 따라 순서가 정해짐

![Untitled 1](https://user-images.githubusercontent.com/61227459/183903382-40de61af-e855-4e83-b864-4ab67801f2dc.png)

![Untitled 2](https://user-images.githubusercontent.com/61227459/183903388-46cfcb86-57bd-4374-86a9-0bbcfec50e9c.png)

실제 처리는 invokeHandlerMethod: Data bind, Model Factory로 모델 생성

![Untitled 3](https://user-images.githubusercontent.com/61227459/183903390-9cf53e72-1e37-4565-b7eb-77fe3ef9d5d1.png)

**브라우저 요청 -> 톰캣서버 -> Host -> Context -> Servlet (DispathcerServlet) -> 결과값 응답 순으로 실행된다**

- Filter: Request 전처리 담당
- Servlet: 사용자 Request 처리
- Listener: Servlet이 달라졌을때 Update


![Untitled 4](https://user-images.githubusercontent.com/61227459/183903394-a26984de-dca3-4a52-a3e5-4f35662bd357.png)

![Untitled 5](https://user-images.githubusercontent.com/61227459/183903399-2571f971-72e7-4fcb-bbc2-9045c99ab8e9.png)

Interceptor와 Filter?

- Interceptor: DispatcherServlet (Spring). 세밀한 처리.
- Filter: WAS. 범용적인 처리

톰캣 UML: [https://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/request-process.png](https://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/request-process.png)

XML files

ServletContext: Servlet이 생성되는 시점에 사용 (Viewresolver etc)

ApplicationContext: Application 당 1개 생성. Spring Container 구성

Web XML: WAS 서버 구동시 사용할 Servlet 목록

ContextLoaderListener: ApplicationContext를 Servlet Container에 등록