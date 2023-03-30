---
title: github action 최적화 (1) - docker build cache 문제
tags: CI docker github_actions
---

<br/>
github action에서 이미지 빌드 시 cache from option 이 먹지 않아 탐구한 내용들 <br/>
<!--more-->

---

DevOps로 github actions를 사용하다가 문제 발생 😤

# 문제
아래 문제로 현재 github action 최적화 작업을 진행하는 중이다.
- 최근 소스코드와 테스트가 동시에 많이 늘어나게 되어, 놀고 있던 머신에 러너 추가
- CI 파이프라인은 고사양 & 단순 PR 체크는 저사양에서 진행하는 것으로 세팅
- 저사양이 테스트가 너무 오래걸리고, 도커 빌드 시 캐시도 못사용하고 있어 답답해 기절 직전

# 문제 파악
github actions의 경우 리모트에서 도커 이미지를 받아 해당 이미지를 기반으로 컨테이너를 생성해(pull -> build -> run) 여기서 테스트를 돌리고 있는데,
테스트도 테스트지만 도커 빌드 시 캐싱을 하지 못해 항상 고정적인 빌드 시간이 소요되는 것이 첫 번째로 해결하고 싶은 문제였다.

캐시가 되면 아래와 같이 캐시된 이미지 레이어가 찍히면서 CACHED 라고 표시가 되어야 하는데, 모든 단계가 다 빌드를 할 때마다 똑같이 실행됨 ㅠ

```bash
#5 [ 2/15] RUN apt-get 패키지
#5 sha256: ******....
#5 CACHED
```

# 과정
이번 기회에 chatGPT한테 도움을 구해봤다. 일단 너무 재밌어서 거의 새벽까지 디버깅을 하게 만듦...
구글링 검색 시 안나오는 것들 (예를 들어 || true, -t 같이 여기저기서 사용되느 공통 옵션)을 너무 잘 설명해줘서 약간 .... 진짜 기가 막히고 코가 막힘. 여튼 제시해준 6가지 방법 중 귀담아 들은 방법은 총 4가지였다.

1. DockerFile과 --cache-from image가 같은 디렉토리에 있는지 확인해
2. --cache-from image가 실제로 존재하는지 확인해
3. --no-cache 옵션으로 rebuild해봐. 그럼 문제 파악 가능해
4. Docker Daemon이 캐시 사용할 수 있게 잘 세팅되어 있어?

# 해결
위 해결 방법을 보고 단서를 얻었다. 일단 docker images로 검색해보면 내가 캐시하고자 하는 이미지는 있었는데, 빌드 시 사용할 캐시 레이어를 못찾는 것 같았다.
추가 질문을 하니 DOCKER_BUILDKIT=1 환경 설정을 하라고 했다.

## Docker BuildKit
- Docker의 새로운 빌드 엔진. 향상된 캐싱, 병렬화, 보안 기능을 제공
- DOCKER_BUILDKIT=1 로 환경 변수를 세팅하면 도커가 이미지를 빌드할 때 BuildKit을 사용

BuildKit 도 똑같이 --cache-from이 있는데, 캐싱 소스로서 쓸 이미지를 지정할 수 있고, 빌드를 속도를 높이기 위해 로컬 캐시를 사용한다고 한다.

```
name: Build Image with cache
env:
  DOCKER_BUILDKIT: 1
run: |
  docker build --cache-from host/image_name:latest -t host/image_name:latest -t host/image_name:version .
```

### BuildKit은 알고있다
BuildKit은 도커 이미지의 cached layers의 위치가 어딘지 알고 있다.
content-addressable(이름, 위치가 아니라 콘텐츠 기반으로 검색 가능한) 한 스토리지 시스템에 캐시를 저장해서 이전에 빌드된 레이어를 효과적으로 재사용할 수 있다고 한다!

## 빌드 이력
추가로 이전에 빌드한 이력이 있어야 캐싱을 할 수 있기 때문에, 캐시로 사용하고자 하는 이미지도 빌드 시 태그에 포함해주었다.

# 결과
캐시 레이어가 어디 있는지 알고, 빌드 이력을 만들자 캐싱이 되어 5분 걸리던 빌드가 1초만에 끝남 ,,, 야호 🥰
