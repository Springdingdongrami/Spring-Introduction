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

# 5. 회원 관리 예제 - 웹 MVC 개발

### 회원 웹 기능 - 홈 화면 추가

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

** 컨트롤러가 정적 파일보다 우선순위가 높다.

### 회원 웹 기능 - 등록

- 회원 등록 폼 컨트롤러 (MemberController)
    
    ```java
    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }
    ```
    

- 회원 등록 컨트롤러
    - 웹 등록 화면에서 데이터를 전달 받을 폼 객체
        
        ```java
        package hello.hellospring.controller;
        
        public class MemberForm {
            private String name;
        
            public String getName() {
                return name;
            }
        
            public void setName(String name) {
                this.name = name;
            }
        }
        ```
        
    - 회원을 실제 등록하는 기능 (MemberController)
        
        ```java
        @PostMapping("/members/new")
        public String create(MemberForm form) {
            Member member = new Member();
            member.setName(form.getName());
        
            memberService.join(member);
        
            return "redirect:/";
        }
        ```
        

### 회원 웹 기능 - 조회

- 회원 컨트롤러에서 조회 기능 (MemberController)
    
    ```java
    @GetMapping(value = "/members")
      public String list(Model model) {
          List<Member> members = memberService.findMembers();
          model.addAttribute("members", members);
          return "members/memberList";
    }
    ```

# 6. 스프링 DB 접근 기술

### H2 데이터베이스 설치

1. 1.4.200 버전 설치 (Platform-Independent Zip)
    - https://www.h2database.com/html/download-archive.html
2. 압축 푼 후 권한 주기
    - cd bin
    - chmod 755 h2.sh
3. 실행
    - ./h2.sh

---

![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/c911a399-f365-41ca-8453-b1f8d5311895)


- JDBC URL → 홈 경로 (내 파일)
- 연결하면 test.mv.db 파일 생김
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/77644e9e-29a9-4045-bd30-0a208efb51d1)

    
- 이후엔 JDBC URL → `jdbc:h2:tcp://localhost/~/test`로 접속
    - 파일에 직접 접근하지 않고 소켓을 통해 접근한다.
    - 여러 곳에서 접근 가능 (충돌 X)
- 테이블 관리를 위해 프로젝트 루트에 sql/ddl.sql 파일 생성해주자.
    
    ```sql
    drop table if exists member CASCADE;
    create table member
    (
        id   bigint generated by default as identity,
        name varchar(255),
        primary key (id)
    );
    ```
    

### 순수 JDBC

** build.gradle에 의존성 추가 (jdbc, h2 데이터베이스 관련 라이브러리)

```
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'
```

** resources/application.properties에 데이터베이스 연결 설정 추가

```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```

- JdbcMemberRepository
    
    ```java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    import org.springframework.jdbc.datasource.DataSourceUtils;
    import javax.sql.DataSource;
    import java.sql.*;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.Optional;
    
    public class JdbcMemberRepository implements MemberRepository {
        private final DataSource dataSource;
    
        public JdbcMemberRepository(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    
        @Override
        public Member save(Member member) {
            String sql = "insert into member(name) values(?)";
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null; // 결과를 받음
    
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql,
                        Statement.RETURN_GENERATED_KEYS); // id key
    
                pstmt.setString(1, member.getName());
    
                pstmt.executeUpdate(); // db 실제 쿼리 수행
                rs = pstmt.getGeneratedKeys();
    
                if (rs.next()) {
                    member.setId(rs.getLong(1));
                } else {
                    throw new SQLException("id 조회 실패");
                }
    
                return member;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs); // 자원 release
            }
        }
    
        @Override
        public Optional<Member> findById(Long id) {
            String sql = "select * from member where id = ?";
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setLong(1, id);
    
                rs = pstmt.executeQuery(); // 조회
    
                if (rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                } else {
                    return Optional.empty();
                }
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
    
        @Override
        public List<Member> findAll() {
            String sql = "select * from member";
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                rs = pstmt.executeQuery();
    
                List<Member> members = new ArrayList<>();
    
                while (rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    members.add(member);
                }
    
                return members;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
    
        @Override
        public Optional<Member> findByName(String name) {
            String sql = "select * from member where name = ?";
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setString(1, name);
                rs = pstmt.executeQuery();
    
                if (rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                }
    
                return Optional.empty();
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
    
        private Connection getConnection() {
            return DataSourceUtils.getConnection(dataSource);
        }
    
        private void close(Connection conn, PreparedStatement pstmt, ResultSet rs) {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn != null) {
                    close(conn);
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    
        private void close(Connection conn) throws SQLException {
            DataSourceUtils.releaseConnection(conn, dataSource);
        }
    }
    ```
    
