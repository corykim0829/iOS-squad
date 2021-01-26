## OAuth

OAuth의 목적은 OAuth를 통해서 `accessToken`을 얻어낼 수 있다는 것이다.

<br>

#### 1. 역할

- Client - 서비스 제공
- Resource Server - Google, Facebook, Twitter + Authorization Server
- Resource Owner - User

Client는 서비스를 제공하며, Client는 Google, Facebook, Twitter와 같은 client의 서비스가 아닌 타서비스의 특정 기능들을 추가적으로 이용하고 싶을 때가 있다.

여기서 Google, Facebook, Twitter와 같은 서비스를 resource server라고 일컫는다. Resource server의 기능을 이용하기 위해서는 사용자(Resource Owner)의 정보가 필요하다.

가장 간단한 방법은 resource owner의 아이디와 비밀번호를 client가 받아서 직접적으로 사용하는 것이지만, 상식적으로 말이 되지 않는 일이다. 이를 해결하기 위해서 OAuth가 사용된다.

<br>

#### 2. 등록

Client가 resource server의 기능들을 사용하기 위해서는 resource server에 등록을 해야한다. 이는 사전에 서버에 승인을 받는 절차를 의미한다.

서비스마다 차이가 있지만, 공통적인 것은 

- `Client ID`
- `Client Secret`
- `Authorized redirect URLs`

위 3가지를 등록할 때 client로부터 받게된다.

Client ID는 client의 application를 식별하는 고유 ID이다.

Client Secret은 client ID에 대한 비밀번호로 외부에 노출이되면 안된다. 이는 엄청난 보안사고로 이어질 수 있다.

Authorized redirect URLs: 직역하면 권한이 있는 리다이렉트 URL로. resource server가 권한을 부여하는 과정에서 **이 주소**(Authorized redirect URLs)로 전달해달라는 의미이다. Resource server는 만약 Authorized redirect URLs가 아닌 다른 곳에서 요청이 오면 무시한다.

<br>

#### 3. Resource Owner의 승인

등록을 마친 후에 resource server와 client는  `Client ID`, `Client Secret`, `Authorized redirect URLs` 를 알고 있다.

Resource server의 기능이 A, B, C, D라고 했을 때, B, C의 기능을 사용하고 싶다면 B, C 기능에 대해서만 최소한의 인증을 받는 것이 서로에게 좋다.

어떤 일이 일어나는 가에 살펴보자. Resource owner는 client에 접속하는 과정에서 resource server에 접근해야하는 작업이 필요하다면, 우리는 먼저 이러한 작업들에 있어서 사전에 resource owner에게 동의를 구하기 위해서 "Login with OOO"이러한 버튼들이 필요하다.

이러한 버튼들의 주소는 다음과 같다. (버튼이 중요한 것은 아니다.)

`https://resource.server/?client_id=1&scope=B,C&redirect_url=https://client/callback`

해당 주소를 통해 resource owner는 resource server에게 요청을 하게 된다. 이 요청을 통해서 만약 로그인이 되어있지 않다면, Resource server는 로그인을 하라는 화면을 보여주게 된다. 여기서 로그인을 성공적으로 하게 되면, Resource server는 다음 절차를 진행한다.

- 요청 URL에서 client_id와 같은 id가 있는지 확인한다.
- 확인 후, 자신이 가지고 있는 client_id의 redirect URL이 접속 시도한 요청의 redirect URL이 같은지 확인한다. (여기서 다르면 요청을 바로 끝낸다.)
- 요청한 scope, 즉 기능을 client에게 허용할 것인지를 resource owner에게 물어본다.(하단 이미지 참고) 



<img src="https://blog.teddykatz.com/assets/img/oauth-flow-prompt.png" width="400px">

- 허용여부를 resource onwer는 resource server에게 보내게 되고 resource server는 user_id, scope: B, C가 저장되게 된다. (resource server의 B, C에 기능에 대해서 허용했다는 전제하에)

<br>

#### 4. Resource Server의 승인

**authorization code**를 Resource Server는 Resource Owner에게 전송

authorization code는 임시 비밀번호의 역할을 한다. authorization code를 아래와 같이 전송한다. 응답할 때 header의 Location에 넣어서 보낸다.

`Location: https://client/callback?code=3`

location은 리다이렉션인데, 웹브라우저에게 해당 주소로 이동하라고 명령하는 것이다. Resource Server가 Resource Owner의 웹브라우저에게

`?code=3` 이는 임시 비밀번호 authorization code이다.

Resource Owner가 인식하지 못하게 이동하게 된다.

Resource Owner는 Client에게 `https://client/callback?code=3` 을 보내게 된다. 이 과정에서 Client는 authorization code를 알게된다.



여기까지 오면 이 때는 authorization code를 발급받아 accessToken을 발급 받기 전 상황이다.

이제 Client는 Owner를 통하지 않고 Resource Server에 직접 접근할 수 있다.

Client가 Resource Server에 요청할 때 먼저 authorization code를 확인하여 자신의 authorization code와 일치하는지 확인하고 어떤 client에게 발급한 것인지 확인하여 secret까지 확인 redirect까지 완전히 일치하는지 확인

모두가 일치하면 다음 단계로 넘어간다.



#### 5. Access Token

Client가 Resource Owner를 통해 authorization code를 받았다. 

그다음은 access token을 발급받는 것, 이것이 목적이었다.

Resource Server는 authorization code를 통해서 인증을 했기 때문에 지워버려야한다. 그래야 다시 인증을 하지 않는다.

resource server는 드디어 accress token을 발급한다. 그 후에 access token을 client에게 준다. client는 이를 저장한다. 

이 accress toekn은 client가 해당 토큰에 접근하면 특정 user id에 해당하는 사용자들에게 어떤 기능들(scope)에 대해서 accessToken을 가진 사람에게 허용해야겠다 라고 동작하게 된다.



#### Reference

- [생활코딩 - OAuth 2.0](https://opentutorials.org/course/3405)