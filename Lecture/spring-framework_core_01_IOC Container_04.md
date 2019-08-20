## 4강 @Component and ComponentScan

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```

### 01. Component Scan
---
- Component Scan의 범위
    - 이전 소스에서 Component Annotation으로 기입된 class를 Bean으로 자동 등록해주는 Component Scan에 대해서 한번 본적이 있다.
    - 그리고 이 Component Scan을 하는 방식 중에서 basePackageClasses Method를 활용하여 해당 class부터 시작해서 그 class가 포함된 패키지 내의 모든 Bean을 등록하는 방식에 대해 조금더 살펴보도록 하자.
        - 2개의 패키지를 생성하여 한 패키지에서 Component Scan을 통해 다른 패키지의 Bean을 생성하고 의존성을 주입할 수 있는지 확인해보자.
        ```java
        // MyService.java
        package com.example.out;

        @Service
        public class MyService {
        }

        // DemospringApplication.java
        package com.example.demospring;

        @SpringBootApplication
        public class DemospringApplication {
            @Autowired
            MyService myService;
            
            public static void main(String[] args) {
                SpringApplication.run(DemospringApplication.class, args);
            }
        }
        ```
        - 위의 DemospringApplication를 실행하면 아래와 같은 에러가 발생하는데, 확인해보면 MyService를 찾지 못해 발생한 에러이다.
        ```
        Field myService in com.example.demospring.DemospringApplication required a bean of type 'com.example.out.MyService' that could not be found.

        The injection point has the following annotations:
            - @org.springframework.beans.factory.annotation.Autowired(required=true)


        Action:

        Consider defining a bean of type 'com.example.out.MyService' in your configuration.
        ```
- Component Scan Filter
    - Component Scan 대상에서 제외하거나 대상으로 포함하는 Filter 기능을 제공한다.
    ```java
    // SpringBootApplication.java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(
        excludeFilters = {@Filter(type = FilterType.CUSTOM,classes = {TypeExcludeFilter.class}),                    @Filter(type = FilterType.CUSTOM,classes = {AutoConfigurationExcludeFilter.class})}
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
    - 위의 소스는 SpringBootApplication Annotation으로 이 Annotation 안에는 ComponentScan Annotation이 포함되어 있으며, ExcludeFilter가 추가되어 있다.
- @Component 의 확장 Annotation
    - @Controller
    - @Service
    - @Repository
    - @Configuration
- Component Scan의 단점
    - 어플리케이션 구동 초기에 해당 Bean을 모두 검색하여 등록하기 때문에, Bean이 많아지면 많아질수록 속도가 느려진다. 하지만 최초 구동할 때에 Bean이 생성되기 때문에 이후에 추가로 Bean을 생성할 필요가 없다는 부분이 있으니.. 사실.. 단점이라기에는 모호한 부분이 있다.
    - 구동 시간이 느려진다는 단점에 대한 해결책은 Spring 5부터 도입된 **Function을 활용한 Bean 생성** 방법이 있는데 이 방식은 기존의 방식과는 달리 Reflection이나 Proxy와 같은 방식을 사용하지 않기 때문에 별도의 리소스를 할당하지 않아도 된다.
    ```java
    // DemospringApplication.java
    @SpringBootApplication
    public class DemospringApplication {

        @Autowired
        MyService myService;
        
        public static void main(String[] args) {
            SpringApplication app = new SpringApplication(DemospringApplication.class);
            // Bean 등록
            app.addInitializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
                ctx.registerBean(MyService.class);
            });
            app.run(args);
        }
    }
    ```    
    - 기존에 Component Scan 방식과는 달리 직접 Bean을 등록하기 때문에 패키지 위치와는 관계 없이 MyService.java 역시 등록할 수 있다.
    - 위와 같은 방식을 사용했을 때의 장점은 아래와 같이 Bean을 등록하는 부분에 로직을 넣을 수 있다는 점이다.
    ```java
    // DemospringApplication.java
    @SpringBootApplication
    public class DemospringApplication {

        @Autowired
        MyService myService;
        
        public static void main(String[] args) {
            SpringApplication app = new SpringApplication(DemospringApplication.class);
            // Bean 등록
            app.addInitializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
                if(true){
                    ctx.registerBean(MyService.class);
                }
            });
            app.run(args);
        }
    }
    ```    
    - 물론 위 방식은 Component Scan 방식에 비해 성능상의 이점은 가져갈 수는 있으나.. 모든 Bean을 수기로 등록해줘야 하므로 추천하는 방식은 아니다..
- Component Scan 동작 원리
    - BeanFactoryPostProcessor를 구현한 ConfigurationClassPostProcessor에 의해 다른 빈들을 만들기 이전에 Component Scan을 진행해서 Bean을 등록한다.
    - 여기에서 다른 빈들이란 Bean Annotation을 사용하여 만들어질 Bean이나 위에서 살펴본 Function을 사용한 Bean 등록 등을 들 수 있다.

