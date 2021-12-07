---
title: gitalk config 설정 방법
tags: jekyll gitalk
---

<br/>
블로그에 gitalk을 연동 시 설정하는 부분에 대해 정리해보았다. <br/>
<!--more-->

# 설정

개인적으로 gitalk config 작성하는 부분이 자세하게 설명이 안 되어있다고 느껴져서, 내가 쓰는 테마의 설정을 가져와 자세하게 설명을 적어보았다.

```python
  gitalk:
    clientID    :  # GitHub Application Client ID
    clientSecret:  # GitHub Application Client Secret
    repository  :  # GitHub repo
    owner       :  # GitHub repo owner
    admin:  # GitHub repo owner and collaborators, only these guys can initialize GitHub issues, IT IS A LIST.
       # - your GitHub Id
```

## 1. github Oauth application
config를 설정하기 전에 github에서 application을 생성해주어야 한다. [링크](https://github.com/settings/applications/new) 에서 아래와 같이 입력해주면 된다.

- Application name: 원하는 이름을 입력 ex. gitalk
- Homepage URL: 깃헙 블로그 주소 ex. https://yenilee.github.io
- Application description: 설명, optional
- Authorization callback URL:  깃헙 블로그 주소 ex. https://yenilee.github.io

## 2. _config.yml 파일 설정

### clientID
위에서 application을 생성하면 나오는 아이디 (아래 노란 색으로 가려진 부분)

### clientSecret
위에서 application을 생성 완료 후 뜨는 화면에서, secret을 생성한 뒤 해당 secret을 복사하고, 아래로 스크롤을 내려 update application 버튼을 눌러준다.
- secret은 만든 뒤에 페이지 이동을 아래와 같이 가려져서 다시 볼 수 없으니, 꼭 복사를 하고 난 뒤에 이동을 해야 한다.

![client](/assets/images/client.png)

### repository
gitalk을 사용하면 댓글이 내가 지정한 repo의 issue에 등록된다. 때문에 아래 조건을 만족해야 한다.
- 사용하는 repo가 public이어야 함
- repo에 issue가 있어야 함 (없을 경우 setting에서 수정 가능)
- repo의 이름만 입력
  - repo에 들어가면 yenilee/gitalk 이런 식으로 되어있는데, gitalk만 입력.

### owner
내 github id 예) yenilee

### admin
내 github id
- 리스트로 입력(`['yenilee']`)하거나, 예시처럼 - yenilee 이런 식으로 입력해주면 됨.


---

구글링해보면 disqus를 사용하는 사람들은 꽤 많은데, gitalk은 검색하면 중국어밖에 안 나온다.
개인적으로 훨씬 깔끔하고 광고도 없어서 바꾸고 나니 아주 만족스럽다 👍🏻



