## 6강 Environment(Profile, Property)

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. Environment
---
- ApplicationContext는 단순히 Bean Factory의 기능만을 갖고 있는 것이 아니라 다양한 역할을 하는데,
그 중에서도 환경에 따라 다른 Bean을 주입하는 등의 역할에 개입하는 EnvironmentCapable Interface를 구현하여 사용하는 역할을 갖고 있다.
- EnvironmentCapable Interface에서 제공하는 기능에는 Profile과 Property가 있다.

### 02. Profile
---
- Bean들의 그룹으로 특정 Bean의 사용 시기를 설정을 미리 작성해 두고, 서비스 구동 시 불러다 사용할 수 있다.
- 이 Profile 기능은 환경에 따라 다른 Bean을 주입할 때도 사용할 수 있다.
- 이러한 기능을 소스를 통해 확인해보자.
    - 사용할 Bean에 대한 Interface 작성
    ```java
    // BookRepository.java
    public interface BookRepository {
    
    }
    ```
    - Interface 구현
        - 구현 시, Profile Annotation을 통해 어떤 Profile일때 사용할 Bean인지 명시한다.
        ```java
        // TestBookRepostory.java
        @Repository
        @Profile("test")
        public class TestBookRepository implements BookRepository{
        }
        ```
    - 실행을 위한 ApplicationRunner 구현
    ```java
    @Component
    public class AppRunner implements ApplicationRunner {

        @Autowired
        ApplicationContext ctx;

        @Autowired
        BookRepository bookRepository;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            Environment environment = ctx.getEnvironment();
            System.out.println(Arrays.toString(environment.getActiveProfiles()));
            System.out.println(Arrays.toString(environment.getDefaultProfiles()));
        }
    }
    ```
    - 어플리케이션 실행을 위한 Main Class 구현
    ```java
    @SpringBootApplication
    public class DemospringApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemospringApplication.class, args);
        }
    }
    ```
    - 위의 어플리케이션을 실행하면 아래와 같은 에러가 발생한다.
    ```
    Field bookRepository in com.example.demospring.AppRunner required a bean of type 'com.example.demospring.BookRepository' that could not be found.

    The injection point has the following annotations:
        - @org.springframework.beans.factory.annotation.Autowired(required=true)


    Action:

    Consider defining a bean of type 'com.example.demospring.BookRepository' in your configuration.
    ```
    - BookRepository Bean을 주입하려고 하였으나 주입해야하는 Bean이 없다는 에러로 TestBookRepository Bean은 "test" Profile 환경에서만 주입되도록 설정이 되어있다.
    - 이를 해결하기 위해 IDE에서 제공하는 Run/Debug Configuration의 VM Option에 아래와 같은 설정을 추가하자.
    ```
    -Dspring.profiles.active="test"
    ```
- 이러한 Profile 기능을 사용하면 테스트 환경에서는 A 라는 Bean을, 배포 환경에서는 B라는 Bean을 사용하는 등의 조작이 가능하다.

### 03. Property
---
- 다양한 방법으로 정의할 수 있는 설정 값을 말하며 계층형으로 정의할 수 있다.
- 각 Property에는 우선 순위가 있으며 아래와 같다.
    - ServletConfig 매개변수
    - ServletContext 매개변수
    - JNDI(java:comp/env/)
    - JVM 시스템 프로퍼티(-Dkey="value")
    - JVM 시스템 환경변수(운영체제 환경 변수)
- 실제로 Property를 등록하는 방법을 소스를 통해 확인해보자.
    - IDE의 Run/Debug Configuration 메뉴를 통한 등록
        - 사용하는 IDE의 Run/Debug Configuration의 VM 옵션에 아래와 같이 파라미터를 등록한다.
        ```
        -Dapp.name=spring5
        ```
        - ApplicationRunner 구현
        ```java
        // AppRunner.java
        @Component
        public class AppRunner implements ApplicationRunner {

            @Autowired
            ApplicationContext ctx;
            
            @Override
            public void run(ApplicationArguments args) throws Exception {
                Environment environment = ctx.getEnvironment();
                System.out.println("app.name : " + environment.getProperty("app.name"));
            }
        }
        ```
        - Main Class 구현
        ```java
        // DemospringApplication.java
        @SpringBootApplication
        public class DemospringApplication {

            public static void main(String[] args) {
                SpringApplication.run(DemospringApplication.class, args);
            }
        }
        ```
        - 위와 같이 구현한 후, 어플리케이션을 실행하면 아래와 같이 출력되는 것을 확인할 수 있다.
        ```
        app.name : spring5
        ```
    - Properties 파일을 통한 등록
        - resource 폴더에 app.properties을 생성한다.
        ```
        app.about=spring
        ```
        - ApplicationRunner 를 구현한다.
        ```java
        @Component
        public class AppRunner implements ApplicationRunner {

            @Autowired
            ApplicationContext ctx;

            @Value("${app.name}")
            String appName;

            @Override
            public void run(ApplicationArguments args) throws Exception {
                Environment environment = ctx.getEnvironment();
                System.out.println("app.name : " + environment.getProperty("app.name"));
                System.out.println("app.about : " + environment.getProperty("app.about"));
                //System.out.println("app name value : " + appName);
            }
        }
        ```
        - Main Class 구현
            - 이전과는 달리 PropertySource Annotation을 통해 설정 파일의 위치를 등록한다.
            ```java
            // DemospringApplication.java
            @SpringBootApplication
            @PropertySource("classpath:/app.properties")
            public class DemospringApplication {

                public static void main(String[] args) {
                    SpringApplication.run(DemospringApplication.class, args);
                }
            }
            ```
        - 위 코드를 실행하면 아래와 같은 결과를 확인할 수 있다.
        ```
        app.name : spring5
        app.about : spring
        ```
        - 위의 결과는 각각 app.name은 VM Option을, app.about은 app.properties를 통해 불러오는데 이 두개의 파라미터 중 어떤 것이 우선 순위가 높은지 확인해보도록 하자.
            - app.properties 수정
            ```
            app.about=spring
            app.name=spring
            ```
            - 어플리케이션을 재실행하면 아래와 같이 app.name이 VM Option에 설정된 값으로 출력되므로 VM Option의 우선 순위가 더 높음을 알 수 있다.
            ```
            app.name : spring5
            app.about : spring
            ```
    - 이전까지는 getProperty Method를 통해 Property의 값을 불러왔는데, 다른 방식으로 불러와보도록하자.
        - Value Annotation을 활용한 Property 값 호출
        ```java
        // AppRunner
        @Component
        public class AppRunner implements ApplicationRunner {

            @Autowired
            ApplicationContext ctx;

            @Value("${app.name}")
            String appName;

            @Override
            public void run(ApplicationArguments args) throws Exception {
                Environment environment = ctx.getEnvironment();
                System.out.println("app.name : " + environment.getProperty("app.name"));
                System.out.println("app.about : " + environment.getProperty("app.about"));
                System.out.println("app name value : " + appName);
            }
        }
        ```
        - 위와 같이 구현 후, 실행해보면 아래와 같이 appName 변수에서 불러온 값을 출력하는 것을 확인할 수 있다.
        ```
        app.name : spring5
        app.about : spring
        app name value : spring5

        ```
