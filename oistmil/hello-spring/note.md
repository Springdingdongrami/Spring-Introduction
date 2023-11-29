# 스프링 입문

# 1. 프로젝트 환경설정

### 환경 설정

Java 11 설치

- MAC, 자바 버전 변경
    
    ```bash
    java --version // 자바 버전 확인
    	
    /usr/libexec/java_home -V // JVM 목록 확인
    
    // JAVA_HOME의 경로를 JDK 11로 잡기
    export JAVA_HOME=$(/usr/libexec/java_home -v 11.0)
    source ~/.bash_profile
    ```
    
- Spring Boot 3.0 ~ → java 16부터 지원
    - Hello-Spring 예제를 위해 Java 11 + Spring Boot 2.7.15로 설정 후 사용

### 라이브러리 살펴보기

의존성을 알아서 주입 시켜줌 → 의존 관계에서 필요한 것들을 알아서 가져오게 됨

→ 이를 확인하는 법 : MAC cmd 2번

spring-boot-starter-logging

*log에 대해 궁금하다면? slf4j, logback을 공부해보기*

 

### View 환경설정

Controller

- 웹 애플리케이션의 첫 번째 진입점

```bash
@Controller
public class HelloController {
    @GetMapping("hello") // get 방식
    public String hello(Model model){
        model.addAttribute("data", "hello");
        return "hello"; // resources의 templates안의 hello를 찾아서 실행시켜라라는 의미. 
				// default로 templates에서 찾음
      
    }
}
```

**return viewName;**

컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버(viewResolver)가 화면을 찾아서 처리한다.

- 스프링 부트 템플릿엔진 기본 viewName 매핑
- resources:templates/ +{ViewName}+ .html

### 빌드하고 실행하기

1. `./gradlew build`
2. `cd build/libs`
3. `java -jar [jar파일명].jar`

잘 되지 않는다면, `./gradlew clean` 후 위 과정을 다시 수행

# 2. 스프링 웹 개발 기초

### 정적 컨텐츠

서버에서 하는 것없이 그대로 고객에게 전달해 것 

스프링에서 정적 컨텐츠  → `resources/static`

- 어떠한 프로그래밍 불가
- 정적 컨텐츠를 그냥 전달
- [localhost:8080/파일명](http://localhost:8080/파일명) 형태로 접근 가능
- 내부 동작
    1. 서버가 파일명 관련한 컨트롤러가 매핑되어있는지를 확인
    2. 파일명으로 매핑된 컨트롤러가 없을 시 resources 내부의 파일을 찾음

### MVC와 템플릿 엔진

MVC : Model-View-Controller

MVC - Model2 방식

- Model1은 View에서 모든 일을 함
- View는 화면을 그리는데 역량 집중 필요
- Controller, Model은 비즈니스 로직, 내부 로직 등에 집중 필요

+)

