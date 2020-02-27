# Automatic Reference Counting

출처 : <https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID52>

Swift는 ARC (Automatic Reference Counting)를 사용하여 앱의 메모리 사용량을 추적하고 관리합니다.
대부분의 경우 이는 Swift에서 메모리 관리가 “작동하는 것”이므로 메모리 관리에 대해 스스로 생각할 필요는 없습니다.
ARC는 해당 인스턴스가 더 이상 필요하지 않을 때 클래스 인스턴스가 사용하는 메모리를 자동으로 해제합니다.

그러나 경우에 따라 메모리를 관리하기 위해 ARC에 당신의 코드 부분 간의 관계에 대한 자세한 정보가 필요합니다.
이 장에서는 이러한 상황에 대해 설명하고 ARC가 모든 앱 메모리를 관리하는 방법을 보여줍니다.

참조 카운팅은 클래스 인스턴스에만 적용됩니다.
struct와 enum는 reference type이 아닌 value type이며 참조로 저장 및 전달되지 않습니다.

# How ARC Works

클래스의 새 인스턴스를 만들 때마다 ARC는 해당 인스턴스에 대한 정보를 저장하기 위해 메모리 청크를 할당합니다.
이 메모리에는 해당 인스턴스와 관련된 저장된 프로퍼티 값과 함께 인스턴스 유형에 대한 정보가 있습니다.

또한 인스턴스가 더 이상 필요하지 않은 경우 ARC는 해당 인스턴스가 사용하는 메모리를 비워서 다른 용도로 메모리를 사용할 수 있습니다. 
이렇게 하면 클래스 인스턴스가 더 이상 필요하지 않을 때 메모리에서 공간을 차지하지 않습니다.

그러나 ARC가 여전히 사용중인 인스턴스를 할당 해제하면 해당 인스턴스의 프로퍼티에 액세스하거나 해당 인스턴스의 메서드를 호출 할 수 없습니다. 실제로 인스턴스에 액세스 하려고 하면 앱이 중단될 가능성이 높습니다.

인스턴스가 여전히 필요한 동안 사라지지 않도록 ARC는 현재 각 클래스 인스턴스를 참조하는 프로퍼티, 상수 및 변수의 수를 추적합니다.
**ARC는 해당 인스턴스에 대한 하나 이상의 활성 참조가 여전히 존재하는 한 인스턴스 할당을 해제하지 않습니다.**

이를 가능하게 하기 위해 클래스 인스턴스를 프로퍼티, 상수 또는 변수에 할당 할 때마다 해당 프로퍼티, 상수 또는 변수가 인스턴스를 강력하게 참조합니다.
참조는 "강력한" 참조라고하며,이 인스턴스는 해당 인스턴스를 확실하게 유지하고 강력한 참조가 남아있는 한 할당을 해제 할 수 없기 때문입니다.

# ARC in Action

다음은 Automatic Reference Counting 작동 방식의 예입니다. 이 예제는 Person이라는 간단한 클래스로 시작합니다. 이 클래스는 name이라는 저장된 상수 프로퍼티을 정의합니다.

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}
```
Person 클래스에는 인스턴스의 name 프로퍼티을 설정하고 초기화가 진행 중임을 나타내는 메시지를 print하는 이니셜 라이저가 있습니다. 
Person 클래스에는 클래스의 인스턴스가 할당 해제 될 때 메시지를 print하는 초기화 해제 기능도 있습니다.


다음 코드 뭉치는 Person? 유형의 세 가지 변수를 정의합니다.이 변수는 후속 코드 뭉치에서 새 Person 인스턴스에 대한 다중 참조를 설정하는 데 사용됩니다.

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?
```

이제 새로운 Person 인스턴스를 만들어 다음 세 변수 중 하나에 할당 할 수 있습니다.

```swift
reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
```
Person 클래스의 이니셜 라이저를 호출 할 때 "John Appleseed is being initialized" 메시지가 print됩니다. 초기화가 완료되었음을 확인합니다.

새 Person 인스턴스가 reference1 변수에 지정되었으므로 이제 reference1에서 새 Person 인스턴스에 대한 강력한 참조가 있습니다.
최소한 하나의 강력한 참조가 있기 때문에 ARC는 이 Person이 메모리에 유지되고 할당 해제되지 않도록 합니다.

동일한 Person 인스턴스를 **두 개의 변수에 더 할당하면** 해당 인스턴스에 대한 **두 개의 강력한 참조가 설정**됩니다.

