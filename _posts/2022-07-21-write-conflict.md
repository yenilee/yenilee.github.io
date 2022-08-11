---
title: 트랜잭션과 Write conflict
tags: mongodb
---

<br/>
Write conflict를 겪게 되면서 정리한 트랜잭션, 충돌에 대한 이야기<br/>
<!--more-->

# 트랜잭션
트랜잭션은 쪼갤 수 없는 업무 처리의 최소 단위이며, 정상 처리가 될 경우 커밋 & 오류가 발생하면서 롤백하게 된다.
DB에 여러 클라이언트가 동시에 조회 또는 갱신 시 발생할 수 있는 데이터 부정합을 방지하기 위해 사용된다.

## 트랜잭션의 조건
트랜잭션이 안전히 수행되기 위해서는 ACID 조건을 충족해야 한다.
- Atomicity 원자성: all or nothing
    - rollback segment: 이전 데이터를 임시로 저장해 원자성을 보장해줌
    - save point: 작업이 확실히 수행되었다고 여겨지는 부분을 지정해, 롤백 시 해당 지점부터 수행
- Consistency 일관성: transaction 전후 상태는 같아야 함
- Isolation 고립성: 트랜잭션 간에는 간섭할 수 없다
- Durability 지속성: 트랜잭션 종료 후 작업 결과는 영구 저장됨

# 스케줄
하나의 트랜잭션이 끝나면 그 다음 트랜잭션을 수행하는 직렬 스케줄(Serial schedule)은 일관성을 높일 수는 있지만 성능은만떨어지기 때문에, 직렬화 스케줄(Serializable schedule)을 통해 병행 처리하면서 적절한 제어 조치를 해 일관성은 유지하면서 성능은 향상시킬 수 있다.
다만 병행 처리는 첫 번째 갱신이 유실되거나, 낡은 자료를 읽는 등의 문제가 발생할 수 있기 때문에 자료를 배타적으로 접근해 직렬성을 확보한다.
- 직렬성: 트랜잭션의 처리 결과가 직렬 처리와 동일한 효과를 가지는 스케줄의 특성
- Locking: 자료를 배타적으로 접근하는 방법이다.
  - Shared Lock: 갱신 불가, 읽혀질 수 있음
  - Exclusive Lock: 갱신 불가, 읽기도 불가 (Dirty Read 방지)

## MVCC
Multi Version Concurrency Control, 트랜잭션의 병렬도가 높아지면 Dirty Read가 생길 수 있기 때문에, 이를 방지하기 위한 동시성 제어 방식이다.
위 원자성을 보존해주는 rollback segment에 이전 데이터를 저장해놓음으로써 읽기의 일관성을 유지시켜주는 것으로 볼 수 있다.


# MongoDB에서의 트랜잭션
공식 문서에 따르면 single document에 대한 operation은 atomic하다. 예를 들어 하나의 document에서 여러 field를 수정할 때 all or nothing이라는 것.
그리고 버전 4.0부터 레플리카 셋 위에서의 multi-document 트랜잭션을, 4.2부터는 샤딩된 클러스터에서의 트랜잭션을 지원하는 분산 트랜잭션을 지원하게 되었다. 현재 문서에서 Distributed Transaction과 Multi-Document Transaction은 동의어로 본다.

아래는 트랜잭션을 동작하기 위한 예제다. 개인적으로 사용해봤을 때에도 MongoClient를 통해 session을 시작하고, 해당 세션에서 트랜잭션을 시작 및 commit 했다.
- 트랜잭션을 시작하고, 특정 동작을 수행하고, commit(또는 에러 시 abort)
- commit 에러인 TransientTransactionError 또는 UnknownTransactionCommitResult 발생 시의 Retry 로직 통합

```python
client = MongoClient(uriString)
wc_majority = WriteConcern("majority", wtimeout=1000)

# Prereq: Create collections.
client.get_database("mydb1", write_concern=wc_majority).foo.insert_one({"abc": 0})
client.get_database("mydb2", write_concern=wc_majority).bar.insert_one({"xyz": 0})

# Step 1: Define the callback that specifies the sequence of operations to perform inside the transactions.
def callback(session):
    collection_one = session.client.mydb1.foo
    collection_two = session.client.mydb2.bar
    # Important:: You must pass the session to the operations.
    collection_one.insert_one({"abc": 1}, session=session)
    collection_two.insert_one({"xyz": 999}, session=session)

# Step 2: Start a client session.
with client.start_session() as session:
    # Step 3: Use with_transaction to start a transaction, execute the callback, and commit (or abort on error).
    session.with_transaction(
        callback,
        read_concern=ReadConcern("local"),
        write_concern=wc_majority,
        read_preference=ReadPreference.PRIMARY,
    )
```

# OperationFailure
아래와 같은 에러가 센트리에 계속 확인되어 신경쓰였는데, [링크](https://medium.com/@marchpig/mongodb-multi-document-transactions-d51e047f811d)에서 봤을 때 Network 문제, Primary Election, Write Conflict는, 트랜잭션을 다시 시도하는 경우 성공할 수 있는 에러이다. (errorLabels의 TransientTransactionError 로 확인)
파일 업로드 시에 해당 에러가 과도하게 많이 발생했는데, 업로드 시 생성된 세션의 트랜잭션이 끝나지 않은 상태에서 다른 세션이 동일 데이터를 변경하려고 할 때 발생한 것으로 보인다.
connnection 설정을 통해 MongoDB Driver에서 재시도를 하도록 설정해놨기 때문에, 실패 이후 시도했을 것으로 판단된다.
해당 에러가 발생하는 상황에 대해서는 코드를 조금 더 자세히 들여다보는 것이 필요할 것 같다.

```
WriteConflict, full error: {'errorLabels': ['TransientTransactionError'] ...}

```

---