```bash
@RequestParam(name = "name", required = true)
// required=false일 시 param이 없어도 실행. true일 시 param이 있어야 실행됨. (default: required=true)
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/448e2cc3-27fb-4d93-8596-85f3bbdf0975/Untitled.png)

### API

json 등을 통해 전달. 데이터를 제공할 때 API 방식을 자주 사용

`**@ResponseBody` 란?**

- **http**의 body부에 이 데이터를 내가 직접 넣어주겠다는 것. *(html의 body 아님)*

```java
@GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello{
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
```

- 결과가 json으로 리턴됨

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/18f8ca31-61fc-4779-b07e-cc25a683a3da/Untitled.png)

서버가 내부적으로 `@ResponseBody` 를 만나면 데이터를 그냥 넘겨야 겠다고 생각함.

return으로 객체가 넘겨지면 json으로 만들어서 반환. (문자의 경우, StringConveter가 동작 / 객체의 경우 JsonConverter가 동작)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/842b1965-c1af-431d-9137-aaa063da2219/Untitled.png)

# 3. 회원 관리 예제 - 백엔드 개발

### 비즈니스 요구사항 정리

*데이터: 회원ID, 이름
기능: 회원 등록, 조회
아직 데이터 저장소가 선정되지 않음(가상의 시나리오)*

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/79d03b4a-ea31-4795-8af9-162b275c1593/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/0c54731c-1895-4cfa-89cf-62e3544b2859/Untitled.png)

### 회원 도메인과 리포지토리 만들기

*+) Optional* 

- `*Optional.ofNullable(store.get(id));` null이 반환될 가능성이 있을 때 형태로 감싸서 전달. 이후 클라이언트에서 어떠한 작업을 할 수 있음*

### 회원 리포지토리 테스트 케이스 작성

<aside>
💡 개발한 기능을 실행해서 테스트 할 때 자바의 main 메서드를 통해서 실행하거나, 웹 애플리케이션의
컨트롤러를 통해서 해당 기능을 실행한다. 이러한 방법은 준비하고 실행하는데 오래 걸리고, 반복 실행하기 어렵고 여러 테스트를 한번에 실행하기 어렵다는 단점이 있다. 자바는 JUnit이라는 프레임워크로 테스트를 실행해서 이러한 문제를 해결한다.

</aside>

`Assertions.*assertEquals*(member, result);`

- `import org.junit.jupiter.api.Assertions;`
- *assertEquals*(**기댓값**, 실제값)

*`**assertThat*(member).isEqualTo(result);**`

- `import static org.assertj.core.api.Assertions.*;`

스프링 테스트 주의

- 메소드를 하나 하나 실행하지 않고, 전체를 한꺼번에 테스트 하면 테스트가 (메소드 선언 등의) 순서대로 이루어진다는 보장 없음
    
    ⇒ **데이터 클리어**가 필요하다.
    
    - 테스트 후 레포지토리를 깔끔하게 지워주는 코드가 필요
    - `**@AfterEach**`
        - 메소드 실행 후 실행되는 콜백 메소드의 역할
        
        ```java
        @AfterEach
            public void afterEach(){
                repository.clearStore();
            }
        ```
        

테스트 틀을 먼저 만들고 구현하는 방법 - ***TDD(테스트 주도 개발)***

### 회원 서비스 개발

```java
result.ifPresent(m -> {
    throw new IllegalStateException("이미 존재하는 회원입니다.");
});
```

- Optional로 리턴되었기 때문에 Optional의 메소드들을 사용할 수 있다.
    - `ifPresent`는 어떠한 값이 있을 때 안의 내용을 실행

***Service - Business Logic에 비슷하게 설계***

### 회원 서비스 테스트

**given - when - then**

given - 이런 상황이 주어졌을 때

when - 이걸 실행 했을 때

then - 이게 나와야 함

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/e44a4034-4ea0-4b34-9d1a-f83609a4cb40/Untitled.png)

`@BeforeEach` 

- 각 테스트 실행 전에 호출된다. 테스트가 서로 영향이 없도록 항상 새로운 객체를 생성하고, 의존관계도 새로 맺어준다.

# 4. 스프링 빈과 의존관계

스프링 빈을 등록하는 두가지 방법

1. 컴포넌트 스캔과 자동 의존관계 설정
2. 자바 코드로 직접 스프링 빈 등록하기

### 컴포넌트 스캔과 자동 의존관계 설정

멤버 컨트롤러가 멤버 서비스를 통해 회원가입, 데이터 조회 ⇒ 멤버 컨트롤러가 멤버 서비스를 의존한다.

스프링 컨테이너가 Controller를 빈으로 관리

`@Component` - 어노테이션이 있으면 스프링 빈으로 자동 등록됨.

- `@Service`, `@Controller`, `@Repository`  ⇒ 모두 스프링 빈으로 자동 등록됨.

Component의 등록 범위

- `@SpringBootApplication` 어노테이션이 등록된 hello.hellospring 부터 시작하여 이 패키지를 포함한 하위 패키지들만 컴포넌트 스캔의 대상이 됨.

스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, 기본으로 싱글톤으로 등록(유일하게 하나만 등록하여 공유함)

⇒ 따라서 같은 스프링이면 모두 같은 인스턴스다.

### 자바 코드로 직접 스프링 빈 등록하기

*애노테이션을 사용하지 않고*

[SpringConfig.java]

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
```

+) 

필드 주입 (생성자가 아닌 선언부에서 `@Autowired` 설정 *- 바람직하진 않음*)

ex) `@Autowired private final MemberService memberService;`

권장하는 스타일 - **생성자에서** 설정

- DI에는 필드 주입, setter 주입, 생성자 주입 3가지 방법이 있음. 그러나 의존 관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 **생성자 주입을 권장**함.

```java
@Autowired
public MemberController(MemberService memberService){
    this.memberService = memberService;
}
```

# 5. 회원 관리 예제 - 웹 MVC 개발

# 6. 스프링 DB 접근 기술

h2 데이터베이스 설치

