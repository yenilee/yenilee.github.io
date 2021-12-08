---
title: 객체지향 | 인터페이스
tags: java 객체지향
---

<br/>
[객체지향 강의](http://www.kocw.net/home/cview.do?cid=42e5d809846bebec) 정리✍🏻 10강 인터페이스 <br/>
<!--more-->


# 인터페이스

- 클래스 간의 상호 작용. 객체와 객체 사이의 상호작용
- 내부에 무엇이 있는지 몰라도 사용 방법(명령)만 알면 쓸 수 있음 예시) USB 인터페이스, 전원 버튼
- 클래스, 인터페이스를 컴파일한 파일의 확장자는 A.class가 됨
- 상속 가능 `extends`, 다중 상속 지원(여러 부모를 가질 수 있다. 단 `interface`에 한해서)
- 사용 순서는 extends, interface 순



## 1. 장점

- 관련성이 떨어지는 클래스를 상속 없이 연결해서 사용 가능
- 하나 이상의 클래스를 통해서 구현해야 하는 메서드 선언 가능
- 클래스 알지 못해도 인터페이스를 통해 접근 가능



## 2. java 제공 인터페이스

- ```public static final``` 로 정의된 변수(클래스 상수)만 사용된다. (생략 가능)
- 인터페이스에는 추상 메서드 `abstract` 키워드)만 들어가야 한다. (생략 가능)



## 3. 구현

- 클래스에서 인터페이스를 이용할 수 있도록 하는 것
- 키워드 Implements를 사용 -> 상속
- 반드시 메서드를 ```public```으로 overriding해야 한다. (부모보다 넓은, 최소 동일한 접근 제한자를 사용해야 한다)

```java
interface A {
			public static final int a = 2;
			public abstract void say();
}
class B implements A {
			public void say() {
						System.out.println("Hello");
			}
}
```



## 3. 예시

```java
interface Greet {
			abctract void greet();
}
interface Talk {
			abstract void talk();
}
class Morning implements Greet, Talk {
			public void greet(){
						System.out.println("안녕")
			}
			public void talk(){
						System.out.println("날씨가 춥지?")
			}
}

class GreetMain {
			public static void main(String args[]) {
						Morning = new Morning();
						m.greet();
						m.talk();
			}
}

```



## 4. 인터페이스 vs 추상메서드

### 공통점

- 추상 메서드 보유



### 차이점

- 추상 클래스는 부분적으로 미완성(abstract인 것과 아닌 것도 있음)이나, 인터페이스는 완전한 미완성
- 인터페이스는 외부 메서드명을 명확하게 하고, 여러 클래스에서 사용되는 상수를 정의
- 추상클래스는 일부 메서드를 하위 클래스 구현에 위임하고, 클래스의 객체 생성을 막는데 사용 (다형성 구현)
- 인터페이스는 다중 상속, 다중 구현만 가능하나 추상클래스는 다중 상속이 되지 않음 (하나의 부모)



![image-20210520221635206](/Users/avery/Library/Application Support/typora-user-images/image-20210520221635206.png)
---
