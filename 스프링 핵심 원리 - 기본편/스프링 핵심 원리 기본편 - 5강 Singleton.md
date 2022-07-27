# 스프링 핵심 원리 기본편 - 5강 Singleton

## 싱글톤 패턴: 클래스 인스턴스가 1개만 생선

1. Eager Initialization: 해당 인스턴스를 안써도 new로 객체 생성됨. 낭비

![Untitled](https://user-images.githubusercontent.com/61227459/181258422-84b20bf6-b0df-44e5-8ff0-54769da88bab.png)

1. Lazy Initialization: public static method인 getinstance가 호출될 때 객체 생성

인스턴스 낭비는 해결되지만 multi-thread 환경에서 정합성 보장 X

![Untitled 1](https://user-images.githubusercontent.com/61227459/181258388-7f471eea-5de7-447e-bdeb-2d06d9b7a5ab.png)

1. Thread Safe Singleton: Synchronized로 인한 성능 저하

![Untitled 2](https://user-images.githubusercontent.com/61227459/181258393-1353fbfb-fa07-4f54-bf14-f7bac58173de.png)

1. Bill Pugh Singleton Implementation: 현재 가장 널리 사용

![Untitled 3](https://user-images.githubusercontent.com/61227459/181258395-743eec89-8cb9-43ce-8f29-c577102fca0a.png)

///// TODO **inner static/inner static method** 

Singleton 패턴 문제점

- 의존 관계에서 구현체 클래스에 의존
- 코드 복잡

싱글톤 컨테이너 (스프링 컨테이너): 상기 문제 해결하면서 싱글톤 보장 

***** Singleton으로 관리되는 객체는 Stateless 해야함 *****

**스프링이 Singleton 유지하는 법**

![Untitled 4](https://user-images.githubusercontent.com/61227459/181258397-7e9445be-b98b-40f5-b960-bce4d1b64f95.png)

MemberRepository는 두 번 생성자가 호출되지만 Singleton을 유지한다

- java → class로 컴파일 (bytecode) 되면서 조작을 가한다
- CGLIB 라이브러리가 AppConfig을 상속하는 다른 클래스를 만들고, 이를 등록한다 (@Configuration Annotation)
- 해당 클래스가 Singleton임을 보장해준다.
- **@Bean 태그로도 bean으로 등록되지만 Configuration Annotation이 없으면 Sigleton 보장이 안된다**

**Proxy와 AOP:** 중복되는 코드를 떼어내어, 객체마다 다른 결과를 반환하게 해주는 개념

스프링은 Proxy라는 객체를 사용하여 AOP를 구현한다.

- 프록시 객체는 Client 객체와 같은 Interface를 구현해야함

![Untitled 5](https://user-images.githubusercontent.com/61227459/181258402-faf7287c-17ff-430b-b1ec-400a00654818.png)

![Untitled 6](https://user-images.githubusercontent.com/61227459/181258404-03aecf00-aefd-453f-9529-41d26317d903.png)

**Factory Bean 메서드**

![Untitled 7](https://user-images.githubusercontent.com/61227459/181258405-fd4e3202-52a0-4e0c-9097-ac3a6c147047.png)

Message 클래스는 생성자가 Private이라 오프젝트 생성이 불가

→ FactoryBean 인터페이스를 구현해서 Bean으로 등록 

![Untitled 8](https://user-images.githubusercontent.com/61227459/181258410-548a238f-44b7-4597-827e-ce062799a6fb.png)

Proxy 구현의 문제

- 프록시 구현을 위해서는 Target 객체를 직접 구현해야함
- 프록시 클래스 내 중복 발생

다이나믹 프록시: Reflection API를 사용하여 동적으로 프록시 객체 생성 가능

![Untitled 9](https://user-images.githubusercontent.com/61227459/181258412-a60ad53c-4222-45d4-954b-360d661d2396.png)

![Untitled 10](https://user-images.githubusercontent.com/61227459/181258413-ecd2b8c3-732e-408c-9eb8-452eeb1387fd.png)

- Proxy 객체를 Dynamic하게 생성하기 위해 (Dynamic Proxy 객체는 Bean 등록 불가) Factory Proxy 사용

![Untitled 11](https://user-images.githubusercontent.com/61227459/181258418-5e7664ad-dec0-493b-981c-01225dd54d94.png)

InvocationHandler는 invoke() 메서드 하나만 가지고 있음.

Client에서 Proxy 객체로 어떤 method를 호출하던지, target 객체의 invoke가 호출됨