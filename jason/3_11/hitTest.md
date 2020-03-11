* [apple document의 번역본](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest)임을 알립니다. 

> UIView의 인스턴스 메소드입니다. 

# hitTest(_:with:)

지정된 Point을 포함하는 뷰 계층 구조 (자체 뷰도 포함)에서 receiver의 가장 먼 subview를 반환합니다.

## Declaration

```swift
func hitTest(_ point: CGPoint, 
        with event: UIEvent?) -> UIView?
```

## Parameters

* point
  * receiver의 로컬 좌표계(local coordinate system)에 지정된 point (bounds).

* event
  * 이 메소드의 호출을 보증한 event. event 처리 코드 외부에서 이 메소드를 호출하는 경우 nil을 지정할 수 있습니다.

## Return Value

현재 뷰의 가장 먼 subview이며 point를 포함하는 뷰 객체입니다.
<br>point가 receiver의 뷰 계층 외부에 있으면 nil을 반환합니다.

## Discussion

이 메소드는 각 서브 뷰의 point (inside : with :) 메소드를 호출하여 터치 이벤트를 수신해야 하는 서브 뷰를 판별하여 뷰 계층 구조를 순회합니다. point (inside : with :)가 true를 반환하면 지정된 point가 포함 된 가장 앞면 뷰(계층에서 가장 먼 subview)가 발견 될 때까지 하위 뷰의 계층 구조가 유사하게 순회됩니다. 뷰에 point가 포함되어 있지 않으면 뷰 계층의 분기(branch)가 무시됩니다. 이 메서드를 직접 호출할 필요는 없지만 하위 뷰에서 터치 이벤트를 숨기려면 이 메서드를 재정의 할 수 있습니다.

이 메소드는 숨겨진 뷰 객체(isHidden = true)나 사용자와의 상호작용을 비활성화(disabled)했거나 알파 레벨이 0.01 이하인 뷰들은 무시합니다. 이 메소드는 hit을 결정할 때 뷰의 콘텐츠를 고려하지 않습니다. 
따라서 지정된 포인트가 해당 뷰 콘텐츠의 투명한 부분에 있어도 뷰는 여전히 반환 될 수 있습니다.

수신자의 bound 밖에 있는 point는 실제로 수신자의 하위 뷰 중 하나에 속하더라도 hit으로 보고되지 않습니다. 이 경우는 현재 뷰의 clipsToBounds 프로퍼티가 false로 설정되어 있고 영향을 받는 하위 뷰가 뷰의 bounds를 넘어 확장 된 경우에 발생할 수 있습니다.