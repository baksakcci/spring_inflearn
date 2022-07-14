## 스프링 컨테이너에 저장된 모든 빈 객체를 조회해보자(꺼내보자)

컨테이너를 생성하고 설정을 지정했으면

이제 데이터를 조회하는 일을 해보자

## 우선 테스트를 위해 코드에 컨테이너를 생성해보자

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```

이렇게 해버리면 문제가 생긴다.

바로 `getBeanDifinition()`메소드를 사용을 못한다는 것.

이유를 살펴보니 `ApplicationContext` `interface`를 상속으로 삼았다는 것.

`getBeanDifinition()`은 `AnnotationConfigApplicationContext` 의 **부모 클래스**에서 상속받은 메소드라

`ApplicationContext` `interface`에는 구현되지 않은 메소드인 것.

이렇게 구조를 짠 이유는 바로

**ISP (인터페이스 분리 원칙 : Interface Segregation Principal)을 지키기 때문이다.**

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```

이렇게 해줘야 한다.

## 모든 빈을 출력할 테스트 코드를 작성해보자

```java
@Test
@DisplayName("모든 빈 출력하기")
void findAllBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("Name = " + beanDefinitionName
                + " object = " + bean);
    }
}
```

컨테이너를 만들었으면 `getBeanDeifinitionNames()`메서드를 통해 모든 빈의 이름을 가져오고 이거를 `String[]`배열에 넣자

이후 `향상된 for문`을 사용하여 배열에서 이름을 하나하나 빼내보자

`getBean()`을 사용하여 매개변수에 빈 이름을 넣고 **객체를 꺼내보자**

꺼낸 빈 객체와 빈 이름을 출력하자

![image-20220706180441105](https://user-images.githubusercontent.com/105288887/177519731-b474fee2-9f32-4810-80e0-60b51e53785a.png)

이렇게 이름과 객체가 출력되는 것을 알 수 있다.

## 우리가 등록한 빈 만을 출력해보자

빈은 총 2가지로 나누어진다

- `ROLE_APPLICATION` : 일반적으로 사용자가 정의한 빈 

- `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈

우리가 등록한 빈 만 출력해보자

```java
@Test
@DisplayName("애플리케이션 빈 출력하기")
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

        if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("Name = " + beanDefinitionName
                    + " object = " + bean);
        }
    }
}
```

`getBeanDefinition()`을 사용해서 매개변수로 이름을 넣고 빈의 메타데이터들을 꺼내와보자

```java
beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION
```

이게 뭔소리나면

`BeanDefinition.ROLE_APPLICATION`은 실제로 `public final static int`타입으로 `0`으로 저장된 것이다.

즉, 빈의 메타데이터에서 `getRole()`메서드를 사용해서 꺼낸게 0이면 **우리가 직접 등록한 애플리케이션 빈이라는 소리**이다.

이후에 위와 똑같이 bean 객체, bean 이름을 가져와서 프린트 한다.

![image-20220706182218871](https://user-images.githubusercontent.com/105288887/177519761-efa48d69-2547-42f4-bf97-7a5dcbba0332.png)

잘 출력되는 것을 알 수 있다.

## getBean()을 통해 빈을 조회해보자

`getBean('빈 이름', '빈 타입')`으로 이름과 타입을 매개변수로 입력해서 조회할 수 있고

`getBean('타입')`타입 만으로 조회할 수 있다.

```java
@Test
@DisplayName("빈 이름, 타입을 이용해 조회")
void findBeanByName() {
    memberService memberService = ac.getBean("memberService", memberService.class);
    System.out.println("memberService = " + memberService +
            "\nmemberService.getClass = " + memberService.getClass());
    assertThat(memberService).isInstanceOf(memberServiceImpl.class);
}
```

우선 위에 `AnnotationConfigApplicationContext` 컨테이너를 만들어주고 시작하자.

`jUnit`을 이용해서 테스트 진행.

그 후 이전에 `AppConfig`에 구현했던 `memberSerivce()`를 컨테이너에서 가져와서 `getBean()`을 통해 조회했다.

그 후 컨테이너에 저장된 객체와 우리가 만들었던 `memberServiceImpl.class`가 같은지 비교하였다.

![image-20220707152034563](https://user-images.githubusercontent.com/105288887/177734875-be1fd399-4474-46fc-925b-974a3af372a7.png)

```java
memberService memberService = ac.getBean(memberService.class);
```

타입 만으로 빈을 조회해도 결과는 똑같다.

```java
memberServiceImpl memberService = ac.getBean(memberServiceImpl.class);
```

인터페이스 말고 **구현체**를 타입으로 가져와도 똑같이 작동한다.

그러나, 역할과 구현으로 나눠지지 않고 구현체를 직접 가져오는 코드가 되기 때문에 좋진 않다.

## getBean()에서 없는 이름을 적고 테스트 해보자

```java
@Test
@DisplayName("다른 빈 이름으로 조회했을 때")
void findBeanByNameX() {
    memberService memberService = ac.getBean("xxxxx", memberService.class);
    assertThat(memberService).isInstanceOf(memberServiceImpl.class);
}
```

다른 이름으로 조회했을 때 오류가 던져진다.

`org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'xxxxx' available`

반대로 이 테스트를 오류없게 나오기 위한 코드를 구현해보자

```java
@Test
@DisplayName("다른 빈 이름으로 조회했을 때")
void findBeanByNameX() {
    assertThrows(NoSuchBeanDefinitionException.class,
            () -> ac.getBean("xxxxx", memberService.class));
}
```

여기서 `assertThrows()` 메서드는 `assertj`에서 `import`한게 아닌,

`junit`에서 `import`한 메서드이다.

첫번째 매개변수인 `NoSuchBeanDefinitionException.class` 오류 클래스가

람다 함수인 `()`의 결과와 똑같다면 제대로 실행 될 것이다.