[https://www.h2database.com](https://www.h2database.com/)
다운로드 및 설치
h2 데이터베이스 버전은 스프링 부트 버전에 맞춘다.
권한 주기: h2 bin 폴더에서 chmod 755 [h2.sh](http://h2.sh/) (윈도우 사용자는 x)
실행: ./h2.sh (윈도우 사용자는 h2.bat)
데이터베이스 파일 생성 방법
jdbc:h2:~/test (최초 한번)
~/test.mv.db 파일 생성 확인
이후부터는 jdbc:h2:tcp://localhost/~/test 이렇게 접속

create table member

```sql
drop table if exists member CASCADE;
create table member
(
 id bigint generated by default as identity,
 name varchar(255),
 primary key (id)
);
```

- generated by default as identity : 값을 지정하지 않으면 db가 자동으로 값을 넣어준다.

### 순수 JDBC

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/dbb60612-0fea-4b6a-9310-7f728194dfb2/Untitled.png)

- 개방-폐쇄 원칙(OCP, Open-Closed Principle)
- 확장에는 열려있고, 수정, 변경에는 닫혀있다.
- 스프링의 DI (Dependencies Injection)을 사용하면 기존 코드를 전혀 손대지 않고, 설정만으로 구현클래스를 변경할 수 있다.

### 스프링 통합 테스트

### 스프링 JdbcTemplate

스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거해준다. 하지만 SQL은 직접 작성해야 한다.

### JPA

- JPA는 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다.
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다.

`spring.jpa.show-sql=true`

- jpa가 날리는 sql을 볼 수 있음

`spring.jpa.hibernate.ddl-auto=none`

- 객체를 보고 자동으로 테이블을 만들어 줄 수 있음(**true 시에**)

`@Entitiy`

- jpa가 관리하는 엔티티가 됨
- code
    
    ```java
    @Entity
    public class Member {
    
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        // @Column(name = "username") // 컬럼명이 다르다면 이와 같이 추가해 줄 수 있음
        private String name;
    
        public Long getId() {
            return id;
        }
    
        public void setId(Long id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    

### 스프링 데이터 JPA

**스프링 데이터 JPA 제공 기능**

- **인터페이스**를 통한 기본적인 CRUD
- findByName() , findByEmail() 처럼 메서드 이름 만으로 조회 기능 제공
- 페이징 기능 자동 제공

*참고: 실무에서는 JPA와 스프링 데이터 JPA를 기본으로 사용하고, 복잡한 동적 쿼리는 Querydsl이라는
라이브러리를 사용하면 된다. Querydsl을 사용하면 쿼리도 자바 코드로 안전하게 작성할 수 있고, 동적
쿼리도 편리하게 작성할 수 있다. 이 조합으로 해결하기 어려운 쿼리는 JPA가 제공하는 네이티브 쿼리를
사용하거나, 앞서 학습한 스프링 JdbcTemplate를 사용하면 된다.*

# 7. AOP

### AOP가 필요한 상황

- 모든 메소드의 호출 시간을 측정하고 싶다면?
- 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern)
- 회원 가입 시간, 회원 조회 시간을 측정하고 싶다면?

**문제**

- 회원가입, 회원 조회에 시간을 측정하는 기능은 핵심 관심 사항이 아니다.
- 시간을 측정하는 로직은 공통 관심 사항이다.
- 시간을 측정하는 로직과 핵심 비즈니스의 로직이 섞여서 유지보수가 어렵다.
- 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 매우 어렵다.
- 시간을 측정하는 로직을 변경할 때 모든 로직을 찾아가면서 변경해야 한다.

⇒ AOP 적용하기

### AOP 적용

AOP : Aspect Oriented Programming

공통 관심 사항과 핵심 관심 사항을 분리

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/6e63057d-7c78-4622-b0b4-9292be669ec6/Untitled.png)

**해결**

- 회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리한다.
- 시간을 측정하는 로직을 별도의 공통 로직으로 만들었다.
- 핵심 관심 사항을 깔끔하게 유지할 수 있다.
- 변경이 필요하면 이 로직만 변경하면 된다.
- 원하는 적용 대상을 선택할 수 있다

스프링 AOP 적용 동작 방식 설명

AOP 적용 전 의존 관계

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/fb02bd4e-a5a3-449b-978c-bf6e0144fe37/Untitled.png)

AOP 적용 후 의존 관계

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/14a99cb1-5101-4c11-904c-a87fb9cac907/7948a19c-04b0-4734-9341-f9f3fc78fd00/Untitled.png)

가짜 멤버 서비스(proxy)를 생성 → 스프링 컨테이너는 스프링 빈을 등록할 때 진짜 스프링 빈이 아닌 가짜 스프링 빈을 세워둠 → 가짜 스프링 빈 실행이 끝나고 나서 진짜 스프링 빈이 동작됨

- MemberController 생성자에서 `System.*out*.println("memberService = " + memberService.getClass());` 으로 프록시 콘솔로 확인해 볼 수 있음

# 다음으로
