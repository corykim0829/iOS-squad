# Delegate pattern in iOS



## What is delegate pattern?

> delegate 위임하다, 대표(자)

델리게이트 패턴은 디자인 패턴 중 하나로, 프로그램 안에서 어떤 객체를 대신하여 행동한다던가, 다른 객체와 협동하여 일할 수 있게끔 권한을 위임하는 패턴이다.

iOS에서 자주 사용하는 delegator인 UITableViewDelegate나 UICollectionViewDelegate의 타입은 프로토콜이다.

즉 위임받는 대리자(주로 사용자가 만든 컨트롤러 객체)가, 위임 받으려는 프로토콜을 채택하는 방식으로 위임의 과정이 이루어진다.

그렇다면 자연히 프로토콜과 델리게이트의 차이점에 대해 의문이 들게 된다.

그냥 해당 프로토콜을 채택해서 사용한다고 말하면 될 것을, 굳이 delegate pattern이라고 명시하며 사용하는 이유는 왜일까?

그것은 델리게이트가 어떻게 동작되는지 살펴보면서 함께 알아보자.





## How works it?

이 패턴에는 크게 두가지 역할이 존재한다.

**위임자** 와 **대리자** 가 바로 그것이다.

**위임자** 

- 대리자는 일반적으로 framework object이다.

- 대리자에 대한 참조를 유지한다.

- 적절한 시점에 대리자에게 메시지를 보낸다. (* 메시지 - 위임자가 다뤄왔거나, 방금 처리한 이벤트에 대한 알림)

- 대부분의 Cocoa framework 클래스의 위임자는 대리자에 의해 발생되는 알림의  관찰자로 자동으로 등록된다.

- 경우에 따라 위임자는 이벤트 처리 방법에 영향을 주는 값을 리턴할 수 있다. 

  

**대리자**

- 일반적으로 사용자가 만든 컨트롤러 객체이다.
- 대리자는 특정 알림 메시지를 받기 위해 프레임워크 클래스에서 선언한 (채택한 프로토콜의 메서드만?) 구현하면 된다.
- 대리자는 어플리케이션 안에 있는 다른 객체나 자신의 모양이나 상태를 업데이트 해 메시지에 응답한다.



여기서 기억해야 할 사항은 위임자와 대리자가 서로 메시지를 보내고 응답하면서 통신을 한다는 것이다.

<img src="https://user-images.githubusercontent.com/40784518/74105996-1956e180-4ba6-11ea-8553-dbd4f62564bb.jpeg" width="70%"></img>



애플 개발자 문서에 있는 예제에 대한 설명을 이 그림으로 대신한다.







우리는 앞서 프로토콜과 델리게이트의 차이점에 대해 의문을 가졌다.

프로토콜은 특정 프로토콜을 구현하는 한 어떤 클래스(or 구조체, 열거형..)를 사용하던 상관 없다. 

다시 말해, 어떤 프로토콜을 만들면 그것을 채택하는 모든 클래스들은 프로토콜에 선언되어 있는 메소드의 구현부만 작성하면 된다.



하지만 델리게이트 디자인 패턴은 특정 프로토콜을 따르는 클래스의 인스턴스를 지정하는 데 사용된다. 

```swift
class MyViewController: UIViewController, UICollectionViewDelegate, UICollectionViewDataSource {
	var collectionView: UICollectionView!		//#1
	
	override func viewDidLoad() 
		collectionView.delegate = self		//#2
		collectionView.dataSource = self
	}
}
```



그저 `UICollectionViewDelegate` 를 채택하고 메소드만 구현하는 것이 아니라,

 `collectionView ` 의 위임자를 self(`MyViewController`)로 지정하면서(#2), 델리게이트 객체의 프로퍼티를 통해 구현 메소드를 실행할 수 있게 된다. 







## Why use it?

델리게이트 패턴을 사용함으로 얻는 이점은, 앞에서 살펴봤듯이 객체가 분리된 방식으로(위임하므로) 대리자와 위임자간의 통신이 이루어져서 객체의 소유자의 구체적인 타입을 알 필요가 없다.

그래서 재사용 및 유지 관리가 훨씬 쉬운 코드를 작성할 수 있기 때문에 delegate pattern을 사용하는 것 같다.





### References

- https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html#//apple_ref/doc/uid/TP40008195-CH14
- https://www.swiftbysundell.com/articles/delegation-in-swift/
- https://www.appcoda.com/swift-delegate/
- https://stackoverflow.com/questions/5431413/difference-between-protocol-and-delegates







