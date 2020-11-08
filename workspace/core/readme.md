##Spring Core Study
`
이 프로젝트는 인프런 김영한님의 스프링 핵심기능 강의를 듣고 실습하는 프로젝트 입니다.
`  

####1. 순수 자바를 활용한 프로젝트 구현
* Project Info
    * Java : JDK 11
    * Project : Gradle Project
    * IDE : IntelliJ Community
    * *[Spring Initializr](start.spring.io)* 활용하여 Dependencies 선택하지 않고 프로젝트 생성
* Business Requirement
    * 회원
        * 회원을 가입하고 조회할 수 있다.
        * 회원은 일반과 VIP 두 가지 등급이 있다.
        * 회원 데이터는 자체 DB를 구출할 수 있고, 외부 시스템과 연동할 수 있다.(미확정)
    * 주문과 할인 정책
        * 회원은 상품을 주문할 수 있다.
        * 회원 등급에 따라 할인 정책을 적용할 수 있다.
        * 할인 정책은 모든 VIP에게 1000원을 할인해주는 고정 금액 할인을 적용해 달라.(변경될 수 있음)
        * 할인 정책의 경우 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다.
          최악의 경우, 할인을 적용하지 않을 수도 있다. 