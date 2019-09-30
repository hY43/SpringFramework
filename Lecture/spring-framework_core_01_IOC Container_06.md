## 6강 Environment 1부. Profile

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. Environment
---
ApplicationContext는 단순히 Bean Factory의 기능만을 갖고 있는 것이 아니라 다양한 역할을 하는데,
그 중에서도 환경에 따라 다른 Bean을 주입하는 등의 역할에 개입하는 EnvironmentCapable Interface를 구현하여 사용하는 역할을 갖고 있다.

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
