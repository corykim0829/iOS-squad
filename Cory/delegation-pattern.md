## Delegation

Delegate의 사전적 의미는 "특정한 일, 의무, 권리 등을 나 대신 할 수 있는 사람에게 주는 것"이다. Delegation의 사전적 의미는 "다른 사람을 위해 하는 행동을 부여하는 것"이라고 등재되어있다.

Delegation은 하나의 객체가 프로그램에서 다른 객체를 대신하여 동작하거나 다른 객체와 함께 동작하는 간단하며 강력한 패턴이다. **Delegating object**는 다른 **delegate 객체**에 참조를 유지하고 적절한 때에 **delegate 객체**에게 메세지를 보낸다. 메세지는 **Delegating object**가 처리할 예정이거나 처리된 이벤트를 전달한다. **delegate**는 보여지는 부분, 앱 안의 자신 또는 다른 객체들의 상태를 업데이트하여 메세지에 응답 할 수 있다. 때로는 값을 리턴할 수 있는데, 그 값은 걸쳐진 이벤트가 어떻게 처리되는 지에 영향을 주기도 한다. Delegation의 가장 큰 장점은 하나의 객체에서 여러 객체의 동작을 쉽게 커스터마이징할 수 있게 해준다는 것이다.

<img src="http://www.knowstack.com/wp-content/uploads/2015/05/Cocoa-Delegate-Design-Pattern-NSWindowDelegate-2.png">

- Delegating Object : 위임 객체
- Delegate : 대리자



#### Delegation and the Cocoa Frameworks

**Delegating object**는 보통 프레임워크 객체이고, **delegate**는 보통 커스텀 컨트롤러 객체이다. 메모리가 잘 관리되는 환경에서는, **delegating object는 약한 참조를 delegate에 유지한다.** garbage-collected 환경에서는, receiver는 강한 참조를 delegate에 유지한다. Delegation 예제는 Foundation, UIKit, AppKit, 그리고 다른 Cocoa, Cocoa Touch 프레임워크에 많이 있습니다.

예시로 **Delegating object**를 AppKit 프레임워크의 NSWindow 클래스의 인스턴스이다. NSWindow는 `windowShouldClose:`라는 메소드를 가지고 있는 프로토콜을 선언한다. 사용자가 윈도우의 닫기를 클릭하면, 윈도우 객체는 `windowshouldClose:` 를 delegate에 보내서 윈도우가 닫혔는지 확인하도록 요청한다. delegate는 Boolean 값을 리턴하여 윈도우 객체의 동작을 제어한다.

- Delegating Object : 프레임워크 객체
- Delegate : 커스텀 컨트롤러 객체

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Art/delegation_2x.png">

#### Delegation and Notification

Delegate는 자동적으로 delegating object의 알림의 observer로 등록이된다. delegate는 특정 알림 메세지를 받기 위해 framework 클래스에 의해 선언된 notification 메소드를 구현만하면 된다.  -> 뭔말인지 모르겠다. ^^

#### Data Source

delegate와 거의 동일하다. 차이점은 delegating object와의 관계이다. UI 제어를 위임받는게 아닌, **data 제어**를 위임받는다. delegating object는 보통 table view와 같은 view 객체인데, 자신의 data sourcr를 가지고 있어 때때로 표시할 data가 있는지 묻는다. Data source도 delegate와 마찬가지로 protocol을 채택하고 최소한 protocol의 필수 메소드를 구현해야한다. Data source는 delegating view에 제공하는 모델 객체들의 메모리를 관리한다.

---

### UIImagePickerController

실제 사용되는 코드로 예시를 들어보자

#### UIViewController를 상속한 ViewController

```swift
class SecondViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate {

    let imagePickerController = UIImagePickerController()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        imagePickerController.delegate = self
    }
  
    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        ...
    }
}
```

<br>

#### UIImagePickerController

```swift
open class UIImagePickerController : UINavigationController, NSCoding {
  
    weak open var delegate: (UIImagePickerControllerDelegate & UINavigationControllerDelegate)?
}
```

<br>

#### UIImagePickerControllerDelegate

```swift
public protocol UIImagePickerControllerDelegate : NSObjectProtocol {

    
    @available(iOS 2.0, *)
    optional func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any])

    @available(iOS 2.0, *)
    optional func imagePickerControllerDidCancel(_ picker: UIImagePickerController)
}
```

<img src="https://github.com/corykim0829/iOS-squad/blob/cory/Images/delegation-pattern-image-picker.png">

Delegate 변수를 가지고 있는 객체가 delegation object이다. 이 delegate를 통해서 위임을 처리할 수 있기 때문이다.



#### References

- [Swift.org](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID276)
- [Cocoa Delegate Design Pattern](http://www.knowstack.com/cocoa-delegate-design-pattern/)
- [Understanding Delegates and Delegation in Swift 4](https://www.appcoda.com/swift-delegate/)

