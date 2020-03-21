

# UIResponder

이벤트에 대한 대응 및 처리를 위한 추상적 인터페이스.

<br>




### Declaration

```swift
class UIResponder: NSObject
```

<br>



### Overview

Responder 객체, 즉 `UIResponder` 인스턴스는 UIKit 앱의 이벤트 처리 백본을 구성한다. `UIApplication` 객체, `UIViewController` 객체 및 모든 `UIView` 객체(`UIWindow` 포함)를 포함한 많은 주요 객체도 Responder 이다. 이벤트가 발생하면, UIKit은 처리하기 위해 앱의 Responder 객체로 이벤트들을 발송한다.

터치 이벤트, 동작 이벤트, remote-control 이벤트, press 이벤트 등 여러종류의 이벤트가 있다. 특정 유형의 사건을 처리하려면 responder가 해당 방법을 재정의해야 한다. 
예를들어 터치 이벤트를 처리하기위해 responder는 `touchBegan(_:with:)`, `touchMoved(_:with:)`, `touchEnd(_:with:)`, `touchCanceled(_:with:)` 메소드를 구현한다. 터치시 responder는 UIKit에서 제공하는 이벤트 정보를 사용해 터치 변경사항을 추적하고 앱 인터페이스를 적절히 업데이트한다. 

UIKit Responder는 이벤트 처리 외에도 처리되지 않은 이벤트를 앱의 다른부분으로 전달하는 관리도 한다. 주어진 responder가 이벤트를 처리하지 않을 경우, 해당 이벤트를 responder chain의 다음 이벤트로 전달한다.
UIKit은 이벤트를 수신하기 위해 어떤 객체가 다음에 있어야 하는지를 결정하기 위해 미리 정의된 규칙을 사용해 responder chain을 동적으로 관리한다. 예를들어, 뷰는 이벤트를 슈퍼뷰로 전달하고, 계층의 루트 뷰는 이벤트를 뷰 컨트롤러로 전달한다.

Responder는 `UIEvent` 객체를 처리하지만 input view를 통해 사용자 정의 입력을 허용할 수도 있다. 시스템의 키보드는 input view의 가장 명백한 예시다. 사용자가 화면에서 `UITextField` 및 `UITextView` 개체를 탭하면 view가 첫번째 responder가 되고, input view인 시스템 키보드가 표시된다. 마찬가지로 다른 responder가 활성화되면 사용자 정의 input view를 생성하고 표시할 수 있다. 사용자 정의 input view를 responder와 연결하려면 해당 view를 responder의 `inputView` 속성에 할당하면 된다. 




### References
- https://developer.apple.com/documentation/uikit/uiresponder