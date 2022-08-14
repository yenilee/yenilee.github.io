---
title: 매직 메서드 `__init_subclass__` 의 동작
tags: python
---

<br/>
클래스 변수가 어디서 생기는 거여...? <br/>
<!--more-->

회사 레거시 코드에서 `__init_subclass__` 를 처음 봤을 땐, init이 들어있는 것으로 보아 인스턴스가 생성될 때 같이 생성될 것이라고 지레짐작했던 것 같다.
그러던 어느날 다른 팀원 분의 질문으로 해당 magic method를 정리하기까지 이르렀다.

문제는 클래스 변수가 빈 dict를 만드는데, 그 어떤 로직에서도 해당 dict를 채워주는 곳이 없던 것이다.
일반적으로 클래스의 속성이나 변수는 인스턴스를 생성했을 때에 생성된다고 생각하기 때문에, 분명 로직 어딘가에 있을 것이라고 생각했다.
그런데 해당 변수를 아무리 검색해도 찾을 수 없다는 것을 알게됐고, 뭔진 모르지만 해당 class를 정의할 때 이미 클래스 변수가 채워진다는 결론에 이르렀다.

결과적으로 찾아봤을 때 `__init_subclass__` 메서드는 부모 클래스를 상속받은 자식 클래스를 정의했을 때 호출되는 것을 알 수 있다.


```python
# 예시
class Parent:
    def __init_subclass__(cls):
        print('Subclass of Parent Created!')

class Child(Parent):
    pass

# 결과: 클래스 정의만으로 Subclass of Parent Created! 가 print 됨.

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
