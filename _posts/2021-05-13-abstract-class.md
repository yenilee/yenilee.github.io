---
title: 객체지향 | 추상 클래스와 인터페이스
tags: java 객체지향
---

<br/>
객체지향 강의 정리✍🏻 9-10강 추상클래스, 인터페이스 <br/>
<!--more-->


# 추상클래스

공통적인 변수나 메서드를 기반으로 선언하는 클래스이며, 객체 생성이 불가능하고 상속해서 사용할 수 있음
- 키워드 abstract 사용
- 추상 메서드를 포함하는 클래스
- 추상 메서드를 override 해주어야 객체 생성할 수 있는 일반 클래스가 됨


# 추상 메서드

- 추상 메서드 정의는 추상 클래스에서만 가능
- 구현 코드가 없고 선언만 된다.
- 반드시 하위 클래스에서 구현(구체화)되어야 함
- 키워드 abstract로 지정
  - 변수 선언에는 abstract 사용할 수 없다. 클래스, 메서드만 가능



## 일반 클래스

```java
class Animal {
	String name;
	void cry() {
    // 이 집합 괄호 {}를 빼면 body가 없이 구현되지 않은 추상 메서드가 됨
  };
}
```

## 추상 클래스

```java
abstract class Animal {
	String name;
	abstract void cry(); //
}
```



### 예제

```java
abstract class Calculation {
			int a,b;
			void setData(int w, int h) {
						a=w;
						b=h;
			}
			abstract void answer();
}

class Plus extends Calculation {
			void answer() {
			System.out.printIn(a+b);
			} // 이 클래스에 abstract keyworkd 붙여도 에러 발생하지 않음. 추상 메서드 없기 때문에
}

class CalMain {
			public static void main(String args[]) {
        		Plus p = new Plus();
        		p.setData(100, 5);
        		p.answer();
      }
}
```


# 인터페이스
상속을 통해 동일한 기능을 강제하는 역할을 함

## 의미
- 클래스 간의 상호 작용. 객체와 객체 사이의 상호작용을 말한다.
- 내부에 무엇이 있는지 몰라도 사용 방법(명령)만 알면 쓸 수 있다. 예시) USB 인터페이스, 전원 버튼

## 구현
- 클래스, 인터페이스를 컴파일한 파일의 확장자는 A.class가 됨
- 상속 가능 `extends`, 다중 상속 지원(여러 부모를 가질 수 있음, 단 `interface`에 한해서)
- 사용 순서는 extends, interface 순
- 키워드 Implements를 사용 -> 상속
- 반드시 메서드를 ```public```으로 overriding (부모보다 넓은, 최소 동일한 접근 제한자를 사용해야 함)


### java 제공 인터페이스

- ```public static final``` 로 정의된 변수(클래스 상수)만 사용 (생략 가능)
- 인터페이스에는 추상 메서드 (`abstract` 키워드)만 들어감 (생략 가능)


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

### 예제

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

## 장점

- 관련성이 떨어지는 클래스를 상속 없이 연결해서 사용 가능
- 하나 이상의 클래스를 통해서 구현해야 하는 메서드 선언 가능
- 클래스 알지 못해도 인터페이스를 통해 접근 가능



# 인터페이스 vs 추상메서드

## 공통점

- 추상 메서드 보유


## 차이점

- 추상 클래스는 부분적으로 미완성(abstract인 것과 아닌 것도 있음)이나, 인터페이스는 완전한 미완성
- 인터페이스는 외부 메서드명을 명확하게 하고, 여러 클래스에서 사용되는 상수를 정의
- 추상클래스는 일부 메서드를 하위 클래스 구현에 위임하고, 클래스의 객체 생성을 막는데 사용 (다형성 구현)
- 인터페이스는 다중 상속, 다중 구현이 가능하나 추상클래스는 다중 상속이 되지 않음 (하나의 부모)



---
