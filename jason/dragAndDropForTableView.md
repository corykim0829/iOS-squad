# Adopting Drag and Drop in a Table View

## Get Started

앱 간에 drag 앤 드롭을 지원하는 iPad에 이 프로젝트를 배포하십시오. 이 프로젝트의 빌드된 앱을 처음 시작하면 각각 텍스트 문자열이있는 여러 행이있는 테이블이 표시됩니다. Reminder App과 같은 텍스트 문자열 편집을 지원하는 두 번째 앱과 함께 이 앱을 사용하십시오. 예를 들어, Reminder App과 함께 이 앱을 사용하여 iPad 화면을 split view로 구성하십시오. 그런 다음이 앱의 행을 Reminder App으로 드래그하거나 Reminder App을 이 앱으로 드래그하십시오.

이 앱은 또한 행을 위 또는 아래로 끌어서 테이블에서 행 재정렬(rearranging)을 지원합니다. 그러나 이 앱의 재정렬(rearranging)에서는 drag 앤 드랍 API 대신 기존 `tableView (_ : canMoveRowAt :)` 및 `tableView (_ : moveRowAt : to :)` 메서드를 사용합니다.

## Enable Drag and Drop Interactions

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tableView.dragInteractionEnabled = true // Enable intra-app drags for iPhone.
    tableView.dragDelegate = self
    tableView.dropDelegate = self
}
```
* dragInteractionEnabled: A Boolean value indicating whether the table view supports drags and drops between apps.(테이블 뷰가 앱 간 drag 앤 드롭을 지원하는지 여부를 나타내는 부울 값입니다.)
* UITableViewDragDelegate: The interface for initiating drags from a table view. (테이블 뷰에서 drag를 시작하기 위한 인터페이스입니다.)
* UITableViewDropDelegate: The interface for handling drops in a table view. (테이블뷰에서 drop을 핸들링하기 위한 인터페이스입니다. )

## Provide Data for a Drag Session

```swift
func tableView(_ tableView: UITableView, itemsForBeginning session: UIDragSession, at indexPath: IndexPath) -> [UIDragItem] {
    return model.dragItems(for: indexPath)
}
```
UIDragSession: The interface for configuring a drag session.
(drag 세션을 구성하기위한 인터페이스입니다.)

\[부모 프로토콜] UIDragDropSession: The common interface for querying the state of both drag sessions and drop sessions.
(drag 세션과 드롭 세션의 상태를 쿼리하기위한 공통 인터페이스입니다.)

```swift
func dragItems(for indexPath: IndexPath) -> [UIDragItem] {
    let placeName = placeNames[indexPath.row]

    let data = placeName.data(using: .utf8)
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
* `NSItemProvider`: An item provider for conveying data or a file between processes during drag and drop or copy/paste activities, or from a host app to an app extension. (끌어서 놓기 또는 복사 / 붙여 넣기 작업 중 또는 호스트 앱에서 앱 확장으로 프로세스 간에 데이터 또는 파일을 전달하는 Item Provider입니다.)

* `kUTTypePlainText`: The type identifier for text with no markup and in an unspecified encoding.

* `registerDataRepresentation(forTypeIdentifier:visibility:loadHandler:)`: Registers a data-backed representation for an item, specifiying item visibility and a load handler. (아이템의 가시성 및 로드 핸들러를 지정하여 아이템에 대한 데이터 기반 표현을 등록합니다.)

## Consume Data from a Drop Session

```swift
func tableView(_ tableView: UITableView, canHandle session: UIDropSession) -> Bool {
    return model.canHandle(session)
}
```
=> `tableView(_:canHandle:)`: Asks your delegate whether it can accept the specified type of data. (지정된 유형의 데이터를 수락 할 수 있는지 Delegate에게 묻습니다.)

UIDropSession: The interface for querying a drop session about its state and associated drag items. (상태 및 관련 drag 항목에 대한 드롭 세션을 쿼리하기위한 인터페이스입니다.)


```swift
func canHandle(_ session: UIDropSession) -> Bool {
    return session.canLoadObjects(ofClass: NSString.self)
}
```

`canLoadObjects(ofClass:)`: Returns a Boolean value that indicates whether at least one drag item in the session can create an instance of the specified class.(세션에서 하나 이상의 drag 항목이 지정된 클래스의 인스턴스를 만들 수 있는지 여부를 나타내는 부울 값을 반환합니다.)

`NSItemProviderReading`: The protocol you implement on a class to allow an item provider to create an instance of the class.
ItemProvider가 클래스의 인스턴스를 작성할 수 있도록 클래스에서 구현하는 프로토콜입니다.

```swift
func tableView(_ tableView: UITableView, dropSessionDidUpdate session: UIDropSession, withDestinationIndexPath destinationIndexPath: IndexPath?) -> UITableViewDropProposal {
    // The .move operation is available only for dragging within a single app.
    if tableView.hasActiveDrag {
        if session.items.count > 1 {
            return UITableViewDropProposal(operation: .cancel)
        } else {
            return UITableViewDropProposal(operation: .move, intent: .insertAtDestinationIndexPath)
        }
    } else {
        return UITableViewDropProposal(operation: .copy, intent: .insertAtDestinationIndexPath)
    }
}
```
* UITableViewDropProposal: Your proposed solution for handling a drop in a table view. (테이블 뷰에서 드랍을 처리하기위한 제안된 솔루션입니다.)

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
* drop 된 데이터를 데이터 구조에 통합하고 테이블을 업데이트합니다.

drop된 컨텐츠를 테이블뷰에 통합하려면 이 메소드를 사용하십시오. 구현시 coordinator 객체의 items 속성을 반복하여 drag 작업과 관련된 UIDragItem 객체를 검색합니다. 해당 객체의 데이터를 사용하여 테이블 뷰의 데이터 소스와 테이블 뷰 자체를 업데이트하십시오.

하나의 아이템을 옮기는 로컬 drag에는 `tableView(_:moveRowAt:to:)` 가 역할을 하지만, 
다른 화면이나 앱과의 drag에는 이 메소드가 호출되는 것 같다.