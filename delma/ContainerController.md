# Behavior of Container View Controller 



## What is Container View Controller

컨테이너 뷰 컨트롤러는 컨텐츠를 화면에 표시하는 방법과 컨텐츠를 분리해서 캡슐화를 향상시킨다. 앱의 데이터를 표시하는 컨텐츠 뷰 컨트롤러와 달리 컨테이너 뷰 컨트롤러는 다른 뷰 컨트롤러를 표시하여 화면에 배치하고 이들 사이의 탐색을 처리한다. 

단일 UI에 여러 뷰 컨트롤러의 컨텐츠를 결합하는 방법으로, 컨테이너 뷰 컨트롤러는 기존 컨텐츠를 기반으로 새로운 UI를 작성하는데 자주 사용된다. 하나 이상의 자식 뷰 컨트롤러의 뷰를 자체 뷰 계층 구조로 통합하여 복합 인터페이스를 관리한다. 각 자식은 자체 뷰 계층 구조를 계속 관리하지만, 컨테이너는 해당 자식 루트 뷰의 위치와 크기를 관리한다.

대표적으로 UIKit의 UINavigationController, UITabBarController, UISplitViewController등이 있다.

<img src="https://user-images.githubusercontent.com/40784518/74506944-66d4a500-4f3e-11ea-8af0-607235d5f9d5.png" width="100%">

## How it works

root view와 컨텐츠를 관리한다는 점에서 Container View Controller는  Content View Controller와 같다. 

 콘텐츠는 다른 뷰 컨트롤러의 뷰로 제한되며 자체 뷰 계층 구조에 포함됩니다. 컨테이너 뷰 컨트롤러는 내장 된 뷰의 크기와 위치를 설정하지만 원본 뷰 컨트롤러는 여전히 해당 뷰 내의 컨텐츠를 관리합니다.

커스텀 컨테이너 뷰 컨트롤러를 만들 때는 항상 컨테이너와 상위의 뷰 컨트롤러 간의 관계를 이해하는 것이 중요하다.  아래의 질문을 하면서 관계를 설정하는 것이 좋다

- 컨테이너와 Child View의 역할은 무엇인가?
- 동시에 몇개의 child가 표시되는가?
- sibling 뷰 컨트롤러 간의 관계가 어떻게 되는가?
- 하위 뷰 컨트롤러는 컨테이너에 어떻게 추가/삭제 되는가?
- child의 크기나 위치가 변할 수 있는가? 어떤 조건에서 그러한 변경이 발생하는가?
- 컨테이너는 decoration, navigation을 자체적으로 제공하는가? 
- 컨테이너와 그 하위 사이에는 어떤 종류의 커뮤니케이션이 필요한가? 컨테이너가 UIViewController 클래스에 정의된 표준 이벤트 이외의 특정 이벤트를 해당 하위 이벤트에 보고해야 하는가?
- 컨테이너 모양을 다른 방식으로 구성할 수 있는가? 그렇다면 어떻게?



컨테이너 뷰 컨트롤러와 모든 하위 뷰 컨트롤러간에 공식적인 부모-자식 관계를 설정해야 한다. 부모-자식 관계는 자식이 관련 시스템 메시지를 받게 한다. 컨테이너의 컨텐츠 영역 아무곳에나 뷰를 배치하고 원하는 뷰 크기를 지정할 수 있다. 



### Add a Child View Controller Programmatically to Your Content

1. 포함 관계를 규정하기 위해 컨테이너 뷰 컨트롤러 안에서 `addChild(_:)` 메소드를 호출한다.
2. 컨테이너 뷰 계층에 차일드의 루트뷰를 추가한다.
3. 차일드 루트뷰에 크기와 위치 설정을 위한 제약 조건을 추가한다.
4. 전환이 완료되었음을 알리기 위해 차일드 뷰 컨트롤러에서 `didMove(toParent:)` 를 호출한다.



### Remove a Child View Controller from Your Content

1. 차일드의 `willMove(toParent:)` 메소드의 값을 nil로 호출한다.
2. 차일드의 루트뷰의 제약(contraints)를 없애거나 비활성화 한다.
3. 뷰 계층에서 차일드의 루트뷰를 제거하기 위해 `removeFromSuperview()` 를 호출한다.
4. 컨테이너와 차일드의 관계를 끊기 위해 차일드에서 `removeFromParent()` 를 호출한다.



### References

- https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html
- https://developer.apple.com/documentation/uikit/view_controllers/creating_a_custom_container_view_controller