## ASWebAuthenticationSession

A session that an app uses to authenticate a user through a web service.



### Declaration

```swift
class ASWebAuthenticationSession : NSObject
```

### Overview

Use an `ASWebAuthenticationSession` instance to authenticate a user through a **web service**, including one run by a **third party**. **Initialize the session** with a URL that points to the authentication webpage. A browser loads and displays the page, from which the user can authenticate. In iOS, the browser is a secure, **embedded web view**. In macOS, the system opens the user’s default browser if it supports web authentication sessions, or Safari otherwise.

제 3자를 포함하여, 웹 서비스를 통해 사용자를 인증하기 위해서 `ASWebAuthenticationSession` 인스턴스를 사용한다. 인증 웹페이지를 가리키는 URL로 session을 초기화한다. 브라우저는 사용자가 인증할 수 있는 페이지를 로드하고 표시한다. iOS에서 브라우저는 안전한 내장 web view이다. In macOS, the system opens the user’s default browser if it supports web authentication sessions, or Safari otherwise.

완료 시, 해당 서비스는 callback URL을 세션에 인증 token을 보내고, 세션은 이 URL을 completion handler를 통해 앱으로 보낸다.

For more details, see [Authenticating a User Through a Web Service](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service).

---

<br>

## Authenticating a User Through a Web Service

앱에서 사용자 인증을 받기 위해 웹 인증 세션을 사용하자.

### Overview

Some websites provide, as a service, a secure mechanism for authenticating users. When the user navigates to the site’s authentication URL, the site presents the user with a form to collect credentials. After validating the credentials, the site redirects the user’s browser, typically using a custom scheme, to a URL that indicates the outcome of the authentication attempt.

#### Create a Web Authentication Session

You can make use of a web authentication service in your app by initializing an [`ASWebAuthenticationSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) instance with a URL that points to the authentication webpage. The page can be one that you maintain, or one operated by a third party. During initialization, indicate the callback scheme that the page uses to return the authentication outcome:

```swift
// Use the URL and callback scheme specified by the authorization provider.
guard let authURL = URL(string: "https://example.com/auth") else { return }
let scheme = "exampleauth"

// Initialize the session.
let session = ASWebAuthenticationSession(url: authURL, callbackURLScheme: scheme)
{ callbackURL, error in
    // Handle the callback.
}
```

Use the initializer’s trailing closure to tell the session how to handle the callback after authentication completes, as described below in [Handle the Callback](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service#3395261).

In macOS, or if you have a deployment target of iOS 13 or later, the session keeps a strong reference to itself until the authentication process completes to prevent the system from deallocating the closure. For earlier iOS deployment targets, your app needs to keep a strong reference to the session until authentication completes.

<br>

#### Provide a Presentation Context

Your app indicates the window that should act as a presentation anchor for the session by adopting the [`ASWebAuthenticationPresentationContextProviding`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationpresentationcontextproviding) protocol. From the [`presentationAnchor(for:)`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationpresentationcontextproviding/3237230-presentationanchor) method, which is the protocol’s one required method, return the window that should act as the anchor:

```swift
extension ViewController: ASWebAuthenticationPresentationContextProviding {
    func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor {
        return view.window!
    }
}
```

After creating the session, set an appropriate context provider instance as the session’s [`presentationContextProvider`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/3237232-presentationcontextprovider) delegate:

```swift
session.presentationContextProvider = viewController
```

<br>

#### Optionally Request Ephemeral Browsing

You can configure the session to request ephemeral browsing by setting the session’s [`prefersEphemeralWebBrowserSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/3237231-prefersephemeralwebbrowsersessio) property to `true`:

```swift
session.prefersEphemeralWebBrowserSession = true
```

This setting asks the browser to avoid using any existing browsing data, like cookies, during the authentication process. It also asks that the browser avoid retaining any data collected during the authentication attempt beyond the lifetime of the attempt, or sharing it with any other session. An ephemeral session improves security but prevents reusing the result of a previously successful authentication, potentially forcing the user to reenter credentials. As a result, it’s typically best to let the user choose whether to request ephemeral browsing.

Safari always respects the request. In macOS, the user can choose a different default browser that might or might not respect the request.

Note

When not using an ephemeral session, all cookies except session cookies are available to the browser.

#### Start the Authentication Flow

After configuring the session, call its [`start()`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/2990953-start) method:

```swift
session.start()
```

In iOS, the session loads the authentication web page that you indicated during initialization in an embedded browser view. In macOS, the session sends the page load request to the user’s default browser if it handles authentication sessions, or to Safari otherwise. In any case, the browser presents the user with the authentication page, which is typically a form for entering a username and password.

You can cancel the authentication attempt from your app before the user finishes by calling the session’s [`cancel()`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/2990951-cancel) method:

```swift
session.cancel()
```

When you cancel, the session automatically dismisses the corresponding browser view.

#### Handle the Callback

After the user authenticates, the authentication provider redirects the browser to a URL that uses the callback scheme. The browser detects the redirect, dismisses itself, and passes the complete URL to your app by calling the closure you specified during initialization.

When you receive this callback, first check for errors. For example, you receive the [`canceledLogin`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsessionerror/2991134-canceledlogin) error if the user aborts the flow by dismissing the browser window. If the error is `nil`, inspect the callback URL to determine the outcome of the authentication:

```swift
guard error == nil, let callbackURL = callbackURL else { return }

// The callback URL format depends on the provider. For this example:
//   exampleauth://auth?token=1234
let queryItems = URLComponents(string: callbackURL.absoluteString)?.queryItems
let token = queryItems?.filter({ $0.name == "token" }).first?.value
```

The above example looks for a token stored as a query parameter. The specific parsing that you have to do depends on how the auth

<br>

#### Original Articles

- [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession)
- [Authenticating a User Through a Web Service](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service)