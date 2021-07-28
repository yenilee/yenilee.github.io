---
title: pytest & mock의 기본 개념 (테스트란?🤔)
tags: python
---

<br/>
테스트 코드를 쓰다 보면 정상 케이스 외에도, 실패 케이스를 확인하거나 검증할 수 없는 케이스를 생략해야 하는 경우가 생긴다.<br/>
오늘 이 두 가지 케이스를 모두 접하게 되어 TIL로 간단히 정리하고자 한다.

<!--more-->

# 테스트란?
[pytest 공식 문서](https://docs.pytest.org/en/6.2.x/fixture.html#fixture)는
테스트를 `특정 행동에 대한 결과`를 보고, `결과가 내가 기대한 것과 동일한지`를 확인하는 것이라고 말한다.

테스트 코드를 쓰는 것이 힘든 이유는 행동(이하 Behavior)은 측정될 수 있는 것이 아니기 때문이다. Behavior는 어떻게 왜 되는지보다 무엇이 되었는지(what is done)가 더 중요하다.  테스트는 아래 네 단계로 나눠볼 수 있다.

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

def test_my_fruit_in_basket(my_fruit, fruit_basket):
    assert my_fruit in fruit_basket # fruit_basket에 my_fruit이 있는지 확인하기 위해 fixture를 만든 것.
```

### Marks
- fixtures나 plugins를 통해 접근할 수 있는 테스트 함수에 `메타 데이터를 적용`하는 데에 사용될 수 있다.
- 말 그대로 해당 테스트를 스킵할 것이다, 실패할 것이다에 대해서 표시를 해두는 것이라고 생각하면 된다.
- 예시는 아래 skip and xfail에서 다뤄볼 것이다.

## Skip and xfail

fixture를 사용해서 정상 테스트를 하는 것은 많이 다뤄봤는데, 실패 케이스에 대해서는 명확하게 공부한 적이 없어서 이 부분을 더 집중해서 다뤄보고자 한다.

skip이나 xfail은 `성공할 수 없는 테스트를 다루는 방식`으로, 특정 플랫폼에서 실행될 수 없거나 실패를 기대하는 테스트 함수에 표시해를 해서 테스트 요약에 나타낸다.

- skip: 특정 조건에 맞아야 성공하고, 그렇지 않으면 테스트도 스킵하는 것이다. 일반적인 예는 윈도우가 아닌 플랫폼에서 윈도우 only test를 skip하는 것이다.
- xfail: 특정 이유로 테스트가 실패하는 것을 기대하는 것을 의미한다. 기능을 위한 테스트가 아직 실행되지 않았거나, 버그가 수정되지 않은 것 등이다.  실패를 기대했는데도 만약 패스한다면, test summary에 xpass로 리포팅되게 된다.

### pytest.mark.skip
조건 없이 테스트를 skip하도록 해준다.

```python
@pytest.mark.skip(reason="no way of currently testing this")
def test_the_unknown():
    ...
```

### pytest.mark.xfail

[공식 문서](https://docs.pytest.org/en/6.2.x/skipping.html#xfail-mark-test-functions-as-expected-to-fail)에 나온 설명은 다음과 같다. 직역해보면
test가 fail할 것이라고 가리키는 것이고, 실패하더라도 traceback을 리포팅하지 않고 문제 없이 패스하게 된다. 터미널에서는 XFAIL(expected to fail)과 XPASS(unexpectedly passing) 섹션에서 결과를 리스팅한다.

>You can use the xfail marker to indicate that you expect a test to fail.
>This test will run but no traceback will be reported when it fails.
>Instead, terminal reporting will list it in the “expected to fail” (XFAIL) or “unexpectedly passing” (XPASS) sections.
>Alternatively, you can also mark a test as XFAIL from within the test or its setup function imperatively

#### raises

해당 테스트가 실패한 이유에 대해 더 자세히 명시하고 싶을 경우, raises 인자에 exception class를 넣는다. (아래 코드에 예시)
exception은 Exception의 subclass여야 한다. 만약 raises에서 언급한 exception이 아닌 에러가 리포팅될 경우 일반 실패 케이스로 처리된다.

#### 주의할 점

내가 이 블로그를 쓰게된 계기인데, `@pytest.mark.xfail` 데코레이터는 만약 내가 기대하는 에러가 발생하지 않더라도 테스트에 초록불이 들어온다.
RunTimeError가 발생할 것으로 기대하는 상황인데 에러가 발생하지 않더라도 문제 없이 통과된다는 의미다.

```python
import pytest

@pytest.mark.xfail(raises=RuntimeError)
def test_function():
    pass
```

정상적으로 에러가 발생하는 케이스라면 테스트를 실행했을 때 해당 테스트에 traceback이 찍혀있어야 하는데, 아무 로그도 없는 것을 보고 당황했다.
동작 방식을 잘 모른다면 pass를 했기 때문에 내가 원하는 대로 테스트되었다고 착각할 수 있을 것 같다는 생각이 들었고, 다음 방식으로 사용하는게 더 좋을 것 같다.

#### 개선점

- 에러 검증을 위해 테스트 함수 내옹을`pass` 보다는 `assert False`를 통해 테스트 통과 여부를 명시적으로 알 수 있도록 해야 한다.
- pytest.mark.xfail은 fail할 것이라고 Mark만 하는 것이기 때문에, 에러가 나지 않아도 통과된다는 점을 인지해야 한다.

```python
import pytest

@pytest.mark.xfail(raises=RuntimeError)
def test_function():
    assert False

    # 검증하려는 로직에서 중간에 에러가 발생하면 assert하는 곳까지 오지 않기 떄문에,
    # False 로 테스트가 실패하게 되면 해당 테스트 함수의 정상 행동인 에러가 발생되지 않았음을 뜻한다.
```

# mock

테스트로 구현할 수 없는 외부 서비스 요청이라든지, fixture로 만들 수 없는 조건이 있을 때 mocking을 하게 된다.
예를 들어 비동기 서비스에 요청을 해서 결과 값을 받아온다고 한다면, 해당 부분을 mocking함으로써 테스트를 만들어볼 수 있다.

## patch

patch 함수 소스코드를 보면 다음과 같이 되어있다.

```python
def patch(
        target, new=DEFAULT, spec=None, create=False,
        spec_set=None, autospec=None, new_callable=None, **kwargs
    ):
```

- 쉽게 말해 테스트 시 검증이 불가한 부분만 변경해주는 것이다.
- 함수 혹은 클래스 데코레이터나 컨텍스트 매니저로 사용될 수 있다.
- 첫 번째 인자가 target인데, function body나 with statement 내에서 target은 new object와 함께 patch된다.
- patch는 프로그래밍에서 부분적 변경을 의미한다. mocking을 통해 부분적으로 변경해서 테스트를 진행하는 것이라고 이해하면 좋을 것 같다.

### new
아래 함수는 http 요청이 필요한 async_request_service 대신, 해당 로직을 담고 있는 함수인 request_function을 new 인자에 넣어 테스트를 하는 예시다.
mock 부분만 보여주기 위한 예시로, new에 대신 실행을 원하는 함수명을 입력한다.

```python
from unittest import mock

@pytest.fitxture
def mock_async_service(self):
  with mock.patch('async_service.request', new='request_function'):
    # target은 new object로 대체되어야 하기 떄문에, improt할 수 있도록 string.string.string 형식으로 입력해야 한다.
    yield
```

만약 해당 함수를 대신해 다른 함수도 실행하지 않고 그냥 패스하려면, 아래처럼 사용할 수도 있다.

```python
from unittest import mock

def dummy(*args, **kwargs):
    pass

@pytest.fitxture
def mock_async_service(self):
  with mock.patch('async_service.request', new=dummy):
    yield
```

### return_value
만약 new에 함수를 입력하지 않는다면, target은 Mock Object로 대체된다. 이 때 Mock이 가지는 atrribute 중 하나가 return_value다.
내가 검증하고자 하는 로직 중 생략하는 부분이 반환해야 할 값이 있다면, 해당 반환 값을 통해서 검증을 이어나가는 용도로 사용할 수 있다.

>If new is omitted, then the target is replaced with an
    AsyncMock if the patched object is an async function or a
    MagicMock otherwise. If patch is used as a decorator and new is
    omitted, the created mock is passed in as an extra argument to the
    decorated function. If patch is used as a context manager the created
    mock is returned by the context manager.


이렇게 길게 쓰려고 했던 포스트는 아니었는데, 사용하면서도 몰랐던 부분이 있었어서 자세히 다루게 되었다.
잘 쓰여진 테스트 코드를 직접 보고 배우고 싶다.

---
