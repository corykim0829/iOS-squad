# Using Responders and the Responder Chain to Handle Events ( 번역 )

앱을 통해 전파되는 이벤트를 처리하는 방법에 대해 알아보십시오.

## 개요 

앱은 responder 객체를 사용하여 이벤트를 수신하고 처리합니다.
responder 객체는 UIResponder 클래스의 모든 instance이며 일반적인 하위 클래스에는 UIView, UIViewController 및 UIApplication이 있습니다.
**responder는 raw 이벤트 데이터를 수신하여 이벤트를 처리하거나 다른 responder 객체로 전달해야 합니다.**
앱이 이벤트를 받으면 UIKit은 자동으로 해당 이벤트를 first responder라고 하는 가장 적합한 responder 객체로 보냅니다.

만약에 이벤트가 처리되지 않았다면 해당 이벤트는 활성화된 responder chain에서 responder에서 responder로 지나갑니다, responder chain은 앱의 responder 객체들의 
동적인 설정입니다.( 동적이란 것은 런타임에 발생한다는 것이다.)
그림 1은 인터페이스에 label, text field, button 그리고 두 개의 배경 뷰를 포함된 앱의 responder들을 보여줍니다.
다이어그램은 또한 responder chain에 따라 이벤트가 한 responder에서 다음 responder로 어떻게 이동하는지 보여줍니다.


**Figure 1** Responder chains in an app
![responder_chain](https://user-images.githubusercontent.com/38216027/75507957-e75ad180-5a25-11ea-80d8-f861853e59ca.png)


텍스트 필드가 이벤트를 처리하지 않으면 UIKit은 이벤트를 텍스트 필드의 부모 UIView 객체로 보낸 다음 window의 root 뷰를 보냅니다.
root view에서 responder chain는 이벤트를 window으로 보내기 전에 자신(root view)의 view controller로 전달합니다. window가 이벤트를 처리할 수 없는 경우, UIKit은 이벤트를 UIApplication 객체 및 해당 객체가 UIResponder의 인스턴스이고 아직 responder chain의 일부가 아닌 경우 앱 Delegate 에게 전달합니다.

## Determining an Event's First Responder

UIKit은 이벤트 type에 따라 이벤트에 대한 **first responder**로 object를 지정합니다. 
<br>이벤트 type은 다음과 같습니다.

| Event Type | First Responder | 
|------------|-----------------|
| Touch events | 터치가 발생한 뷰 |
| Press events | focus(실물)를 가진 객체 |
| Shake-motion events | 당신 (또는 UIKit)이 지정한 객체. |
| Remote-control events | 당신 (또는 UIKit)이 지정한 객체. | 
| Editing menu messages | 당신 (또는 UIKit)이 지정한 객체. |

> what is Press events?

이 이벤트는 하드웨어 버튼과 관련된 이벤트입니다. 

> what is remote-control event?

remote-control event는 장치에서 멀티미디어를 제어하기 위해 헤드셋 또는 외부 액세서리에서 수신된 명령으로 시작됩니다.

> 주의
<br>가속도계, 자이로 스코프 및 자력계와 관련된 Motion 이벤트들은 responder chain을 따르지 않습니다. 대신에, Core motion deliver가 지정된 객체에 직접 event를 전달합니다.

Control는 action message들을 사용하여 target 객체와 직접 의사소통합니다.
사용자가 control과 상호작용할 때, control은 target 객체에 action message를 보냅니다. 
**action message는 이벤트가 아니지만, 그들은 여전히 responder chain의 이점을 취합니다.**
control의 target 객체가 nil일때, UIKit는 target 객체에서 시작하여 적절한 액션 메서드를 구현하는 객체를 찾을 때까지 responder chain을 통과합니다.
예를 들어, UIKit의 editing menu는 이 동작을 사용하여 `cut (_ :), copy (_ :) 또는 paste (_ :)` 와 같은 이름을 가진 메소드를 구현하는 responder 객체를 탐색합니다.

**Gesture recognizer는 그들의 뷰가 이벤트를 수신하기 전에 먼저 touch 와 press 이벤트들을 수신합니다.** 
**만약에 뷰의 Gusture recognizer가 일련의 터치를 인식하지 못하면 UIKit는 터치를 뷰로 보냅니다.(Gesture recognizer가 인식한다면 안보낸다는 뜻)** 뷰가 터치를 처리하지 않으면 UIKit은 responder chain에 전달합니다.

## Determining Which Responder Contained a Touch Event

UIKit는 뷰 기반의 hit-testing을 사용하여 터치 이벤트가 어디에 일어나는지 결정합니다.
특히 UIKit은 터치 위치를 뷰 계층 구조의 뷰 객체의 bounds와 비교합니다. 
UIView의 hitTest (_ : with :) 메소드는 지정된 터치를 포함하는 가장 깊은 서브 뷰를 찾아서 터치 이벤트의 first responder가 되는 뷰 계층 구조를 탐색합니다.

> Note 
<br>터치 위치가 뷰의 bounds를 벗어나면 hitTest (_ : with :) 메서드는 해당 뷰와 모든 하위 뷰를 무시합니다.
결과적으로 뷰의 clipsToBounds 속성이 false 인 경우 터치를 포함하더라도 해당 뷰의 bounds를 벗어난 subview는 반환되지 않습니다.

터치가 발생하면 UIKit는 UITouch 객체를 만들어, 뷰와 연결합니다.
터치의 위치나 다른 파라미터가 변하면, UIKit은 똑같은 UITouch 객체를 새로운 정보로 업데이트합니다(**손가락이 화면에 닿고 떼어질 때까지 동일한 터치 객체를 사용한다**, 예: 터치하고 움직이면 터치의 위치도 변한다.).
<br>변하지 않는 프로퍼티는 오직 해당 뷰(UITouch 객체의 프로퍼티이다, view: The view to which touches are being delivered, if any.)입니다. (터치 위치가 원래 뷰의 외부로 움직이더라도 터치한 뷰 프로퍼티는의 값은 변하지 않습니다.) 터치가 끝날때 UIKit는 UITouch 객체를 해제합니다. 

## Altering the Responder Chain

responder 객체의 next 프로퍼티를 재정의하여 responder chain을 변경할 수 있습니다. 이렇게 하면 next responder는 당신이 설정한 개체입니다.

많은 UIKit 클래스는 이미 이 프로퍼티(next)을 재정의하고 다음과 같은 특정 객체를 반환합니다.

* UIView objects. 
  * 만약 해당 뷰가 view controller의 root view인 경우, next responder는 view controller 입니다. 
  * 그렇지 않으면 next responder는 해당 뷰의 superview입니다. 

* UIViewController objects.
    * 만약 view controller의 view가 window의 root view인 경우, next responder는 window 객체입니다. 
    * 만약 view controller가 또 다른 view controller(= container view controller)에 의해 제공된다면, next responder는 제시된 view controller(= container view controller) 입니다.  

* UIWindow objects. 
   * window 객체의 next responder는 UIApplication 객체입니다.

* UIApplication object. 
   * next responder는 the app delegate입니다. 하지만 app delegate가 UIResponder의 인스턴스이고 뷰, 뷰 컨트롤러 또는 앱 객체 자체가 아닌 경우에만 가능합니다. 

## Reference

출처 : <https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events>