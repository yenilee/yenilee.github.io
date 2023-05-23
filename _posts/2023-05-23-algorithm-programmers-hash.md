---
title: 프로그래머스 베스트 앨범 문제 풀이
tags: algorithm python
---

<br/>
프로그래머스 해시 관련 문제를 풀다가 정리 <br/>
<!--more-->

---

# 문제
장르 별로 가장 많이 재생된 노래를 두 개씩 앨범에 추가하는 것이 문제의 핵심인데, 노래를 추가할 때 조건이 다음과 같다.

1. 많이 재생된 장르를 먼저 추가
2. 장르 내에서는 재생 횟수 기준으로 추가
3. 장르 내에서 재생 횟수가 같은으면 고유 번호(노래 이름)가 낮은 노래를 먼저 추가

# 풀이 과정
1. 반복문을 줄이기 위해 장르, 플레이리스트는 zip과 enumerate로 한번에 관련 값들을 저장
   - 노래 이름이 리스트 인덱스와 동일하기 때문에 enumerate 사용이 가능.
2. Counter의 most_common 을 활용해 가장 재생 횟수가 많은 장르부터 앨범에 추가할 수 있도록 함
3. 만약 장르의 노래가 하나일 경우에는 정렬하지 않고 바로 추가 후 다음 장르로
4. 장르의 노래가 여러개면 sorted를 통해 정렬
   - 처음에는 노래를 저장하는 자료구조를 dict나 list로 생각했는데, 그러면 정렬이 어려워짐
   - sorted를 찾아보다가 tuple로 저장할 경우 인덱스로 정렬이 가능하고, 여러개의 키로 정렬할 수 있어(재생 횟수는 내림차순 + 노래명은 오름차순) 위 3번 이슈도 해결
5. 노래를 저장하는 자료 구조를 결정하니 바로 통과됨

# 문제 풀이

```python
from collections import Counter, defaultdict

def solution(genres, plays):
    answer = []
    genres_count = Counter()
    count_by_genre_songs = defaultdict(list)

    for song, (genre, played) in enumerate(zip(genres, plays)):
        genres_count[genre] += played
        count_by_genre_songs[genre].append((song, played))

    for genre, _ in count_by_genre.most_common():
        songs = count_by_genre_songs[genre]
        if len(songs) == 1:
            answer.append(songs[0][0])
        else:
            sorted_songs = sorted(songs, key=lambda song: (-song[1], song[0]))
            answer.append(sorted_songs[0][0])
            answer.append(sorted_songs[1][0])
    return answer

```