```swift
reference2 = reference1
reference3 = reference1
```
이제 이 단일 Person 인스턴스에 대한 세 개의 강력한 참조가 있습니다.

두 개의 변수에 nil을 지정하여이 두 개의 강력한 참조 (원본 참조 포함)를 깨면 하나의 강한 참조가 남고 Person 인스턴스는 할당 해제되지 않습니다.
```swift
reference1 = nil
reference2 = nil
```

ARC는 세 번째이자 강력한 참조가 깨질 때까지 Person 인스턴스를 할당 해제하지 않습니다.
세번째가 깨지는 시점에서 더 이상 Person 인스턴스를 사용하지 않는 것이 분명합니다.
```swift
reference3 = nil
// Prints "John Appleseed is being deinitialized"
```

# Strong Reference Cycles Between Class Instances

위의 예에서 ARC는 생성 한 새로운 Person 인스턴스에 대한 참조 수를 추적하고 더 이상 필요하지 않은 경우 해당 Person 인스턴스를 할당 해제 할 수 있습니다.
그러나 클래스의 인스턴스가 절대 Reference Count가 0이 되지 않아 앱이 끝날때까지 메모리에 해제되지 않는 경우가 있습니다. 
이는 두 클래스 인스턴스가 서로에 대한 강력한 참조를 보유하여 각 인스턴스가 다른 인스턴스를 계속 유지하는 경우에 발생할 수 있습니다. 이를 강력한 참조 주기(strong reference cycle)라고 합니다.

클래스 간의 관계 중 일부를 강력한 참조가 아닌 약하거나 unowned 참조로 정의하여 강력한 참조 주기(strong reference cycle)를 해결합니다.
이 프로세스는 클래스 인스턴스 간의 강력한 참조주기 해결에 설명되어 있습니다. 그러나 강력한 참조주기를 해결하는 방법을 배우기 전에 이러한 cycle이 어떻게 발생하는지 이해하는 것이 좋습니다.

다음은 우연하게 강력한 참조주기를 생성하는 방법에 대한 예입니다. 이 예에서는 Person 및 Apartment라는 두 개의 클래스를 정의합니다. Apartment 클래스는 Apartment 블록과 해당 거주자를 모델링합니다.

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
모든 Person 인스턴스에는 String 유형의 name 프로퍼티과 초기에 nil 인 선택적 Apartment 프로퍼티이 있습니다. Person이 항상 Apartment를 가지고 있지는 않기 때문에 Apartment 프로퍼티은 선택 사항입니다.
마찬가지로 모든 Apartment 인스턴스에는 String 유형의 'unit' 프로퍼티이 있으며 초기에는 nil 인 선택적 'tenant'프로퍼티이 있습니다. Apartment에는 항상 tenant이 있을 수 없으므로 tenant 프로퍼티은 선택 사항입니다.
이 두 클래스 모두 deinitializer를 정의하여 해당 클래스의 인스턴스가 해제되었다는 사실을 print합니다. 이를 통해 Person 및 Apartment 인스턴스가 예상대로 할당 해제되고 있는지 확인할 수 있습니다.

이 다음 코드 뭉치는 john 및 unit4A라는 선택적 유형의 두 변수를 정의합니는. 
이 변수는 아래의 특정 Apartment 및 개인 인스턴스로 설정됩니다. 
이 두 변수는 모두 옵션이기 때문에 초기 값이 nil입니다.

```swift
var john: Person?
var unit4A: Apartment?
```
이제 특정 Person 인스턴스 및 Apartment 인스턴스를 작성하고 이러한 새 인스턴스를 john 및 unit4A 변수에 지정할 수 있습니다.

```swift
john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")
```

다음은 이 두 인스턴스를 만들고 할당한 후 강력한 참조가 어떻게 보이는지 보여줍니다. 
john 변수는 이제 새로운 Person 인스턴스에 대한 강력한 참조를 가지며 unit4A 변수는 새로운 Apartment 인스턴스에 대한 강력한 참조를 갖습니다.

