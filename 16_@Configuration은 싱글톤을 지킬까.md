## AppConfig 파일을 들여다 보자
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
@configuration을 사용해서 스프링 컨테이너의 설정정보라는 것을 표시하였다.

자세히 보면 `memberService`도 `MemberRepository()`를 호출하고, `orderService`도 `memberRepository()`를 호출한다.

즉, `new memoryMemberRepository()`가 총 2번 일어나서 각각 다른 객체를 생성하는 것을 볼 수 있다.

## 실제 `spring Container`가 다른 객체를 생성하는지, 싱글톤을 지키는지 확인해보자

같은 객체를 생성하는지 확인하기 위해 2가지 테스트를 진행 할 것이다.
- 객체 참조값이 같은지 테스트
- 컨테이너에서 빈을 생성할 때 몇번 호출하는지 테스트

```java
// Configuration Test 용도
public memberRepository getMemberRepository() {
    return memberRepository;
}
```
테스트를 위해 위 코드를 각각 `memberServiceImpl`, `orderServiceImpl`에 추가하였다.

이후 테스트 코드를 작성했다.

```java
public class ConfigurationSingletonTest {
    @Test
    void ConfigurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        memberServiceImpl memberService = ac.getBean("memberService", memberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        memberRepository memberRepository = ac.getBean("MemberRepository", memberRepository.class);

        System.out.println("memberService -> memberRepository" + memberService.getMemberRepository());
        System.out.println("orderService -> memberRepository " + orderService.getMemberRepository());
        System.out.println("MemberRepository = " + memberRepository);

        Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}
```
그림

위 코드와 그림을 보면 이해하기 쉽다.

`memoryMemberRepository` 객체를 2번 생성했는데 `Spring Container`를 거치면 과연 2개의 다른 객체가 생성될 것인지 같은 객체를 사용할 것인지를 테스트 하는 것이다.

```
memberService -> memberRepositoryhello.core.member.memoryMemberRepository@119020fb
orderService -> memberRepository hello.core.member.memoryMemberRepository@119020fb
MemberRepository = hello.core.member.memoryMemberRepository@119020fb
```
정말 아름답게 같은 객체를 사용하는 것을 알 수 있다.

이번에는 컨테이너에서 빈을 생성할 때 빈을 몇 번 호출하는지 테스트 해보자

아까 `AppConfig` 코드에 
```
System.out.println("call AppConfig.memberService");
```
이런식으로 각 메서드마다 `call` 했는지 표시할 것이다.

테스트 하기 전 원래 시나리오 대로라면,

```
// memberService 빈 생성
call AppConfig.memberService
call AppConfig.MemberRepository
// MemberRepository 빈 생성
call AppConfig.MemberRepository
// orderService 빈 생성
call AppConfig.orderService
call AppConfig.MemberRepository
```
이런식으로 호출될 것이다.

하지만 실제로 테스트 코드를 돌려보면
```
18:08:27.767 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'memberService'
call AppConfig.memberService
18:08:27.785 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'MemberRepository'
call AppConfig.MemberRepository
18:08:27.789 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'orderService'
call AppConfig.orderService
```
Spring Container는 단 3번을 호출한다.

뭐지?

## @Configuration의 동작 방식을 알아보자

먼저 `AppConfig`도 스프링 빈에 등록된다는 사실을 알았으니 조회해보자

```java
@Test
void ConfigurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean);
}
```

```text
bean = hello.core.AppConfig$$EnhancerBySpringCGLIB$$250bb178@55562aa9
```
원래 우리가 기대하는 객체는 `hello.core.AppConfig`인데

뒤에 무언가 더 붙어있는 새로운 객체가 생성되었음을 알 수 있다.

뭘까?

**이는 바로 싱글톤을 지키기 위해서 만들어진 객체이다.**

[그림]

그림과 같이 CGLIB라는 클래스 바이트코드를 조작하는 라이브러리를 사용해서 스프링이 직접 싱글톤을 보장해주는 객체로 변신하여 만들어준 것이다.

이런 식으로 싱글톤을 동적으로 생성하여 보장해 줄 것이다.

```java
@Bean
public MemberRepository memberRepository() {

    if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
        return 스프링 컨테이너에서 찾아서 반환;
    } else { //스프링 컨테이너에 없으면
        기존 로직을 호출해서 MemoryMemberRepository를 생성 
        스프링 컨테이너에 등록
        return 반환
    }
}
```

`@Bean` 이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 

스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

## @Configuration을 한번 빼보고 실행해보자

`AppConfig`에서 `@Configuration`을 빼보고

위 테스트 코드를 실행시켜보자

```text
bean = hello.core.AppConfig@7ca20101
```
CGLIB를 이용하지 않는 순수 자바 코드의 객체를 반환했다.

```text
memberService -> memberRepositoryhello.core.member.memoryMemberRepository@1f3f02ee
orderService -> memberRepository hello.core.member.memoryMemberRepository@1fde5d22
MemberRepository = hello.core.member.memoryMemberRepository@5dcb4f5f
```
다른 테스트도 마천가지로 `@Bean`이 붙은 메서드의 순수 자바 코드를 그대로 실행시켜서

싱글톤이 지켜지지 않고 객체를 추가로 생성된 모습을 볼 수 있다.

## 정리

- `@Configuration`이 없어도 스프링 빈으로 등록되어 `의존관계 주입`이 가능하지만, `싱글톤`이 지켜지지 않는다. 즉, 싱글톤을 지킬 수 있게 하는 역할을 한다.

- 설정정보를 `@Configuration`을 통해 무조껀 등록하다록 하자.