## 10강 Resource 추상화

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. Resource 추상화 대상
---
- java.net.URL을 org.springframework.core.io.Resource 로 감싸서 추상화 함.
- 추상화한 이유
    - java.net.URL이 classpath 기준으로 Resource를 불러오는 기능이 없음.
        - 즉 이전 강의에서 XML 파일을 통해 Resource를 가져오는 ClassPathXmlApplicationContext 등의 객체는 내부적으로 해당 인터페이스를 구현해서 동작한다.
    

### 02. Resource의 구현체
---
- URIResource : http, https, ftp등의 프로토콜을 지원하는 구현체
- ClassPathResource : classpath:라는 접두어를 지원하는 구현체
- ServletContextResource : 웹 어플리케이션 루트에서 상대 경로로 리소스를 찾아주는 구현체
- FileSystemResource 

### 03. Resource 읽어오기
---
- 읽을 Resource는 ApplicationContext의 타입에 따라 결정이 된다.
    - 예를 들어 ClassPathXmlApplicationContext를 사용하면 ClassPathResource를 사용하게 되고, FileSystemXmlApplicationContext를 사용하면 FileSystemResource를 사용하게 된다.
    - 추가로 WebApplicationContext Interface를 사용하게 되면, ServletContextResource를 사용한다.
    - 위의 사항에 대해 소스를 통해 사용 방식을 확인해보자.
    ```
    ApplicationContext ctx = new ClassPathXmlApplicationContext("test.xml");
    ```
    - 위와 같이 코드를 작성하게 되면 Classpath 방식으로 해당 Resource를 가져오게 된다.
- 단 ApplicationContext의 타입과 무관하게 Resource 타입을 강제하려면 java.net.URL 접두어 중 하나를 사용하면 강제할 수 있다.
    - 소스를 통해 확인해보자.
    
