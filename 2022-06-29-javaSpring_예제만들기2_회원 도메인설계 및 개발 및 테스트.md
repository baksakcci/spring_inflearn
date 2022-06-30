---
layout: page
title: "vanillaJava 2 회원 도메인 설계"
categories: javaSpring
---

## 회원 도메인 요구사항을 보며 협력관계를 파악한다.

![image-20220629161657877](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629161657877.png)

요구사항을 보고 어떻게 설계할 것인지 구상 후 다이어그램을 그린다.

![image-20220629162406719](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629162406719.png)

## 협력관계를 바탕으로 구체적인 회원 클래스 다이어그램을 그린다.

![image-20220629163104289](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629163104289.png)

클래스들의 의존 관계들을 그려놓았다.

## 실제 사용할 객체 다이어 그램을 그려보자

![image-20220629163148950](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629163148950.png)

service는 한개만 구현할 것이니까 impl를 쓰고

repository는 우선 local memory 상에서 해결해보자

## 회원 entity를 코드로 구현해보자

![image-20220629165503585](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629165503585.png)

우선 member라는 패키지 안에 grade, member를 각각 enum, class로 만들어주었다.

```java
package hello.core.member;

public enum grade {
    BASIC,
    VIP
}
```

```java
package hello.core.member;

import java.lang.reflect.Member;

public class member {
    private Long id;
    private String name;
    private grade grade;
    
   	// 이후는 constructor, getter/setter 기능을 이용해 구현
```

## 회원 저장소 interface를 코드로 구현해보자

repository에는 service에게 정보를 **주고 받는** 기능을 구현하면 된다.

```java
package hello.core.member;

import java.lang.reflect.Member;

public interface memberRepository {

    void save(member member); // 저장

    member findById(Long memberId); // 내보내기
}
```

## 위를 바탕으로 실제 구현체를 코드로 구현해보자

```java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class memoryMemberRepository implements memberRepository {

    private static Map<Long, member> store = new HashMap<>();

    @Override
    public void save(member member) {
        store.put(member.getId(), member);
    }

    @Override
    public member findById(Long memberId) {
        return store.get(memberId);
    }
}
```

인터페이스 상속하고 메서드들을 자동으로 구현시켜주고

Map 자료구조를 사용해서 회원 정보를 저장하게 함

## 회원 서비스 인터페이스를 만들어보자

```java
package hello.core.member;

public interface memberService {

    void join(member member);

    member findMember(Long memberId);
}
```

회원가입과 회원조회 기능을 구현하였다.

## 위를 바탕으로 회원 서비스 구현체를 만들어보자

```java
package hello.core.member;

public class memberServiceImpl implements memberService {

    private final memberRepository memberRepository = new memoryMemberRepository();

    @Override
    public void join(member member) {
        memberRepository.save(member);
    }

    @Override
    public member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

최종적으로 6개의 파일이 만들어졌다.

![image-20220629174729198](../images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629174729198.png)

이제 이들을 각각 테스트 해보자

## 회원 도메인을 직접 테스트해보자

이때 우리는 순수 자바로만 개발한다고 했으므로

스프링 스타터를 사용하지 않고 실행하는 main 클래스를 만들것이다.

```java
package hello.core;

import hello.core.member.grade;
import hello.core.member.member;
import hello.core.member.memberService;
import hello.core.member.memberServiceImpl;

public class MemberApp {
    public static void main(String[] args) {
        memberService memberService = new memberServiceImpl();
        
		//memberService로 저장 전의 순수 회원정보
        member member = new member(1L, "memberA", grade.VIP); 
        
        // 회원가입
        memberService.join(member);
        
		//memberService로 저장한 후의 회원정보
        member findMember = memberService.findMember(1L); 
        System.out.println("newMember = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```

순수 member와 회원가입 절차를 거친 후 memberRepository에 저장된 member의 이름이 같은지 테스트하는 코드이다.

## Junit을 이용해서 테스트 해보자

```java
package hello.core.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class memberServiceTest {

    memberService memberService = new memberServiceImpl();
    @Test
    void join() {
        // given
        member member = new member(1L, "memberA", grade.VIP);

        // when
        memberService.join(member);
        member findMember = memberService.findMember(1L);

        // then
        Assertions.assertThat(member).isEqualTo(findMember);
    }
}
```

![image-20220629180952801](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629180952801.png)

테스트 후 초록색 체크모양이 뜨니 성공적으로 실행되었음을 알 수 있다.

## 한계점

OCP, DIP를 지키지 못했음을 알 수 있다.

![image-20220629181651131](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629181651131.png)

memberServiceImpl가 memberRepository 인터페이스를 의존하면서 동시에 memoryMemberRepository 구현체까지 의존하게 된다.

![image-20220629182344318](./images/2022-06-29-javaSpring_%EC%98%88%EC%A0%9C%EB%A7%8C%EB%93%A4%EA%B8%B02_%ED%9A%8C%EC%9B%90%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84%EB%B0%8F%EA%B0%9C%EB%B0%9C%EB%B0%8F%ED%85%8C%EC%8A%A4%ED%8A%B8/image-20220629182344318.png)

**추상화에도 의존하고 구체화에도 의존하게 되는 문제점 발생**

