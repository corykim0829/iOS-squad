## ASWebAuthenticationSession

앱이 웹 서비스를 통해 사용자를 인증하는 세션

#### Declaration

```swift
class ASWebAuthenticationSession : NSObject
```

#### Overview

써드 파티에서 실행되는 것을 포함하여, 웹 서비스를 통해 사용자를 인증하기 위해서 `ASWebAuthenticationSession` 인스턴스를 사용해야 합니다. 인증 웹페이지를 가리키는 URL로 **session을 초기화**합니다. 브라우저는 사용자가 인증을 할 수 있는 페이지를 로드하고 표시합니다. iOS에서 브라우저는 안전하고 내장된 web view입니다.

완료 시, 해당 서비스는 세션에 callback URL을 보내는데 인증 토큰(authentication token)도 함께 전송하고, 세션은 이 URL을 completion handler를 통해 앱으로 다시 보냅니다.

---

<br>

## Authenticating a User Through a Web Service

앱에서 사용자 인증을 받기 위해 웹 인증 세션을 사용하자.

#### Overview

어떤 웹사이트는 서비스로 사용자를 인증하는 안전한 메커니즘을 제공하고 있습니다. 사용자가 사이트 인증 URL로 이동하면, 사이트는 사용자로부터 자격(credential)을 얻기 위해 양식을 보여줍니다. 자격을 확인한 후, 사이트는 보통 **custom scheme**을 사용하는 사용자의 브라우저를 인증 시도에 대한 결과를 가리키는 URL로 리다이렉트합니다.

- custom scheme: iOS에서 딥링크를 지원하는 방법 중 하나

- 딥링크: 특정 페이지에 도달할 수 있는 링크



#### Create a Web Authentication Session

웹 인증 서비스를 앱에 사용하기 위해서는 인증 웹페이지를 가리키는 `ASWebAuthenticationSession` 인스턴스를 초기화를 해야합니다. 페이지는 우리가 유지해야하는 것 또는 써드 파티에서 실행되는 것도 있습니다. 초기화 중에는, 페이지 인증 결과를 반환하기 위해 사용하는 **callback scheme**을 의미한다.

```swift
// URL 과 인증 provider에 의해 명시된 callback scheme을 사용해야 합니다.
guard let authURL = URL(string: "https://example.com/auth") else { return }
let scheme = "exampleauth"

// 세션 초기화
let session = ASWebAuthenticationSession(url: authURL, callbackURLScheme: scheme)
{ callbackURL, error in
    // callback 처리
}
```

