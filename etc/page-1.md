# Jwt와 Refresh Token



### Jwt

#### Jwt 란?

Jwt(Json Web Token)은 당사자간에 전송할 클레임을 나타내는 수단이고, Jwt의 클레임은 Json Web Signature(Jws)로 디지털 서명된 json 객체로 인코딩되어 사용된다.\
쉽게 말해 통신에 필요한 정보를 디지털 서명된 json 객체로 만들어 사용되는 토큰이다.

[jot 이라고 읽는다](https://datatracker.ietf.org/doc/html/rfc7519#section-1)

#### Jwt 구조

Jwt는 크게 세 부분으로 나뉜다.

*   Header: 토큰의 타입, 사용된 알고리즘 정보를 갖고 있다.

    ```sh
    {
        "alg": "HS256",
        "typ": "JWT"
    }
    ```
*   Payload: 클레임이라고 불리는 데이터 조각들이 포함되어있다. 클레임은 registered, public, private 세 종류로 구분된다.

    ```sh
    {
        "sub": "1234567890",
        "name": "John Doe",
        "admin": true
    }
    ```
*   Signature: Header와 Payload를 결합하고 서버의 비밀키를 사용해 저장된 알고리즘으로 해싱하여 생성된 값. 이 서명을 통해 토큰의 무결성과 인증이 보장된다.

    ```sh
    HMACSHA256(
        base64UrlEncode(header) + "." +
        base64UrlEncode(payload),
        secret)
    ```

각 부분은 base64로 인코딩되어 `.`으로 구분된 문자열로 출력된다.

<div align="left">

<figure><img src="../.gitbook/assets/jwt1.png" alt="" width="563"><figcaption></figcaption></figure>

</div>

#### Jwt를 왜 사용하는가?

그렇다면 Jwt를 왜 사용할까? 사용자가 로그인을 하면 사용자 인증 정보를 서버(세션)에 두고 사용하면 편하고 좋지 않을까?

물론 세션을 사용하면 서버내에서 사용자의 상태를 유지하면서 관리할 수 있기 때문에 좋은 방법이다. 하지만 단점도 존재한다.

* 사용자 정보를 세션에 저장하고 사용되니 사용자가 많아지면 서버에 부담이 가해진다.
* 분산 서버의 경우 서버간 세션을 동기화하는데 비용이 든다.

등등의 단점들을 Jwt가 보완해준다.\
Jwt는 어떤 이점이 있길래?

* 인증 정보가 토큰에 존재하기 때문에 서버는 사용자 인증 정보를 유지할 필요가 없어져서(Stateless) 서버의 확장성 및 유연성이 높아진다.
* 토큰으로 사용자 인증을 수행하기 때문에 다양한 유형의 클라이언트에서 쉽게 접근할 수 있다.

등등, 다양한 이점이 존재하지만 **사용자 인증 정보를 서버내에서 유지하지 않는 `무상태성(Stateless)`**이 가장 큰 장점이라고 할 수 있다.\
하지만 당연히 단점도 존재한다.

* 사용자 인증 정보를 담은 토큰을 탈취당하면 서버에선 대처할 수 있는 방법이 없다.(아예 없는건 아님)

토큰이 악의적 사용자에게 탈취당한다면 출입 자유이용권을 갖게되는 셈이다.\
이런 보안 취약점을 보완하고자 사용되는 방법 중 하나가 바로 `access token`과 `refresh token`의 분리이다.

#### Access Token과 Refresh Token

사용자가 인증(로그인)에 성공하면 필요한 인증 정보를 포함하는 access token과 access token을 재발급 받을 수 있는 refresh token을 발급한다. access token은 만료 시간이 짧게 설정되어있어서 access token이 만료되었을 때 사용자는 refresh token을 사용해 새로운 access token을 발급받을 수 있게된다.&#x20;

refresh token을 통해 access token의 만료 시간이 짧아져서 access token이 탈취당하더라도 위험 시간은 크게 줄어들게 되고 access token이 만료되었을 때 다시 로그인할 필요 없이 refresh token을 통해 새로운 access token을 발급받을 수 있어 사용자 경험이 유연해진다.



### Refresh Token

그렇다면 refresh token을 사용하기 때문에 access token의 만료시간이 짧아져서 안전해진걸까?\
access token과 refresh token을 구분했으니 토큰이 안전해졌을까?\
access token과 refresh token의 구분으로 더 좋은 설계가 됐을까?

그렇지 않다.

access token의 만료시간이 짧다한들 만료 시간이 지나지 않은 access token을 탈취당하게 된다면 악의적 사용자는 자유롭게 access token을 사용할 수 있게된다.\
access token과 refresh token을 구분지었지만 access token 뿐만 아니라 refresh token 또한 탈취당한다면  위험한 상황이 벌어질 수 있다.\
만약 리소스 서버와 권한 부여 서버가 동일한 엔티티인 경우, 보안 수준이 동일할 수 있기때문에 access token과 refresh token의 구분으로 얻을 수 있는 보안적 이점이 크지 않을 수 있다.

#### 토큰 보안

토큰 인증 방식에 대한 많은 참고 자료들을 찾아봤지만 결과는 "완벽한 보안은 존재하지 않는다"라는 결론에 이르렀다. 토큰 인증 방식은 세션 인증 방식의 보안적 이점은 포기한 채 토큰 인증 방식을 통해 가져올 수 있는 이점을 통해 서버의 확장성과 유연성을 증대시키는 방법이다.

그렇다면 토큰 인증 방식은 사용하면 안되는 방법인걸까?

당연히 그렇지 않다. 완벽한 보안이 존재하지 않을 뿐이지 충분히 보안을 위한 다양한 방법이 제시되어있고 사용되고 있다.\
그렇다면 어떤 방법을 통해 토큰 보안을 지킬 수 있을까?

*   쿠키 사용

    refresh token을 쿠키에 저장 및 secure, httponly, samesite 등의 옵션과 `csrf token`등을 활용하여 `csrf`공격에 대한 예방과 `xss`공격에 어느정도 예방할 수 있다.
*   토큰 사용 정보

    토큰이 사용된 ip 주소 등의 정보를 토대로 악의적인 접근을 차단하는 방법이다.\
    토큰에 대한 자체적인 보안 처리보단 사용자의 활동을 토대로 이루어지는 보안 방법이다.
*   https 사용

    https를 사용해 데이터를 암호화하여 통신
*   만료 시간 단축

    토큰의 만료 시간을 짧게 유지하여 토큰이 악용될 수 있는 시간을 최소화 한다.
*   RTR(Refresh Token Rotation)

    access token을 재발급할 때 refresh token도 새로 발급하여 refresh token을 일회성으로 사용하는 방법이다. 하지만 새로운 refresh token이 발급되었다고 해서 refresh token이 만료되는 것은 아니다. 한 번 발급된 토큰은 토큰 그 자체로 유효하기 때문이다. 그렇다면 어떻게 해야할까?\
    트레이드 오프가 필요하다.\
    새로운 refresh token과 기존 refresh token을 구분지어서 새로운 refresh token이 발급되면 기존의 refresh token을 만료 시키는 과정이 필요하다. 토큰 자체를 직접적으로 만료 시킬 수 있는 방법은 존재하지 않고 refresh token정보를 서버 어딘가 저장해두고 각 토큰을 구분지어야 한다는 뜻이다.\
    refresh token정보를 서버 데이터베이스에 저장하고 관련 처리를 한다면 jwt의 이점인 무상태가 아니게 되는거 아닐까? 이러한 부분이 타협점이 필요하다.\
    refresh token 정보를 데이터베이스에 저장한다면 충분한 보안적 이점을 얻을 수 있다.\
    새로운 refresh token이 발급되었다면 기존의 refresh token에 대한 무효 태그를 남길 수 있고, 악의적 요청이 포착된다면 해당 refresh token과 관련된 모든 요청을 차단할 수 있다.

이 외에도 다양한 보안 방법이 존재하지만, 앞서 말한 것처럼 완벽한 보안은 존재하지 않는다.&#x20;

토큰 활용에 대한 이점을 갖기 위해 보안에 대한 처리를 그만큼 철저히 해야 하는 것이다.



***

#### 참고

[https://hudi.blog/refresh-token/](https://hudi.blog/refresh-token/)\
\
[https://stackoverflow.com/questions/32060478/is-a-refresh-token-really-necessary-when-using-jwt-token-authentication](https://stackoverflow.com/questions/32060478/is-a-refresh-token-really-necessary-when-using-jwt-token-authentication)\
\
[https://pragmaticwebsecurity.com/articles/oauthoidc/refresh-token-protection-implications.html](https://pragmaticwebsecurity.com/articles/oauthoidc/refresh-token-protection-implications.html)\
\
[https://betterprogramming.pub/should-we-store-tokens-in-db-af30212b7f22](https://betterprogramming.pub/should-we-store-tokens-in-db-af30212b7f22)\
\
[https://stackoverflow.com/questions/3487991/why-does-oauth-v2-have-both-access-and-refresh-tokens/12885823](https://stackoverflow.com/questions/3487991/why-does-oauth-v2-have-both-access-and-refresh-tokens/12885823)\
\
[https://dev.to/cotter/localstorage-vs-cookies-all-you-need-to-know-about-storing-jwt-tokens-securely-in-the-front-end-15id](https://dev.to/cotter/localstorage-vs-cookies-all-you-need-to-know-about-storing-jwt-tokens-securely-in-the-front-end-15id)













