---
title: 객체지향 | 다형성
tags: java 객체지향
---

<br/>
[객체지향 강의](http://www.kocw.net/home/cview.do?cid=42e5d809846bebec) 정리✍🏻 11강 다형성 <br/>
<!--more-->



# 다형성 (polymorphism)

- 여러가지 형태를 가지는 것
- 하나의 메서드나 클래스가 다양한 방법으로 동작하는 것 (overriding, overloading)
- 객체의 타입이 다르면 똑같은 메시지에도 다른 동작을 하는 것
  - 메시지 전달: 메시지 호출 (method invocation)
  - `Animal  abstract sound(), Cat sound(), Dog sound()`
- 하나의 클래스의 객체로 다른 클래스의 객체를 참조할 수 있는 특성
- 슈퍼 클래스는 서브 클래스의 생성자로 생성이 가능하나, 서브클래스는 슈퍼 클래스의 생성자로 생성될 수 없다.



## 상향 형변환(Cast)

- 서브 클래스의 객체는 슈퍼 클래스 객체처럼 취급 가능
- super class 참조 변수로 sub class 객체를 참조할 수 있음 (아래 2번 예제)
- 대입은 할 수 있으나 super class의 속성까지 가질 수는 없다.

```java
class Shape {}
class Ractangle extends Shape {}
class ShapeMain {
			public static void main(String[] args) {
						Shape s;
						Rectangle r;
						s = new Shape(); // (1) ok Shape s = new Shape()
						s = new Rectangle(); // (2)ok Rectangle 생성자를 통해 객체를 만듦
						r = new Shape(); // (3) Error Shape 생성자를 통해 객체를 만듦
			}
}
```



## Casting 예제

```java
abstract class Shape{
			int x, y;
}
class Rectangle extends Shape{
			int z;
}
class RectMain {
			public static void main(String args[]) {
						Shape s = new Shape; // Error. abstract class 객체 생성 불가
						Rectangle r = new Rectangle();
						s = r; // ok. 자기 자신의 객체만 못 받지 sub class의 객체는 대입 가능
        		s.x=5;
        		s.y=10;
        		s.z=15; // Error. Shape 클래스 s에 Rectangle 객체를 대입한 것이기 때문에 Shape은 x,y만 가지므로 불가한거임

        		r=(Rectangle) s; // 캐스팅: Shape을 Rectangle로 형변환 (type이 호환되지 않는 문제 해결. 하위 객체로 캐스팅)
        		System.out.println(r.z);
			}
}

```



## 객체 다형성

- 상속 받은 클래스의 객체는 Super class와 하위 클래스(this)로 모두 취급 가능
- sub class의 객체는 super class의 객체로 생성 가능하다
- super class로 선언하면 하위 모든 클래스의 객체를 사용할 수 있다. (파라미터를 super class로 넣을 경우 모든 sub class를 인자로 넣을 수 있음)

```java
X a = new Y();
// X: Super class
// Y: Sub class
```



```java
abstract class Calc {
			int x,y;
			abstract int result();
			void printResult() {
						System.out.println(result());
			}
			void setData(int a, int b) {
						x=a,
						y=b
			}
}
class Plus extends Calc {
			int result() {
						return(x+y);
			}
}
class Minus extends Calc {
			int result() {
						return(x-y);
			}
}
class CalcMain {
			public static void main(String args[]) {
						int a=50, b=20;
						Calc c1 = Plus();
						Calc c2 = Minus();
						c1.setData(a, b)
						c2.setData(a, b);
						c1.printResult(); // 70
						c2.printResult(); // 30
			}
}
```



# Object 클래스

- java에 존재하는 모든 클래스의 super class
- 참조 변수를 사용할 때 무슨 클래스를 넣을지를 모른다면 object class를 넣으면 됨





# 예외 처리

1. 자바는 문법적 오류(구문)만 캐치하기 때문에 내용 상의 문제(배열의 인덱스 같은)는 잡아내지 않아 문제 없이 컴파일 된다.
2. 내용 상의 문제는 실행할 때 발생하게 된다.
3. 예외: 프로그램 실행 시 발생하는 이상 상태 -> `Exception class`에서 객체가 생성된다. -> 프로그램 비정상 종료
4. 예외 처리 영역을 설정하는 것에 따라 결과가 달라질 수 있다.



```java
try {
			// 예외 발생 가능한 명령어
} catch (Exception e) {
			// 예외를 처리할 Exception 객체를 매개 변수로 받는다
			// 예외 발생 시 수행할 명령어를 넣는다
} finally {
			// 예외 발생 유무와 상관 없이 실행된다
  		// finally 구문은 생략 가능
}
```



```java
class ErrorMain {
			public static void main(String args[]) {
        		try{
            			sayAgain(args[0]);
            } catch(Exception e) {
              		System.out.pringln("예외발생")
            }
			}
			static void sayAgain(String s) throws Exception {
						System.out.println(s); // throws Exception이 있으면 해당 함수는 try catch 구문 안에서 사용되어야 함
			}
}
```


---
