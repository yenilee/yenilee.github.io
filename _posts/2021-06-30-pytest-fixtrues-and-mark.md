---
title: Pytest | 테스트 코드 작성 기본
tags: python
---

<br/>
테스트 코드를 쓰다 보면 정상 케이스 외에도, 실패 케이스를 확인하거나 검증할 수 없는 케이스를 생략해야 하는 경우가 생긴다.<br/>
오늘 이 두 가지 케이스를 모두 접하게 되어 TIL로 간단히 정리하고자 한다.

<!--more-->

# 테스트란?
pytest 공식 문서는 테스트를 `특정 행동에 대한 결과`를 보고, `결과가 내가 기대한 것과 동일한지`를 확인하는 것이라고 말한다.

- Behavior은 특정 상황이나 자극에 대해 시스템이 반응하는 방식이다.
- 테스트를 쓰는 것이 어려운 것은, Behavior가 경험적으로 측정될 수 있는 것이 아니기 때문이다.
- Behavior은 어떻게 왜 되는지보다(how or why) 무엇이 되었는지(what is done)가 더 중요하다.

테스트는 아래 네 단계로 나눠볼 수 있다.

## 테스트의 단계

### Arrange
테스트를 위해 모든 것을 준비하는 단계다. act를 제외한 거의 모든 것이라고 할 수 있다. 도미노를 무너뜨리기 위해 블럭을 하나 씩 쌓는 것처럼,
존재하지 않는 유저의 인증을 만들거나 URL의 query를 만드는 등 모든 것을 준비하는 단계를 말한다.

### Act

우리가 테스트하고자 하는 Behavior를 시작하는 단계다. 이 Behavior가 테스트 아래에서 변경된 시스템의 상태를 가져오게 되고,
결과적으로 변경된 상태(resulting changed state)는 우리가 Behavoir에 대해 판단할 수 있는 근거가 된다. 보통 function이나 method call의 형식이다.

### Assert

결과적으로 변경된 상태를 보고 우리가 기대하는 상태와 일치하는 지를 보는 단계다. 테스트에서는 `assert`를 사용해서 측정한다.

### Cleanup

정해진 scope 내에서만 실행되고, 테스트 이후 스스로를 정리함으로써 다른 테스트에 영향이 가지 않게 하는 단계다.

# pytest
pytest 공식 문서의 큰 꼭지 중에서도 내가 자주 쓰는 Marks와 Fixtures를 정리해봤다.

## 기본 개념
### Fixtures

- 위에서 말한 arrange 단계에 필요한 데이터를 만든다. 테스트를 하기 위해 필요한 모든 것이라고 할 수 있다.
- 일정하고 반복 가능한 결과를 생성하고 안정적으로 실행되기 위해 정의된다.
- 테스트 함수나 다른 fixture에 argument로 사용된다.
- `@pytest.fixture` 데코레이터를 붙여서 사용한다.

```python
import pytest

class Fruit:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return self.name == other.name

@pytest.fixture
def my_fruit():
    return Fruit("apple")

@pytest.fixture
def fruit_basket(my_fruit):
    return [Fruit("banana"), my_fruit]

# 테스트하는 부분
def test_my_fruit_in_basket(my_fruit, fruit_basket):
    assert my_fruit in fruit_basket # fruit_basket에 my_fruit이 있는지 확인하기 위해 fixture를 만든 것.
```

### Marks
- fixtures나 plugins를 통해 접근할 수 있는 테스트 함수에 `메타 데이터를 적용`하는 데에 사용될 수 있다.
- 말 그대로 해당 테스트를 스킵할 것이다, 실패할 것이다에 대해서 표시를 해두는 것이라고 생각하면 된다.
- 예시는 아래 skip and xfail에서 다뤄볼 것이다.

## 테스트 실패를 처리하는 방법
skip이나 xfail은 `성공할 수 없는 테스트를 다루는 방식`으로, 특정 플랫폼에서 실행될 수 없거나 실패를 기대하는 테스트 함수에 표시해를 해서 테스트 요약에 나타낸다.

- skip: 특정 조건에 맞아야 성공하고, 그렇지 않으면 테스트도 스킵하는 것이다. 일반적인 예는 윈도우가 아닌 플랫폼에서 윈도우 only test를 skip하거나, 테스트하는 순간 가용이 불가한 외부 자원에 따라 skip하는 경우다.
- xfail: 테스트가 실패할 것으로 기대하는 것을 의미한다. 아직 구현되지 않은 기능을 테스트하거나, 수정되지 않은 버그로 인해 실패 결과를 기대하는 것 등이다. 실패를 기대했는데 만약 패스한다면, test summary에 xpass로 리포팅되게 된다.

### 1. pytest.mark.skip
조건 없이 테스트를 skip하도록 해준다.

```python
@pytest.mark.skip(reason="no way of currently testing this")
def test_the_unknown():
    ...
```

### 2. pytest.mark.xfail

공식 문서에 나온 설명은 다음과 같다. 직역해보면 test가 fail할 것이라고 가리키는 것이고, 실패하더라도 traceback을 리포팅하지 않고 문제 없이 패스하게 된다. 터미널에서는 XFAIL(expected to fail)과 XPASS(unexpectedly passing) 섹션에서 결과를 리스팅한다.

>You can use the xfail marker to indicate that you expect a test to fail.
>This test will run but no traceback will be reported when it fails.
>Instead, terminal reporting will list it in the “expected to fail” (XFAIL) or “unexpectedly passing” (XPASS) sections.
>Alternatively, you can also mark a test as XFAIL from within the test or its setup function imperatively

### 3. pytest.raises

예외 발생을 assert해준다. 위 케이스는 실패 케이스를 예상하고 무시해주는 기능인 반면, pytest.raises는 내가 코드에서 고의적으로 낸 예외 케이스에 대해서 잡아줄 때 사용하는 방법이다.
고의적 발생이라고 하면 이해가 안 갈 수도 있는데, 예를 들어 특정 값을 나눌 때 0으로 나누는 케이스가 생기는 것을 방지하기 위해 아래 코드처럼 ZeroDivisionError를 발생시키도록 코드를 짰다면,
테스트 케이스에서 해당 에러가 잘 발생하는지를 체크하는 용도로 사용하는 것이다.

반면 xfail은 디펜던시에 의한 버그나, 고쳐지지 않은 버그에 대한 문서화의 목적으로 사용하는 것으로 보는 것이 좋다.

```python
import pytest
with pytest.raises(ZeroDivisionError):
    1/0
```

skip이나 xfail을 mark할 경우, 무시하고 지나가면 안 되는 상황인데 테스트에서 pass해 버그를 놓치는 경우가 생길 수 있다.
때문에 반드시 문서에 나온 목적대로 사용하지 않는다면 유의해서 사용하는 것이 좋을 것 같다.
