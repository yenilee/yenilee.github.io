---
title: Odd numbered Node.js versions will not enter LTS status 😟
tags: node 버전관리
---

<br/>
터미널 에러를 통해 공부한 노드 패키지의 배포 phase와 버전 관리 방식 <br/>
<!--more-->

---

# 문제
가끔 화면을 띄울 일이 있어 node를 깔아놓았는데, 터미널을 시작할 떄마다 아래 메시지가 뜨기 시작했다.

```bash
Node.js version v19.6.1 detected.
Odd numbered Node.js versions will not enter LTS status and should not be used for production.
For more information, please see https://nodejs.org/en/about/releases/.
```

로컬에서만 사용하고 있기도 하고 바빠서 무시하다가 심심해서 검색을 시작했다.

## LTS
Long Term Support는 일반적 수준보다 더 오래 지원되는 버전을 뜻한다고 한다.
쉘에 나와있던 해당 링크에 들어갔더니 node 버전 별로 LTS가 시작되는 시점과 유지보수 시작 시점, 종료 시점까지 명시되어 있었다.
내가 다운받았던 버전 19.x은 22년 10월에 release 되었으나, LTS는 없이 유지 보수만 23년 4월부터 시작으로 나와있었다.

[링크](https://github.com/nodejs/Release/issues/76)는 노드 깃헙에서 LTS와 Maintenance의 차이점에 대해 답변한 글이다.


## 로그의 의미
해당 글을 읽고 로그를 다시 보니 Odd numbers, 즉 홀수로 시작하는 버전은 LTS를 지원하지 않으니 운영에서 사용하지 말라는 뜻이었다.

## node release phases
패키지를 다운로드만 받아봤지 패키지의 수명에 대해서는 깊이 생각해본적이 없는데, 배포 스케줄이 생각보다 더 자세하구나 새삼 알게 됐다.
릴리즈 페이지를 살펴보면 상태를 따로 정의해놓고, 버전 별로 상태를 정확하게 명시해놓았다.

- Current :
  - 노드 메인 브랜치에 들어갈 non-majer (non-breaking) 변화의 대부분을 포함해야 함
- Active LTS
  - LTS 팀이 새로운 기능, 버그 픽스, 업데이트를 하고 릴리즈 라인에 적합하고 안정되었다고 결정됨
- Maintenance
  - 치명적인 버그나 보안 문제만 해결

# 과정
로그를 없애려면 노드 버전을 바꿔주면 되는 부분으로 확인되어 노드 버전 매니저를 깔았다.

```bash
sudo npm install -g n
sudo n stable
```

n을 치면 아래와 같이 선택할 수 있는 노드 버전이 나오게 된다.
특정 버전을 선택하면 업데이트가 될줄 알았는데, Permission Denided가 엄청 뜨면서 active한 버전은 여전이 18.15.0 이었다.

![n](/assets/images/n.png)

# 해결
생각해보니 node를 brew로 깔았던 것 같아서 node를 통해 업데이트를 해주었다

```bash
 # 깔려있는 버전 확인 가능
brew search node

# 만약 원하는 버전이 없다면 원하는 버전으로 설치, 있으면 생략
brew install node@16

# 현재 사용하고 있는 node 버전을 언셋
brew unlink node

# 사용하고자 하는 버전으로 세팅
brew link --overwrite node@16
```

# 결과
`node -v` 입력 시 16대 버전으로 확인, `source ~/.zshrc`로 쉘 새로고침 시 더이상 메시지가 확인되지 않음!
