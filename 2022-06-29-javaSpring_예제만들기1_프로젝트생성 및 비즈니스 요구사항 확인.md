## start.spring.io를 이용해서 프로젝트를 생성하자

![image-20220629143625141](https://user-images.githubusercontent.com/105288887/176652030-084e0561-cf84-49b8-849d-6d77a8a2a262.png)

gradle, spring 최신버전, java11, jar,

기타 다른 라이브러리 의존관계를 다운받지 않아서 core와 boot만 있는 것을 볼 수 있다.

![image-20220629200653179](https://user-images.githubusercontent.com/105288887/176652062-a30d204d-b46d-473d-b8e4-571ba1e21b41.png)

**스프링 없는 순수한 자바로만 개발을 진행한다는 점을 기억, 프로젝트 환경설정을 편리하게 하려고 스프링 부트 사용함.**

Spring을 gradle을 통해서 빌드하게 되면 좀 더 느리기 때문에 intelliJ로 바로 빌드하는 것이 좋다.

![image-20220629143043075](https://user-images.githubusercontent.com/105288887/176652070-968d0638-cf5f-4fa6-bcd5-73dba80229a1.png)

**build.gradle 파일 내부**

```java
plugins {
	id 'org.springframework.boot' version '2.7.1'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'hello' // project name
version = '0.0.1-SNAPSHOT' 
sourceCompatibility = '11' // java version

repositories {
	mavenCentral()
}

// 의존관계: 아무것도 추가하지 않음
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

## 비즈니스 요구사항을 확인해보자

![image-20220629144124202](https://user-images.githubusercontent.com/105288887/176652077-0244eeb1-9cb1-44af-96d9-9ab1c1bdf5f2.png)

미확정된, 추후 변동 가능한 항목들은 지금 개발하기 어려운 부분

**but**, **객체지향 설계**를 이용하면 충분히 만들 수 있다.

**역할과 구현**을 이용하자
