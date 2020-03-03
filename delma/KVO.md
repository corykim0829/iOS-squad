# KVO (Key - Value Observing)



## What is key-value observing?

변수가 변경될 때마다 코드가 실행되도록 코드를 변수에 첨부하는 기능이다. 모델과 뷰 등 논리적으로 분리된 앱 부분간에 변경사항을 전달하는데 유용하다.

실행 될 코드가 타입이 선언된 곳 바깥에 있다는 점만 제외하면, 프로퍼티 옵저버(willSet, didSet)과 유사하다.

KVO는 Objective-C 런타임에 의존하기 때문에, 순수한 swift 코드에서는 그리 좋지 않다. `NSObject`를  상속받은 `@objc`를 사용해야 하고, 각각의 프로퍼티에 `@objc dynamic` 표시를 해야한다.

<br>

<br>



## Annotate a Property for Key-Value Observing

`@objc`속성과 `dynamic` 한정어를 모두 추가해 KVO를 이용해 관찰하려는 속성을 표시한다.  아래 예제를 참고해보자

```swift
class MyObjectToObserve: NSObject {
  @objc dynamic var myDate = NSDate(timeIntervalsince1970: 0)	//1970
  func updateDate() {
    myDate = myDate.addingTimeInterval(Double(2 << 30)) 	//Adds about 68 years
  }
}
```

<br>

<br>



## Define and Observer

옵저버 클래스의 인스턴스는 하나 이상의 속성에 대한 변경사항에 대한 정보를 관리한다. 옵저버를 만들 때, `observe(_:options:changeHandler:)`메소드와 관찰하려는 속성을 나타내는 key path를 호출함으로 관찰을 시작한다. 

아래의 예에서key path  `\.ojbectToObserve.myDate` 는 `MyObjectToObserve`의 `myDate`속성을 참조한다



```swift
class MyObserver: NSObject {
    @objc var objectToObserve: MyObjectToObserve
    var observation: NSKeyValueObservation?
    
    init(object: MyObjectToObserve) {
        objectToObserve = object
        super.init()
        
        observation = observe(
            \.objectToObserve.myDate,
            options: [.old, .new]
        ) { object, change in
            print("myDate changed from: \(change.oldValue!), updated to: \(change.newValue!)")
        }
    }
  
  	deinit {
      observation.removeAll()
    }
}
```

`NSKeyValueObservedChange` 인스턴스의 `oldValue`,`newValue`프로퍼티를 통해서관찰중인 속성의 변경된 사항을 확인할 수 있다.

속성이 어떻게 변경되었는지 알 필요가 없는 경우, `options` 매개변수를 생략하면 된다. `options`매개변수를 생략하는 경우,`oldValue`,`newValue`속성의 값은 nil이 된다



관찰자의 역할이 끝나면 이를 꼭 해제시켜준다.

<br>

<br>



## Associate the Observer with the Property to Observe

옵저버의 이니셜라이저에 관찰하려는 객체(observed)를 등록함으로써 observer - observed 간에 연결이 이뤄진다

```swift
let observed = MyObjectToObserve()
let observer = MyObserver(object: observed)
```

<br>

<br>



## Respond to Property Change

key-value observing을 사용하도록 설정된 객체(앞서 계속 observed라고 얘기한 것)는 속성이 변경된 것에 대해 옵저버에게 알린다. 

아래 예시는 `updateDate` 메소드를 호출함으로써 `myDate` 속성을 변경한다. 이 메소드는 옵저버의 변경 핸들러를 자동적으로 호출한다. 

옵저버의 변경 핸들러에 관찰 대상의 값이 변경될 때마다 실행되어야 할 일들을 넣어 자동적으로 실행할 수 있게 할 수 있다.



```swift
observed.updateDate() // Triggers the observer's change handler.
// Prints "myDate changed from: 1970-01-01 00:00:00 +0000, updated to: 2038-01-19 03:14:08 +0000"
```











### References



### References

- https://developer.apple.com/documentation/swift/cocoa_design_patterns/using_key-value_observing_in_swift
- https://www.hackingwithswift.com/example-code/language/what-is-key-value-observing