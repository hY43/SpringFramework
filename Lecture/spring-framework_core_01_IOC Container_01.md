
## 1강 IOC Container and Bean

```
인프런 백기선님의 "스프링 프레임워크 핵심 기술" 강의를 들으며 스프링 프레임워크의 기본 사항들을 정리한 내용입니다.
좀더 상세한 내용은 강의를 수강해주세요~
```

### 01. IOC (Inversion Of Control)
---

번역하면 제어의 역전이라는 말의 약어이며 다른 말로는 Dependency Injection이라는 의존성 주입이라고도 하는 개념이다.

일반적으로 책을 보면 저 의존성 주입이라는 말은 *"어떤 객체가 사용하고자 하는 의존 객체를 직접 만들어 사용하는 것(ex) new ArrayList())이 아니라 어떤 장치(Ex) 생성자, Setter 등)를 통해 객체를 주입하는 방식"*을 말한다고 설명한다.

그리고 이러한 방식은 단순히 스프링에서만 사용되는 개념이 아니라 비슷한 역할을 하는 장치만 구현한다면 사용도 가능하다.

단순히.. 글로만 나열되어 있으면 이해가 어려우므로 이에 대해서는 Spring IOC Container를 설명하며 소스 코드를 통해 확인해보자.

### 02. IOC Container
---
* IOC 기능을 제공하는 *Bean Object*를 담고 있는 일종의 저장소라고 할 수 있다.
*  최상위 인터페이스로는 BeanFactory가 있으며 이를 구현하여 Bean Object에 의존성을 주입하는 등의 기능을 제공한다.
BeanFactory를 상속하는 인터페이스의 예로는 ApplicationContext가 있다.
* Bean Configuration File로 부터 Bean 정의를 읽어서 Bean을 구성하고 제공한다.    

### 03. Bean Object
---
* IOC Container가 관리하는 Object
* Bean Object의 장점
    * 의존성 관리
    * 스코프
        * 싱글톤 타입 : 최초 1회에만 객체를 생성/사용하여 불필요한 객체 생성을 방지한다.
        * 프로포토타입 : 매번 객체를 생성하여 사용한다.
    * Life Cycle Interface
        * Bean이 특정 시점에 특정 동작을 하도록 설정이 가능하다
        * ex) PostConstruct Annotation


### 04. POJO Object
---
* Plain Old Java Obejct의 약어로 Spring IOC Container에 의해 관리되는 Bean이 아닌 Object로,  예를 들어 멤버 변수와 Get/Set 메소드만을 갖는 DTO 역시 이에 해당된다.

### 05. Bean Object VS POJO Object
---
* Bean Object와 POJO Object를 구분하는 방법은?
가장 간단한 방법은 Component Annotation를 확장하여 만든 Annotation으로 표기된 Class는 Bean이라고 생각하면 된다.

예를 들어 아래의 소스코드를 참고해보자.
```java
@Repository
public class BookRepository {


}
```
위의 BookREpository Class는 Bean이라고 할 수 있는데 그 이유는 @Repository가 아래와 같이 Component를 확장하여 만든 Annotation이기 때문이다.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

### 05. ApplicationContext 인터페이스
* BeanFactory 인터페이스를 상속받은 인터페이스로 IOC Container의 기능을 포함하는 인터페이스이다.
* 아래와 같은 기능들을 제공하며 해당 기능들에 대해서는 차후에 소스 코드를 통해 정리하도록 하자.
    * IOC Container
    * MessageSource(i18n) -> 다국어 처리 기능
    * 이벤트 발행 기능
    * 리소스 로딩 기능