![refer_count](https://user-images.githubusercontent.com/38216027/74526438-bc24ac80-4f66-11ea-9b04-a761148f7188.png)

이제 두 인스턴스를 서로 연결하여 그 Person이 Apartment를 소유하고 Apartment가 tenant을 갖도록 할 수 있습니다.
```swift
john!.apartment = unit4A
unit4A!.tenant = john
```

다음은 두 인스턴스를 서로 연결 한 후 강력한 참조가 어떻게 나타나는지 보여줍니다.

![referenceCycle02_2x](https://user-images.githubusercontent.com/38216027/74527040-170ad380-4f68-11ea-8cb3-138fcc7e8b2b.png)

불행히도 이 두 인스턴스를 연결하면 두 인스턴스간에 강력한 참조주기가 생성됩니다. Person 인스턴스는 이제 Apartment 인스턴스에 대한 강력한 참조를 가지며 Apartment 인스턴스는 Person 인스턴스에 대한 강력한 참조를 갖습니다. 따라서 john 및 unit4A 변수가 보유한 강력한 참조를 해제해도 참조 카운트가 0으로 떨어지지 않으며 ARC에서 인스턴스를 할당 해제하지 않습니다.

```swift
john = nil
unit4A = nil
```

이 두 변수를 nil로 설정해도 초기화 해제기(deinit)u가 호출되지 않은 것을 주목하십시오.
강력한 참조주기는 Person 및 Apartment 인스턴스가 할당 해제되지 않도록 하여 앱에서 메모리 누수(memory leak)를 유발합니다.

다음은 john 및 unit4A 변수를 nil로 설정 한 후 강력한 참조가 어떻게 보이는지 보여줍니다.

![referenceCycle03_2x](https://user-images.githubusercontent.com/38216027/74527595-7c12f900-4f69-11ea-87c9-287ea900fdc4.png)
Person 인스턴스와 Apartment 인스턴스 사이의 강력한 참조는 유지되며 깨지지 않습니다.

# Resolving Strong Reference Cycles Between Class Instances

Swift는 클래스 타입의 프로퍼티을 작업할 때 weak reference 와 owned reference 이 두 가지 방법으로 강력한 참조주기를 해결합니다.

weak 참조와 unowned 참조는 참조 cycle의 한 인스턴스가 다른 인스턴스를 강력하게 유지하지 않고 다른 인스턴스를 참조할 수 있도록 합니다. 
그러면 인스턴스는 강력한 참조 cycle를 만들지 않고 서로를 참조 할 수 있습니다.

다른 인스턴스의 수명이 짧을 때 (즉, 다른 인스턴스를 먼저 할당 해제 할 수 있는 경우) Weak 참조를 사용하십시오. 
위의 Apartment 예에서, Apartment가 일생 동안 어떤 시점에 tenant도 가질 수 없는 것이 적절하므로 이 경우 기준이 weak 참조가 참조 cycle를 깨뜨리는 적절한 방법입니다. 
반대로, 다른 인스턴스의 수명이 동일하거나 수명이 길면 unowned 참조를 사용하십시오.

weak 참조는 참조하는 인스턴스를 강력하게 유지하지 않는 참조이므로 ARC가 참조 된 인스턴스의 처리를 중지하지 않습니다.
이 동작은 참조가 강력한 참조 cycle의 일부가 되는 것을 방지합니다. weak 키워드를 프로퍼티 또는 변수 선언 앞에 배치하여 weak 참조를 나타냅니다.

weak 참조는 참조하는 인스턴스를 강력하게 유지하지 않기 때문에 weak 참조가 여전히 참조하는 동안 해당 인스턴스의 할당이 해제될 수 있습니다.
**따라서 ARC는 참조하는 인스턴스가 할당 해제될 때 자동으로 weak 참조를 nil로 설정합니다.
또한 weak 참조는 런타임에 값을 nil로 변경해야 하므로 항상 optional 상수가 아닌 optional 변수로 선언됩니다.**

```swift
weak 참조는 var 이어야하고, optional 이어야 합니다.  
```

다른 optional 값과 마찬가지로 weak 참조에 값이 있는지 확인할 수 있으며, 더 이상 존재하지 않는 유효하지 않은 인스턴스에 대한 참조로 끝나지 않습니다.(you will never end up with a reference to an invalid instance that no longer exists.)

> note : ARC가 weak 참조가 nil로 할당되면 프로퍼티 관찰자는 호출되지 않습니다.

아래의 예는 위의 Person 및 Apartment 예와 동일하지만 한 가지 중요한 차이점이 있습니다.
이번에는 Apartment 유형 tenant 프로퍼티이 weak 참조로 선언되었습니다.

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
    weak var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

두 변수 (john 및 unit4A)의 강력한 참조와 두 인스턴스 간의 링크는 이전과 같이 작성됩니다.

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```
다음은 두 인스턴스를 서로 연결 한 후 참조가 어떻게 보이는지 보여줍니다.
Person 인스턴스는 여전히 Apartment 인스턴스에 대한 강력한 참조를 갖지만 이제 Apartment 인스턴스는 Person 인스턴스에 대한 참조가 약합니다. 
즉, john 변수가 보유한 강력한 참조를 nil로 설정하여 중단하면 Person 인스턴스에 대한 더 이상 강력한 참조가 없습니다.

```swift
john = nil
// Prints "John Appleseed is being deinitialized"
```
Person 인스턴스에 대한 참조가 더 이상 없으므로 할당이 해제되고 tenant 프로퍼티이 nil로 설정됩니다.

![weakReference02_2x](https://user-images.githubusercontent.com/38216027/74532403-7d94ef00-4f72-11ea-9171-ad068a9ada3b.png)

Apartment 인스턴스에 대한 유일한 강력한 참조는 unit4A 변수에서입니다. 
강력한 참조를 깨뜨리면 더 이상 Apartment 인스턴스에 대한 참조가 없습니다.

```swift
unit4A = nil
// Prints "Apartment 4A is being deinitialized"
```
더 이상 Apartment 인스턴스에 대한 참조가 없으므로 할당 해제됩니다.

![weakReference03_2x](https://user-images.githubusercontent.com/38216027/74579629-427cd500-4fdf-11ea-96c3-429c1e99b298.png)

> NOTE : GC을 사용하는 시스템에서는 메모리 압력이 GC을 트리거 할 때만 강한 참조가 없는 객체가 할당 해제되기 때문에, weak 포인터는 때때로 간단한 캐싱 메커니즘을 구현하는데 사용되는 경우가 있었습니다. 
그러나 ARC를 사용하면 마지막 강력한 참조가 제거되자마자 값이 할당 해제되어 weak 참조가 이러한 목적에 적합하지 않게됩니다.


* 질문 : C ++ 11의 스마트 포인터를 연구하기 시작했지만 std :: weak_ptr을 유용하게 사용하지 못합니다. std :: weak_ptr이 유용 / 필요할 때 누군가 말해 줄 수 있습니까?
<br><br> 
좋은 예는 캐시입니다.
최근에 액세스 한 객체의 경우 메모리에 유지하려면 객체에 대한 강력한 포인터를 갖습니다. 주기적으로 캐시를 스캔하고 최근에 액세스하지 않은 객체를 결정합니다. 메모리에 저장할 필요가 없으므로 강력한 포인터를 제거하십시오.
그러나 해당 객체가 사용 중이고 다른 코드에 강한 포인터가 있으면 어떻게 될까요? 캐시가 객체에 대한 유일한 포인터를 제거하면 다시는 찾을 수 없습니다. 따라서 캐시는 메모리에 남아 있는지 확인해야하는 객체에 대한 weak 포인터를 유지합니다.
이것은 weak 포인터가 하는 것입니다. 여전히 객체가 있으면 객체를 찾을 수는 있지만 다른 물체가 필요하지 않으면 그대로 두지 않습니다.

# Unowned References

weak 참조처럼, unowned 참조는 참조하는 인스턴스를 강력하게 유지하지 않습니다.
그러나 약한 참조와 달리 소유하지 않은 참조는 다른 인스턴스의 수명이 동일하거나 수명이 길면 사용됩니다. 
unowned 키워드를 프로퍼티 또는 변수 선언 앞에 배치하여 unowned 참조를 나타냅니다.

unowned 참조에는 항상 값이 있어야합니다. 결과적으로 ARC는 unowned 참조 값을 nil로 설정하지 않습니다. 즉, unowned 참조는 non-optional 타입을 사용하여 정의됩니다.

> IMPORTANT
<br>참조가 항상 할당 해제되지 않은 인스턴스를 참조하는 경우에만 unowned 참조를 사용하십시오.
<br><br>해당 인스턴스가 할당 해제 된 후 unowned 참조의 값에 접근하려고 하면 런타임 오류가 발생합니다.

다음 예제는 Customer 및 CreditCard 의 두 클래스를 정의합니다. 
이 두 클래스는 은행 고객과 해당 고객의 신용 카드를 모델링합니다. 
이 두 클래스는 각각 다른 클래스의 인스턴스를 프로퍼티으로 저장합니다. 
이 관계는 강력한 참조주기(Strong Reference Cycle)를 만들 가능성이 있습니다.

Customer과 CreditCard의 관계는 위의 weak 참조 예에서 볼 수있는 Apartment와 Person의 관계와 약간 다릅니다. 이 데이터 모델에서 고객은 신용 카드를 보유하거나 보유하지 않을 수 있지만 신용 카드는 항상 고객과 연결됩니다.
CreditCard 인스턴스는 참조한 Customer의 수명보다 오래 지속되지 않습니다. 이를 나타내기 위해 Customer 클래스에는 선택적 카드 프로퍼티이 있지만 CreditCard 클래스에는 소유되지 않은 (non-optional) 고객 프로퍼티이 있습니다.

또한 새 CreditCard 인스턴스는 숫자 값과 고객 인스턴스를 custom CreditCard 초기화 프로그램에 전달하여만 작성할 수 있습니다. 이렇게하면 CreditCard 인스턴스가 작성 될 때 CreditCard 인스턴스에 항상 연관된 고객 인스턴스가 있습니다.

**신용 카드에는 항상 고객이 있기 때문에 강력한 참조 cycle를 피하기 위해 고객 프로퍼티을 unowned 참조로 정의합니다.**

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}
```

> NOTE : CreditCard 클래스의 숫자 프로퍼티은 32 비트 및 64 비트 시스템 모두에 16 자리 카드 번호를 저장할 수있을 정도로 큰 숫자 프로퍼티의 용량을 보장하기 위해 Int가 아닌 UInt64 유형으로 정의됩니다.


다음 코드 뭉치는 john이라는 optional Customer 변수를 정의합니다.이 변수는 특정 고객에 대한 참조를 저장하는 데 사용됩니다.
이 변수는 optional type이기 때문에 초기 값은 nil입니다.

```swift
var john: Customer?
```

이제 고객 인스턴스를 생성하고 이를 사용하여 해당 고객의 카드 프로퍼티으로 새 CreditCard 인스턴스를 초기화하고 할당할 수 있습니다.

```swift
john = Cutomer(name : "John Appleseed")
john.card = CreditCard(number : 1234_5678_9012_3456 , customer : john)
```

다음은 두 인스턴스를 연결 한 후 참조 모양입니다.

![unownedReference01_2x](https://user-images.githubusercontent.com/38216027/74582377-381e0380-4ffe-11ea-99dc-0f35e7ecb433.png)

Customer 인스턴스는 이제 CreditCard 인스턴스에 대한 강력한 참조를 가지며 CreditCard 인스턴스는 Customer 인스턴스에 대한 unowned 참조를 갖습니다.

unowned 고객 참조이기 때문에, john 변수가 보유한 강력한 참조를 깨 뜨리면 Customer 인스턴스에 대한 더 이상 강력한 참조가 없습니다.

![unownedReference02_2x](https://user-images.githubusercontent.com/38216027/74582397-7d423580-4ffe-11ea-9b96-94775a01fdcc.png)

Customer 인스턴스에 대한 참조가 더 이상 없으므로 할당이 해제됩니다. 이후에는 CreditCard 인스턴스에 대한 강력한 참조가 더 이상 없으므로 CreditCard 인스턴스도 할당이 해제됩니다.

```swift
john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
```

위의 마지막 코드 뭉치는 john 변수가 nil로 설정된 후 Customer 인스턴스 및 CreditCard 인스턴스의 이니셜 라이저가 "초기화되지 않은" 메시지를 print함을 보여줍니다.

> NOTE : 위의 예는 안전한 소유되지 않은 참조를 사용하는 방법을 보여줍니다.
Swift는 또한 성능상의 이유로 런타임 안전 검사를 비활성화해야하는 경우 안전하지 않은 참조를 제공합니다. **따라서 모든 안전하지 않은 작업과 마찬가지로 안전을 위해 해당 코드를 확인해야 할 책임이 있습니다.(한마디로 뻑안나게 unowned 키워드를 잘써라)** unowned (unsafe)를 작성하여 안전하지 않은 소유되지 않은 참조를 나타냅니다. 참조하는 인스턴스가 할당 해제 된 후 안전하지 않은 소유되지 않은 참조에 액세스하려고하면 프로그램은 인스턴스가 있던 메모리 위치 (안전하지 않은 작업)에 액세스하려고 시도합니다.(그리고 뻑이 납니다.)

# Unowned References and Implicitly Unwrapped Optional Properties

위의 weak 참조와 unowned 참조에 대한 예는 위의 두 가지 예시처럼 strong reference cycle을 깨야할 필요성이 있는 시나리오에서 교과서처럼 쓰이는 방법입니다.

Person 과 Apartment 의 예는 nil이 허락되는(즉 optional 타입인) 두 가지 프로퍼티가 strong reference cycle을 가질 잠재성이 있는 상황을 보여줍니다.
따라서 이 시나리오는 weak 참조로 해결하는 것이 매우 적절합니다. (둘 다 옵셔널?인 경우에는 weak를 써라)

Customer 과 CreditCard의 예는 하나의 프로퍼티가 nil이 될 수 있고, 또 다른 프로퍼티가 nil이 될 수 없는 경우에 strong reference cycle을 가질 잠재성 있는 상황을 보여줍니다. 
따라서 이 시나리오는 unowned 참조로 해결하는 것이 매우 적절합니다.(하나는 옵셔널?이고, 하나는 옵셔널 타입이 아닌 경우에 unowned를 써라)

그러나, **여기에는 세번째 시나리오가 있습니다. 두 개의 프로퍼티가 항상 값을 가져야 하고 , 두 개의 프로퍼티 한번 초기화된 이후에는 nil이 절대로 되선 안되는 경우입니다.**
<br>이 시나리오에서는,  이 시나리오에서는 한 클래스의 unowned 프로퍼티를 다른 클래스의 **옵셔널!** 타입의 프로퍼티와 결합하는 것이 좋습니다.

이를 통해 초기화가 완료되면 strong reference cycle를 피하면서 두 프로퍼티 모두에 직접 액세스 할 수 있습니다 (without optional unwrapping). 이 섹션에서는 그러한 관계를 설정하는 방법을 보여줍니다.


아래 예제는 Country 및 City 의 두 클래스를 정의하며 각 클래스는 다른 클래스의 인스턴스를 프로퍼티로 저장합티다. 이 데이터 모델에서 모든 국가에는 항상 capital city가 있어야 하며 모든 city는 항상 country에 속해야 합니다. 이를 나타내기 위해 Country 클래스에는 capitalCity 프로퍼티가 있고 City 클래스에는 country 프로퍼티가 있습니다.

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

두 클래스 사이의 상호 의존성을 설정하기 위해 City의 초기화 프로그램은 Country 인스턴스를 가져 와서 이 인스턴스를 country 프로퍼티에 저장합니다.
<br>City의 이니셜 라이저는 Country의 이니셜 라이저 내에서 호출됩니다. 
그러나 `2단계 초기화`에 설명 된대로 Country의 이니셜 라이저는 새로운 Country 인스턴스가 완전히 초기화 될 때까지 자체를 City 이니셜 라이저로 전달할 수 없습니다.

이 요구사항에 대처하기 위해 Country의 capitalCity 프로퍼티를 암시적으로 unwraped 인 optional 프로퍼티로 선언하고 타입 키워드 끝에 느낌표가 표시됩니다(City!).
즉, 다른 옵션과 같이 capitalCity 프로퍼티의 기본값은 nil이지만 암시적으로 unwraped optional에 설명 된대로 값을 unwrap하지 않고도 액세스 할 수 있습니다.

capitalCity의 default 값은 nil이므로 Country 인스턴스가 **initializer 내에서 name 프로퍼티을 설정하면 새 Country 인스턴스가 완전히 초기화된 것으로 간주됩니다.** 
이는 Country initializer가 name 프로퍼티가 설정되는 즉시 암시적인 자신의 프로퍼티를 참조하고 pass할 수 있음을 의미합니다. 
따라서 Country initializer는 Country initializer가 **자신의 capitalCity 프로퍼티를 설정할 때 City initializer의 매개변수 중 하나로 self를 전달할 수 있습니다.**

이 모든 것이 strong reference cycle를 만들지 않고 단일 문(single statement)으로 Country 및 City 인스턴스를 만들 수 있으며 느낌표를 사용하여 옵션 값을 풀지 않고도 capitalCity 프로퍼티에 직접 액세스 할 수 있습니다.

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"
```

In the example above, the use of an implicitly unwrapped optional means that all of the two-phase class initializer requirements are satisfied. The capitalCity property can be used and accessed like a non-optional value once initialization is complete, while still avoiding a strong reference cycle.

위의 예시에서, implicitly unwrapped optional 타입(옵셔널!)의 사용은 2단계 클래스 초기화의 요구사항을 모두 만족합니다. capitalCity 프로퍼티는 한번 초기화가 완벽하게 되면 non-optional 타입처럼 사용되고 접근될 수 있습니다, strong reference cycle을 피하면서 말이죠. 
