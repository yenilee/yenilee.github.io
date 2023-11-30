---
title: Sanic 프레임워크 특징 & 비동기 개념
tags: sanic python async
---

<br/>
프레임워크와 관련된 개념에 대한 정리 <br/>
<!--more-->

---
# Summary

- `non-blocking` & `speedy` 프레임워크
- python asyncio 라이브러리를 기반으로 구현되어 있음
- 프레임워크이면서 `웹서버`임 (ASGI 지원)
- flask와 유사한 api를 제공, `가벼움`


# Related concept


## GIL로 인한 블로킹 이슈

- 파이썬은 global interpreter lock으로 인해 성능이 저하될 수 있음
  - 인터프리터에서 스레드간 일관성 유지하기 위해 한번에 하나의 스레드만 파이썬 코드만 실행하도록 강제하기 때문
- I/O 작업 중에는 블로킹 되지 않음 → 대기 시간을 소모하는 특성 때문
    - 멀티스레딩: 대기 시간 동안 다른 스레드 실행
    - 논블로킹(비동기) 프로그래밍: I/O 작업 중 대기하지 않고 다른 작업 수행
- CPU 작업 (CPU자원을 많이 사용하는 작업)은 블로킹 되어 성능 향상 어려움 → 멀티프로세스 사용

## ASGI
- Asynchronous 서버 게이트웨이 인터페이스: 비동기 웹프레임워크와 사용됨
- WSGI의 확장. python 앱은 보통 gunicorn 으로 멀티프로세스를 사용하는데,  sanic은 내장 asgi 서버가 있고 여러개의 워커를 실행함
    - WSGI : 웹서버 게이트웨이 인터페이스. 웹 서버와 웹 애플리케이션 통신을 위한 인터페이스
    - gunicorn은 wsgi 서버로, wsgi 애플리케이션을 실행하기 위한 서버
- ASGI 지원으로 논블로킹 I/O 가능
    - 이벤트 루프를 통한 비동기 이벤트 기반 프로그래밍

```python
from sanic import Sanic
from sanic.response import text

# 애플리케이션 생성
app = Sanic("MyHelloWorldApp")

# 라우트 정의
@app.route("/")
async def hello(request):
    return text("Hello, Sanic!")
```

- `sanic server.app` 실행 시
    - asgi 서버 시작 → 설정된 애플리케이션 로드 → 요청 처리 및 로그 출력

### ASGI 특징

- 실시간 웹소켓 통신
- 비동기 I/O: I/O 작업이 블로킹 되지 않고 다른 작업을 수행함
    - I/O 작업: 파일, 네트워크, 데이터베이스 등(파일 읽기/쓰기, DB 쿼리, 외부 api 호출 등)외부 자원과의 상호작용이 많은 작업
- 이벤트 기반 프로그래밍: 이벤트 루프를 통해 비동기 이벤트 기반 프로그래밍 지원
    - 이벤트 루프: 비동기 프로그래밍의 핵심 개념
        - 이벤트 큐에 등록된 이벤트를 순차적으로 처리함
        - 논블로킹 방식으로 동작함 (작업 완료까지 기다리지 않고 다른 작업 수행하며 이벤트 처리)
        - 콜백 함수를 통해 비동기 작업의 결과를 처리함

## 코루틴

- 중지되고 재개될 수 있는 함수. (진입점이 여러개, 중단 지점에서 재개될 수 있는 함수)
- async def 키워드를 사용해서 정의할 수 있음
- 비동기 함수 내에서 다른 비동기 함수를 호출할 때는 await 키워드 사용
    - `some_async_io_function` 이 완료될 때까지 기다린 뒤에 결과를 받아옴

    ```python
    @app.route('/async_io')
    async def async_io(request):
        result = await some_async_io_function()
        return json({'result': result})
    ```


# Basics

## 핸들러
- 비동기 코루틴 함수
- Request 인스턴스를 인수로 받는 모든 함수
- HTTP Response 혹은 코루틴을 반환
- 함수에 엔드포인트에 연결만 하면 끝!

```python
from sanic.response import text

@app.get("/foo")
async def foo_handler(request):
    return text("I said foo!")
```

## 미들웨어

- 웹 애플리케이션의 요청과 응답 사이에서 동작하는 컴포넌트
    - 인증, 로깅, 에러 처리, 캐싱, 데이터 가공 등
- 미들웨어를 register 하고 언제 수행될지만 정하면 됨
    - app.on_request
    - app.on_response
