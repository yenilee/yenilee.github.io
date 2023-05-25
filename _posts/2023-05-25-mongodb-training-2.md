---
title: MongoDB 잘! 써보기 (2) 디자인 패턴
tags: mongodb index
---

<br/>
mongodb training session에서 배운 내용을 요약 정리 🤓 <br/>
<!--more-->

---

[해당 게시글](https://yenilee.github.io/2023/05/25/mongodb-training-1.html)에서 이어지는 글.

# MongoDB에서의 디자인 패턴
## RDBMS
- RDBMS에는 모델링 시 제3정규형, 제6정규형 등 정해진 답이 존재한다.
- 사용보다 데이터를 중심으로 디자인된다.

## Document
- 사용 패턴에 따른 여러가지 디자인 옵션이 존재한다.
- 데이터보다 애플리케이션에 초점을 맞추며, `application-level에 맞게 스키마를 정의`한다.

### BSON
- mongodb는 데이터를 bson 형식으로 저장한다.
- bson에서 특정 field를 찾을 때, 도큐먼트가 다 메모리에 올라오고 맞는 field가 나올 때까지 jump
  - 때문에 너무 많은 field를 flat하게 저장하면 불리하다.

### Array
- mongodb에서는 array를 잘 쓰는게 매우 중요
- whole array를 업데이트하면 성능상 문제가 생기므로 operator를 활용해서 업데이트할 것
  - $elematch, $all, $size, $push, $pop, $pull, $pullALl, $each
- arrayr는 document 사이즈를 초과하지 않도록 bounded(제한) 되어야 한다.
  - 끝 없이 저장되는 데이터를 list로 추가할 경우 document size limit 초과하게될 수 있음

# Embedding vs Linking
## Embedding
- 장점
  - 연관된 모든 데이터를 하나의 쿼리로 조회 가능
  - 하나의 document에서 처리되기 때문에 atomic 한 operation이 가능
    - mongodb는 단일 document 단위의 트랜잭션을 지원하기 때문에, transaction 필요 없음
- 단점
  - 데이터 duplication의 가능성
  - 큰 도큐먼트는 오버헤드를 의미할 수 있음
  - 16 MB document size limit이 존재
- When to embed
  - 보통의 use case에서 사용
    - client에 보내는 response에 따라 start는 embed로 하고, 필요에 따라 link로 빼는 것을 추천
  - document가 작을 때
  - 연관된 데이터가 자주 함께 조회될 때 ex. 게시물 조회 시 항상 댓글 보여지는 경우

## Linking
- 장점
  - 도큐먼트 사이즈가 작아져 16MB limit 초과할 가능성 없음
  - duplication 데이터 없음
- 단점
  - 필요 데이터 조회를 위해 쿼리를 두번 하거나 aggregation 해야됨.
  - multi document는 transaction 없이는 관련 정보를 atomic하게 업데이트할 수 없음
- When to link?
  - 하나 또는 양쪽의 document가 너무 거대한 경우
  - 서로 거의 접근되지 않는 경우
  - document limit 16MB를 넘지 않을 경우
  - write가 자주 발생하는 경우

# Common Schema Patterns

## Attributes
- 비슷한 field가 많고 이로 인해 인덱스도 많아지는 경우
- 데이터를 단일 field가 아닌 array 형태로 저장 -> elematch를 통해 쿼리
- array에 fixed schema로 데이터를 저장할 경우 인덱스 하나로 매칭 쿼리를 커버할 수 있다.

## Extended Reference
- Reference field는 데이터를 normalize를 해주지만 데이터 패치 시 비효율적임
- 관련 데이터를 빠르게 조회하기 위해 자주 사용되는 정보를 복제해옴
- 조회 효율을 위한 비정규화 방식이라고 보면 될듯

## Subset
- 워킹셋이 너무 크고, 모든 documents의 field들이 다 사용되진 않을 때
- 자주 사용되는 데이터 일부(subset)와, 자주 사용되지 않는 나머지로 colection을 split함
- 또는 자주 사용되는 데이터만 embedding 해서 쿼리의 round trip을 줄임
- ex. 상품, 리뷰 collection이 따로 존재하고 리뷰가 너무 많을 때, 최근 10개의 리뷰를 상품에 저장해 한번에 쿼리

## Bucket
- documents가 너무 많거나 너무 많아서 embed가 불가할 때
- 데이터를 그루핑해서 document 별로 나누어 저장
- ex. 특정 영화의 관객수를 일별로 저장하면 document가 너무 많고, 영화 하나에 모든 일별 데이터를 저장하기도 너무 많아 embed 불가 -> 월 별로 데이터를 저장한다면 embed도 size limit 초과하지 않으면서 document 개수도 과도하게 많아지지 않음
- application 단에서 구현할 수도 있으나, mongo 5.0 부터 Timeseries collection 사용 시 DB 단에서 알아서 처리해준다고 함

## Computed
- read 비중 높고 write 비중이 낮은 데이터에 대해, 미리 계산해놓고 바로 조회할 수 있도록 하는 것
- 특정 field의 총합을 구할 때, 조회 시 쿼리하는게 아니라 미리 sum을 해놓고 조회할 때 바로 가져오는 것
- 읽기에 최적화된 패턴

## Schema Versioning
- 모델에 schema version identifier를 둬서 스키마 변경하는 방법
- downtime 없이 seamless하게 모델 수정할 수 있음

# 끝으로
index와 디자인 패턴은 서로 보완해주는 관계라고 할 수 있다.
index는 필요한 데이터만 로드할 수 있도록 쿼리 성능을 올려주지만, 만약 100만건의 데이터를 다 읽어야 한다면 index는 있어도 느리다.
-> 100만건을 읽는 상황이 오지 않도록 하는 것이 디자인 패턴을 통해 모델링을 변경하는 방법이다.

소스 코드에 적용해볼만한 부분이 많을 것 같아 기대된다 🤭
