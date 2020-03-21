# Responder Chain을 이용해 Event Handling 하기





### Overview

앱은 Responder 객체를 사용해 이벤트를 수신하고 처리한다. Responder 객체는 UIResponder 클래스의 모든 인스턴스이며, 공통 하위 분류로는 UIView, UIViewController, UIApplication 등이 있다. Responder는 raw event data를 수신하고 이벤트를 처리하거나 다른 responder 객체로 전달해야 한다. 앱이 이벤트를 수신하면, UIKit은 자동적으로 그 이벤트를 first responder로 알려진 가장 적절한 responder 객체로 안내한다.

처리되지 않은 이벤트는 active responder chain 안에 있는 responder로부터 responder에게 전달되며, 이는 앱 responder 객체의 동적 구성이다. 아래 그림1은 인터페이스에 레이블, 텍스트필드, 버튼 및 두개의 배경 뷰가 포함된 앱의 responder를 보여준다. 도표는 사건들이 responder chain에 따라 한 responder에서 다음 responder로 이동하는 방법을 보여준다.

<br>

![](https://images.velog.io/images/delmasong/post/0121fcd4-c087-4cc7-8dcc-145de3fc012c/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-03-21%20%EC%98%A4%ED%9B%84%209.56.36.png)

<br>

텍스트 필드가 이벤트를 처리하지 않으면 UIKit은 텍스트 필드의 상위 UIView 객체에 이벤트를 전송한 다음 window의 root view에 전송한다. 루트 뷰에서 responder chain은 이벤트를 window로 유도하기 전에 (연결된)View Controller로 전환한다. window가 이벤트를 처리할 수 없는 경우 UIKit은 UIApplication 객체에 이벤트를 전달하고, 해당 객체가 UIResponder의 인스턴스이며, responder chain의 일부가 아닌 경우 app delegate에게 이벤트를 전달할 수 있다. 

<br>
<br>

### Determining an Event's First Responder
UIKit은 해당 이벤트 유형을 기반으로 first responder 객체를 지정한다.

| Event type            | First responder                           |
| --------------------- | ----------------------------------------- |
| Touch events          | The view in which the touch occurred.     |
| Press events          | The object that has focus.                |
| Shake-motion events   | The object that you (or UIKit) designate. |
| Remote-control events | The object that you (or UIKit) designate. |
| Editing menu messages | The object that you (or UIKit) designate. |

<br>


> Note
> 가속도계, 자이로스코프 및 자기계 관련 동작 이벤트는 Responder chain을 따르지 않는다. 대신에, 코어 모션은 이벤트를 지정된 객체에 직접 전달한다. 자세한 내용은 Core Motion Framework

컨트롤은 액션 메시지를 사용해 연결된 대상 개체와 직접 통신한다. 사용자가 컨트롤과 상호작용을 할 때 컨트롤은 대상 개체에 액션 메시지를 보낸다. 액션 메시지는 이벤트는 아니지만 responder chain을 이용할 수 있다. 컨트롤의 타겟 객체가 nil인 경우, UIKit은 대상 객체에서 시작해 적절한 동작 방법을 구현하는 객체를 찾을 때까지 responder chain을 횡단(traverses) 한다. 

Gesture recognizer는 view 전에 터치 이벤트와 프레스 이벤트를 받는다. 뷰의 제스처 인식기가 일련의 터치 순서를 인식하지 못하면 UIKit은 뷰에 터치 기능을 전송한다. 뷰가 터치를 처리하지 못하면 UIKit은 이들을 Responder chain으로 전달한다. 제스처 인식기를 사용하여 이벤트를 처리하는 방법에 대한 자세한 내용은 [Handling UIKit Gestures](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_uikit_gestures) 참고


<br>
<br>

### Determining Which Responder Contained a Touch Event
UIKit은 터치 이벤트가 발생하는 위치를 결정하기 위해 뷰 기반 히트 테스트를 사용한다. 구체적으로는 UIKit은 터치 위치를 뷰 계층 구조에서 뷰 객체의 bounds와 비교한다. `UIView`의 `hitTest(_:with:)` 메소드는 뷰 계층을 가로지르며(traverse), 터치 이벤트의 first responder가 되는 가장 깊은 하위 뷰를 찾는다. 

> Note
> 터치 위치가 뷰의 범위를 벗어나면 `hitTest(_:with:)`메서드는 해당 뷰와 해당 하위 뷰를 모두 무시한다. 결과적으로 뷰의 clipsToBounds 속성이 false 일때, 해당 뷰의 bounds를 벗어난 하위 뷰는 터치 기능이 포함되어 있더라도 반환되지 않는다. 히트 테스트 동작에 대한 자세한 내용은 `UIView`의 `hitTest(_:with:)`메소드 설명을 참조. 

터치 시 UIKit은 UITouch 객체를 생성해 뷰와 연결한다. 터치 위치나 다른 파타미터가 변경되면 UIKit은 동일한 UITouch 개체를 새로운 정보로 업데이트 한다. 유일하게 변하지 않는 속성은 뷰다.(터치 위치가 원래 뷰를 벗어나도 터치 뷰 속성의 값은 변경되지 않는다.) 터치 종료시 UIKit은 UITouch 객체를 해제한다.

<br>
<br>

### Altering the Responder Chain

Responder chain의 다음 속성을 오버라이드 해, responder chain을 변경할 수 있다. 이렇게 변경하면, 다음 responder는 리턴되는 객체이다.

많은 UIKit 클래스는 이미 이 속성을 재정의하고 다음을 포함한 특정 개체를 반환한다.

- `UIView` objects
뷰가 뷰컨트롤러의 루트 뷰인 경우, 다음 responder는 뷰 컨트롤러, 그렇지 않으면 다음 responder는 뷰의 슈퍼뷰가 된다. (뷰->슈퍼뷰->루트뷰->뷰컨트롤러 식으로 reponder chain이 올라감)
- `UIViewControlle` objects
뷰 컨트롤러의 뷰가 window의 루트뷰인 경우, 다음 responder는 window 객체이다.
다른 뷰 컨트롤러에 의해 뷰 컨트롤러가 제시된(was presented) 경우, 다음 responder는 제시된(presenting) 뷰 컨트롤러이다.
- `UIViewdow` objects
window의 다음 responder는 UIApplication 객체이다
- `UIApplication` objects
다음 responder는 app delegate 이다. 오직 app delegate가 `UIResponder`의 인스턴스이고 뷰, 뷰컨트롤러, 앱 자체가 아닌 경우에만 해당한다.



### Reference
- https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events