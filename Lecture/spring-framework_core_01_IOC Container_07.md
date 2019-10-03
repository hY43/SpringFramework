## 7강 MessageSource

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. MessageSource
---
- ApplicationContext가 구현하고 있는 Interface 중 하나로 국제화(i18n), 즉 다국어 기능을 제공한다.

### 02. MessageSource 실습
---
- 소스를 통해 다국어 기능을 작성해보자.
- Message에 대한 설정 파일 작성(resource 폴더 밑에 생성한다.)
    - messages.properties : 기본 파일
    ```
    greeting=Hello {0}
    ```
    - messages_ko_KR.properties : 한국어에 대한 다국어 파일
    ```
    greeting=안녕 {0}
    ```
    - 위의 설정 파일들에서 {0}는 해당 메시지를 호출할 때 같이 보낸 파라미터를 말하며 괄호 안의 숫자는 몇 번째 파라미터인지를 의미한다.
- ApplicationRunner 구현
```java
// AppRunner.java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception{
        System.out.println("ko.kr : " + messageSource.getMessage("greeting", new String[]{"sangwon"}, Locale.KOREA));
        System.out.println("default : " + messageSource.getMessage("greeting", new String[]{"sangwon"}, Locale.getDefault()));
    }
}
```
- Main Class 구현
```java
@SpringBootApplication
public class DemospringApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemospringApplication.class, args);
    }
}
```
- 실행 결과
    - 위의 코드를 실행하면 아래와 같이 출력되는 것을 확인할 수 있다.
    ```
    안녕 sangwon
    Hello sangwon
    ```
    - 단 Locale.getDefault()는 사용자 별로 사용하는 운영체제의 언어 셋에 따라 Hello가 출력될 수도 안녕이 출력될 수도 있다.
### 03. Reload되는 MessageSource
---
- 메시지가 바뀔때마다 서버를 재기동하는 점은 부담이 되기 때문에 제공하는 ReloadableResourceBundleMessageSource Class가 있는데 이를 확인해보자.
- Main Class 구현
```java
@SpringBootApplication
public class DemospringApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemospringApplication.class, args);
    }

    @Bean
    public MessageSource messageSource(){
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(3);
        return messageSource;
    }
}
```
- ApplicationRunner 구현
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception{
        while(true){
            System.out.println("ko.kr : " + messageSource.getMessage("greeting", new String[]{"sangwon"}, Locale.KOREA));
            System.out.println("default : " + messageSource.getMessage("greeting", new String[]{"sangwon"}, Locale.getDefault()));
            Thread.sleep(1000l);
        }
    }
}
```
- 위의 소스에 대한 테스트 방법은 아래와 같다.
    - 서버 기동
    - messages.properties 혹은 messages_ko_KR.properties 파일 내용 수정
    - Build
    - 결과 확인

