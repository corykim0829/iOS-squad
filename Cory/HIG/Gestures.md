## Gestures

> 용어 정리

- Immersive app : 몰입도가 높은 앱
- screen-edge : 스크린-테두리

> 본문

사람들은 터치스크린에서 제스쳐를 통해 iOS 디바이스와 인터랙션한다. 이러한 제스쳐들은 컨텐츠와 밀접한 개인적 관계를 끌어내며 화면 객체들의 직접적인 조작 감각을 높여준다.

**일반적으로 표준 제스쳐를 사용하자.** 사람들은 표준 제스쳐에 익숙하며 같은 일을 하는 것에 대해 다른 방법을 배워야 하는 것을 이해하지 못한다. 게임과 다른 몰입도가 높은 앱들에서는, 커스텀 제스쳐는 경험의 재미요소가 될 수 있다. 하지만 다른 앱에서는, 표준 제스쳐를 사용하여 추가적인 비용(제스쳐를 찾거나 기억해야하는 것)을 사용하지 않는게 최선이다.

**비표준 액션을 처리하기 위해 표준 제스쳐를 사용하는 것을 피하자.** 게임 플레이가 있는 게임 앱이 아니라면, 표준 제스쳐의 의미를 재정의하는 것은 혼란과 복잡성을 초래한다.

**시스템 스크린-테두리 제스쳐를 방해하는 것을 피하자.** 디바이스에 따라, 스크린-테두리 제스쳐는 홈 화면, 앱 전환 화면, 알림 센터, 제어 센터 그리고 독(Dock)을 제공한다. 사람들은 모든 앱에서 이 제스처들이 동작할것이라 생각한다. 드물게 게임과 같은 몰입도가 높은 앱은 시스템 제스처보다 우선순위가 높은 커스텀 스크린-테두리 제스처를 필요로 하는데, 첫번째 스와이프는 앱의 커스텁 제스처를 호출하고 두번째 스와이프는 시스템 제스처를 호출한다. 이 행동(a.k.a 테두리 보호)은 사람들로 하여금 시스템-단계 액션에 접근하는 것을 어렵게 하기 때문에 절제되어(sparingly) 구현되어야한다. 개발자 가이드로는 UIViewController의 [preferredScreenEdgesDeferringSystemGestures()](https://developer.apple.com/documentation/uikit/uiviewcontroller/2887512-preferredscreenedgesdeferringsys) 메소드를 참고하자.

**인터페이스 기반의 네비게이션과 액션을 대체하는게 아닌, 보충해 줄 제스처 단축버튼(shortcut)을 제공하자.** 가능하면, 어떤 방법이 한두개의 추가적인 탭을 의미하더라도, 네비게이트 및 액션을 실행하기 위한 간단하고 가시적인 방법을 제공하자.(Whenever possible, offer a simple, visible way to navigate or perform an action, even if it means an extra tap or two.) 다수의 시스템 앱은 이전 스크린으로 돌아갈 수 있는 명백하고 탭할 수 있는 버튼이 있는 네비게이션 바를 제공한다. 하지만 사용자들은 스크린의 측면을 스와이프하여 뒤로 돌아갈 수도 있다. iPad에서 사람들은 홈 버튼을 누르거나 네 손가락을 이용한 핀치 제스처로 홈 화면으로 나갈 수 있다.

**어떤 앱의 경우 경험을 높이기 위해 다중손가락 제스처를 사용하자.**  한번에 여러 손가락을 포함하는 제스처는 모든 앱에 적합하지는 않지만, 어떤 앱에서는 경험을 풍부하게 해주는데, 게임이나 그리기 앱과 같은 경우이다. 예를 들어, 어떤 게임은 조이스틱과 발사 버튼과 같이 동시에 작동될 수 있는 화면의 여러개의 컨트롤을 포함할 수 있다.

개발자 가이드로는, [UIGestureRecognizer](https://developer.apple.com/documentation/uikit/uigesturerecognizer)를 참고하면 좋다.

### 표준 제스처(Standard Gestures)

사람들은 일반적으로 다음 표준 제스처들이 모든 앱에서 시스템을 통해 똑같은 동작을 하길 기대한다.

<br>

- **Tap** : 컨트롤 또는 항목을 선택을 활성화한다.
- **Drag** : 어떤 요소를 이동시키거나 스크린에서 드래그한다.

<img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Tap_2x.png" width="240px"> <img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Drag_2x.png" width="240px">

<br>

- **Flick** : 빠르게 스크롤하거나 이동한다.
- **Swipe** : 한 손가락으로 수행하면 이전 화면으로 돌아가거나, split view controller에서 숨겨진 뷰를 표시하거나, 테이블뷰 행에서 삭제 버튼을 표시하거나 엿보기 작업을 표시한다. iPad에서 네 손가락으로 수행하면 앱을 전환한다.

<img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Flick_2x.png" width="240px"> <img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Swipe_2x.png" width="240px">

<br>

- **Double tap** : 컨텐츠 또는 이미지를 확대 및 중앙에 맞추거나 이미 확대된 경우 축소한다.
- **Pinch** : 바깥으로 핀치하면 확대하고 안으로 핀치하면 축소한다.

<img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_DoubleTap_2x.png" width="240px"> <img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Pinch_2x.png" width="240px">

<br>

- **Touch and hold** : 수정가능하거나 선택가능한 텍스트를 수행할 때, 커서가 위치한 확대된 뷰를 표시한다. collection view와 같은 특정 뷰에서 수행되면, 항목이 재배치할 수 있는 상태로 진입한다.
- **Shake :** 실행 취소나 취소 복원을 실행한다. 

<img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_TouchHold_2x.png" width="240px"> <img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/Gestures_Shake_2x.png" width="240px">

<br>

- **Rotate** : 이미지나 뷰를 회전시킨다.
<img src="https://developer.apple.com/design/human-interface-guidelines/ios/images/gestures-rotation_2x.png" width="240px">

