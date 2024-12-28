# SpringBootCoreGuideLogs

(장정우 저자의) 스프링부트 핵심가이드 Review Repository

20241227

**의존성 주입 방법 3가지**
1. Constructor
2. 필드 객체 선언
3. Setter 메서드

**내가 주로 사용하는 방식은 Constructor를 사용한다. 특히, Lombok의 @RequiredArgsConstructor를 사용해왔다. 필요한 객체 없이 대상 객체를 초기화할 수 없도록 설계할 수 있기 때문에 스프링에서는 이 방식을 권장한다고 한다.**


**스프링부트가 스프링보다 편리한 이유**

1. 의존성 관리에 있어서 스프링부트는 스프링보다 편리하다. 왜냐하면 **spring-boot-starter라는 의존성 때문이다. 이것은 각 라이브리러의 기능과 관련해서 자주 사용되고 서로 호환되는 버전의 모듈 조합을 제공한다.**
2. 스프링 부트는 스프링 프레임워크의 기능을 사용하기 위해 필요한 설정을 자동으로 지원한다. @SpringBootApplication 어노테이션을 보면 알 수 있다. 이 어노테이션은 @SpringBootConfiguration, @EnableAuthoConfiguration, @ComponentScan 등으로 구성된다.
  @ComponentScan 어노테이션은 @Component 시리즈의 어노테이션(@Controller, @RestController, @Service, @Repository, @Configuration)이 붙은 클래스를 발견해 빈으로 등록한다.
  @EnableAutoConfiguration 어노테이션을 통해서 spring.factories 파일이 추가되어 자동 설정이 적용된다.
3. 내장 WAS
  spring-boot-starter-web 의존성은 톰캣을 내장한다.
4. 자체 모니터링(스프링부트 액추에이터)도구로 스레드, 메모리, 세션 등의 주요 요소를 모니터링한다.

---

20241228

**테스트 코드 작성 학습 내용 정리**

1. 테스트 코드에서 의존성 주입 간편하게 하기
   기존에는 그냥 검색결과를 따라하면서 @Mock, @MockBean 어노테이션에 대한 학습이 부족했다. 기존에는 @Mock 어노테이션을 (모르면서 그냥 돌아가니까 다행이라는 생각으로) 주로 사용했다. @Mock의 경우 테스트 클래스 내에서 모의 객체를 생성하고 Spring ApplicationContext와는 무관하다. 그래서 아래와 같이 직접 코드를 작성해서 의존성을 주입하도록 했어야 했다.

```java
private MyDependency dependency;

@Mock
private DependencyPart1 part1;
@Mock
private DependencyPart2 part2;

@BeforeEach
    void setUp() {
        dependency = new MyDependency(part1, part2);
    }
```

하지만, 이와 같은 방법은 개발하면서 생각한 과정과 너무 괴리감이 있다고 느꼈다. 개발하면서는 주로 빈 객체를 스프링 컨텍스트에 등록하기 때문에, 이런 과정들이 다 필요가 없다고 생각했기 때문이다. 그래서 책을 읽으면서, 이와 같이 테스트 코드를 변경하려고 하였다. 

그 과정에서 @MockBean이 사용된 것을 보았고, 이 어노테이션이 붙었을 때, 해당 클래스의 객체를 스프링 컨텍스트에 등록하고, 해당 객체가 필요할 때, 주입받을 수 있다는 것을 알게 되었다. 그래서 @AutoWired 어노테이션을 붙여 자동으로 의존성 주입이 될 수 있게, 개발과정과 같은 맥락으로 테스트 코드를 작성할 수 있게 되었다. 아래는 수정코드이다. 

```java
    @Autowired
    private MyDependency dependency;
    @Autowired
    private DependencyPart1 part1;
    @Autowired
    private DependencyPart2 part2;

    //@BeforeEach로 직접 의존성 주입 관리를 할 필요가 없음
```

핵심 요약
@Mock: Mockito에서 제공하는 어노테이션으로, 테스트 클래스 내에서 모의 객체를 생성합니다. 이 객체는 Spring ApplicationContext와는 독립적이며, 수동으로 의존성을 주입해야 합니다.
@MockBean: Spring Boot에서 제공하는 어노테이션으로, 테스트 컨텍스트에 모의 객체를 등록합니다. 이 객체는 Spring의 의존성 주입을 통해 자동으로 주입될 수 있습니다.

2. 컨트롤러 단위테스트를 방해한 JPA, Spring Security
   책의 테스트 코드를 내 프로젝트 테스트 코드에 따라하면서 겪은 시행착오들이 있다. 하나는 컨트롤러 단위테스트를 진행하면서 데이터베이스? 정확히는 jpaAuditingHandler와의 의존성에서 시작했다. 떠오른 것은 각 객체의 생성, 삭제, 수정 시각을 저장하기 위해 @EnableJpaAuditing를 사용한다는 것이었다. 근데 이게 단위테스트에서 문제를 일으켰다.

Cannot resolve reference to bean 'jpaMappingContext' while setting constructor argument

검색 끝에, 이 또한 @MockBean으로 가짜 객체로 스프링 컨텍스트에 등록하여 해결할 수 있었다. 
@MockBean(JpaMetamodelMappingContext.class)

같은 맥락에서 MockMvc를 사용해서 모의 Http 요청을 보내고 해당 요청에 대한 응답을 검증하였다. 첫 실행에서 302코드를 받았다. 

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /api/quiz-content/review-content/123
    Session Attrs = {SPRING_SECURITY_SAVED_REQUEST=DefaultSavedRequest [http://localhost/api/quiz-content/review-content/123?continue]}

MockHttpServletResponse:
           Status = 302
             Body = 
    Forwarded URL = null
   Redirected URL = http://localhost/oauth2/authorization/kakao
   
여기서 해결 아이디어를 얻은 내용은 스프링 시큐리티, auth였다. 사실 어쩌면 예상을 조금은 했었다. 인증이 필요한 요청인데, 모의 요청에서 또한 필요하지 않았을까 생각했다. 그래서 테스트 코드에서 시큐리티에 대한 설정을 해제할 수 있는 방법에 대해서 검색하였고, @AutoConfigureMockMvc(addFilters = false) 어노테이션을 붙여서 해결하였다. 
