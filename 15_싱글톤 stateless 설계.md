## 싱글톤 패턴, 컨테이너 전부 설계할 때 고려해야 할 점이 있다.

많은 사람들이 동시에 요청하는 `Web`환경에서

싱글톤 패턴을 사용한 컨테이너는 객체가 1개만 존재하게 되는데,

`stateful`(상태 유지)하게 설계해버리면 문제가 생긴다.

코드로 확인해보자

```java
public class StatefulService {
    private int price;

    public void order(String name, int price) {
        System.out.println("name = " + name + "price = " + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}
```

객체 상태를 유지하게 코드를 위와같이 짜면 

`price`가 런타임에서 계속 유지`stateful`하게 된다.

```java
    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        // thread A
        statefulService1.order("userA", 10000);
        // thread B
        statefulService2.order("userA", 20000);

        // thread A
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig {
        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
```

위에서 서비스를 `stateful`하게 설게한 후

싱글톤이 적용된 `spring container` 를 사용해서 테스트 코드를 작성해보았다.

결과

```
name = userAprice = 10000
name = userAprice = 20000
price = 20000
```

그림

그림과 같이 `threadA`에서 10000원이란 값이 나와야 하는데 `stateful`하게 설계한 탓에 20000원이란 잘못된 값이 나오게 된 것이다.

이 경우 상당한 문제가 될 수 있다.

## 이를 어떻게 해결해야 할까요?

무상태(`stateless`)로 설계해야 한다

이를 위한 몇가지 지켜야 할 점이 있다.

- 특정 클라이언트에 의존적인 필드가 있으면 안된다.

- **특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!**

- 가급적 읽기만 가능해야 한다.

- 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

지금 위 코드를 보면 `client`단에서 `price`값을 직접 수정하는 것을 볼 수 있다.

이를 수정해야 한다.

```java
public class StatefulService {
    private int price;

    public int order(String name, int price) {
        System.out.println("name = " + name + "price = " + price);
        //this.price = price;
        return price;
    }
}
```

`order()`를 직접 값을 반환하게 만듬으로써 클라이언트가 `price`값을 수정 못하게 설계하면 된다.

```java
// thread A
int userAPrice = statefulService1.order("userA", 10000);
// thread B
int userBPrice = statefulService2.order("userA", 20000);
```

이런식으로 test코드를 바꿔주면 된다.

```
name = userAprice = 10000
name = userAprice = 20000
price = 10000
```

정상적으로 출력이 되었다.

## 요약

`container`를 사용할 때 `singleton`으로 설계되었기 때문에 객체 1개로 여러 사용자가 사용하게 되는데,

`stateful`하게 설계하면 안된다. 무조껀 `stateless`하게 설계해야 한다.