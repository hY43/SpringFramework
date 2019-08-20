## 3강 Autowire Annotation

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```

### 01. Autowired Annotation
---
- 이 Annotation은 Bean으로 등록된 객체들에 대해서 의존성을 주입해주는 역할을 한다.
- 사용 가능한 위치는 아래와 같다.
    - 단 아래와 같은 방식을 사용하기 위해서는 BookRepository Class가 Bean으로 등록이 되어있어야 한다.
    ```java
    @Repository
    public class BookRepository {
    }
    ```
    - Constructor
    ```java
    @Service
    public class BookService {

        BookRepository bookRepository;

        @Autowired
        public BookService(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }
    ```
    - Setter
    ```java
    @Service
    public class BookService {
        BookRepository bookRepository;

        @Autowired
        public void setBookRepository(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }
    ```
    - Field
    ```java
    @Service
    public class BookService {
        @Autowired
        BookRepository bookRepository;
    }
    ```
    - 만약 BookRepository Class가 Bean으로 등록되어 있지 않다면(@Repository를 사용하지 않는다면), BookService 객체 생성시에 Autowired를 통해 BookRepository에 의존성을 주입하다가 에러가 발생한다.
    이를 해결하기 위해서는 BookRepository Class를 Bean으로 등록하거나 @Autowired에 아래와 같이 조건을 주어 에러를 방지할 수는 있다.
    ```java
    @Service
    public class BookService {
        BookRepository bookRepository;

        @Autowired(required = false)
        public BookService(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
    }
    ```

    ### 02. Autowired Annotation - 같은 타입의 Bean이 여러 개인 경우
    ---
    - 같은 타입의 Bean이 여러 개 있는 경우에는 실제 Bean을 주입할 때 어떤 Bean을 주입 받을지 판단하지 못해 에러를 발생 시킨다.
    아래 소스를 통해 확인해보자.
        - BookRepository Interface 와 상속 class 구현
        ```java
        // BookRepository.java
        public interface BookRepository {
        }

        // MyBookRepository.java
        @Repository
        public class MyBookRepository implements BookRepository {
        }

        // SangwonBookRepository.java        
        @Repository
        public class SangwonBookRepository implements BookRepository {
        }
        ```
        - 의존성을 주입 받을 BookService class 구현
        ```java
        // BookService.java
        @Service
        public class BookService {
            @Autowired
            BookRepository bookRepository;

            public void printBookRepository(){
                System.out.println(bookRepository.getClass());
            }
        }
        ```
        - Runner class 및 Main class 구현
        ```java
        // BookServiceRunner.java
        @Component
        public class BookServiceRunner implements ApplicationRunner {

            @Autowired
            BookService bookService;

            @Override
            public void run(ApplicationArguments args) throws Exception {
                bookService.printBookRepository();
            }
        }

        // Main class
        @SpringBootApplication
        public class DemospringApplication {

            public static void main(String[] args) {
                SpringApplication.run(DemospringApplication.class, args);
            }

        }
        ```
        - 위와 같이 클래스들을 작성한 후 실행하면, 아래와 같은 에러가 발생한다.        
        ```
        Field bookRepository in com.example.demospring.BookService required a single bean, but 2 were found:
        - myBookRepository: defined in file [C:\workspace\bookRent\demospring\target\classes\com\example\demospring\MyBookRepository.class]
        - sangwonBookRepository: defined in file [C:\workspace\bookRent\demospring\target\classes\com\example\demospring\SangwonBookRepository.class]

        Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
        ```
        - 위의 에러는 2개의 Bean이 있어 어떤 것을 사용할지 확정하라는 에러로, @Primary 혹은 @Qualifier 를 사용하여 해결할 수 있다는 설명이다.
    - @Primary 를 활용한 Bean 확정
        - Primary Annotation을 사용하여 복수의 Bean이 있을 때 어떤 Bean을 사용하여 의존성을 주입할지 결정할 수 있다.
        - Primary Annotation를 추가하여 구현
        ```java
        // MyBookRepository.java
        @Repository
        @Primary
        public class MyBookRepository implements BookRepository {
        }
        ```
        - 위와 같이 사용할 Bean에 Primary Annotation을 추가하고 다시 실행해보면 정상 동작하는 것을 확인할 수 있다.

    - 여러 개의 Bean을 모두 받아서 사용
        - 이 방법의 경우 List를 통해 모든 종류의 Bean을 받아서 사용하고자 하는 Bean을 사용하는 방식이다.
        ```java
        // BookService.java
        @Service
        public class BookService {
            @Autowired
            List<BookRepository> bookRepositories;

            public void printBookRepository(){
                bookRepositories.forEach(System.out::println);
            }
        }
        ```
        - 위와 같이 BookRepository를 List로 받아서 BookRepository 타입의 모든 Bean을 주입받는다.

### 03. Autowired 동작 원리
---
- 이전까지 Autowired Annotation에 대해 확인해보았는데, 이 Autowired가 어떤 방식으로 동작하는지 알아보자.
- BeanPostProcessor
    - Bean이 생성되고 난 후, Initialization Interface를 통해 초기화가 되는데 이 Interface 실행 전후로 동작하는 Life Cycle Interface이다.
    - Autowired Annotation을 사용하면 그 중에서도 **AutowiredAnnotationBeanPostProcessor** 에 의해서 Bean이 생성된 직후에 의존성을 주입해주게 된다.
- Bean Initialization
    - PostConstruct Annotation 을 사용한 초기화
    ```java
    // BookService.java
    @Service
    public class BookService {
        @Autowired
        List<BookRepository> bookRepositories;

        public void printBookRepository(){
            bookRepositories.forEach(System.out::println);
        }

        @PostConstruct
        public void setUp(){
            // BookService Bean이 생성된 직후에 진행할 Process 구현
        }
    }
     ```
     - InitializingBean 을 사용한 초기화
     ```java
    @Service
    public class BookService implements InitializingBean {
        @Autowired
        List<BookRepository> bookRepositories;

        public void printBookRepository(){
            bookRepositories.forEach(System.out::println);
        }

        @Override
        public void afterPropertiesSet() throws Exception {
            // BookService Bean이 생성된 직후에 진행할 Process 구현
        }
    }
     ```
        
