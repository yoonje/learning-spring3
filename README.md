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
  - 라이프 사이클 인터페이스를 지원
    - `PostConstruct` 등등..
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

### 빈의 스코프

### Enviroment

### MessageSource

### ApplicationEventPublisher

### ResourceLoader

Spring Resource Validation
=======


Spring Data Binding
=======


Spring SpEL
=======

Spring AOP
=======

Spring Null Safety
=======

