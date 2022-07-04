## IntelliJ에서 단축키가 어떤 것이 있는지 검색하는 기능이다

![image-20220704164818593](https://user-images.githubusercontent.com/105288887/177117212-6c8786fc-8d37-4855-a19f-dfb3e1938be1.png)

Setting에 들어가서 Keymap에 들어간 후 원하는 기능을 검색하면 오른쪽에 단축키 세팅이 나온다.

## Generate 단축키

Alt+Insert 누르면 Generate 창이 나온다.

![image-20220704164841248](https://user-images.githubusercontent.com/105288887/177117223-b4b395cb-4673-4967-ae22-a7ac3b72df0a.png)

- getter/setter : get,set 자동구현
- constructor : 생성자 자동구현
- toString : 출력할 때 좀 더 편하게 보기 위한 메서드

## 코드에 오류 발생했을 때 해결 단축키

Alt + Enter 누르면 어떻게 해결할 지 나온다.

* 인터페이스 상속받을 때 구현 메서드들 적을 때 유용
* 자동 import

## 코드 빌드, 실행 단축키

Ctrl + Shift + F10

## 테스트 코드 작성 단축키

테스트 하고싶은 메서드에다가

Ctrl + shift + T

![image-20220701162902867](https://user-images.githubusercontent.com/105288887/176997091-a376d399-e184-4af9-a9f2-2c31a5d66372.png)

Test Class를 만들어준다.

## 메서드가 어떤 경로에서 오는지 확인하는 단축키

경로를 알고싶은 메서드에다가 Ctrl 누른 상태로 클릭

## 리팩토링 단축키

**메서드 추출 리팩토링**

![image-20220702191108936](https://user-images.githubusercontent.com/105288887/176997094-1bb487c6-ca31-4405-976e-ef2ae43902ae.png)

추출하고 싶은 메서드를 드래그 해서

Ctrl + Alt + M

![image-20220702191236197](https://user-images.githubusercontent.com/105288887/176997099-5e302902-e68f-4b9d-9d21-d1a69ca9b24b.png)

리턴 타입과 이름을 정의한 후 리팩터링 하면 관련 메서드들 **한꺼번에 다 변경**

## 클래스 의존성 다이어그램 보는법

다이어그램을 보고자 하는 파일이나 패키지에 마우스 오른쪽 클릭 후 Show Diagram 클릭

![image-20220704165236650](https://user-images.githubusercontent.com/105288887/177117232-a3fa95c7-a5b8-43f0-b0fa-522e04d05de0.png)

그 후 자세한 디펜던시를 보고 싶을 땐 다시한번 오른쪽 클릭 후 Show Dependencies 클릭

![image-20220704165249826](https://user-images.githubusercontent.com/105288887/177117240-62b476da-27cb-4394-aa2f-c1be0791edb6.png)

이런식으로 볼 수 있게 된다.

![image-20220704165344234](https://user-images.githubusercontent.com/105288887/177117246-cbf19cdd-44d9-4f3a-abe8-3bc95a7fceed.png)
