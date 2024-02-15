# OAuth2

### OAuth2 란?

OAuth2는 인증을 위한 오픈 스탠다드 프로토콜이다. OAuth2를 사용하면 사용자의 비밀번호를 **제3자에게 노출하지 않고 인증**을 수행할 수 있고, 사용자의 권한을 제어할 수 있다.

### OAuth2의 구성 요소

OAuth2는 4가지 구성 요소로 이루어져 있다.

* **Resource Owner**: 자원의 소유자. (사용자)
* **Client**: 사용자의 정보를 이용하려는 서비스를 의미한다. (애플리케이션)
* **Authorization Server**: 사용자의 인증을 담당하는 서버를 의미한다. (ex. Google, Naver)
* **Resource Server**: 사용자의 정보를 가지고 있는 서버를 의미한다.

### OAuth2 인증 방식

OAuth2는 인증 방식에 따라 4가지로 나뉜다.

* **Authorization Code Grant**
* Implicit Grant
* Resource Owner Password Credentials Grant
* Client Credentials Grant

#### Authorization Code Grant

Authorization Code Grant는 가장 보편적인 인증 방식이다.\
사용자가 client에게 인증을 요청하면 client는 authorization server에게 인증 요청을 보낸다.\
authorization server는 사용자에게 인증을 요구하고, 사용자가 인증을 하면 authorization server는 client에게 callback url로 인증 코드를 전달한다.\
client는 authorization server에게 인증 코드로 access token을 요청한다.\
authorization server는 client에게 access token을 전달한다.\
client는 access token을 이용하여 resource server에게 사용자 정보를 요청한다.\
resource server는 client에게 사용자 정보를 응답하고, client는 사용자에게 응답을 전달한다.

<div align="left">

<figure><img src="../.gitbook/assets/Authorization Code Grant.png" alt="" width="534"><figcaption></figcaption></figure>

</div>

#### Implicit Grant

Implicit Grant는 Authorization Code Grant와 다르게 authorization server가 client에게 인증 코드를 전달하지 않고 access token을 바로 전달한다.\
refresh token 사용이 불가능하다는 단점이 있다.\
client는 client\_secret을 사용하지 않는다.\
access token을 획득하는 과정이 간소화되어 있지만, access token을 url에 노출하기 때문에 보안에 취약하다.

<div align="left">

<figure><img src="../.gitbook/assets/Implicit Grant.png" alt="" width="534"><figcaption></figcaption></figure>

</div>

#### Resource Owner Password Credentials Grant

Resource Owner Password Credentials Grant는 사용자가 아이디와 비밀번호를 client에게 전달 후 client가 직접 authorization server에게 전달하여 access token을 획득하는 방식이다.\
client가 사용자의 아이디와 비밀번호를 알고 있어야 하기 때문에 보안에 취약하다.\
client가 신뢰할 수 없는 경우에는 사용하지 않는 것이 좋다.

<div align="left">

<figure><img src="../.gitbook/assets/Resource Owner Password Credentials Grant.png" alt="" width="534"><figcaption></figcaption></figure>

</div>

#### Client Credentials Grant

Client Credentials Grant는 client가 사용자와의 상호작용 없이 서버 to 서버로 인증을 수행하는 방식이다.

<div align="left">

<figure><img src="../.gitbook/assets/Client Credentials Grant.png" alt="" width="384"><figcaption></figcaption></figure>

</div>

***

