---
layout: page
title: "java return문"
categories: javaBasic
---

#### 이게 뭐에요?

메서드 내에서 특정 작업을 수행한 후 결과값을 도출해내고 싶을 때 사용

정확히는 **현재 실행중인 메서드를 종료하고 호출한 메서드로 되돌아가는 역할**

```java
return;
```



#### 어떻게 동작하는 거에요?

```java
void printName(void) {
    System.out.println("박상혁");
    return;
}
```

printName이란 메소드는 "박상혁"이라는 문자열을 출력해주는 역할을 담당

이 때 메소드를 종료하고 원래 호출했던 장소로 되돌아가는 역할을 담당

반환타입이 void인데도 return문을 쓰는 이유는 

**원래는 반환값의 유무에 관계없이 모든 메서드에 적어도 하나의 return문이 있어야 된다는 규칙 때문**

반환값의 유무에 따라

```java
return;
return x;
```

로 쓰인다. 반환값이 없으면 **그냥 메서드를 종료**



#### 언제 사용해야 할까요?

매개 변수의 **유효성 검사**를 위해 사용

메서드를 작성할 때

**매개변수의 값이 적절한 것인지 확인하는 것**이 제일 중요

```java
float divide(int x, int y) {
	if(y==0) {
        System.out.println("0으로 나눌 수 없습니다.");
        return;
    }
    
    return x / (float)y;
}
```

0으로 나누는 것이 금지되어 있기 때문에

**매개변수 y가 0인지 아닌지 확인**

이 때 return문을 **작업 중단 시점**에 넣어 사용 가능.

덕분에 매개변수의 유효성을 판단할 수 있다