# 더 자바, 애플리케이션을 테스트하는 다양한 방법 - 1강 JUnit 5

# 모듈 구조

- Jupiter
    - TestEngine API 구현체로 JUnit 5를 제공
- Vintage
    - JUnit 4와 3을 지원하는 TestEngine 구현체
- JUnit Platform
    - 테스트를 실행해주는 런처 제공
    - TestEngine API 제공

<aside>
💡 JAVA Reflection을 활용하면서 public을 쓸 필요가 없어짐

</aside>

# 기본 애노테이션

- `@Test`
    - 테스트 단위
- `@BeforeAll`
    - 모든 테스트를 실행 전에 한번
    - static을 활용
    - return 안됨
- `@AfterAll`
    - 모든 테스트를 실행 후에 한번
    - static을 활용
    - return 안됨
- `@BeforeEach`
    - 각 테스트를 실행 전에 한번씩
    - static 필요없음
- `@AfterEach`
    - 각 테스트를 실행 후에 한번씩
    - static 필요없음
- `@Disabled`
    - 테스트를 실행하지 않음

# 테스트 이름 표기

![Untitled](https://user-images.githubusercontent.com/61227459/190576775-2893de04-b9cc-4f21-bc5d-263d5e565eb0.png)

- 카멜케이스 보다는 언더바로

## `@DisplayNameGeneration`

![Untitled 1](https://user-images.githubusercontent.com/61227459/190576735-3cf524a5-114e-4ab1-ad94-a7ab7077b001.png)

- Method와 Class 레퍼런스를 사용해서 테스트 이름을 표기하는 방법 설정
- 기본 구현체로 ReplaceUnderscores 제공
    - `@DisplayNameGeneration(DisplayNameGeneration.ReplaceUnderscores.class)`

## `@DisplayName`

![Untitled 2](https://user-images.githubusercontent.com/61227459/190576737-012a7ead-065b-435b-9ae7-5e491758b364.png)

- 어떤 테스트인지 테스트 이름을 보다 쉽게 표현할 수 있는 방법을 제공하는 애노테이션
- `@DisplayNameGeneration` 보다 우선 순위가 높음

# Assertion

<aside>
💡 마지막 매개변수로 Supplier<String> 타입의 인스턴스를 람다 형태로 제공 가능

→ 복잡한 메시지 생성해야 하는 경우 사용하면 실패한 경우에만 해당 메시지를 만들게 할 수 있음

→ 즉, 람다를 쓰면 실패 시에만 메세지 만들고 람다를 안 쓰면 성공/실패 매번 생성

</aside>

| --- | --- |

```java
// assertAll을 쓰지 않으면 첫번째 테스트에서 막힐 수 있음
// assertNotNull에서 막히면 assertTrue가 실행이 안되어서 다른 테스트 실행 불가
assertAll(
	() -> assertNotNull(study),
	() -> assertTrue(""),
	```
);
```

# 조건에 따라 테스트 실행

## `assumeTrue`(조건)

- 조건이 false면 `assumeTrue` 이후로는 실행이 안됨

## `assumingThat`(조건, 테스트)

- 조건이 true일 때 `assumingThat` 안에 있는 테스트 실행

## `@Enabled___` / `@Disabled___`

- `OnOS`
    - OS 별로 설정 (OS.MAC, OS.Linux, etc…)
- `OnJre`
    - JAVA 버전 (JRE.JAVA_11, etc…)
- `IfEnvironmentVariable`
    - 환경변수 참조하여 실행 여부 결정

# 태깅과 필터링

> 테스트 그룹을 만들고 원하는 테스트 그룹만 테스트를 실행
> 

## `@Tag`

- 테스트 메소드에 태그를 추가
- 하나의 테스트 메소드에 여러 태그를 사용

## IntelliJ에서 설정

Run/Debug Config. → Test kind → Tags


![Untitled 3](https://user-images.githubusercontent.com/61227459/190576739-6b750c8b-7496-4579-960e-e2db606ec11d.png)

## Maven에서 필터링

```xml
<profiles>
	<profile>
		<id>임의의 아이디</id>
			<build>
				<plugins>
					<plugin>
					    <artifactId>maven-surefire-plugin</artifactId>
					    <configuration>
					        <groups>fast | slow</groups>
					    </configuration>
					</plugin>
				</plugins>
			</build>
	</profile>
</profiles>
-----------------------------------------------------------------------
./mvnw test -P <id 태그>
```

## 커스텀 태그

```java
// 별도 생성
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Tag("fast")
@Test
public @interface FastTest {
}
-----------------------------------
@FastTest
@DisplayName("스터디 만들기 fast")
void create_new_study() {
}
```

- 별도 `@interface` 생성
- `@Retention` : 어느 시점까지 어노테이션의 메모리를 가져갈 지 설정
    - RetentionPolicy.SOURCE : 소스 코드(.java)까지 남아있는다.
    - RetentionPolicy.CLASS : 클래스 파일(.class)까지 남아있는다.(=바이트 코드)
    - RetentionPolicy.RUNTIME : 런타임까지 남아있는다.(=사실상 안 사라진다.)
- `@Target` : 어디에서 사용 가능한가
    - ex. 필드, 메소드, 클래스, 파라미터 등


![Untitled 4](https://user-images.githubusercontent.com/61227459/190576742-514aef4a-59b3-43dc-87e4-e480af55e44c.png)

# 테스트 반복

## `@RepeatedTest()`

- 반복 횟수와 반복 테스트 이름을 설정
    - {displayName}
    - {currentRepetition}
    - {totalRepetitions}
- `RepetitionInfo` 타입의 인자를 받을 수 있음

```java
@DisplayName("스터디 만들기")
@RepeatedTest(value="10", name="{displayName}, {currentRepetition}/{totalRepetitions}")
void repeatTest(RepetitionInfo repetitionInfo){
	System.out.println("now" + repetitionInfo.getCurrentRepetition());
	System.out.println("total" + repetitionInfo.getTotalRepetitions());
}
```

![Untitled 5](https://user-images.githubusercontent.com/61227459/190576743-44603db7-f877-4c41-8b16-0841648f751a.png)

## `@ParameterizedTest`

![Untitled 6](https://user-images.githubusercontent.com/61227459/190576745-4a133c36-e1de-43ce-9fe3-dafefeadda1e.png)

![Untitled 7](https://user-images.githubusercontent.com/61227459/190576747-cc1e1c9d-f6ce-4323-ad92-7776d3c493c3.png)

## 인자값의 소스

- `@ValueSource`
    - 메스드에 인자 하나만 넣을 수 있음
- `@NullSource` + `@EmptySource` = `@NullAndEmptySource`
    - 추가적으로 테스트 하나 더
    - null 또는 빈칸으로
- `@EnumSource`
- `@MethodSource`
- `@CsvSource`
- `@CvsFileSource`
- `@ArgumentSource`

## 인자 값 타입 변환

### 암묵적인 타입 변환

- [레퍼런스](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit) 참고
    
    ![Untitled 8](https://user-images.githubusercontent.com/61227459/190576749-c97171e3-90b2-4dae-be1a-453c58d1279f.png)

### 명시적인 타입 변환

![Untitled 9](https://user-images.githubusercontent.com/61227459/190576753-ac159050-8a96-4aeb-b5da-a71e226e661a.png)

![Untitled 10](https://user-images.githubusercontent.com/61227459/190576755-552b622c-29da-4ae4-87f9-a7d5ac76eced.png)

![Untitled 11](https://user-images.githubusercontent.com/61227459/190576758-8893400e-6ff7-4a22-ae62-479579c00c86.png)

- `SimpleArgumentConverter` 상속 받은 구현체 제공
    - `@ConvertWith` 를 활용
    - 하나의 args에만 적용
        - ex. Study study 하나여서 사용 가능
    - 이상의 경우 `ArgumentsAccessor` 사용

## 인자 값 조합

### `ArgumentsAccessor`

![Untitled 12](https://user-images.githubusercontent.com/61227459/190576761-4a11f410-e2f9-4e3d-85f3-d247e2f9fd5a.png)

- `@CsvSource` 활용
- 각 인자 값들을

### 커스텀 `Accessor`

![Untitled 13](https://user-images.githubusercontent.com/61227459/190576765-3e5251c6-d87f-4b4b-b99c-62d3dbb50ceb.png)

![Untitled 14](https://user-images.githubusercontent.com/61227459/190576770-46c0f6b2-bc44-4042-b7e1-e5addadcc042.png)

![Untitled 15](https://user-images.githubusercontent.com/61227459/190576772-504afc1e-86db-49b4-912f-6a6af6936a7b.png)

- `ArgumentsAggregator` 인터페이스 구현
    - public으로 class을 선언해야 활용 가능 (외부 class인 경우)
- `@AggregateWith` 활용

# 테스트 인스턴스

<aside>
💡 각 `@Test`마다 `this`를 출력하면 hashcode가 다른 것을 볼 수 있는데 이는 각 테스트마다 독립적으로 실행 된다는 것을 확인 가능
→ 테스트 메소드를 독립적으로 실행하여 예상치 못한 부작용을 방지하기 위함

</aside>

```java
int value = 1;

@Test
void test1{
	System.out.println(value++);
}

@Test
void test2{
	System.out.println(value++);
}
-------------------------------------
test1이나 test2는 같은 값을 출력 : 1
```

## `@TestInstance(Lifecycle.PER_CLASS)`

- 테스트 클래스당 인스턴스를 하나만 만들어 사용
- 경우에 따라, 테스트 간에 공유하는 모든 상태를 `@BeforeEach` 또는 `@AfterEach`에서 초기화 할 필요 있음
    - 각각의 test에서 같은 인스턴스를 공유하기 때문에 hashcode도 같음
- `@BeforeAll`과 `@AfterAll`을 인스턴스 메소드 또는 인터페이스에 정의한 `default`메소드로 정의 가능
    - `static`를 앞에 붙일 필요가 없어짐

# 테스트 순서

`@TestInstance(Lifecycle.PER_CLASS)`와 함께 `@TestMethodOrder`를 사용하면 원하는 순서대로 실행 가능

- `@Order()` 활용하면 됨

# junit-platform.properties

junit.jupiter.testinstance.lifecycle.default = per_class

- 테스트 인스턴스 라이프사이클 설정

junit.jupiter.extensions.autodetection.enabled = true

- 확장팩 자동 감지 기능

junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition

- `@Disabled` 무시하고 실행하기

`junit.jupiter.displayname.generator.default = \org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores`

- 테스트 이름 표기 전략 설정
    - `ReplaceUnderscores` : replaceUnderscores 대신 함
- \ → 줄 바꿈 역할