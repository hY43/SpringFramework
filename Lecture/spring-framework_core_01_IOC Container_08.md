## 8강 ApplicationEventPublisher

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```
### 01. ApplicationEventPublisher
---
- ApplicationContext가 상속받고 있는 클래스로 옵저버 패턴의 구현체로 이벤트 기반의 프로그래밍에 활용되는 구현체이다.

### 02. Spring 4.2 이전의 이벤트 프로그래밍
---
- Spring 4.2 이전에는 이벤트 기반의 프로그래밍을 하기 위해서는 Event Class는 ApplicationEvent를 상속, 이 이벤트를 처리할 Handler Class는 ApplicationListener<> 를 구현하여 사용하도록 되어있었다.
- Event Class 구현
```java
public class MyEvent extends ApplicationEvent {

    private int data;

    public MyEvent(Object source) {
        super(source);
    }

    public MyEvent(Object source, int data) {
        super(source);
        this.data = data;
    }

    public int getData(){
        return this.data;
    }
}
```
- Event를 실행할 ApplicationRunner 구현
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationEventPublisher publishEvent;

    @Override
    public void run(ApplicationArguments args) {
        publishEvent.publishEvent(new MyEvent(this, 100));
    }
}
```
- 실제로 Event를 처리할 Handler 구현
```java
@Component
public class MyEventHandler implements ApplicationListener<MyEvent> {

    @Override
    public void onApplicationEvent(MyEvent myEvent) {
        System.out.println("이벤트 받았다. 데이터는 " + myEvent.getData());
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
- 위 코드를 실행하면 아래와 같이 실행된다.
```
이벤트 받았다. 데이터는100
```
- 하지만 위와 같이 개발하게 되면 Event와 EventHandler Class들은 내부에 Spring의 소스가 녹아져있기 때문에 독립적이지 못하고, 유지보수 및 테스트에 용이하지 않다.

### 03. Spring 4.2 이후의 이벤트 프로그래밍
---
- Spring 4.2 이후에는 Event와 관련된 Class를 상속받지 않아도 이벤트 기반의 프로그래밍을 할 수 있도록 되었다.
    - Event Class 구현
        - 아래의 Event Class의 구조가 바로 Spring이 지향하는 POJO를 의미한다.
            - 개발자의 코드에 Framework의 코드가 노출되지 않는 것.
            - 테스트와 유지보수에 용이하다.
    ```java
    public class MyEvent {

        private Object source;
        private int data;

        public MyEvent(Object source, int data) {
            this.source = source;
            this.data = data;
        }

        public int getData(){
            return this.data;
        }

        public Object getSource(){
            return this.source;
        }
    }
    ```
    - Handler Class 구현
        - 실제로 이벤트를 처리할 Method에 EventListener Annotation을 통해 명시해준다.
    ```java
    @Component
    public class MyEventHandler{

        @EventListener
        public void handle(MyEvent event){
            System.out.println("이벤트를 받았다. 데이터는 " + event.getData());
        }
    }
    ```
    - 기존에 작성된 Main Class를 실행해보면 같은 결과를 보여준다.
### 04. 다양한 방식의 이벤트 기반의 프로그래밍
---    
- 한개의 Event에 대한 Handler가 여러개인 경우.
    - 새로운 MyEvent를 처리할 Handler 구현
    ```java
    @Component
    public class AnotherHandler {
        
        @EventListener
        public void handle(MyEvent event){
            System.out.println("또다른 이벤트 핸들러입니다. 값은 " + event.getData());
        }
    }
    ```
    - 이를 실행하면 아래와 같은 결과가 나오는데, 각각의 Handler는 멀티 스레드 방식이 아니라 순차적으로 호출된다.
    ```
    또다른 이벤트 핸들러입니다. 값은 100
    이벤트를 받았다. 데이터는 100    
    ```
    - 각각의 handler Method에 아래와 같은 코드를 추가해보자.
    ```java
    // MyEventHandler.java
    @Component
    public class MyEventHandler{

        @EventListener
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("이벤트를 받았다. 데이터는 " + event.getData());
        }
    }

    // AnotherHandler.java
    @Component
    public class AnotherHandler {

        @EventListener
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("또다른 이벤트 핸들러입니다. 값은 " + event.getData());
        }
    }
    ```
    - 위 소스를 실행하면, 아래와 같이 동일한 Main Thread를 통해 순차적으로 실행되었다는 것을 알 수 있다.
    ```
    Thread[main,5,main]
    또다른 이벤트 핸들러입니다. 값은 100
    Thread[main,5,main]
    이벤트를 받았다. 데이터는 100
    ```    
    - 위의 우선 순위를 변경하고자 한다면 Handler Method에 Order Annotation을 사용하면 된다.
    ```java
    // MyEventHandler.java
    @Component
    public class MyEventHandler{

        @EventListener
        @Order(Ordered.HIGHEST_PRECEDENCE)
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("이벤트를 받았다. 데이터는 " + event.getData());
        }
    }

    // AnotherHandler.java
    @Component
    public class AnotherHandler {

        @EventListener
        @Order(Ordered.HIGHEST_PRECEDENCE + 2)
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("또다른 이벤트 핸들러입니다. 값은 " + event.getData());
        }
    }
    ```
    - 다시 실행해보면 아래와 같이 순서가 바뀐 것을 확인할 수 있다.
    ```
    Thread[main,5,main]
    이벤트를 받았다. 데이터는 100
    Thread[main,5,main]
    또다른 이벤트 핸들러입니다. 값은 100
    ```
- 비동기 방식의 이벤트 프로그래밍
    - 우선 각각의 Event Handler에 Async Annotation을 명시한다.
    ```java
    // MyEventHandler.java
    @Component
    public class MyEventHandler{

        @EventListener
        @Async
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("이벤트를 받았다. 데이터는 " + event.getData());
        }
    }

    // AnotherHandler.java
    @Component
    public class AnotherHandler {

        @EventListener
        @Async
        public void handle(MyEvent event){
            System.out.println(Thread.currentThread().toString());
            System.out.println("또다른 이벤트 핸들러입니다. 값은 " + event.getData());
        }
    }    
    ```
    - 비동기방식으로 처리하도록 Main Class를 수정한다.(EnableAsync Annotation)
    ```java
    @SpringBootApplication
    @EnableAsync
    public class DemospringApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemospringApplication.class, args);
        }
    }
    ```
    - 실행하면 아래와 같이 MultiThread로 이벤트를 처리하는 것을 확인할 수 있다.
    ```
    Thread[task-2,5,main]
    이벤트를 받았다. 데이터는 100
    Thread[task-1,5,main]
    또다른 이벤트 핸들러입니다. 값은 100
    ```
