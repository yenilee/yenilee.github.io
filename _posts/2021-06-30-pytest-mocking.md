---
title: pytest 로직 일부분을 mocking하는 방법 🕶
tags: python
---

<br/>
외부 서비스 호출 등 테스트 시에 실제 실행되면 안 되는 부분을 mocking하는 방법을 간단히 정리!

<!--more-->

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

---