- SpringConfig
    
    ```java
    package hello.hellospring;
    
    import hello.hellospring.repository.JdbcMemberRepository;
    import hello.hellospring.repository.MemberRepository;
    import hello.hellospring.repository.MemoryMemberRepository;
    import hello.hellospring.service.MemberService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    import javax.sql.DataSource;
    
    @Configuration
    public class SpringConfig {
    
        private DataSource dataSource;
    
        @Autowired
        public SpringConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository() {
            // return new MemoryMemberRepository();
            return new JdbcMemberRepository(dataSource);
        }
    
    }
    ```
    

→ 데이터를 DB에 저장하므로 스프링 서버를 다시 실행해도 데이터가 안전하게 저장된다.

![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/b646f190-3a9f-499f-9f65-cdf5d7b1f597)


- 개방-폐쇄 원칙 (OCP)
    - 확장에는 열려 있고, 수정에는 닫혀 있다.
- 스프링의 DI → 기존 코드를 전혀 손대지 않고, 설정만으로 구현 클래스를 변경할 수 있다.

### 스프링 통합 테스트

- `@SpringBootTest`
    - 스프링 컨테이너와 테스트를 함께 실행한다.
- `@Transactional`
    - 테스트 시작 전에 트랜잭션을 시작하고, 테스트 끝나면 rollback (DB에 데이터 반영 X)
    - 테스트 반복 가능

** 가급적이면 스프링 컨테이너 없이 순수한 단위 테스트를 하도록.. (순수한 자바코드로 테스트)

### 스프링 JdbcTemplate

- 스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거해준다.

- JdbcTemplateMemberRepository
    
    ```java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.jdbc.core.RowMapper;
    import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
    import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
    
    import javax.sql.DataSource;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    import java.util.Optional;
    
    public class JdbcTemplateMemberRepository implements MemberRepository {
    
        private final JdbcTemplate jdbcTemplate;
    
        // @Autowired
        // 생성자가 하나일 때 @Autowired 생략 가능
        public JdbcTemplateMemberRepository(DataSource dataSource) {
            this.jdbcTemplate = new JdbcTemplate(dataSource);
        }
    
        @Override
        public Member save(Member member) {
            SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
            jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
    
            Map<String, Object> parameters = new HashMap<>();
            parameters.put("name", member.getName());
    
            Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
            member.setId(key.longValue());
    
            return member;
        }
    
        @Override
        public Optional<Member> findById(Long id) {
            List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
            return result.stream().findAny();
        }
    
        @Override
        public Optional<Member> findByName(String name) {
            List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
            return result.stream().findAny();
        }
    
        @Override
        public List<Member> findAll() {
            return jdbcTemplate.query("select * from member", memberRowMapper());
        }
    
        private RowMapper<Member> memberRowMapper() {
            return (rs, rowNum) -> {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return member;
            };
        }
    }
    ```
    
