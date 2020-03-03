> apple article 번역본임을 알려드립니다. 
<br>출처 : <https://developer.apple.com/documentation/swift/cocoa_design_patterns/using_key-value_observing_in_swift>

# Swift에서 Key-Value Observing 사용하기

* 다른 객체의 프로퍼티들의 변화에 대해 객체에 알립니다.  

## 개요

key-value observing은 다른 객체의 프로퍼티 변경에 대해 객체에 알리기 위해 사용하는 Cocoa 프로그래밍 패턴입니다. 
Model과 View 등 논리적으로 분리된 앱 부분 간에 변경 사항을 전달하는 데 유용합니다. NSObject에서 상속된 클래스에서만 key-value-observing 을 사용할 수 있습니다.

## key-value observing을 위한 프로퍼티에 annotate 달기

@objc 프로퍼티와 dynamic modifier를 모두 사용하여 key-value observing을 통해 관찰하려는 프로퍼티을 표시하십시오. 
아래 예제는 관찰할 수 있는 myDate 프로퍼티를 가진 MyObjectToObserve 클래스를 정의합니다. (=> 관찰당하는 놈)

```swift
class MyObjectToObserve: NSObject {
    @objc dynamic var myDate = NSDate(timeIntervalSince1970: 0) // 1970
    func updateDate() {
        myDate = myDate.addingTimeInterval(Double(2 << 30)) // Adds about 68 years.
    }
}
```

## Observer 정의하기 

옵저버 클래스의 인스턴스는 하나 이상의 프로퍼티에 대한 변경 사항에 대한 정보를 관리합니다.
옵저버를 만들 때 관찰하려는 프로퍼티을 참조하는 키 경로(key path)와 함께 observe (_ : options : changeHandler :) 메서드를 호출하여 관찰(observation)을 시작합니다.

아래 예에서 \ .objectToObserve.myDate 키 경로(key path)는 MyObjectToObserve의 myDate 프로퍼티를 참조합니다.

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
}
```

NSKeyValueObservation 인스턴스의 oldValue 및 newValue 프로퍼티를 사용하여 관찰중인 프로퍼티에 대해 변경된 내용을 볼 수 있습니다.

프로퍼티가 어떻게 변경되었는지 알 필요가 없으면 options 매개 변수를 생략하십시오. options 매개 변수를 생략하면 new 및 old 프로퍼티 값이 저장되지 않으므로 oldValue 및 newValue 속성이 nil이됩니다.


Associate the Observer with the Property to Observe

## Observer를 관찰할 프로퍼티와 연결하기 

객체를 observer를 생성할 때 매개변수로 전달하여 관찰하려는 프로퍼티를 observer와 연결합니다.

```swift
let observed = MyObjectToObserve()
let observer = MyObserver(object: observed)
```

## 프로퍼티 변화에 응답 

위에서 관찰한 것처럼 key-value observation을 사용하도록 설정된 객체는 observer에게 프로퍼티 변경에 대해 알립니다. 
아래 예제는 updateDate 메소드를 호출하여 myDate 프로퍼티를 변경합니다. 
**이 메소드 호출은 자동적으로 observer의 change 핸들러를 트리거합니다.**

```swift
observed.updateDate() // Triggers the observer's change handler.
// Prints "myDate changed from: 1970-01-01 00:00:00 +0000, updated to: 2038-01-19 03:14:08 +0000"
```

위의 예는 Date의 new value과 old value을 모두 인쇄하여 프로퍼티 변경에 응답합니다.
