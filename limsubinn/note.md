# 1. 프로젝트 환경설정

### 라이브러리 살펴보기

Gradle은 의존관계가 있는 라이브러리를 함께 다운로드한다.

*스프링 부트 라이브러리*

- spring-boot-starter-web
    - spring-boot-starter-tomcat: 톰캣(웹서버)
    - spring-webmvc: 스프링 웹 MVC
- spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
        - logback, slf4j

*테스트 라이브러리*

- spring-boot-starter-test
    - junit: 테스트 프레임워크
    - mockito: 목 라이브러리
    - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test: 스프링 통합 테스트 지원

### thymeleaf 템플릿엔진 동작 확인

<img width="704" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/e0bb19b2-dbbf-4f91-9f36-60dbaa1d5685">


- 컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버(`viewResolver`)가 화면을 찾아서 처리한다.
    - 스프링 부트 템플릿엔진 기본 viewName 매핑
    - `resources:templates/` + {viewName} + `.html`

- 참고: `spring-boot-devtools` 라이브러리를 추가하면, html 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능하다.

### 빌드하고 실행하기

```
% ./gradlew build
% cd build/libs
% java -jar hello-spring-0.0.1-SNAPSHOT.jar
실행 확인
```

<img width="528" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/2bda366c-1865-4cc4-b01d-58273781b7e2">

# 2. 스프링 웹 개발 기초

### 정적 컨텐츠

<img width="703" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/325c4200-86e6-423e-9f3c-cbc36151623a">


### MVC와 템플릿 엔진

- MVC: Model - View - Controller

<img width="710" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/6a0cdf04-2a16-41cf-83e7-ea4b6466da74">


** 정적 컨텐츠 → 변환하지 않고 넘김

** 템플릿 엔진 → 변환하고 넘김

### API

<img width="712" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/cf88df07-63c9-432c-b01e-a621d4be3020">


- `@ResponseBody`
    - http의 body에 문자 내용을 직접 반환
    - `viewResolver` 대신 `HttpMessageConverter`가 동작
    - 기본 문자 처리 → `StringHttpMessageConverter`
    - 기본 객체 처리 → `MappingJackson2HttpMessageConverter` (json)
    

** 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보를 조합해서 `HttpMessageConverter`가 선택된다.

# 3. 회원 관리 예제 - 백엔드 개발

### 비즈니스 요구사항 정리

- 데이터: 회원ID, 이름
- 기능: 회원 등록, 조회

<일반적인 웹 애플리케이션 계층 구조>

<img width="712" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/96822c15-ed74-4c98-b3d3-f0d930b0f49e">


- 컨트롤러
- 서비스: 핵심 비즈니스 로직 구현
- 리포지토리: DB 접근 - 도메인 객체를 DB에 저장하고 관리
- 도메인: 비즈니스 도메인 객체. ex) 회원, 주문, 쿠폰 등

<클래스 의존관계>

<img width="710" alt="image" src="https://github.com/Springdingdongrami/spring-introduction/assets/66028419/e669f9a9-82e0-415a-b4a1-b738799bd767">


- 인터페이스로 구현 클래스를 변경할 수 있도록 설계 (아직 데이터 저장소가 선정되지 않음)

### 회원 도메인과 리포지토리 만들기

- `Optional<T>` : 값의 존재나 부재 여부를 표현하는 컨테이너 클래스
    
    값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다.
    
    - of : null값 비허용 (null값을 인자로 받으면 NullPointerException)
    - ofNullable : null값 허용
    - isPresent() : 값을 포함하면 true, 값을 포함하지 않으면 false 반환
    - ifPresent(Consumer<T> block) : 값이 있으면 주어진 블록을 실행 (없으면 아무 일도 없음)
    - T get() : 값이 존재하면 값을 반환, 값이 없으면 NoSuchElementException
    - T orElse(T other) : 값이 있으면 값을 반환, 없으면 기본값을 반환

- `stream` : ‘데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소’
    - filter: 람다를 인수로 받아 스트림에서 특정 요소를 제외시킨다.
    - map: 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출한다.
    - limit: 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소한다.
    - collect: 스트림을 다른 형식으로 변환한다.
    
    ** 검색과 매칭 : anyMatch, allMatch, noneMatch, findAny
    

