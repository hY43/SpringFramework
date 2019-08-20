## 5강 Bean Scope

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```

### 01. Bean Scope
---
- Bean Scope란 Bean의 생성 범위를 나타내며, Singleton, Prototype 등이 있다.

### 02. Singleton Scope
---
- 이전 강의까지 사용해온 Bean들은 별도의 설정을 하지 않았으므로 기본 옵션인 Singleton Scope으로 생성되었었다.
- Singleton Scope이란 Application 전체에 걸쳐서 해당 Bean의 Instance가 단 한개만 생성되는 방식을 말한다.
- 소스를 통해 정말 한개만 생성하여 계속 사용하는지 확인해보자.
    - Singleton Scope을 갖는 Bean 구현
        - 두 개의 Bean을 생성하되, 한 개의 Bean이 다른 Bean을 갖고 있도록 설계하여 구현한다.
        ```java
        // Proto.java
        @Component
        public class Proto {
        }

        // Single.java
        @Component
        public class Single {

            @Autowired
            Proto proto;

            public Proto getProto() {
                return proto;
            }
        }
        ```
    - ApplicationRunner 구현
        - Application이 구동되는 시점에서 Singleton Scope Bean을 체크하기 위해 Runner를 구현한다.
        ```java
        // AppRunner.java
        @Component
        public class AppRunner implements ApplicationRunner {
            @Autowired
            Single single;

            @Autowired
            Proto proto;

            @Override
            public void run(ApplicationArguments args) throws Exception {
                System.out.println("proto Bean : " + proto);
                System.out.println("single Bean의 proto Bean" + single.getProto());
                System.out.println(proto.equals(single.getProto()));
            }
        }
        ```
    - Main Class 구현
        - 위의 Application을 구동하기 위한 Main Class를 구현한다.
        ```java
        @SpringBootApplication
        public class DemospringApplication {

            public static void main(String[] args) {
                SpringApplication.run(DemospringApplication.class, args);
            }
        }
        ```
    - Application을 실행해보면 아래와 같이 Proto Bean과 Single Bean이 갖고 있는 Proto Bean이 동일한 객체임을 확인할 수 있다.
    ```
    proto Bean : com.example.demospring.Proto@51abf713
    single Bean의 proto Beancom.example.demospring.Proto@51abf713
    true
    ```

### 03. ProtoType Scope
---
- ProtoType Scope이란 Bean Instance를 매번 생성하는 방식을 말한다.
- Singleton Scope와 소스로 비교해서 확인해보자.
    - 위에서 실습한 Proto Bean을 ProtoType Scope Bean으로 아래와 같이 수정한다.
    ```java
    // Proto.java
    @Component
    @Scope("prototype")
    public class Proto {
    }
    ```
    - ApplicationContext에서 직접 Bean을 꺼내서 확인하기 위해 ApplicationRunner를 구현한다.
    ```java
    // AppRunner.java
    @Component
    public class AppRunner implements ApplicationRunner {

        @Autowired
        ApplicationContext ctx;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            System.out.println("proto");
            System.out.println(ctx.getBean(Proto.class));
            System.out.println(ctx.getBean(Proto.class));
            System.out.println(ctx.getBean(Proto.class));

            System.out.println("single");
            System.out.println(ctx.getBean(Single.class));
            System.out.println(ctx.getBean(Single.class));
            System.out.println(ctx.getBean(Single.class));
        }
    }
    ```
    - 수정 후, Application을 실행하면 아래와 같이 Singleton Scope은 한 개의 객체로 계속 사용하는 반면 ProtoType Scope은 매번 새로운 객체를 만들어 사용하는 것을 확인할 수 있다.
    ```
    proto
    com.example.demospring.Proto@159e366
    com.example.demospring.Proto@57dc9128
    com.example.demospring.Proto@24528a25
    single
    com.example.demospring.Single@17ae98d7
    com.example.demospring.Single@17ae98d7
    com.example.demospring.Single@17ae98d7
    ```
### 04. ProtoType Scope Bean에서 Singleton Scope Bean을 참조한다면??
---
- 결론부터 말하자면 아무런 문제 없다.
    - 그 이유는 Singleton은 최초 한번 생성 후, 계속 메모리에 상주해있기 때문에 매번 생성되는 ProtoType Bean에서는 그냥 불러서 사용하면 되기 때문이다.

- 소스를 통해 확인해보자.
    - Singleton Bean을 갖는 ProtoType Bean 구현
    ```java
    // Proto.java
    @Component
    @Scope("prototype")
    public class Proto {

        @Autowired
        Single single;

        public Single getSingle() {
            return single;
        }
    }
    ```
    - ApplicationContext에서 직접 Bean을 꺼내서 확인하기 위해 ApplicationRunner를 구현한다.
    ```java
    // AppRunner.java    
    @Component
    public class AppRunner implements ApplicationRunner {

        @Autowired
        ApplicationContext ctx;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            Proto proto1 = ctx.getBean(Proto.class);
            Proto proto2 = ctx.getBean(Proto.class);
            Proto proto3 = ctx.getBean(Proto.class);
            System.out.println("proto1 : " + proto1 + ", proto1의 single : " + proto1.getSingle());
            System.out.println("proto2 : " + proto2 + ", proto2의 single : " + proto2.getSingle());
            System.out.println("proto3 : " + proto3 + ", proto3의 single : " + proto3.getSingle());
        }
    }
    ```
    - Application을 실행해보면 아래과 같이 ProtoType Bean은 계속 생성되는 반면에 ProtoType Bean이 갖고 있는 Singleton Bean은 동일한 것을 확인할 수 있다.
    ```
    proto1 : com.example.demospring.Proto@271f18d3, proto1의 single : com.example.demospring.Single@6bd51ed8
    proto2 : com.example.demospring.Proto@61e3a1fd, proto2의 single : com.example.demospring.Single@6bd51ed8
    proto3 : com.example.demospring.Proto@51abf713, proto3의 single : com.example.demospring.Single@6bd51ed8
    ```

### 05. Singleton Scope Bean에서 ProtoType Scope Bean을 참조한다면??
---
- Singleton Scope의 경우, 객체를 최초 1회만 생성하기 때문에 Singleton Bean이 갖는 ProtoType Bean 역시 최초 1회만 생성되고 새로 생성하려해도 갱신이 되지 않는다.
- 이 문제에 대해 소스를 통해 확인해보자.
    - ProtoType Bean과 ProtoType Bean을 갖는 Singleton Bean을 각각 구현한다.
    ```java
    // Proto.java
    @Component
    @Scope("prototype")
    public class Proto {
    }

    // Single.java
    @Component
    public class Single {
        @Autowired
        Proto proto;

        public Proto getProto() {
            return proto;
        }
    }
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
            System.out.println("proto");
            System.out.println(ctx.getBean(Proto.class));
            System.out.println(ctx.getBean(Proto.class));
            System.out.println(ctx.getBean(Proto.class));

            System.out.println("single");
            System.out.println(ctx.getBean(Single.class));
            System.out.println(ctx.getBean(Single.class));
            System.out.println(ctx.getBean(Single.class));

            System.out.println("single.getProto");
            System.out.println(ctx.getBean(Single.class).getProto());
            System.out.println(ctx.getBean(Single.class).getProto());
            System.out.println(ctx.getBean(Single.class).getProto());
        }
    }
    ```
    - 실행 결과를 확인하면 아래와 같이 Singleton Bean이 참조하는 ProtoType Bean은 갱신되지 않는 것을 확인할 수 있다.
    ```
    proto
    com.example.demospring.Proto@4d4d48a6
    com.example.demospring.Proto@315df4bb
    com.example.demospring.Proto@3fc08eec
    single
    com.example.demospring.Single@5cad8b7d
    com.example.demospring.Single@5cad8b7d
    com.example.demospring.Single@5cad8b7d
    single.getProto
    com.example.demospring.Proto@7b02e036
    com.example.demospring.Proto@7b02e036
    com.example.demospring.Proto@7b02e036
    ```
- 이러한 갱신 문제는 ProxyMode를 사용하여 해결이 가능한데, 그 방법은 ProtoType Bean을 아래와 같이 구현해주면 된다.
    - ProxyMode를 추가하여 ProtoType Bean 구현
    ```java
    // Proto.java
    @Component
    @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public class Proto {
    }
    ```
    - 다시 실행해보면 아래와 같이 갱신되는 것을 확인할 수 있다.
    ```
    proto
    com.example.demospring.Proto@3773862a
    com.example.demospring.Proto@2472c7d8
    com.example.demospring.Proto@589b028e
    single
    com.example.demospring.Single@22175d4f
    com.example.demospring.Single@22175d4f
    com.example.demospring.Single@22175d4f
    single.getProto
    com.example.demospring.Proto@9fecdf1
    com.example.demospring.Proto@3b809711
    com.example.demospring.Proto@3b0f7d9d
    ```
    - 위 코드에 대해 설명을 덧붙이면 ***"Proxy로 해당 Bean을 감싸라"*** 라는 의미로 위와 같이 구현하면, CG Library에 의해 Proxy Bean이 생성되고, 다른 인스턴스들은 이 Proxy Bean을 통해서 실제 Bean에 접근하게 된다.(Proxy Pattern 참조)
- 또다른 방법으로는 ObjectProvider로 감싸서 ProtoType Bean을 참조하는 방법이 있다.
    - Singleton Bean에서 ObjectProvider로 ProtoType Bean을 감싸서 참조하도록 구현한다.
    ```java
    // Single.java
    @Component
    public class Single {
        @Autowired
        ObjectProvider<Proto> proto;

        public Proto getProto() {
            return proto.getIfAvailable();
        }
    }
    ```
    - 이 방법 역시.. 해결은 가능하나 Bean 자체에 Spring의 소스코드가 들어가기 때문에 추천하지는 않는다.

### 06. Singleton Scope Bean 사용시 고려 사항
---
- Singleton Scope이 ProtoType Scope 보다 범위가 넓기 때문에(생존 주기가 길다.) 갱신 등의 문제에 대해 항상 체크해야한다.
- 프로퍼티가 공유되기 때문에, Thread Safe한 방식으로 개발해야한다.
    - 예를 들어 아래와 같이 Singleton Scope Bean이 구현되었다고 생각해보자.
    ```java
    // Single.java
    @Component
    public class Single {
        @Autowired
        ObjectProvider<Proto> proto;

        int number = 0;

        public Proto getProto() {
            return proto.getIfAvailable();
        }
    }
    ```
    - 위의 경우에 Single이라는 Bean은 Singleton Bean이기 때문에 Application이 구동하는 중에는 어디서 접근해도 같은 Bean에 접근된다.
    이 때문에 number값이 매우 불안정하기 때문에 이에 대한 대비책 역시 필요하다.
