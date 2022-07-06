## 지금까지 짜왔던 코드를 스프링으로 전환해보자

`AppConfig`에서 직접 객체 생성 및 의존성 주입을 해왔다면

이제는 `SpringContainer`에서 **설정정보를 등록하고 관리**하도록 해보자

```java
@Configuration
public class AppConfig {
    @Bean
    public memberService memberService() {
        return new memberServiceImpl(MemberRepository());
    }
    @Bean
    public memberRepository MemberRepository() {
        return new memoryMemberRepository();
    }
    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(MemberRepository(), DiscountPolicy());
    }
    @Bean
    public DiscountPolicy DiscountPolicy()  { return new FixDiscountPolicy();
    }
}
```

`@Configuration`, `@Bean` 어노테이션을 각각 이런식으로 붙이면 된다.

이후 실제로 사용할 때 이렇게 하면 된다.

```java
// AppConfig appConfig = new AppConfig();
// memberService memberService = appConfig.memberService();

ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
memberService memberService = applicationContext.getBean("memberService",memberService.class);
```

`AppConfig` 대신에 `ApplicationContext`라는 스프링 컨테이너가 객체생성과 주입을 대신해준다.

![image-20220706132708111](https://user-images.githubusercontent.com/105288887/177519277-16f06869-b6ec-4194-9fcd-5e6358f82465.png)

스프링 컨테이너의 등장으로 앞으로 어떻게 사용하고 어떠한 장점이 있는지 알아볼 예정이다.

