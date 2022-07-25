# 스프링 핵심 원리 기본편 - 4강 Application Context

**ApplicationContext (Spring Container): 설정 정보를 참고해서 의존 관계를 맺어주고, Bean 객체들을 저장**

사용 이유

→ new 키워드로 객체를 생성하게 된다면 Singleton 유지 불가

→ 의존성 주입 DI의 간편화

![Untitled](https://user-images.githubusercontent.com/61227459/180781248-c910a9da-d11a-4547-a600-c93a9287a44a.png)

![Untitled 1](https://user-images.githubusercontent.com/61227459/180781229-8b638d97-eb03-42a8-b1d2-e6fa832044a8.png)

Bean Factory: Bean 관리 및 조회

Application Context: 국제화 기능 (언어 변경), 환경 변수 (로컬, 개발 운영 구분), 이벤트 모델 지원, 리소스 조회 등의 Interface를 추가로 구현

구현 클래스 (xml, java config 등): ApplicationContext를 구현하여, Bean Definition을 반환.

**Bean Definition을 사용하여 스프링 컨테이너는 Bean을 생성하고 등록한다**

**팩토리 메서드: 객체 생성을 하위 클래스에게 위임하여 처리하는 패턴**

- 동적으로 객체를 다르게 생성해야 할 경우
- 확장이 예측될 경우

![Untitled 2](https://user-images.githubusercontent.com/61227459/180781236-064241b6-9181-4d38-a734-f44faa97ff1e.png)

ElevatorScheduler: 다양한 Scheduler의 Interface 역할

ScheduleFactory: ElevatorManager (Client)로 부터 전달 받은 정보 (시간)을 사용하여 Scheduler 객체를 반환

ElevatorManager: SchedulerFactory로 부터 전달 받은 객체를 사용하여 알고리즘 수행. 결과 전달

→ 신규 Scheduler가 생성되어도, Manager는 바뀌지 않음. SchedulerFactory만 수정

**전략패턴: 특정 컨텍스트에서 알고리즘을 별도로 분리하는 설계**

**템플릿 메서드 (콜백): 전략패턴의 일종.변화하는 부분을 인터페이스로 선언하고, 이를 구현하는 구현체 객체들을 사용하여 다른 결과를 도출.** 

![Untitled 3](https://user-images.githubusercontent.com/61227459/180781239-35a0f7b5-939d-4348-b97d-b557b03ed8a8.png)

다른 Print문을 얻고자 하면 새로운 메서드를 추가하고, Main의 호출문도 바꾸고, 증복되는 코드도 매우 많음. 

**템플릿 콜백 적용**

![Untitled 4](https://user-images.githubusercontent.com/61227459/180781243-bdd4e27c-9100-42f0-87ca-b4e141812d3f.png)

변경되는 부분을 따로 Interface로 선언 (void b() 메서드)

Interface를 상속하여 구현하는 클래스의 객체를 (Callback) 사용하여 Template를 호출

→ 신규 메서드 추가시엔, Inteface를 구현하는 클래스만 추가. 

간단한 구현체라면 보통 익명 클래스 활용

![Untitled 5](https://user-images.githubusercontent.com/61227459/180781245-c382dbe6-2a57-4305-bb8d-5612018eaa4a.png)