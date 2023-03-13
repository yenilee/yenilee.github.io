---
title: Access token 과 ID token의 차이점
tags: 인증/권한 oauth
---

<br/>
구글 로그인 개발 중 access token, id token 관련해 찾으며 정리한 글 <br/>
<!--more-->

---

3줄 요약
- 인증은 로그인 등 자격 증명으로 사용자를 식별하는 것
- 권한은 특정 리소스에 대해 접근 권한을 갖고 있는지 확인하는 것
- Access token은 권한, ID token은 인증

# 인증과 권한
- 인증 (Authentification)
  - 로그인과 같이 사용자 신원을 확인하는 프로세스
  - 비밀번호 인증, 소셜 인증, 생체 인증 등을 통해 사용자 식별
    - 소셜 인증의 경우 구글, 페이스북, 트위터와 같은 소셜 플랫폼의 기존 자격 증명을 통해 사용자 식별
- 권한 (Authorization)
  - 사용자가 특정 리소스에 대해 특정 접근 권한을 갖고 있는지 확인
  - Oauth 등을 통해 사용자의 권한 확인

# IDP (IDentify Provider)
- 여권을 통해 내 신원을 확인해 여러 나라를 여행다닐 수 있는 것처럼, IDP를 통해 인터넷에서 내 신원을 확인
- SAML, Oauth 2.0, OIDC 같은 표준을 기반으로 사용자를 인증하거나, 특정 리소스에 대한 접근 권한을 준다.

# Access token
- client application이 사용자 자원에 접근할 수 있도록 권한(authorization)을 준다.
- 사용자 인증과 동의를 획득한 이후에 authorization 서버에 의해 발행된다.

## OAuth(Open Authorization)
- oauth2 context에서 access token은 사용자를 대신해 특정 action을 수행하기 위한 자원에 접근하는 것을 허용하는걸 말한다.
  - 예를 들어 링크드인에 올린 글을 트위터에도 올리고 싶을 경우, 트위터에 글을 올릴 수 있도록 링크드인에게 권한을 부여해주는 것
  - 글을 쓰는 것만 허용하며, 삭제나 프로필 수정, 다른 권한 없는 action은 할 수 없음
- OAuth 전에는 아이디, 비밀번호 방식이었으며 보안에 취약한 구조였음
- 해당 개방형 표준을 통해 인터넷 사용자들이 ID/비밀번호를 제공하지 않고도 다양한 플랫폼에서 인증과 권한을 획득할 수 있게 됨

### Oauth 2.0
- 1.0의 문제들(복잡한 구현, 애플리케이션 지원 부족 등)을 개선한 버전
- Oauth 2.0의 핵심 명세는 토큰과 관련된 형식에 대해 언급해놓지 않았다.

## Format
```json
{
  "iss": "https://tenant.auth0.com/",
  "sub": "MCHiB...OsMFrzyb@clients",
  "aud": "https://glossary.com",
  "exp": 1627566690,
  "iat": 1627653090,
  "scope": "create:term update:term",
  "gty": "authorization_code"
}
```
- string이기만 하면 어떤 포맷이든 상관이 없으나, 통상적으로 JWT가 사용된다.
- scopes가 포함된다.
  - third-party 애플리케이션(권한 요청자)이 특정 작업(create, update 등)만 할 수 있도록 허용해주는 메커니즘

## Do and Don't
- ✅ API를 호출할 때 사용한다.
  -  access token이 토큰이 탈취되더라도, sender와 바인딩 되어있기 때문에 요청 client가 아니면 api를 호출할 수 없다.
- ✅ 클라이언트의 접근 가능 여부를 확인할 때 사용
- ✅ 서버 사이드(resource server, 권한을 받고자 하는 서버)에서 해당 내용을 inspect
- ❌ 클라이언트 사이드(권한을 요청하는 주체)에서 해당 내용을 inspect
  - 정해진 포맷이 아니기 때문에 변경될 수 있고, 그 경우 예상치 못하게 동작에 문제가 생길 수 있기 때문

# ID token
- 유저가 인증(authentificated) 되었음을 증명
- OpenID Connect(OIDC)에 의해 도입되었으며 구글, 페이스북, Auth0 과 같은 identity providers에 의해 사용되는 공개 표준(open standard)
- 사용자는 브라우저를 통해 OpenID provider를 대상으로 인증하고, 웹 애플리케이션에 대한 접근을 받는다.


## Format
```json
{
  "iss": "http://my-domain.auth0.com",
  "sub": "auth0|123456",
  "aud": "1234abcdef",
  "exp": 1311281970,
  "iat": 1311280970,
  "name": "Jane Doe",
  "given_name": "Jane",
  "family_name": "Doe"
}
```
- JWT로 인코딩되어있다.
  - 해당 속성들을 claims라고 부른다. 사용자에 대한 claims가 사용자의 identify를 정의한다.
    - OpenID Connect specifications는 사용자 claims를 보유하는게 필수는 아니다.
    - 최소한의 구조에서는 사용자의 데이터도 없고 인증 관련 정보만 존재한다.
- aud: audience of the token, 즉 해당 토큰의 수신자 = client ID
- private key를 사용해 발행 주체에 의해 서명될 수 있다.

## Do and Don't
- ✅ 사용자가 인증되었는지 확인하는 경우
- ✅ 사용자의 프로필 데이터를 조회해야 하는 경우
- ❌ API 호출 시 사용
  -  id token을 탈취한 뒤, 해당 id token으로 api를 호출할 수 있기 때문이다.
- ❌ 클라이언트의 접근 가능 여부를 확인할 때 사용


# 참고
[ID Token and Access Token: What's the Difference?](https://auth0.com/blog/id-token-access-token-what-is-the-difference/)
[편의성을 높인 ID 인증 관리 - OIDC(OpenID Connect)가 주목 받는 이유](https://www.samsungsds.com/kr/insights/oidc.html)
