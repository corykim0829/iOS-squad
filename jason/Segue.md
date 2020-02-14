# Using Segues

segues를 사용하여 앱 인터페이스의 흐름을 정의하십시오.
segue는 앱 스토리 보드 파일에서 두 개의 view 컨트롤러 간의 전환(transition)을 정의합니다.
segue의 시작점은 segue를 시작하는 버튼, 테이블 행(table row) 또는 제스처 인식기(gesture recognizer)입니다. 
segue의 끝점은 표시하려는 view 컨트롤러입니다. segue는 항상 새로운 view 컨트롤러를 제공하지만 unwind segue를 사용하여 view 컨트롤러를 닫을 수도 있습니다.

<img width="727" alt="스크린샷 2020-02-14 오후 3 04 41" src="https://user-images.githubusercontent.com/38216027/74505791-61299000-4f3b-11ea-9d2f-6d2e93f58ced.png">

프로그래밍 방식으로 segue를 트리거 할 필요는 없습니다.
런타임시 UIKit은 view 컨트롤러와 연결된 segue를 로드하고 해당 요소를 해당 요소에 연결합니다.
사용자가 요소와 상호 작용할 때 UIKit은 적절한 view 컨트롤러를 로드하고 앱에 segue가 발생한다는 것을 알리고 전환을 실행합니다. 
UIKit에서 보낸 알림을 사용하여 데이터를 새 view 컨트롤러에 전달하거나 segue가 완전히 발생하지 않도록 할 수 있습니다.


> Table 9-1 Adaptive segue types

|   segue type   | behavior | 
|----------------|----------|
| Show (Push) | 이 segue는 타겟 view 컨트롤러의 showViewController : sender : 메소드를 사용하여 새 컨텐츠를 표시합니다. 대부분의 view 컨트롤러에서 이 segue는 source view 컨트롤러를 통해 새로운 콘텐츠를 모달 형식으로 제공합니다. 일부 뷰 컨트롤러는 특히 메서드를 재정의하고 다른 동작을 구현하는 데 사용합니다. **예를 들어, 내비게이션 컨트롤러는 새 뷰 컨트롤러를 내비게이션 스택으로 푸시합니다.** UIKit은 targetViewControllerForAction : sender : 메소드를 사용하여 source 뷰 컨트롤러를 찾습니다.| 
| Show Detail (Replace) | 이 segue는 타겟 view 컨트롤러의 showDetailViewController : sender : 메소드를 사용하여 새 컨텐츠를 표시합니다. 이 segue는 UISplitViewController 객체에 포함 된 view 컨트롤러에만 해당됩니다. 이 segue와 함께 분할 view 컨트롤러는 두 번째 자식 뷰 컨트롤러 (세부 컨트롤러)를 새 콘텐츠로 바꿉니다. 대부분의 다른 뷰 컨트롤러는 새로운 컨텐츠를 모달로 표시합니다. UIKit은 targetViewControllerForAction : sender : 메소드를 사용하여 source 뷰 컨트롤러를 찾습니다. |
| Present Modally | 이 segue는 지정된 프리젠 테이션 및 전환 스타일을 사용하여 view 컨트롤러를 모달로 표시합니다. 적절한 프리젠테이션 컨텍스트를 정의하는 view 컨트롤러가 실제 프리젠 테이션을 처리합니다. |
| Present as Popover | 수평으로 규칙적인 환경에서는 view 컨트롤러가 팝 오버로 나타납니다. 가로로 컴팩트 한 환경에서는 view 컨트롤러가 전체 화면 모달 프레젠테이션을 사용하여 표시됩니다. |

> 분할 view 컨트롤러 
<br>마스터-디테일 인터페이스에서 두 개의 하위 view 컨트롤러를 관리하는 **컨테이너 view 컨트롤러**입니다.

> showDetailViewController:sender:method
<br>이 방법을 사용하면 실제로 화면에 해당 view 컨트롤러를 표시하는 프로세스에서 view 컨트롤러를 표시 할 필요성을 분리 할 수 ​​있습니다. 
이 방법을 사용하면 view 컨트롤러가 내비게이션 컨트롤러 또는 스플릿 view 컨트롤러에 내장되어 있는지 알 필요가 없습니다.

segue를 생성 한 후 segue 객체를 선택하고 속성 관리자를 사용하여 식별자를 할당합니다. segue 중에 식별자를 사용하여 트리거 된 segue를 확인할 수 있습니다. 이는 view 컨트롤러가 여러 segue를 지원하는 경우 특히 유용합니다. 식별자는 segue가 수행 될 때 view 컨트롤러에 전달 된 UIStoryboardSegue 객체에 포함됩니다.

# Modifying a Segue’s Behavior at Runtime

아래 그림은 segue가 트리거 될 때 발생하는 상황을 보여줍니다. 대부분의 작업은 새로운 view 컨트롤러로의 전환을 관리하는 프리젠 테이션 view 컨트롤러에서 발생합니다. 새 view 컨트롤러의 구성은 뷰 컨트롤러를 직접 생성하여 제시 할 때와 동일한 프로세스를 따릅니다. segue는 스토리 보드에서 구성되므로 segue와 관련된 두 뷰 컨트롤러는 모두 동일한 스토리 보드에 있어야합니다.

<img width="800" alt="display segue" src=https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_displaying-view-controller-using-segue_9-4_2x.png>

segue 동안 UIKit은 현재 view 컨트롤러의 메소드를 호출하여 segue의 결과에 영향을 줄 수 있는 기회를 제공합니다.

* shouldPerformSegueWithIdentifier : sender : 메소드는 segue가 발생하는 것을 막을 수있는 기회 를 제공합니다. 이 방법에서 NO를 반환하면 segue가 조용히 실패하지만 다른 동작이 발생하는 것을 막을 수는 없습니다. 예를 들어, 테이블 행을 탭하면 테이블에서 관련 delegate 메서드를 호출 할 수 있습니다.

* source view 컨트롤러의 PreparingForSegue : sender : 메소드를 사용하면 source view 컨트롤러에서 destination view 컨트롤러로 데이터를 전달할 수 있습니다. 메소드에 전달 된 UIStoryboardSegue 오브젝트에는 다른 segue 관련 정보와 함께 대상보기 컨트롤러에 대한 참조가 포함되어 있습니다.
