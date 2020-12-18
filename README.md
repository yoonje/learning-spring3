# Spring Framework 정리 자료
스프링 프레임워크 핵심 기술 정리 문서

Table of contents
=================
<!--ts-->
   * [Spring IoC Container](#Spring-IoC-Container)
   * [Spring Resource Validation](#Spring-Resource-Validation)
   * [Spring Data Binding](#Spring-Data-Binding)
   * [Spring SpEL](#Spring-SpEL)
   * [Spring AOP](#Spring-AOP)
   * [Spring Null Safety](#Spring-Null-Safety)
<!--te-->

Spring IoC Container
=======
### 스프링 IoC 컨테이너와 빈
- `Inversion of Control`: 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 ​의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법​​
- 의존성: 특정 객체에서 A라는 객체에서 B라는 객체의 기능을 사용할 때 의존성이 생기는데, 스프링을 활용하지 않으면 매우 의존성 관리가 어려움 
- 생성자 의존성 주입을 하는 경우
  ```java
  // BookService를 활용하려면 BookRepository가 반드시 존재해야하는 의존성이 있고 이를 생성자에서 할당 받을 수 있게 함
  @Service
  public class BookService {
      private BookRepository bookRepository;

      @Autowired
      public BookService(BookRepository bookRepository){
          this.bookRepository = bookRepository;
      }
  }
  ``` 
- 생성자 의존성 주입 사용하지 않는 경우
  ```java
  // BookService를 활용하려면 BookRepository가 반드시 존재해야하는 의존성이 있지만 의존성을 바꿀 수 없게 고정되어 있고 계속 생성됨 
  @Service
  public class BookService {
      private BookRepository bookRepository = new BookRepository();
  }
  ```
- `IoC Cotainer`: 의존 관계 주입을 하는 객체들을 담고 있는 컨테이너
- `Bean`: 스프링 IoC 컨테이너가 관리하는 객체
- IoC Container를 사용하는 이유
  - 객체의 관리를 컨테이너를 통해서 프레임워크의 강력한 기능을 지원
  - 애플리이션 전반에서 하나만 만들어서 사용하는 `싱글톤 스코프로 제공`
  - `라이프 사이클` 인터페이스를 지원
    - @PostConstruct 등등..
- BeanFactory: 스프링 IoC 컨테이너의 최상위 인터페이스
- Application Context: BeanFactory 상속 받은 인터페이스로 BeanFactory 보다 더 다양한 기능 제공

### ApplicationContext와 다양한 빈 설정 방법
1. applicationContext.xml 파일에 등록하는 방법(스프링)
  - resource 디렉토리 밑에 설정 파일에 `<bean></bean>`을 통해서 만듦
    - `bean`의 id 속성은 객체 이름, class는 객체의 패키지 경로
    - `bean`의 하위에 `<property></property>`로 속성을 넣을 수 있는데 property의 name은 setter에서 가져온 정보, ref는 참조할 다른 빈을 의미 
  - ApplicationContext 객체를 통해서 빈을 가져와서 활용이 가능
2. component-scan을 활용하는 방법(스프링)
  - applicationContext.xml 안에 패키지 경로를 명시해서 빈으로 등록 가능
  - `@Component` 어노테이션을 기본으로 빈으로 등록 가능
  - `@Service`, `@Repository` 등등의 방법이 존재
3. 자바 파일을 활용하는 방법(스프링)
  - Java 파일에 `@Configuration` 어노테이션을 먹인 설정 파일에서 빈으로 등록 가능
  - `@Bean` 어노테이션을 생성자에 함께 사용하여 등록
  - AnnotationConfigApplicationContext 자바 설정 파일을 넣고 ApplicationContext를 만들어야함
4. 2번과 3번을 혼용한 방법
  - 자바 파일에서 @ComponentScan() 어노테이션으로 패키지나 클래스를 지정하여 빈 등록 어노테이션들을 스캐닝할 수 있음
5. 최신 방법(스프링 부트)
  - `@SpringBootApplication`이 붙어 있으면 그 안에 `@ComponentScan`이 있어서 기본 프로젝트는 그 안에 어노테이션이 들어간 빈 등록이 적용이 되어 있고 ApplicationContext까지 만들어 줌
  
### @Autowire
- @Autowire를 사용하면 벌어지는 일
  - 필요한 의존 객체의 `타입에 해당하는 빈을 찾아 할당(주입) 해줌`
- @Autowire를 사용하는 위치
  - 생성자 (스프링 4.3 부터는 생략 가능)
  - 세터 (별로 사용하지 않음)
  - 필드
- 다형성을 포함한 같은 타입의 빈이 여러개 일 때 @Autowire에서 구별하는 법
  - @Primary
  - 해당 타입 의빈 모두 주입 받기
  - `@Qualifier (빈 이름으로 주입)`
- 동작 원리
  - BeanPostProcessor
    - 새로 만든 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
  - AutowiredAnnotationBeanPostProcessor​ extends BeanPostProcessor
    - 스프링이 제공하는 @Autowired와 @Value 애노테이션 @Inject 애노테이션을 지원하는 애노테이션 처리기

### @Component와 컴포넌트 스캔
- `@ComponentScan` 주요 기능
  - 스캔 시작 위치 설정
  - 어떤 애노테이션을 스캔할지 또는 하지 않을지 필터 설정이 가능
  - @SpringBootApplication 안에 있음
- 대표적인 `@Component`의 종류
  - @Repository
  - @Service
  - @Controller
  - @Configuration
  - @Bean

### 빈의 스코프
- 스코프(@Scope)
  - 싱글톤(기본 값): 해당 빈의 인스턴스가 애플리케이션에 단 하나
  - 프로토타입(@Scope("prototype")): 해당 빈의 인스턴스가 애플리케이션에 여러 개
    - Request
    - Session
    - WebSocket
- 주의 사항
  - 프로토타입 빈이 싱글톤 빈을 참조하면 문제가 없음
  - 싱글톤 빈이 프로토타입 빈을 참조하면 프로토타입 빈의 업데이트에서 문제가 생길 수 있음 -> Provider / scoped-proxy / Object-Provider 등으로 보완햐줘야함
  - 프로퍼티를 공유
  - `ApplicationContext 초기 구동시 인스턴스 생성`

### Enviroment
- 프로파일
  - `빈들의 그룹`
  - ApplicationContext의 Environment​의 역할은 활성화할 프로파일 확인 및 설정
- 프로파일을 사용하는 경우 
  - 테스트 환경에서는 A라는 빈을 사용하고, 배포 환경에서는 B라는 빈을 쓰고 싶을 때처럼 환경에 따라 사용할 빈들이 달라질 때 사용
- 프로파일 정의하기
  - 클래스에 정의
    - @Configuration @Profile(“test”)
    - @Component @Profile(“test”)
  - 메소드에 정의
    - @Bean @Profile(“test”)
- 프로파일 설정하기
  - 인텔레리제이 vm option에서 `-Dspring.profiles.avtive=”A,B,...”`
  - 인텔레리제이 Active Profiles에서 `A`
- 프로파일 표현식
  - ! (not)
  - & (and)
  - |(or)
- 프로퍼티
  - `다양한 방법으로 정의할 수 있는 설정 값`
  - ApplicationContext의 Environment​의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기
- 프로퍼티에서 값을 주는 방법
  - vm 옵션으로 주기
  - @PropertySource에서 파일을 읽고 프로퍼티를 추가하기
  - 스프링 부트에서는 application.properties으로 기본 프로퍼티 소스 지원 

### MessageSource
- MessageSource
  - `ApplicationContext의 MessageSource는 국제화 기능을 제공하는 인터페이스`
  - getMessage(String code, Object[] args, String, default, Locale, loc)을 통해서 메세지를 얻을 수 있음
- 스프링 부트를 사용한다면 별다른 설정 필요없이 messages.properties를 이용해서 사용
  - messages.properties
  - messages_ko_kr.properties

### ApplicationEventPublisher
- ApplicationEventPublisher
  - `ApplicationContext의 ​ApplicationEventPublisher는 이벤트 프로그래밍을 위한 기능을 제공하는 인터페이스`
- 이벤트 만들기
  - 스프링 4.2부터는 POJO로 이벤트 클래스를 만들어 됨
- 이벤트 발생시키기
  - 이벤트를 발생시키는 빈에서 `ApplicationRunner`를 implsments하고 run 메소드를 오버라이딩할 때 거기서 ApplicationEventPublisher.publishEvent();를 활용
- 이벤트 처리하기
  - 스프링 4.2부터는 ​빈으로 등록한 핸들러 클래스의 메소드에서 @EventListener​를 빈의 메소드로 사용
  - 순서를 정하고 싶다면 `@Order`와 함께 사용
  - 기본적으로는 synchronized 비동기적으로 실행하고 싶다면 `@Async`와 함께 사용
- 스프링이 제공하는 기본 이벤트
  - ContextRefreshedEvent: ApplicationContext를 초기화 했더나 리프래시 했을 때 발생
  - ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작
신호를 받은 시점에 발생
  - ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지
신호를 받은 시점에 발생
  - ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에
발생
  - RequestHandledEvent: HTTP 요청을 처리했을 때 발생

### ResourceLoader
- ResourceLoader
  - `ApplicationContext의 ResourceLoader는 리소스를 읽어오는 기능을 제공하는 인터페이스`
  - Resource getResource(java.lang.String location)를 통해서 리소르를 일거올 수 있음
- 리소스 읽어오기
  - 파일 시스템에서 읽어오기
  - 클래스패스에서 읽어오기
  - URL로 읽어오기
  - 상대/절대 경로로 읽어오기


Spring Resource Validation
=======
- Resource
  - java.net.URL을 추상화하여 다양한 기능을 제공하는 인터페이스
  - 클래 스패스 기준으로 리소스 읽어오는 기능과 ServletContext를 기준으로 상대 경로로 읽어오는 기능 추가
  - 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용 
- Resource의 주요 메소드
  - getInputStream()
  - exitst()
  - isOpen()
  - getDescription(): 전체 경로 포함한 파일 이름 또는 실제 URL
- `Resource의 주요 구현체`
  - UrlResource: ​java.net.URL​ 참고, 기본으로 지원하는 프로토콜 http, https, ftp, file, jar. 
  - ClassPathResource: 지원하는 접두어 classpath:
  - FileSystemResource
  - ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스 찾음
- 리소스 읽어오기
  - Resource의 타입은 `locaion 문자`열과 `​ApplicationContext의 타입`에​​ 따라 결정
  - ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+ classpath:)중 하나를 사용
    - classpath:​​me/whiteship/config.xml -> ClassPathResource
    - file://​​/some/resource/path/config.xml -> FileSystemResource
  - 접두어가 없으면 ServletContextResource이 기본 리소스 타입
- Validator
  - 애플리케이션에서 사용하는 객체 검증용 인터페이스
  - 모든 계층(웹, 서비스, 데이터)에서 사용
  - DataBinder에 들어가 바인딩 할 때 같이 사용
  - Bean Validation을 위해 구현체로 hibernate-validator 사용
- Validator를 활용하기 위해 오버라이딩해야하는 메소드
  - `boolean supports(Class clazz)`: 어떤 타입의 객체를 검증할 때 사용할 것인지 결정
  - `void validate(Object obj, Errors e)`: 실제 검증 로직을 이 안에서 구현
    - 구현할 때 ValidationUtils 사용하며 편리
- 스프링 부트 2.0.5 이상 버전을 사용할 때
  - LocalValidatorFactoryBean ​​빈으로 자동 등록으로 등록되므로 Validator를 의존 주입 받아와서 사용 가능
  - 간단한 검증 로직에서는 Validator를 오버라이딩 하지 않고, 애노테이션만으로 검증이 가능

Spring Data Binding
=======
- 데이터 바인딩
  - 입력값은 대부분 문자열인데, 그 값을 객체가 가지고 있는 int, long, Boolean, Date 또는 도메인 타입으로도 변환해서 넣어주는 기능
- `PropertyEditor`
  - 고전적인 데이터 바인딩의 추상화 인터페이스
  - 쓰레드-세이프 하지 않아서 빈으로 등록하지 않아야함
  - Object와 String 간의 변환만 할 수 있어, 사용 범위가 제한적임
- Converter와 Formattter
  - `Converter`
    - ConversionService의 일종으로 S타입을 T타입으로 변환할 수 있는 현대 데이터 바인딩의 일반적인 변환기
    - ConverterRegistry​에 등록해서 사용
    - 기본적인 자료형에 대해서는 이미 등록이 되어 있음
    - 스프링 부트에서는 빈이으로 설정되어 있으면 자동 등록이 되어 따로 Web Mvc 설정에서 registry에 컨버터를 등록할 이유가 없음
    - 쓰레드-세이프하여 빈으로 등록 가능
    ```java
    public class StringToEventConverter implements Converter<String, Event> {
      @Override
      public Event convert(String source) {
         Event event = new Event(); 
         event.setId(Integer.parseInt(source)); 
         return event;
         } 
    }
    ```
  - `Formatter`
    - ConversionService의 일종으로 Converter보다 더 웹 쪽에 특화된 데이터 바인딩의 추상화 인터페이스
    - Object와 String 간의 변환을 담당
    - 문자열을 Locale에 따라 다국화하는 기능도 제공
    - FormatterRegistry​에 등록해서 사용
    - 스프링 부트에서는 빈이으로 설정되어 있으면 자동 등록이 되어 따로 Web Mvc 설정에서 registry에 포맷터를 등록할 이유가 없음
    - 쓰레드-세이프하여 빈으로 등록 가능
    ```java
    public class EventFormatter implements Formatter<Event> {
      @Override
      public Event parse(String text, Locale locale) throws ParseException {
        Event event = new Event();
        int id = Integer.parseInt(text); event.setId(id);
        return event;
      }
    
      @Override
      public String print(Event object, Locale locale) {
        return object.getId().toString(); 
      }
    }
    ```

Spring SpEL
=======
- SpEL
  - 스프링 프로젝트 전반에 걸쳐 사용
  - 연산자와 메소드 호출을 지원하며, 문자열 템플릿 기능도 제공
- 문법
  - #{“표현식"}
  - #{${my.data} + 1}처럼 표현식은 프로퍼티를 가지게 하여 혼용이 가능(반대는 안됨)
- 주로 사용되는 곳
  - 스프링
    - @Value 애노테이션
    - @ConditionalOnExpression 애노테이션
  - 스프링 시큐리티
    - 메소드 시큐리티, @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
    - XML 인터셉터 URL 설정
  - 스프링 데이터
    - @Query 애노테이션
  - Thymeleaf



Spring AOP
=======
- `Aspect-oriendted Programming(AOP)`은 OOP를 보완하는 수단으로, 흩어진 Aspect를 모아서 `모듈화` 할 수 있는 프로그래밍 기법
- 스프링 AOP
  - `스프링 AOP`는 AOP의 기본적인 구현체를 제공하며 다른 구현체인 `AspecJ`와 연동할 수 있는 기능 제공
  - 캐시와 트랜잭션에서 매우 많이 쓰임
- AOP 주요 개념
  - `Aspect`: 각각의 모듈로 `Advice와 Pointcut을 합친 것`
  - `Target`: Aspect를 적용할 대상
  - `Advice`: 조인 포인트에 삽입되어져 `적용할 기능이 있는 코드`
  - `Join point`: 합류점으로 `Advice가 낄어들 수 있는 지점`, 메소드 실행 시점이나 생성자 호출 시점, 필드에서 값을 가져갈 때 등등이 주요 Joint point
  - `Pointcut`: Joint point의 subset으로 `실제로 Advice가 적용되는 Jointpoint`
- AOP 적용 방법
  - 컴파일 시점: 자바 파일을 클래스 파일로 만들 때 바이트 코드 자체에 적용하는 방법(컴파일을 한번 더 해야함)
  - 로드 타임 시점: AspecJ가 사용하는 방법으로, 클래스 파일을 로딩하는 시점에 바이트 코드에 위빙하는 방법(클래스 로딩 시점의 부하가 생기고 로드 타임 위버 설정을 해야함)
  - 런타임 시점: 스프링 AOP가 사용하는 방법으로, 애플리케이션 안에서 A 객체를 만들 때 A 객체를 감싼 `프록시 객체`를 만들어서 사용하는 방법(객체를 생성할 때 비용을 더 들지만 쉽고 별다른 설정이 필요하지 않음)
- 스프링 AOP 특징
  - 프록시 기반의 AOP 구현체
  - 스프링 빈에만 AOP를 적용할 수 있음
  - 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 `흔한 문제`에 대한 해결책을 제공
  - 스프링 IoC 컨테이너가 제공하는 라이프 사이클 같은 기능과 `Dynamic 프록시`를 사용하여 여러 복잡한 문제 해결
  - 스프링 IoC 컨테이너는 동적 프록시 빈으로 등록
  - cf) 동적 프록시: 동적으로 프록시 객체 생성하는 방법으로 자바는 `인터페이스 기반 프록시 생성`
- 스프링 AOP 활용하기 
  - 애스팩트 정의
    - @Aspect를 활용하여 정의하고 @Component로 빈으로 등록
  - 포인트컷 정의
    -  포인트컷 정의의 주요 표현식으로는 execution, @annotation, bean이 있고 &&, ||, !를 통해서 포인트 컷 조합을 활용할 수 있음
  - 어드바이스 정의
    - 적용할 기능이 있는 코드 작성
    - @Around를 활용하고 그 안에 표현식으로 실행할 곳인 `Pointcut을 지정할 수 있음`
    - 간단하게 실행 시점에 따른 어드바이스를 정의하려면 @Before, @AfterReturning, @AfterThrowing를 활용할 수 있음

Spring Null Safety
=======
- Null 관련 어노테이션: 스프링 프레임워크 5에 추가된 Null 관련 애노테이션
  - @NonNull
  - @Nullable
  - @NonNullApi (패키지 레벨 설정)
  - @NonNullFields (패키지 레벨 설정)
- Null 관련 어노테이션의 목적
  - (툴의 지원을 받아) 컴파일 시점에 최대한 NullPointerException을 방지하는 것
- 사용 하는 방식
  - 인텔리제이에서 Nullable/NonNull 설정에서 스프링 Null을 추가해줘야함
