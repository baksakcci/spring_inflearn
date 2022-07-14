## 빈 저장소 안에 같은 타입이 2개 이상 있다면 어떻게 조회할까?

**배경**

테스트 하기 위해 임의의 설정정보를 테스트 코드 안에다가만 추가했다.

```java
@Configuration
static class SameBeanConfig {

    @Bean
    public memberRepository MemberRepository1() {
        return new memoryMemberRepository();
    }

    @Bean
    public memberRepository MemberRepository2() {
        return new memoryMemberRepository();
    }
}
```

같은 타입이 2개 있는 설정 정보 `SameBeanConfig`를 만들었다.



**`getBean()` type으로 조회**

```java
@Test
@DisplayName("타입으로 조회할 때 같은 타입이 2개 이상 있다면 오류 발생")
void findBeanByTypeDuplicate() {
    memberRepository memberRepository = ac.getBean(memberRepository.class);
    Assertions.assertThat(memberRepository).isInstanceOf(memoryMemberRepository.class);
}
```

`org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'hello.core.member.memberRepository' available: expected single matching bean but found 2: MemberRepository1,MemberRepository2`

위와 같이 `NoUniqueBeanDefinitionException` 에러가 발생한다.

즉, 같은 타입이 2개 이상 있을 때 `getBean()` 메서드로 조회할 수 없다.



**`getBean()` 이름으로 조회**

당연한 말이겠지만, 이름으로 조회하면 된다.



## 같은 타입이라도 다 조회하고 싶은데 어떻게 해야할까?

**`getBeansOfType()` 메서드 사용**

```java
@Test
@DisplayName("같은 타입들끼리 모두 조회하고 싶다")
void findBeanByTypeAll() {
    Map<String, memberRepository> beansOfType = ac.getBeansOfType(memberRepository.class);
    for (String key : beansOfType.keySet()) {
        System.out.println("key = " + key); // key
    }
    System.out.println("beansOfType = " + beansOfType); // value
	Assertions.assertThat(beansOfType.size()).isEqualTo(2);
}
```

위에서 스프링 빈 저장소에 저장했던 `memberRepository1, 2`를 한번에 조회하고자 한다.

이 두 메서드의 타입을 매개변수로 메서드를 사용해보면

`Map<String, memberRepository>`로 받아오는 것을 볼 수 있다.

여기서 `key`값은 스프링 빈 이름, `value`값은 빈 객체이다.

```java
key = MemberRepository1
key = MemberRepository2
beansOfType = {MemberRepository1=hello.core.member.memoryMemberRepository@5c10f1c3, MemberRepository2=hello.core.member.memoryMemberRepository@7ac2e39b}
```

이렇게 잘 출력이 된걸 알 수 있다.

검증은 이 `beansOfType`의 사이즈가 2라는 것을 증명하면 된다.

## 빈 저장소에 자식이 2개 이상있는 부모 타입을 어떻게 조회할까?

**배경**

```java
@Configuration
static class TestConfig {
    @Bean
    public DiscountPolicy fixDiscountPolicy() {
        return new FixDiscountPolicy();
    }

    @Bean
    public DiscountPolicy rateDiscountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

설정 정보를 예제에 맞게 작성했다.

같은 부모에 서로다른 자식

일단 **"부모타입을 조회하면 자식들도 함께 조회된다."** 라는 사실을 기억하고 들어가자.

**`getBean()` 사용**

```java
@Test
@DisplayName("부모 타입으로 조회 시 자식이 둘 이상 있으면 오류 발생")
void findBeanByParentTypeDuplicate() {
    DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
    Assertions.assertThat(bean).isInstanceOf(FixDiscountPolicy.class);
}
```

오류가 발생한다.

`org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'hello.core.discount.DiscountPolicy' available: expected single matching bean but found 2: fixDiscountPolicy,rateDiscountPolicy`

위와 같은 이유이다.

**타입 대신에 이름을 조회하는 방법이 있다**

위의 케이스와 동일

**하위 타입으로 조회하는 방법이 있다**

좋은 방법은 아니다. 역할과 구현을 나누지 못했기 때문

**부모 전체를 조회하는 방법**

위와 마찬가지로 `getBeansOfType()` 메서드를 이용하면 된다.

```java
@Test
@DisplayName("부모 타입 전체를 조회")
void findBeanByParentTypeAll() {
    Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
    for (String key : beansOfType.keySet()) {
        System.out.println("key = " + key);
        System.out.println("beansOfType = " + beansOfType.get(key));
    }
    Assertions.assertThat(beansOfType.size()).isEqualTo(2);
}
```

## 부모 전체를 Object 클래스 타입으로 조회해보자

![image-20220707174646484](https://user-images.githubusercontent.com/105288887/177735102-000db1a4-5348-4540-ad86-90925376c0b1.png)

모든 객체의 최상위 클래스인 `Object` 타입으로 조회해보자

```java
@Test
@DisplayName("부모 타입 전체를 조회 - 최상위 클래스 Object 객체로 조회")
void findBeanByParentTypeAllObject() {
    Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
    for (String key : beansOfType.keySet()) {
        System.out.println("key = " + key);
        System.out.println("beansOfType = " + beansOfType.get(key));
    }
}
```

컨테이너는 그대로 두고

컨테이너를 `getBeansOfType`으로 조회할 때 최상위 클래스인 `Object` 타입으로 조회하면

컨테이너 안에 저장된 모든 빈들을 조회할 수 있다.

```java
key = org.springframework.context.annotation.internalConfigurationAnnotationProcessor
beansOfType = org.springframework.context.annotation.ConfigurationClassPostProcessor@1b2c4efb
key = org.springframework.context.annotation.internalAutowiredAnnotationProcessor
beansOfType = org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@c35172e
key = org.springframework.context.annotation.internalCommonAnnotationProcessor
beansOfType = org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@c2db68f
key = org.springframework.context.event.internalEventListenerProcessor
...
그리고 중간에
key = fixDiscountPolicy
beansOfType = hello.core.discount.FixDiscountPolicy@3668d4
key = rateDiscountPolicy
beansOfType = hello.core.discount.RateDiscountPolicy@1c3b9394
있다.
```

## 이게 어떤점에서 중요하나요?

실무에서도 `getBean()`을 쓸 일은 거의 없지만

나중에 나오는 `@AutoWired`같이 자동으로 의존관계 주입해주는 기술들을 익히기 위한

기본, 배경 지식으로 알아두는 것이 좋다.
