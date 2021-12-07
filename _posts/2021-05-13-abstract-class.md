---
title: 객체지향 | 추상 클래스
tags: java 객체지향
---

<br/>
[객체지향 강의](http://www.kocw.net/home/cview.do?cid=42e5d809846bebec) 정리✍🏻 9강 추상클래스, 추상 메서드 <br/>
<!--more-->


# 추상클래스(abstract class)

- 추상 메서드를 포함하는 클래스 (일반 클래스도 포함)
- 키워드 abstract 사용
- 객체 생성 불가능 (AIAssistModel!)
- 추상 메서드가 계속 존재하면 추상 클래스가 되기 때문에 override 해주어야 객체 생성할 수 있는 일반 클래스가 됨



# 추상 메서드

- 추상 메서드 정의는 추상 클래스에서만 가능하다.
- 구현 코드가 없고 선언만 된다.
- 반드시 하위 클래스에서 구현(구체화)되어야 한다.
- 키워드 abstract로 지정 (변수 선언에는 abstract 사용할 수 없다. 클래스, 메서드만 가능)



### 일반 클래스

```java
class Animal {
	String name;
	void cry() {
    // 이 집합 괄호 {}를 빼면 body가 없이 구현되지 않은 추상 메서드가 됨
  };
}
```

### 추상 클래스

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



---