- SpringConfig
    
    ```java
    package hello.hellospring;
    
    import hello.hellospring.repository.JdbcMemberRepository;
    import hello.hellospring.repository.JdbcTemplateMemberRepository;
    import hello.hellospring.repository.MemberRepository;
    import hello.hellospring.repository.MemoryMemberRepository;
    import hello.hellospring.service.MemberService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    import javax.sql.DataSource;
    
    @Configuration
    public class SpringConfig {
    
        private DataSource dataSource;
    
        @Autowired
        public SpringConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository() {
            return new JdbcTemplateMemberRepository(dataSource);
        }
    
    }
    ```
    

### JPA

- JPA는 기존의 반복 코드 + 기본적인 SQL 직접 만들어서 실행해준다.
- JPA를 사용하면 SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있다.
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다.

** build.gradle에 의존성 추가 (JPA 관련 라이브러리)

```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

** application.properties

```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.jpa.show-sql = true
spring.jpa.hibernate.ddl-auto=none
```

- `show-sql`
    - JPA가 생성하는 SQL을 출력한다.
- `ddl-auto`
    - JPA는 테이블을 자동으로 생성하는 기능을 제공하는데 `none`을 사용하면 해당 기능을 끈다.
    - create를 사용하면 엔티티 정보를 바탕으로 테이블도 직접 생성해준다.

- Member
    
    ```java
    package hello.hellospring.domain;
    
    import jakarta.persistence.Entity;
    import jakarta.persistence.GeneratedValue;
    import jakarta.persistence.GenerationType;
    import jakarta.persistence.Id;
    
    @Entity
    public class Member {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY) // IDENTITY -> DB가 id를 자동으로 생성
        private Long id;
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
    
- JpaMemberRepository
    
    ```java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    import jakarta.persistence.EntityManager;
    
    import java.util.List;
    import java.util.Optional;
    
    public class JpaMemberRepository implements MemberRepository {
    
        private final EntityManager em;
    
        public JpaMemberRepository(EntityManager em) {
            this.em = em;
        }
    
        @Override
        public Member save(Member member) {
            em.persist(member);
            return member;
        }
    
        @Override
        public Optional<Member> findById(Long id) {
            Member member = em.find(Member.class, id);
            return Optional.ofNullable(member);
        }
    
        @Override
        public Optional<Member> findByName(String name) {
            List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                    .setParameter("name", name)
                    .getResultList();
            return result.stream().findAny();
        }
    
        @Override
        public List<Member> findAll() {
            return em.createQuery("select m from Member m", Member.class)
                    .getResultList();
        }
    }
    ```
    
- SpringConfig
    
    ```java
    package hello.hellospring;
    
    import hello.hellospring.repository.*;
    import hello.hellospring.service.MemberService;
    import jakarta.persistence.EntityManager;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    import javax.sql.DataSource;
    
    @Configuration
    public class SpringConfig {
        
        private EntityManager em;
    
        @Autowired
        public SpringConfig(EntityManager em) {
            this.em = em;
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository() {
            return new JpaMemberRepository(em);
        }
    
    }
    ```
    

** 서비스에 `@Transactional` 붙여주기

### 스프링 데이터 JPA

- SpringDataJpaMemberRepository
    
    ```java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    import org.springframework.data.jpa.repository.JpaRepository;
    
    import java.util.Optional;
    
    public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {
        @Override
        Optional<Member> findByName(String name);
    }
    ```
    
- SpringConfig
    
    ```java
    package hello.hellospring;
    
    import hello.hellospring.repository.*;
    import hello.hellospring.service.MemberService;
    import jakarta.persistence.EntityManager;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    import javax.sql.DataSource;
    
    @Configuration
    public class SpringConfig {
    
        private final MemberRepository memberRepository;
    
        @Autowired
        public SpringConfig(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository);
        }
    
    }
    ```
    

- 스프링 데이터 JPA가 `SpringDataJpaMemberRepository`를 스프링 빈으로 자동 등록