### 회원 리포지토리 테스트 케이스 작성

- 자바 JUnit 프레임워크로 테스트를 실행 
→ 반복 실행하기 어렵고 여러 테스트를 한번에 실행하기 어려운 문제를 해결

- 테스트 코드 - 순서 보장 X
    
    → 서로 의존관계 없이 설계해줘야 한다.
    
    → 테스트 하나가 끝나고 나면 데이터를 지워줘야 한다.
    
- 테스트 어노테이션
    - `@Test`: 테스트를 수행하는 메서드
    - `@Before`: 테스트 메서드가 실행되기 전에 실행 (BeforeEach / BeforeAll)
    - `@After`: 테스트 메서드가 실행된 후에 실행 (AfterEach / AfterAll)

### 회원 서비스 개발 & 테스트 케이스 작성

- cmd + option + m → 메서드 추출
- cmd + shift + t → 테스트 클래스 작성
- memberService 입장에서 memberRepository를 외부에서 넣어줌 → DI

# 4. 스프링 빈과 의존관계

- 스프링 빈을 등록하는 2가지 방법
    1. 컴포넌트 스캔과 자동 의존관계 설정
    2. 자바 코드로 직접 스프링 빈 등록하기

- 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용한다. 그리고 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다.

### 컴포넌트 스캔과 자동 의존관계 설정

- 회원 컨트롤러가 회원 서비스와 회원 리포지토리를 사용할 수 있게 의존관계를 추가한다.
- **의존관계 주입(의존성 주입)** : Dependency Injection, DI
    - 애플리케이션 실행 시점(런타임)에 객체를 직접 생성하는게 아니라 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결되는 것
    - 이를 통해 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
    - 의존성 주입을 통해 모듈 간의 결합도가 낮아지고 유연성이 높아진다.

- `@Component` - 컴포넌트 스캔 (스프링 빈으로 자동 등록된다.)
    - `@Controller`, `@Service`, `@Repository` (@Component 포함)
- `@Autowired` 의존 관계 주입(연결)
    
    1) **생성자 주입** ⭐️⭐️⭐️
    
    - Constructor 생성자를 통해 의존 관계를 주입하는 방법
    - 불변으로 설계가 가능하다.
    - 필드에 final 키워드를 사용할 수 있다.
    → 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
    - 클래스의 생성자가 하나이고, 그 생성자로 주입받을 객체가 빈으로 등록되어 있다면 어노테이션 생략이 가능하다.
    
    2) 수정자 주입
    
    - 의존해야 하는 객체를 선언한 후 해당 변수에 대한 Setter 메서드를 생성하고, 해당 Setter에 @Autowired를 붙여서 사용한다.
    - 선택적이고 변화 가능한 의존 관계에 사용한다.
    
    3) 필드 주입
    
    - 변수에 @Autowired를 붙여서 사용한다.
    - 제일 간단하지만, 단점이 많음! 사용하지 않는 것을 권장함!
    
    * **빈 등록 충돌**: 타입이 같은 빈을 찾아 주입하기 때문에 어떤 빈을 주입해줘야 할지 판단하지 못함
        → `@Primary` 어노테이션을 통해 우선 순위 지정 가능!
    
    * `@Autowired`를 통한 DI는 스프링이 관리하는 객체에서만 동작한다. 스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않는다.
    

- 스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, 기본으로 싱글톤으로 등록한다. 
→ 같은 스프링 빈이면 모두 같은 인스턴스다.

### 자바 코드로 직접 스프링 빈 등록하기

- @Service, @Repository, @Autowired 어노테이션을 모두 제거하고 진행한다.

- `@Configuration`
    - 해당 어노테이션이 붙은 클래스를 설정(구성) 정보로 사용한다.
- `@Bean`
    - 해당 어노테이션이 붙은 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.
    

```java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {
    
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
}
```

→ MemberService, MemberRepository를 스프링 빈으로 등록한다.

```java
package hello.hellospring.controller;

import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

→ 컴포넌트 스캔이기 때문에 `@Autowired`를 통해 의존관계를 주입한다.
