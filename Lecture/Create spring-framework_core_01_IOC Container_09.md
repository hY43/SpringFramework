## 9강 ResourceLoader

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. ResourceLoader
---
- ApplicationContext가 상속받고 있는 클래스로 자원(리소스)를 읽어오는 기능을 제공한다.

### 02. ResourceLoader 실습
- 사용하고자 하는 Resource를 생성한다.
    - Resource 폴더에 test.txt 파일을 아래와 같이 생성한다.
    ```
    hello spring
    ```
    - ApplicationRunner 구현
        - Files.readString Method는 자바 11부터 사용이 가능함.
        ```java
        @Component
        public class AppRunner implements ApplicationRunner {

            @Autowired
            ResourceLoader resourceLoader;

            @Override
            public void run(ApplicationArguments args) throws IOException {
                Resource resource = resourceLoader.getResource("classpath:test.txt");
                System.out.println(resource.exists());
                System.out.println(resource.getDescription());
                System.out.println(Files.readString(Path.of(resource.getURI())));
            }
        }
        ```
    - Main Class
    ```java
    @SpringBootApplication
    public class DemospringApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemospringApplication.class, args);
        }
    }
    ```
    - 실행하면 아래와 같은 결과를 출력한다.
    ```
    true
    class path resource [test.txt]
    hello spring
    ```
