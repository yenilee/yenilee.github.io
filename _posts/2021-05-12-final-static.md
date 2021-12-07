---
title: 객체지향 | 다형성
tags: java 객체지향
---

<br/>
[객체지향 강의](http://www.kocw.net/home/cview.do?cid=42e5d809846bebec) 정리✍🏻 8강 수식자 <br/>
<!--more-->


# final과 static 수식자

## final

1. field: field 값 변경 불가능 (상수)
2. method: overrideing으로 확장 불가 (하위 클래스에서 재정의 불가)
3. class: 상속 불가능



## static

1. field, method 앞에 붙여서 사용한다

2. field: static이 붙으면 클래스를 상속한 모든 객체들이 공유하는 값이 된다.

   동일한 메모리를 가리키고 있기 때문인데, 메모리 효율보다는 공유를 목적으로 많이 사용된다.

3. method: 객체를 생성할 필요가 없이 바로 class.method로 접근 가능하다.

   static method 안에서는 static 변수만 사용할 수 있다.

```java
public class HousePark  {
    static String lastname = "박";

    public static void main(String[] args) {
        HousePark pey = new HousePark();
        HousePark pes = new HousePark();
    }
}
```



## singleton pattern

단 하나의 객체만 생성하도록 강제하는 패턴. 클래스를 통해 생성할 수 있는 객체(인스턴스)가 단 하나인 것이다.

```java
class Singleton {
    private static Singleton one;
    private Singleton() {
    }

    public static Singleton getInstance() {
        if(one==null) {
            one = new Singleton();
        }
        return one;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



---
