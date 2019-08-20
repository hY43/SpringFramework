## 2강 Spring IOC Container and Bean

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```

### 01. XML 파일을 활용한 Bean 관리 - Bean 수기 작성
---

- XML 파일을 이용하여 Bean 객체에 의존성을 주입하는 방식으로 소스를 통해 확인해보자.
    - BookRepository Class 생성
    - BookService Class 생성
        - BookRepository를 멤버 변수로 갖도록 생성
    ```java
    // BookRepository.java
    public class BookRepository {
    }

    // BookService.java
    public class BookService {
        BookRepository bookRepository;

        public void setBookRepository(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }
    ```
    - application.xml 생성
        - Bean 의존성 주입용 설정 파일이 존재하지 않는다면, BookService 객체의 bookRepository 변수는 별도로 Setter를 호출하지 않는한 선언만 되어있을 뿐, 실제 객체는 생성되지 않는다.
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="bookService"
                class="com.example.applicationcontext.demo.BookService">
                <property name="bookRepository" ref="bookRepository"/>
            </bean>

            <bean id="bookRepository"
                class="com.example.applicationcontext.demo.BookRepository"/>
        </beans>
        ```
        - 위와 XML과 같이 설정파일을 생성하게 되는데, bean id가 bookService로 되어 있는 태그를 확인해보자.
        
        - 객체를 주입하기 위해 property를 주어 name에는 setter가 존재하는 멤버 변수명이 들어가고 ref에는 설정파일에 존재하는 bean의 id 값이 들어가게 된다.

    - Main Class를 통한 테스트
    ```java
    public class DemoApplication {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
            String[] beanDefinitionNames = context.getBeanDefinitionNames();
            System.out.println(Arrays.toString(beanDefinitionNames));;
            BookService bookService = (BookService) context.getBean("bookService");
            System.out.println(bookService.bookRepository != null);
        }
    }
    ```
    - 실행 결과 확인
        - 실행의 결과는 아래와 같이 생성된 객체의 이름과 정상적으로 의존성이 주입되었다는 의미로 true가 출력된다.
        ```
        [bookService, bookRepository]
        true

        ```
    - XML에 수기로 bean을 기입하는 방식의 단점
        - 매번 의존성을 주입해야 하는 Class를 bean 태그를 사용하여 매번 추가해주어야 한다.   

### 02. XML 파일을 활용한 Bean 관리 - Component Scan
---
- 위의 XML 설정 파일에 수기로 bean을 등록하는 방식은 새로 Class가 추가 될때마다 등록해줘야하는 단점이 있으며, 이를 보완하기 위한 방법으로 XML 설정 파일에 Component-scan을 사용하여 특정 package 이하의 Bean을 자동으로 등록하고 Autowired Annotation을 사용하여 의존성을 주입해주는 방법이 있다.

- 소스를 통해 방법을 확인해보자.
    - application.xml 파일 생성
        - base-package를 두어 해당 package 이하의 Component Annotation을 사용하거나 Component Annotation을 확장한 Annotation을 사용하는 Class를 Bean으로 등록해준다.
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

            <context:component-scan base-package="com.example.applicationcontext.demo"/>
        </beans>
        ```
    - BookRepository.java 생성
        - Component Annotation을 확장한 Repository Annotation을 사용하여 파일을 생성해준다.
    - BookService.java 생성
        - Component Annotation을 확장한 Service Annotation을 사용하여 파일을 생성해준다.
        - 단 BookSerice.java에서는 BookRepository 객체에 의존성을 주입받아야 하므로, @Autowired 를 사용하여 의존석을 주입받는다.        
        ```java
        // BookRepository.java
        @Repository
        public class BookRepository {
        }

        // BookSerice.java
        @Service
        public class BookService {

            @Autowired
            BookRepository bookRepository;

            public void setBookRepository(BookRepository bookRepository) {
                this.bookRepository = bookRepository;
            }
        }
        ```
    - 실행 결과 확인
        - 이전의 Main Class를 사용하여 실행해보면 동일한 결과가 나오게 된다.

### 03. java 설정 파일을 활용한 Bean 관리 - Bean 수기 작성
---        
- 이전의 XML 설정 파일을 이용하는 방식 말고도 java 파일로 설정 파일을 만들어 Bean을 관리할 수 있다.
- 소스를 통해 방법을 확인해보자.
    - ApplicationConfig.java 생성
        - Configuration Annotation을 사용하여 해당 파일이 설정 파일임을 명시한다.
        - 코드 내부에 Bean Annotation을 사용하여 Bean을 등록한다.
        ```java
        @Configuration
        public class ApplicationConfig {
            @Bean
            public BookRepository bookRepository(){
                return new BookRepository();
            }

            @Bean
            public BookService bookService(){
                BookService bookService = new BookService();
                bookService.setBookRepository(bookRepository());
                return bookService;
            }
        }
        ```
    - BookService.java 생성
    - BookRepository.java 생성
    ```java
    // BookService.java
    public class BookService {
        BookRepository bookRepository;

        public void setBookRepository(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }

    // BookRepository.java
    public class BookRepository {
    }
    ```
    - Main Class를 통핸 테스트
        - 실행 내용은 동일하나 사용하는 ApplicationContext를 java 파일을 읽는 방식으로 변경한다.
        ```java
        public class DemoApplication {
            public static void main(String[] args) {
                ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
                String[] beanDefinitionNames = context.getBeanDefinitionNames();
                System.out.println(Arrays.toString(beanDefinitionNames));;
                BookService bookService = (BookService) context.getBean("bookService");
                System.out.println(bookService.bookRepository != null);
            }
        }
        ```
    - 이 방식의 단점은 기존 XML에 Bean을 수기로 등록하는 것과 마찬가지로 사용하고자 하는 Bean이 늘어나면 추가로 Bean Annotation을 사용하여 등록해줘야 한다는 점이다.

### 04. java 설정 파일을 활용한 Bean 관리 - Component Scan
---
- java 설정 파일 사용시에도 Component Scan을 활용하여 Bean을 생성할 수 있다.
- 소스 코드를 통해 확인해보자.
    - ApplicationConfig.java 생성
        - ComponentScan Annotation을 사용하여 어떤 Package 내에서 Component Scan을 진행할지 명시해준다.
        ```java
        @Configuration
        @ComponentScan(basePackageClasses = DemoApplication.class)
        public class ApplicationConfig {

        }
        ```
        - BookRepository.java 생성
        - Component Annotation을 확장한 Repository Annotation을 사용하여 파일을 생성해준다.
    - BookService.java 생성
        - Component Annotation을 확장한 Service Annotation을 사용하여 파일을 생성해준다.
        - 단 BookSerice.java에서는 BookRepository 객체에 의존성을 주입받아야 하므로, @Autowired 를 사용하여 의존석을 주입받는다.        
        ```java
        // BookRepository.java
        @Repository
        public class BookRepository {
        }

        // BookSerice.java
        @Service
        public class BookService {

            @Autowired
            BookRepository bookRepository;

            public void setBookRepository(BookRepository bookRepository) {
                this.bookRepository = bookRepository;
            }
        }
        ```
    - 이전에 사용했던 Main Class를 사용하여 테스트를 진행한다.
### 05. SpringBoot의 Bean 관리
- 이전 까지 진행된 방식은 Spring에서 제공하는 Bean 설정에 대한 방법이었다. 하지만 SpringBoot에서는 사용자가 별도의 설정 파일을 생성하지 않고 SpringBoot가 직접 파일을 만들어 관리해준다.
- 소스를 통해 확인해보자.
    - Main Class 생성
        - SpringBootApplication Annotation을 사용하여 클래스를 생성한다.    
        ```java
        @SpringBootApplication
        public class DemoApplication {
            public static void main(String[] args) {

            }
        }
        ```
    - SpringBootApplication Annotation의 내부를 한번 확인해보자.
        - 아래와 같이 SpringBootApplicatoin Annotation은 SpringBootConfiguration Annotation과 ComponentScan Annotation 을 확장하여 만들어진 Annotation으로 해당 Annotation을 사용하면 Main Class 자체를 Bean 설정 파일로 등록해주는 효과를 볼 수 있다.
        ```java
        @Target({ElementType.TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @SpringBootConfiguration
        @EnableAutoConfiguration
        @ComponentScan(
            excludeFilters = {@Filter(
            type = FilterType.CUSTOM,
            classes = {TypeExcludeFilter.class}
        ), @Filter(
            type = FilterType.CUSTOM,
            classes = {AutoConfigurationExcludeFilter.class}
        )}
        )
        public @interface SpringBootApplication {
            @AliasFor(
                annotation = EnableAutoConfiguration.class
            )
            Class<?>[] exclude() default {};

            @AliasFor(
                annotation = EnableAutoConfiguration.class
            )
            String[] excludeName() default {};

            @AliasFor(
                annotation = ComponentScan.class,
                attribute = "basePackages"
            )
            String[] scanBasePackages() default {};

            @AliasFor(
                annotation = ComponentScan.class,
                attribute = "basePackageClasses"
            )
            Class<?>[] scanBasePackageClasses() default {};
        }
        ```        
