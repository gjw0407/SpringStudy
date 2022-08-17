# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 - 4강 MVC 프레임워크 만들기 & 5강 MVC 구조 이해

Servlet/JSP

- Controller, Service + View를 모두 처리하다보니 코드가 복잡
- 유지 보수 Cycle이 다름. View만 고치면 되는데 Controller까지 영향을 받음

![Untitled](https://user-images.githubusercontent.com/61227459/185145515-3c4a8dc2-e179-49e7-92f0-1e3b7c16eae6.png)

기존의 Monolithic한 구조에서 Business 로직 (Controller + Service + Repository)와 View 로직 분리

![Untitled 1](https://user-images.githubusercontent.com/61227459/185145493-ba892c42-6854-44b5-bf47-d5958520c994.png)

구조가 분리되다 보니 View를 생성할 때 사용되는 값들을 담아서 공유하는 객체 필요 (Model)

Controller + View을 분리하고 이 둘을 Model로 연결

**Spring MVC**

![Untitled 2](https://user-images.githubusercontent.com/61227459/185145503-5ee2ad17-dca9-4ed1-9b34-a35917670873.png)

1. HandlerMapping

Component(Controller, Service, Repository)로 등록된 Bean들을 ApplicationContext에서 조회

현재 Request를 처리할 수 있는 Bean Object를 반환

![Untitled 3](https://user-images.githubusercontent.com/61227459/185145506-20ac6f66-d737-4f7e-b730-70584acbb058.png)

0: RequestMapping이나 Controller Annotation이 있는 Bean들이 등록

*1: bean의 이름이 URL과 동일할 때 @Component(”/mypath/myurl”) 일 때

1. HandlerAdapter

반환된 Handler를 다룰 수 있는 Adapter(Handler가 구현하고 있는 클래스 객체)를 조회

- 스프링은 확장을 위해 Controller를 직접 연결하는 것이 아닌, Interface를 찾은 뒤에 구현체를 호출해서 Request를 처리한다. 이 때 어떤 Interface를 구현하고 있는지를 찾는 과정이 HandlerAdapter (support method)

![Untitled 4](https://user-images.githubusercontent.com/61227459/185145508-e9d13f84-4568-4dbd-84e5-5592ef386bcf.png)

0: @RequestMapping 붙은 Bean들을 처리

1: HttpRequestHandler (Servlet과 비슷)를 처리

2: SimpleControllerHandlerAdapter: Controller 

1. handle(handler)

이전 단계에서 찾아낸 handler의 interface에서 handle 메서드를 호출 (handler가 인자로 포함)

1. handle

Inteface 내에서 실제 handler의 handle을 호출 (즉 Request를 처리하는 핵심 코드를 호출) 

- 굳이 Inteface를 가운데 둬서 분리하는 이유는 OCP와 SRP를 준수하기 위함

1. ModelAndView

Business Logic를 거친 뒤 결과값을 Model에 넣어 반환하고, 이를 render 해줄 view의 이름을 ModelAndView라는 객체에 한꺼번에 담아서 반환한다. 

- 지금은 Model과 View를 같이 반환하지만, Model만 DispatcherServlet에서 생성해서 Handler에게 Parameter로 전달해줄 수 있다. 그러면 View name만 반환해주면 된다.

1. viewResolver

현재 반환된 view name은 Logical한 view name이다. 

Prefix와 Suffix를 붙여서 physical name으로 변환해준다.

![Untitled 5](https://user-images.githubusercontent.com/61227459/185145512-45def402-0580-4966-9fa1-ec886a390749.png)

View Resolver는 특정 view template을 처리할 수 있는 view를 반환한다. 

- JSP: InternalResourceViewResolver는 InternalResourceView를 반환하고, 이 View는 JSP를 처리하기 위해 해당 view 객체의 render함수에서 forward()를 호출한다.
    
    JSTL 라이브러리가 있으면 InternalResourceView를 상속받은 JstlView를 반환한다.
    
    다른 View는 forward 없이 바로 render 진행하지만 JSP는 실제 JSP로 이동해서 render를 진행한다 (forwarD)
    
- Thymeleaf: ThymeleafViewResolver를 등록한다.

1. View 반환

viewResolver에서 반환한 Physical view name와 render method를 가진 객체

1. render(model)

view의 render 함수에 model, request, response를 넣고 호출 

model의 attribute를 request로 모두 복사한 뒤 physical view name로 forward