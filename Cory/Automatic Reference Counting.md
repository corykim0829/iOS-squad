# Automatic Reference Counting

Reference counting은 오직 클래스의 인스턴스에만 적용된다. 구조체와 열거형은 reference타입이 아닌 value타입으로, 참조에 의해 저장되거나 전달되지 않는다.

ARC가 아직 사용중인 인스턴스의 할당을 해제하면, 인스턴스의 프로퍼티로 접근이나 인스턴의 메소드를 호출하는 것은 불가능하다. 실제로 그 인스턴스에 접근하려고 하면 앱의 충돌이 일어날 가능성이 크다.

이를 방지하기 위해, ARC는 현재 클래스 인스턴스를 참조하고 있는 많은 프로퍼티, 상수, 변수들을 추적한다 ARC는 존재하는 인스턴스에 활성화된 참조가 하나라도 있다면 그 인스턴스를 할당을 해제시키지 않는다.

이를 가능하게 하기 위해서 클래스 인스턴스를 프로퍼티, 상수, 변수에 할당할 때마다 해당 프로퍼티, 상수, 변수는 **강한 참조**를 인스턴스에 생성한다.

<br>

### Strong Reference Cycles Between Class Instances

강한 참조가 0으로 될 수 없는 코드가 있다. 두개의 클래스 인스턴스가 서로를 강한 참조를 하게 되면 계속해서 서로를 살아있게 한다. 이는 **강한 참조 순환**이라고 불린다.

강한 참조 순환은 클래스들 사이의 관계를 강한 참조가 아닌 **weak** 또는 **unowned 참조**를 정의함으로써 강한 참조를 해결할 수 있다.

#### strong reference cycle

아래 코드는 강한 참조가 생기는 예제이다. 

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

인스턴스 생성

```swift
var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")
```



<img src="https://docs.swift.org/swift-book/_images/referenceCycle01_2x.png">

```swift
john!.apartment = unit4A
unit4A!.tenant = john
```

<img src="https://docs.swift.org/swift-book/_images/referenceCycle02_2x.png">

`john`과 `unit4A` 변수에 의한 강한 참조를 끊어 reference count를 0으로 떨어트려도, 인스턴스들은 ARC에 의해 메모리에서 해제되지 않는다. 두 변수를 nil로 선언해줬을 때 두 인스턴스의 **deinitializer**는 호출된다. 이러한 strong reference cycle은 두 인스턴스가 할당 해제되는 것을 방해하며 앱의 **memory leak(메모리 누수)**를 발생시킨다.

```swift
john = nil
unit4A = nil
```

<img src="https://docs.swift.org/swift-book/_images/referenceCycle03_2x.png">

<br>

### Resolving Strong Reference Cycles Between Class Instances