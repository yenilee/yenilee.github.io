---
title: gitalk 적용되지 않는 문제 (feat. 깃헙 블로그 댓글)
tags: jekyll gitalk
---

<br/>
블로그에 gitalk을 연동하려다가 생긴 문제와 설정에 대해 간단히 정리해보았다. <br/>
<!--more-->

# 문제
처음에는 블로그 댓글 서비스로 disqus를 썼는데, 무료 계정으로 사용하다보니 광고가 게시글 당 3개씩 붙어서 gitalk을 써보기로 했다.
현재 쓰고 있는 TeXt Theme에서 지원을 하길래 쉽게 적용될 줄 알았는데, 코멘트 section은 바로 나타났지만 로그인하려고 하면 자꾸 `gitalking`이라는 글씨와 함께 동그라미가 돌아가면서 아래와 같이 403 에러가 떴다.

![gitalk_fail](/assets/images/gitalk_fail.png)

처음에는 내가 만든 github application의 client secret이 문제인 줄 알고 다시 생성하면서 인증을 해봤다.
그런데 내가 쓰고 있는 테마의 홈페이지도 댓글이 동일하게 로그인이 안 되고 있었다.
인증 문제가 아니라 [gitalk의 버그](https://github.com/gitalk/gitalk/issues/433) 였고, 생각보다 아주 간단하게 해결되었다.

# 해결 방법

결론부터 말하면 gitalk의 버전을 1.2.7로 업데이트하면 된다. gitalk은 해당 인증 이슈로 인해 약 5달 전 1.2.7로 [버전을 업데이트](https://github.com/gitalk/gitalk/commit/68786ee5d28b1e93fc7dcb57edabc70370d80cca) 했다.

그래서 패키지를 가져오는 것으로 추정되는 부분의 모든 gitalk 버전을 1.2.7로 바꿔주었다.
아래처럼 gitalk의 모든 버전이 1.2.2로 되어있었고, 이를 1.7.2로 바꿔주니 gitalk에서 해당 이슈를 fix한 버전으로 가져와서 정상적으로 로그인이 될 수 있었다.

```python
sources:
  bootcdn:
    font_awesome: 'https://cdn.bootcdn.net/ajax/libs/font-awesome/5.15.1/css/all.css'
    jquery: 'https://cdn.bootcss.com/jquery/3.1.1/jquery.min.js'
    leancloud_js_sdk: '//cdn.jsdelivr.net/npm/leancloud-storage@3.13.2/dist/av-min.js'
    chart: 'https://cdn.bootcss.com/Chart.js/2.7.2/Chart.bundle.min.js'
    gitalk:
      js: 'https://cdn.bootcss.com/gitalk/1.2.2/gitalk.min.js'
      css: 'https://cdn.bootcss.com/gitalk/1.2.2/gitalk.min.css'
    valine: 'https://unpkg.com/valine/dist/Valine.min.js' # bootcdn not available
    mathjax: 'https://cdn.bootcss.com/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML'
    mermaid: 'https://cdn.bootcss.com/mermaid/8.0.0-rc.8/mermaid.min.js'
  unpkg:
    font_awesome: 'https://use.fontawesome.com/releases/v5.15.1/css/all.css'
    jquery: 'https://unpkg.com/jquery@3.3.1/dist/jquery.min.js'
    leancloud_js_sdk: '//cdn.jsdelivr.net/npm/leancloud-storage@3.13.2/dist/av-min.js'
    chart: 'https://unpkg.com/chart.js@2.7.2/dist/Chart.min.js'
    gitalk:
      js: 'https://unpkg.com/gitalk@1.2.2/dist/gitalk.min.js'
      css: 'https://unpkg.com/gitalk@1.2.2/dist/gitalk.css'
    valine: 'https//unpkg.com/valine/dist/Valine.min.js'
    mathjax: 'https://unpkg.com/mathjax@2.7.4/unpacked/MathJax.js?config=TeX-MML-AM_CHTML'
    mermaid: 'https://unpkg.com/mermaid@8.0.0-rc.8/dist/mermaid.min.js'
```

## 문제 설명
개발자 도구에서 확인해보면 문제가 발생했을 때 아래 요청에 대해 403 에러가 뜬 것을 확인할 수 있다.

```python
https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token
```
[관련 블로그를 번역해보면](https://www.chenhanpeng.com/create-own-cors-anywhere-to-resolve-the-request-with-403/) cors-anywhere는 도메인 간 문제를 해결하기 위해 설계된 리버스 프록시이며 오픈 소스 프레임워크인데,
gitalk에서 cors-anywhere.herokuapp.com 서버를 써오다가 2021.1.31부터 [해당 서버가 개방형 프록시 역할을 하지 않으면서](https://github.com/Rob--W/cors-anywhere/issues/301)  인증 문제가 발생한 것으로 보인다.

예전에 다른 블로그에 작성했었던 [CORS 게시물](https://velog.io/@matisse/TIL-CORS) 참고해보면, CORS는 클라이언트와 서버가 요청 응답을 주고받을을 때 리소스 접근이 허용된 출처인지 확인하기 위한 구조다.
origin이 다른 내 블로그와 github간의 resource를 주고받기 위해서는 리버스 프록시가 필요하다. 그래서 gitalk은  proxy를 아래와 같이 변경했다.

```python
https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token`
```



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



