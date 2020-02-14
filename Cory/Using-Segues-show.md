## Using Segues

세그웨이를 사용하여 앱의 인터페이스 흐름을 정의할 수 있다. 세그웨이는 앱의 **스토리보드** 파일에 있는 view controller들 간의 전환을 정의할 수 있다. 세그웨이의 시작점은 버튼, 테이블 행 또는 gesture recognizer를 이용해 초기화할 수 있다. 세그웨이의 끝점은 화면에 표시하고 싶은 view controller이다. 세그웨이는 항상 새로운 view controller를 나타내지만, `unwind` 세그웨이를 사용하여 view controller를 닫을 수 있다.

- Segue는 스토리보드의 view controller 전환을 위한 것

세그웨이는 프로그래밍 방식으로 작동시킬 필요는 없다. 런타임에 **UIKit**이 view controller와 관련된 세그웨이를 로드하고 로드된 세그웨이를 **해당 요소**에 연결해준다. 사용자가 연결된 요소에 상호작용을 주면, UIKit은 적절한 view controller를 로드하고, 우리의 앱에 세그웨이 발생을 알려주고 마지막으로 view controller 전환을 실행한다. UIKit으로 부터 받은 알림을 새로운 view controller로 데이터를 전달하는 데 사용하거나 완전히 발생하지 않도록 할 수 있다. 



> Creating a Segue Between View Controllers

스토리보드에서 추가하면 된다.

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_creating_2x.png">

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_creating_relationships_2x.png" width="240px">

> Adaptive Segue

세그웨이에서 관계 타입을 선택할 때, 가능할 때마다 **적응형 세그웨이(Adaptive Segue)**를 선택하는게 좋다. 적응형 세그웨이는 현재 환경에 맞게 자동으로 행동을 조절한다. 예를들어 `Show` 세그웨이는 현재 나타나져있는 view controller에 따라 바뀌게 된다. 비적응형 세그웨이는 적응형 세그웨이를 지원하지 않는 iOS7에서도 실행되어야하는 앱을 위해 제공된다. 

<br>

- #### Show(Push)

  - 현재 view controller의 [`show(_:sender:)`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621377-show) 메소드를 사용하여 새로운 컨텐츠를 보여준다. 대부분의 view controller에서 이 세그웨이는 새로운 컨텐츠를 modal로 제공한다. 일부 view controller는 특정하게 메소드를 오버라이드하여 사용하여 다른 동작을 구현한다. 예를 들어 navigation controller에서는 새로운 view controller를 navigation stack에 push한다.

- #### Show Detail (Replace)

  - 현재 view controller의 [`showDetailViewController(_:sender:)`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621432-showdetailviewcontroller) 메소드를 사용하여 새로운 컨텐츠를 보여준다. 이 세그웨이는 오직 `UISplitViewController` 객체 안에서 추가된 view controller와 관련이 있다. 이 세그웨이를 사용하면 split view controller는 두번째 child view controller를 새로운 컨텐츠로 변경한다. 대부분의 다른 view controller에서는 새로운 컨텐츠를 modal로 보여준다.

- #### PresentModally

  - 특정 presentation과 transition 스타일을 사용하여 modal로 view controller를 보여준다. 적절한 presentation context를 정의하는 view controller가 실제 presentation을 처리한다.

- #### Present as Popover

  - 수평적으로 규칙적인 환경(horizontally regular environment)에서는, view controller는 popover로 나타난다. 수평적으로 컴팩트한 환경(horizontally compact environment)에서는, view controller는 full-screen modal으로 표시한다.

<br>

## show()

> show(_:sender) in UIViewController

#### Discussion

You use this method to decouple the need to display a view controller from the process of actually presenting that view controller onscreen. Using this method, a view controller does not need to know whether it is embedded inside a navigation controller or split-view controller. It calls the same method for both. The [`UISplitViewController`](https://developer.apple.com/documentation/uikit/uisplitviewcontroller) and [`UINavigationController`](https://developer.apple.com/documentation/uikit/uinavigationcontroller) classes **override** this method and handle the presentation according to their design. For example, a navigation controller overrides this method and uses it to push `vc` onto its navigation stack.

<br>

> show(_:sender) in UINavigationController

#### Discussion

This method pushes `vc` onto the navigation stack in a similar way as the [`pushViewController(_:animated:)`](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621887-pushviewcontroller) method. You can call this method directly if you want but typically this method is called from elsewhere in the view controller hierarchy when a new view controller needs to be shown.

The Show segue uses this method to display a new view controller.

<br>

## UINavigationController Document

#### Adapting to Different Environments

The navigation interface remains the same in both horizontally compact and horizontally regular environments. When toggling between the two environments, only the size of the navigation controller’s view changes. The navigation controller does not change its view hierarchy or the layout of its views.

When configuring segues between view controllers on a navigation stack, the standard Show and Show Detail segues behave as follows:

- Show segue—The navigation controller pushes the specified view controller onto its navigation stack.
- Show Detail segue—The navigation controller presents the specified view controller modally.

The behaviors of other segue types are unchanged.

<br>

> Modifying a Segue’s Behavior at Runtime

아래 그림은 세그웨이가 어떻게 발생되는지 보여준다. 대부분의 작업은 보여지고있는 view controller에서 일어나며, 이 view controller는 새로운 view controller로의 전환을 관리한다. 새로운 view controller의 구성은 스토리보드를 사용하지 않고 직접 view controller를 만들어서 표시하려고 할 때와 동일한 프로세스를 따른다. 세그웨이는 스토리보드에서 구성되기 때문에,세그웨이에 연결된 두 view controller는 모두 같은 스토리보드에 있어야한다.

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_displaying-view-controller-using-segue_9-4_2x.png">

세그웨이 중, UIKit은 segue결과에 영향을 줄 수 있는 기회를 주기 위해서 현재 view controller의 메소드를 호출한다.

- `shouldPerformSegue(withIdentifier:sender:)`
- `prepare(for:sender:)`
  - Notifies the view controller that a segue is about to be performed.

#### References

- [Using Segues in View Controller Programming Guide for iOS](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/UsingSegues.html)
- [show(_:sender:) - UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621377-show)
- [show(_:sender:) - UINavigationController](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621872-show)
- [UINavigationController Document - Adapting to Different Environments](https://developer.apple.com/documentation/uikit/uinavigationcontroller)