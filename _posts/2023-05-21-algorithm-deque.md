---
title: python queue 사용 방법, 시간 복잡도
tags: algorithm python
---

<br/>
프로그래머스 스택/큐 관련 문제를 풀다가 정리 <br/>
<!--more-->

---

# 정의
- queue는 선입 선출(First In First Out)이며 양방향으로 데이터 추가 제거가 가능하다. (stack은 후입 선출 Last In First Out, 단방향)
- 구현 사례는 비동기 메시징(dramatiq 등), 알고리즘의 경우 너비 우선 탐색 등에서 활용된다.

# 구현 방법
1. list 구현
   - 구현은 가능하지만 list 첫 번째 값을 추가하거나 제거할 때 O(n)의 시간 복잡도 발생
   - 무작위 접근 또한 O(n)
    ```python
   queue = [1,2,3,4]
   queue.append(5) # 맨 뒤에 값 추가
   queue.insert(0, 2) # 첫 번째에 값 추가
   queue.pop() # 맨 뒤의 값 제거
   queue.pop(0) # 맨 앞의 값 제거
    ```

2. deque 구현
   - Double-Ended queue 의 약자
   - list 첫 번째 값을 추가, 제거 시 O(1)으로 성능상의 이점 가지며, thread safe하다.
   - 무작위 접근은 마찬가지로 O(n)
     - 내부적으로 linked list를 사용하기 때문에, 특정 인덱스 값에 도달하려면 길이만큼 순회를 해야 함.
    ```python
    from collections import deque
    queue = deque([1,2,3,4])
    queue.append(5) # 맨 뒤에 값 추가
    queue.appendleft(2) # 첫 번째에 값 추가
    queue.pop() # 맨 뒤의 값 제거
    queue.popleft() # 맨 앞의 값 제거
     ```

3. Queue 구현
   - 방향성 없음, 인덱스 접근 불가
    ```python
    from queue import Queue
    queue = Queue()
    queue.put(1) # 값 추가
    queue.put(2) # 값 추가
    queue.get() # 1 (앞에서부터 제거)
    queue.get() # 2
     ```
# 공식 문서
위 설명에서 언급된 시간 복잡도는 [링크]()의 공식 문서에서 확인이 가능하다.

> Deques are a generalization of stacks and queues (the name is pronounced “deck” and is short for “double-ended queue”).
>
> Deques support thread-safe, memory efficient appends and pops from either side of the deque with approximately the same O(1) performance in either direction.
>
> Though list objects support similar operations, they are optimized for fast fixed-length operations and incur O(n) memory movement costs for pop(0) and insert(0, v) operations which change both the size and position of the underlying data representation.

# 문제 예시
프로그래머스 올바른 괄호 문제. [링크](https://school.programmers.co.kr/learn/courses/30/lessons/12909) 괄호가 정해진 순서대로 나열되어 있어야 True를 반환함
- ")" 으로 시작하면 처음부터 열리지가 않았기 때문에 바로 False를 리턴함
- "("으로 시작할 경우 queue에 값을 넣어줌
- 첫 번쨰 값이 "("이고 들어오는 괄호가 ")"일 경우 queue에서 제거 (짝지어져 있기 때문에)
- 주어진 값을 모두 순회했을 때 값이 남아있다면 제대로 닫히지 않은 것이기 때문에 False 리턴

```python
from collections import deque

def solution(s):
    if s[0] == ")":
        return False

    queue = deque()
    for bracket in s:
        if bracket == "(":
            queue.append(bracket)
        else:
            if queue and queue[0] == "(":
                queue.popleft()
    return not bool(queue)
```
