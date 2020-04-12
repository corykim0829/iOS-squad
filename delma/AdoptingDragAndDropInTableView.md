# Adopting Drag and Drop in a Table View

table view에서 드래그 앤 드롭을 활성화하고 구현하는 방법



### Overview

이 샘플 코드 프로젝트는 UITableView 인스턴스를 사용해 테이블 뷰에서 드래그 앤 드랍이 되도록 만드는 방법을 보여준다.

드래그앤 드롭을 활성화하려면 테이블 뷰를 자체 드래그 델리게이트 및 드롭 델리게이트로 지정해야한다. 데이터를 제공하거나 사용하려면 델리게이트의 드래그 앤 드롭 메소드를 구현하면 된다.

테이블 뷰에서 드래그 앤 드롭을 채택하는 것은 커스텀 뷰에서 드래그 앤 드롭을 만드는 방법과 비교해 몇가지 중요한 면에서 다르다. 단계를 비교하려면 [Adopting Drag and Drop in a Custom View ](https://developer.apple.com/documentation/uikit/drag_and_drop/adopting_drag_and_drop_in_a_custom_view) 를 참조하면 된다. 



### Get Started

이 프로젝트를 처음 실행하면, 행이 여러개 있고 각각 텍스트 문자열이 있는 테이블이 나타난다. 

이 앱은 행을 위나 아래로 끌어서 테이블의 행을 재정렬하는 것을 지원한다. 그러나 이 앱의 재정렬은 드래그 앤 드롭 API 대신 기존 [tableView(_:canMoveRowAt:)](https://developer.apple.com/documentation/uikit/uitableviewdatasource/1614927-tableview)와 [tableView(_:moveRowAt:to:)](https://developer.apple.com/documentation/uikit/uitableviewdatasource/1614867-tableview) 방법을 사용한다. 



### Enable Drag and Drop Interactions

드래그와 드롭을 사용하려면 테이블뷰를 own drag와 drop 델리게이트로 지정해야 한다. 이 코드를 위한 편한 위치는 앱의 viewDidLoad() 메서드에 있다. 이 코드는 드래그 앤 드랍을 모두 활성화 한다. 

```swift
	override func viewDidLoad() {
		super.viewDidLoad()
		
		tableView.dragInteractionEnabled = true
		tableView.dragDelegate = true
		tableView.dropDelgate = true
	
	}
```



custom view와 다르게 테이블뷰는 상호작용을 추가하는 interaction property가 없다. 대신, 테이블뷰는 drag delegate와 drop delegate를 직접 사용한다.



### Provide Data for a Drag Session

테이블 뷰에서 드래그하기 위한 데이터를 제공하려면 [tableView(_:itemsForBegining:at:)](https://developer.apple.com/documentation/uikit/uitableviewdragdelegate/2897492-tableview) 메소드를 구현해야 한다. 다음은 이 코드의 모델과 무관한 부분이다.

```swift
func tableView(_ tableView: UITableView, itemsForBeginning session: UIDragSession, at indexPath: IndexPath) -> [UIDragItem] {
    return model.dragItems(for: indexPath)
}
```

tableView(_:itemsForBeginning:at:) 메서드에서 사용하는 다음과 같은 기능은 이 샘플 코드 프로젝트에서 데이터 모델의 인터페이스 역할을 한다.

```swift
func dragItems(for indexPath: IndexPath) -> [UIDragItem] {
    let placeName = placeNames[indexPath.row]	//선택된 행의 데이터를

    let data = placeName.data(using: .utf8)	//Data로 인코딩
    let itemProvider = NSItemProvider()
    
    itemProvider.registerDataRepresentation(forTypeIdentifier: kUTTypePlainText as String, visibility: .all) { completion in
        completion(data, nil)
        return nil
    }

    return [
        UIDragItem(itemProvider: itemProvider)
    ]
}
```



### Consume Data from a Drop Session

tableView에서 drop session 데이터를 사용하려면 세가지 델리게이트 메소드를 구현해야 한다.



첫째, (어떤 타입의 오브젝트를 핸들 가능한 상태로 지정할건지 결정) 앱의 상태나 다른 요구사항에 따라 드래그 아이템을 거부할 수 있다. 이 프로젝트의 구현을 통해 사용자는 NSString 클래스의 인스턴스만 삭제할 수 있다. (세션에 로드할 수 있는 오브젝트로 NSString을 지정해놓았기 때문)

```swift
func tableView(_ tableView: UITableView, canHandle session: UIDropSession) -> Bool {
    return model.canHandle(session)
}
```

tableView(_:canHandler:)메서드에서 사용하는 다음과 같은 기능은 데이터 모델의 인터페이스 역할을 한다.

```swift
func canHandle(_ session: UIDropSession) -> Bool {
    return session.canLoadObjects(ofClass: NSString.self)
}
```



둘째, 데이터를 복사해 어떻게 사용하기를 원하는지 시스템에 말해야한다. drop의 선택지에서 선택해 지정해야 한다.

```swift
func tableView(_ tableView: UITableView, dropSessionDidUpdate session: UIDropSession, withDestinationIndexPath destinationIndexPath: IndexPath?) -> UITableViewDropProposal {
    // The .move operation is available only for dragging within a single app.
    if tableView.hasActiveDrag {	//테이블뷰에 드래그가 활성화 되었다면
        if session.items.count > 1 {	//그 아이템이 1개 초과라면
            return UITableViewDropProposal(operation: .cancel)	//걍 취소
        } else {		//1개 이하라면 이동
            return UITableViewDropProposal(operation: .move, intent: .insertAtDestinationIndexPath)
        }
    } else {	//활성화되지 않았다면 걍 카피 오퍼레이션
        return UITableViewDropProposal(operation: .copy, intent: .insertAtDestinationIndexPath)
    }
}
```





마지막으로 사용자가 화면에서 손가락을 들어 드래그 앤 드롭을 하려 할 때, 테이블 뷰에 드래그 항목의 특정 데이터 표시를 요청할 수 있는 기회가 있다.

```swift
/**
     This delegate method is the only opportunity for accessing and loading
     the data representations offered in the drag item. The drop coordinator
     supports accessing the dropped items, updating the table view, and specifying
     optional animations. Local drags with one item go through the existing
     `tableView(_:moveRowAt:to:)` method on the data source.
*/
func tableView(_ tableView: UITableView, performDropWith coordinator: UITableViewDropCoordinator) {
    let destinationIndexPath: IndexPath
    
    if let indexPath = coordinator.destinationIndexPath {
        destinationIndexPath = indexPath
    } else {
        // Get last index path of table view.
        let section = tableView.numberOfSections - 1
        let row = tableView.numberOfRows(inSection: section)
        destinationIndexPath = IndexPath(row: row, section: section)
    }
    
    coordinator.session.loadObjects(ofClass: NSString.self) { items in
        // Consume drag items.
        let stringItems = items as! [String]
        
        var indexPaths = [IndexPath]()
        for (index, item) in stringItems.enumerated() {
            let indexPath = IndexPath(row: destinationIndexPath.row + index, section: destinationIndexPath.section)
            self.model.addItem(item, at: indexPath.row)
            indexPaths.append(indexPath)
        }

        tableView.insertRows(at: indexPaths, with: .automatic)
    }
}
```

