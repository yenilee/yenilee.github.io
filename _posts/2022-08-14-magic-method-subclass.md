---
title: 매직 메서드 __init_subclass__ 의 동작
tags: python
---

<br/>
클래스 변수가 어디서 생기는 거지...? 🤔 <br/>
<!--more-->

---

회사 레거시 코드에서 `__init_subclass__` 를 처음 봤을 땐, init이 들어있는 것으로 보아 인스턴스가 생성될 때 같이 생성되는 프로퍼티일 것이라 짐작했다.
그러던 어느날 다른 팀원 분의 질문으로 해당 magic method를 정리하기까지 이르렀다.

문제가 된 부분은 클래스 변수가 빈 dict를 만드는데, 로직을 보면 해당 dict에서 여러 값들을 꺼내주고 있었고 그 어떤 로직에서도 해당 dict를 채워주는 곳이 없던 것이다.
일반적으로 클래스의 속성이나 변수는 인스턴스를 생성했을 때에 생성된다고 생각하기 때문에, 분명 로직 어딘가에 있을 것이라고 생각했다.
그런데 해당 변수를 아무리 검색해도 찾을 수 없다는 것을 알게됐고, 해당 class를 정의할 때 이미 클래스 변수가 채워진다는 결론에 이르렀다.

결론적으로 `__init_subclass__` 메서드는 부모 클래스를 상속받은 자식 클래스를 정의했을 때 호출된다는 것을 알게되었다.


```python
# 예시
class Parent:
    def __init_subclass__(cls):
        print('Subclass of Parent Created!')

class Child(Parent):
    pass

# 결과: 상속 받은 클래스는 정의만으로 init_subclass method가 호출됨. 'Subclass of Parent Created!' 가 print 되는 것

```

만약 자식 클래스에서 `__init_subclass__`가 재정의될 경우, 자식 클래스의 subclass hierarchy rule이 적용된다.
child -> parent 순으로 찾기 때문에, child에서 해당 메서드가 정의되어있을 경우 child의 내용으로 적용된다.


```python
# 예시
class Parent:
    def __init_subclass__(cls):
        print('AAA')

class Child(Parent):
    def __init_subclass__(cls):
        print('BBB')

# 결과: AAA가 print됨

class Grandchild(Child):
    pass

# 결과: BBB가 print됨

```

# 언제 사용?
버전 3.6에서 메타 클래스의 대안으로 소개된 것이라고 한다. [PEP 487](https://peps.python.org/pep-0487/)에는 서브 클래스를 등록하거나, 서브 클래스들의 default attribute를 세팅할 때 유용하다고 언급되어 있다.
실제 내부 코드에서도 부모 클래스를 상속하는 여러 자식 클래스가 있고, 해당 클래스가 공유하는 클래스 변수가 있을 경우에 사용되고 있다.

[강의](https://www.youtube.com/watch?v=hk85RUtQsBI&t=230s)에서 나온 예시는 아래와 같다.

```python

class PEP487:
    def __init_subclass_(cls, whom, **kwargs):
      super().__init_subclass__(**kwargs)
      cls.hello = lambda: print(f"Hello, {whom}")

class HelloWorld(PEP487, whom="World")
    pass

>>> HelloWorld.hello()

# Hello, World

```

## 메타 클래스
클래스를 만들어주는 클래스다. type을 사용해 동적으로 생성하거나, type을 상속받아 구현한다.

```python
# 동적 구현의 예
Parent = type('Parent', (), {'is_parent':True})
Child = type('Child', (Parent,), {'is_parent':False})

>>> Parent.is_parent # True
>>> Child.is_parent # False
```