- 스프링 데이터 JPA 제공 클래스
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/eaa85e39-868f-4bf7-98e9-666666644d53)

    
- 스프링 데이터 JPA 제공 기능
    - 인터페이스를 통한 CRUD
    - findByName(), findByEmail() 처럼 메서드 이름 만으로 조회 기능 제공
    - 페이징 기능 자동 제공

** 동적 쿼리 → Querydsl

# 7. AOP

### AOP가 필요한 상황

- MemberService 회원 조회 시간 측정 추가
    
    ```java
    package hello.hellospring.service;
    
    import hello.hellospring.domain.Member;
    import hello.hellospring.repository.MemberRepository;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    
    import java.util.List;
    import java.util.Optional;
    
    //@Service
    @Transactional
    public class MemberService {
    
        private final MemberRepository memberRepository;
    
    //    @Autowired
        public MemberService(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        /**
         * 회원 가입
         */
        public Long join(Member member) {
            long start = System.currentTimeMillis();
    
            try {
                validateDuplicateMember(member); // 중복 회원 검증
                memberRepository.save(member);
                return member.getId();
            } finally {
                long finish = System.currentTimeMillis();
                long timeMs = finish - start;
                System.out.println("join " + timeMs + "ms");
            }
        }
    
        private void validateDuplicateMember(Member member) {
            memberRepository.findByName(member.getName())
                    .ifPresent(m -> {
                        throw new IllegalStateException("이미 존재하는 회원입니다.");
                    });
        }
    
        /**
         * 전체 회원 조회
         */
        public List<Member> findMembers() {
            long start = System.currentTimeMillis();
    
            try {
                return memberRepository.findAll();
            } finally {
                long finish = System.currentTimeMillis();
                long timeMs = finish - start;
                System.out.println("findMembers " + timeMs + "ms");
            }
        }
    
        public Optional<Member> findOne(Long memberId) {
            return memberRepository.findById(memberId);
        }
    }
    ```
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/cd36fe39-64d7-41e0-a01d-62f8c7c55e1a)

    

- 문제점
    - 시간 측정 → 핵심 관심 사항이 아니다.
    - 시간을 측정하는 로직은 공통 관심 사항이다.
    - 시간을 측정하는 로직과 핵심 비즈니스 로직이 섞여서 유지보수가 어렵다.
    - 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 어렵다.
    - 시간을 측정하는 로직을 변경할 때 모든 로직을 찾아가면서 변경해야 한다.

⇒ AOP 적용해보자.

### AOP 적용

- AOP (Aspect Oriented Programming)
    - 공통 관심 사항 / 핵심 관심 사항 분리
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/8a157f32-0561-47f9-87d3-ac4e94ec6819)

    

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component // 스프링 빈 등록
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..))") // 공통 관심 사항 적용 (패키지 하위 모두 적용)
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());

        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

** 스프링 빈 등록 → SpringConfig 파일에서도 가능

```java
package hello.hellospring;

import hello.hellospring.aop.TimeTraceAop;
import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import jakarta.persistence.EntityManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class SpringConfig {

    private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }

    @Bean
    public TimeTraceAop timeTraceAop() {
        return new TimeTraceAop();
    }
}
```

- 문제점 해결
    - 회원가입, 회원 조회 등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리했다.
    - 시간을 측정하는 로직을 별도의 공통 로직으로 만들었다.
    - 핵심 관심 사항을 깔끔하게 유지할 수 있다.
    - 변경이 필요하면 이 로직만 변경하면 된다.
    - 원하는 적용 대상을 선택할 수 있다.

- AOP 적용 전 의존 관계
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/c6b2e948-6418-451e-a746-81803feef20d)



- AOP 적용 후 의존 관계
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/c6780b61-d18b-458f-9a14-31f68467ad4c)

    

- AOP 적용 전 전체 그림
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/8c68d2dd-c013-4a2d-b14b-7aa969f395dc)

    

- AOP 적용 후 전체 그림
    
    ![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/6406674f-fa51-42c5-8c39-de1ffb0e5cac)

