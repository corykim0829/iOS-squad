# Delegate pattern in iOS



## What is delegate pattern?

> delegate 위임하다, 대표(자)

델리게이트 패턴은 디자인 패턴 중 하나로, 프로그램 안에서 어떤 객체를 대신하여 행동한다던가, 다른 객체와 협동하여 일할 수 있게끔 권한을 위임하는 패턴이다.

iOS에서 자주 사용하는 delegator인 UITableViewDelegate나 UICollectionViewDelegate의 타입은 프로토콜이다.

즉 위임받는 대리자(주로 사용자가 만든 컨트롤러 객체)가, 위임 받으려는 프로토콜을 채택하는 방식으로 위임의 과정이 이루어진다.







## How works it?

이 패턴에는 크게 두가지 역할이 존재한다.

**위임자** 와 **대리자** 가 바로 그것이다.

**위임자** 

- 위임자는 일반적으로 framework object이다.

- 대리자에 대한 참조를 유지한다.

- 적절한 시점에 대리자에게 메시지를 보낸다. (* 메시지 - 위임자가 다뤄왔거나, 방금 처리한 이벤트에 대한 알림)

- 대부분의 Cocoa framework 클래스의 위임자는 대리자에 의해 발생되는 알림의  관찰자로 자동으로 등록된다.

- 경우에 따라 위임자는 이벤트 처리 방법에 영향을 주는 값을 리턴할 수 있다. 

  

**대리자**

- 일반적으로 사용자가 만든 컨트롤러 객체이다.
- 대리자는 특정 알림 메시지를 받기 위해 프레임워크 클래스에서 선언한 (채택한 프로토콜의 메서드만?) 구현하면 된다.
- 대리자는 어플리케이션 안에 있는 다른 객체나 자신의 모양이나 상태를 업데이트 해 메시지에 응답한다.



여기서 기억해야 할 사항은 위임자와 대리자가 서로 메시지를 보내고 응답하면서 통신을 한다는 것이다.



애플 개발자 문서에 있는 예제를 함께 알아보자.



![스크린샷 2020-02-10 오전 11 19 09](https://user-images.githubusercontent.com/40784518/74116514-310a8600-4bf7-11ea-8be8-192e02a7d8bb.png)

위임자 `NSWindow` 는 Appkit 프레임워크 클래스의 인스턴스이다. 

프로그래머가 만들고 있는 custom view controller에서 `NSWindow` 의 일을 위임하는 `NSWindowDelegate` 프로토콜을 채택한다.

사용자가 왼쪽에 있는 창에서 '닫기' 상자를 클릭하면 `windowShowClose(_:)` 가 대리자에게 확인을 요청한다

custom view controller에서는 로직에 따라 Boolean 값을 리턴하고, NSWindow는 리턴 받은 값에 의해 제어된다.





<img src="https://user-images.githubusercontent.com/40784518/74138619-4bfbeb00-4c35-11ea-8510-a29f8dfc3e35.jpeg" width="70%"></img>





위의 그림과 같이 위임자는 대리자에게 할 일을 위임해주고 위임 객체에 어떤 변화가 있을 때, 

대리자에게 확인 요청을 한다. 그리고 대리자가 리턴하는 값에 따라 위임 객체의 동작을 제어하게 된다.





## Why use it?

델리게이트 패턴에 대해 알아보면서 프로토콜을 채택해서 사용한다고 말하면 될 것을, 왜 굳이 delegate pattern이라고 따로 만들어서 사용하는 이유에 대해 궁금했다.

프로토콜은 swift의 언어적인 특성이고, 델리게이트 패턴을 사용하기 위한 방법으로 프로토콜을 이용했다고 생각하면 될 것 같다.



```swift
protocol BossDelegate {
  func manageTasks()
}

struct Secretary: BossDelegate{
  func manageTasks() {
    print("오늘 할일이 산더미에요. 아자!")
  }
}

struct Assistance: BossDelegate{
  func manageTasks() {
    print("쉬엄쉬엄 하셔도 됩니다 하핫")
  }
}

struct Boss {
  var delegate: BossDelegate       //#1
}

var boss = Boss(delegate: Secretary()) //#2
boss.delegate = Assistance() //#3

boss.delegate.manageTasks()
```



위 코드를 보면 `Secretary` 와 `Assistance` 가 `BossDelegate` 프로토콜을 채택하고 있다.

`Secretary` 와 `Assistance` 둘 다 `manageTasks()` 를 구현하며 해당 프로토콜을 준수하는데, 메소드 내부의 상세 구현은 다르다.

`Boss` 에서 `delegate` 의 타입을 `Secretary` 나 `Assistance` 로 할 수도 있지만, `BossDelegate` 프로토콜로 지정함에 따라(#1) 해당 프로토콜을 채택하는 어떤 구조체(or 클래스 등)도 다 들어올 수 있게 된다.

 

이처럼 델리게이트 패턴을 사용함으로 얻는 이점은, 앞에서 살펴봤듯이 객체가 분리된 방식으로(위임하므로) 대리자와 위임자간의 통신이 이루어져서 객체의 소유자의 구체적인 타입을 알 필요가 없다.

그래서 재사용 및 유지 관리가 훨씬 쉬운 코드를 작성할 수 있기 때문에 delegate pattern을 사용하는 것 같다.





### References

- https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html#//apple_ref/doc/uid/TP40008195-CH14
- https://www.swiftbysundell.com/articles/delegation-in-swift/
- https://www.appcoda.com/swift-delegate/
- https://stackoverflow.com/questions/5431413/difference-between-protocol-and-delegates