[Handle the Callback](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service#3395261) 에 설명되어있는 것 처럼, 인증을 완료한 후에 callback을 어떻게 처리할지에 대해서는 이니셜라이저의 후행 클로저를 사용해야합니다. macOS 또는 배포 타겟이 iOS13 또는 이후 버전을 가지고 있다면, 시스템이 클로저를 할당 해제하지 못하도록 인증 프로세스가 완료될 때까지 자체적으로 강한 참조를 유지합니다. 이전 iOS 배포 타겟의 경우 인증이 완료될 때까지 앱이 **세션을 강하게 참조해야합니다.** 

<br>

#### Provide a Presentation Context

앱에서 [`ASWebAuthenticationPresentationContextProviding`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationpresentationcontextproviding) 프로토콜을 채택하여 세션의 presentation anchor로 작동해야하는 창을 나타냅니다. 프로토콜의 필수 메소드인 [`presentationAnchor(for:)`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationpresentationcontextproviding/3237230-presentationanchor) 메소드에서 anchor 역할을 해야하는 창을 반환해야 합니다:

```swift
extension ViewController: ASWebAuthenticationPresentationContextProviding {
    func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor {
        return view.window!
    }
}
```

세션을 생성한 후에, 적절한 context provider 인스턴스를 세션의  [`presentationContextProvider`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/3237232-presentationcontextprovider) delegate로 설정합니다:

```swift
session.presentationContextProvider = viewController
```

<br>

#### Optionally Request Ephemeral Browsing

세션의  [`prefersEphemeralWebBrowserSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/3237231-prefersephemeralwebbrowsersessio) 프로퍼티를 true로 설정하여 임시 브라우징을 요청하도록 세션을 구성할 수 있다.

```swift
session.prefersEphemeralWebBrowserSession = true
```

이 설정은 인증 프로세스 중에 쿠키와 같은 기존의 브라우징 데이터를 사용하지 않도록 브라우저에 요청합니다. 또한 브라우저가 인증 시도 중에 수집된 데이터를 수명 이상으로 유지하거나 다른 세션과 공유하지 않도록 요청합니다. 임시 세션은 보안을 향상시키지만 이전에 성공한 인증 결과를 재사용하지 못하게 하여 사용자가 자격 증명을 다시 입력하게 할 수 있습니다. 따라서 일반적으로 임시 브라우징을 요청할지 안할지에 대한 여부를 사용자가 선택하도록 하는 것이 가장 좋습니다.

Safari는 항상 요청을 존중(respect)합니다. macOS에서는, 사용자는 기본 브라우저를 선택할 수 있어 요청을 존중하지 않을 수도 있습니다.

> Note
>
> 임시 세션을 사용하지 않으면, 세션 쿠키를 제외한 모든 쿠키를 브라우제엇 사용할 수 있습니다.

#### Start the Authentication Flow

세션 구성을 마치고 세션의 [`start()`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/2990953-start) 메소드를 호출합니다:

```swift
session.start()
```

iOS에서 세션은 우리가 내장된 브라우저 뷰의 초기화 중 표시된 인증 웹 페이지를 로드합니다. macOS에서 세션은 인증 세션을 처리하는 경우 페이지 로드 요청을 사용자의 기본 브라우저로 보내거나 그렇지 않으면 Safari로 보냅니다. 어쨌든 브라우저는 사용자에게 인증 페이지를 제공합니다. 인증 페이지는 일반적으로 사용자 이름과 비밀번호를 입력하기 위한 양식입니다.

앱에서 사용자가 인증을 마무리하기 전에 [`cancel()`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/2990951-cancel) 메소드를 호출하여 인증 시도를 취소시킬 수 있습니다.

```swift
session.cancel()
```

인증을 취소하면, 세션은 자동적으로 대응하는 브라우저 뷰를 자동적으로 dismiss한다.

#### Handle the Callback

사용자 인증 후에, 인증 provider는 callback scheme을 사용하는 URL로 브라우저를 리다이렉트 한다. 브라우저는 리다이렉트를 감지하여 스스로 dismiss하며, 초기화에서 명시한 클로저를 호출하여 complete URL을 앱에 전달한다.

이 callback을 받으면, 먼저 에러를 검사해야합니다. 예를 들어 사용자가 브라우저 창을 닫아 인증 흐름(flow)이 중단된다면, [`canceledLogin`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsessionerror/2991134-canceledlogin) 에러를 받게됩니다. 에러가 `nil`이면, 인증의 결과를 판단하기 위해 callback URL을 검사한다.

```swift
guard error == nil, let callbackURL = callbackURL else { return }

// callback URL 형식은 provider에 따라 다르다. 이 예제에서는:
//   exampleauth://auth?token=1234
let queryItems = URLComponents(string: callbackURL.absoluteString)?.queryItems
let token = queryItems?.filter({ $0.name == "token" }).first?.value
```

위 예제 query 파라미터로 저장된 토큰을 찾습니다. 수행해야 할 특정 파싱 인증 provider가 callback URL을 구성하는 방법에 따라 다릅니다.

<br>

#### Original Articles

- [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession)
- [Authenticating a User Through a Web Service](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service)

#### References

- [Universal Link & Custom URL Scheme](https://ehdrjsdlzzzz.github.io/2019/11/25/Universal-Link-Custom-URL-Scheme/)
- [유니버셜링크 vs. 커스텀URL스킴 비교 분석](https://www.letmecompile.com/universal-link-vs-custom-url-scheme/)
- [인터넷 주소의 의미 (URL / URI)](https://takeknowledge.tistory.com/29)
- [유니버설 링크, URI 스킴, 앱 링크 및 딥 링크:](https://brunch.co.kr/@davidgsyun/3)

