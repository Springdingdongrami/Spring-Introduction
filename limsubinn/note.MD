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

![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/c545e4df-07a8-4642-b36c-e1926e857c62)


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

![image](https://github.com/Springdingdongrami/spring-introduction/assets/66028419/91ae9528-e389-4ce8-978b-f287e72d313a)
