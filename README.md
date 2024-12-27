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

