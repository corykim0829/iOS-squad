# Automatic Reference Counting



Swift는 ARC를 사용해 앱의 메모리 사용량을 추적하고 관리한다. 대부분의 경우 메모리 관리란 swift에서 '작동하는 것'이므로, 사용자가 메모리 관리에 대해 생각하지 않아도 된다.

ARC는 해당 인스턴스가 더 이상 필요하지 않을 때 클래스 인스턴스가 사용하는 메모리를 자동으로 해제한다.

그러나 몇가지 경우에는 ARC가 코드 간의 관계에 관해 더 많은 정보를 요구한다. 

ARC는 클래스 인스턴스에만 적용된다. 구조체와 열거형은 참조 유형이 아닌 값 유형이므로 참조로 저장,전달되지 않는다.



ARC와 가비지 컬렉션의 차이

| 메모리 관리 기법 | ARC                                                          | 가비지 컬렉션                                                |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 참조 카운팅 시점 | 컴파일 시                                                    | 프로그램 동작 중                                             |
| 장점             | - 컴파일 당시 이미 인스턴스의 해제 시점이 정해져 있어, 인스턴스가 언제 메모리에서 해제될지 예측할 수 있다. <br />- 메모리 관리를 위한 시스템 자원을 추가할 필요가 없다. | - 상호 참조 상황 등의 복잡한 상황에서도 인스턴스를 해제할 수 있는 가능성이 더 높다. |
| 단점             | - ARC의 작동 규칙을 모르고 사용하면 인스턴스가 메모리에서 영원히 해제되지 않을 가능성이 있다. | - 프로그램 동작 외에 메모리 감시를 위한 추가 자원이 필요하므로 한정적인 자원 환경에서는 성능 저하가 발생할 수 있다.<br /> - 명확한 규칙이 없기 때문에 인스턴스가 정확히 언제 메모리에서 해제될지 예측하기 어렵다. |





## ARC 작동 방식

클래스의 새 인스턴스를 만들 때매다 ARC는 해당 인스턴스에 대한 정보를 저장하기 위해 메모리 chunk를 할당한다.  이 메모리에는 해당 인스턴스와 관련된 저장 속성 값과 함께 인스턴스 유형에 대한 정보가 있다.

인스턴스가 더 이상 필요하지 않은 경우 ARC는 해당 인스턴스가 사용하는 메모리를 비워 다른 용도로 사용할 수 있다. 이렇게 하면 클래스 인스턴스가 더이상 필요하지 않을 때 메모리에서 공간을 차지하지 않는다.

그러나 ARC가 사용중인 인스턴스를 해제하면 해당 인스턴스의 속성에 접근하거나 해당 인스턴스의 메서드를 호출할 수 없다. 실제로 인스턴스에 액세스하려고 하면 앱이 중단 될 가능성이 높다.

인스턴스가 필요한동안 사라지지 않도록 ARC는 현재 각 클래스 인스턴스를 참조하는 속성, 상수 및 변수의 수를 추적한다. ARC는 해당 인스턴스에 대한 하나 이상의 참조가 존재하는 한 인스턴스 할당을 해제하지 않는다.

이를 가능하게 하기 위해 클래스 인스턴스를 속성, 상수 또는 변수에 할당 할 때마다 해당 속성, 상수 또는 변수가 인스턴스를 *강력하게* 참조한다. 참조는 "강한" 참조라고 하며, 이 인스턴스는 해당 인스턴스를 확실하게 유지하고 강한 참조가 남아있는 한 할당을 해제할 수 없다.



## 강한 참조

인스턴스는 참조 횟수가 0이 되는 순간 메모리에서 해제되는데, 인스턴스를 다른 인스턴스의 프로퍼티나 변수, 상수 등에 할당할 때 강한참조를 사용하면 참조 횟수가 1 증가한다. 또, 강한참조를 사용하는 프로퍼티, 변수, 상수 등에 nil을 할당하면 원래 자신에게 할당되어있던 인스턴스의 참조 횟수가 1 감소한다.

참조의 기본은 강한참조이므로 클래스 타입의 프로퍼티, 변수, 상수 등을 선언할 때 별도의 식별자를 명시하지 않으면 강한 참조를 한다. 

```swift
class Person{
  let name: String
  
  init(name: String) {
    self.name = name
    print("\(name) is being initialized")
  }
  
  deinit {
    print("\(name) is being deinitialized")
  }
}

var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "delma")
//delma is being initialized
//인스턴스 참조 횟수: 1

reference2 = reference1 //인스턴스 참조 횟수: 2
reference3 = reference1 //인스턴스 참조 횟수: 3

reference3 = nil  //인스턴스의 참조 횟수: 2
reference2 = nil  //인스턴스의 참조 횟수: 1
referencel = nil  //인스턴스의 참조 횟수: 0
//delma is being deinitialized
```



<br>

### 강한참조 순환 문제

인스턴스끼리 서로 강한참조를 할 때 강한참조 순환(Strong Reference Cycle)이 일어날 수 있다. 

```swift
class Person {
	let name: String 
  
  init(name: String) {
		self.name =name
  }
  
var room: Room? 
  
  deinit {
		print("\(name) is being deinitialized")
  }
  
}  

class Room {
	let number: String
	init (number: String) {
    self.number =number
  }
  
var host: Person?
	
  deinit {
	print("Room \(number) is being deinitialized")
  }
}


var delma: Persong? = Person(name:"delma")		//Person 인스턴스의 참조 횟수: 1
var room: Room? = Room(number: "1708")	//Room 인스턴스의 참조 횟수: 1

room?.host = delma 		//Person 인스턴스의 참조 횟수: 2
delma?.room = room		//Room 인스턴스의 참조 횟수: 2

delma = nil		//Person 인스턴스의 참조 횟수: 1
room = nil		//Room 인스턴스의 참조 횟수: 1

//Person 인스턴스와 Room 인스턴스를 참조할 방법 상실 - 메모리에 잔존 
```





*강한참조 순환 문제를 수동으로 해결*

```swift
var delma: Person? = Person(name: "delma")	//Person 인스턴스의 참조 횟수: 1
var room: Room? = Room(number: "1708")		//Room 인스턴스의 참조 횟수: 1

room?.host = delma	//Person 인스턴스의 참조 횟수: 2
delma?.room = room	//Room 인스턴스의 참조 횟수: 2

delma?.room = nil		//Room 인스턴스의 참조 횟수: 1
delma = nil		//Person 인스턴스의 참조 횟수: 1

room?.host = nil		//Person 인스턴스의 참조 횟수: 0
// delma is being deinitialized

room = nil		//Room 인스턴스의 참조 횟수: 0
//Room 1708 is being deinitialized
```



변수 또는 프로퍼티에 nil을 할당하면 참조 횟수가 감소하므로 해당 방법으로 인스턴스를 메모리에서 해제시킬 수 있다. 허나 매번 nil을 할당하는 것이 번거로울 수 있다.



## 약한 참조

약한 참조(Weak Reference)는 강한참조와 달리 자신이 참조하는 인스턴스의 참조 횟수를 증가시키지 않는다. 참조 타입의 프로퍼티나 변수의 선언 앞에 **weak** 키워드를 붙이면 그 프로퍼티나 변수는 자신이 참조하는 인스턴스를 약한참조 한다.

> **약한 참조와 상수, 옵셔널**
>
> 자신이 참조하던 인스턴스가 메모리에서 해제되면 nil이 할당될 수 있어야 하므로 약한참조는 상수에서 쓰일 수 없고 옵셔널이어야 한다.



```swift
class Room{
  let number: String
  
  init(number: String) {
    self.number = number
  }
  
  weak var host: Person?
  
  deinit {
    print("Room \(number) is being deinitailized")
  }
}

var delma: Person? = Person(name: "delma")	//Person 인스턴스의 참조 횟수: 1
var room: Room? = Room(number: "1708")		//Room 인스턴스의 참조 횟수: 1

room?.host = delma		//Person 인스턴스의 참조 횟수: 1
delma?.room = room 		//Room 인스턴스의 참조 횟수: 2

delma = nil 	//Person 인스턴스의 참조 횟수: 0, Room 인스턴스의 참조 횟수: 1
//delma is being deinitialized
print(room?.host)		//nil

room = nil	//Room 인스턴스의 참조 횟수: 0
//Room 1708 is being deinitialized
```





delma 변수가 참조했던 인스턴스의 참조횟수가 0이 되면서 메모리에서 해제 될 때, 인스턴스 room의 프로퍼티가 참조하는 인스턴스의 참조횟수도 1이 감소되었다. 이를 통해 **인스턴스가 메모리에서 해제 될 때, 자신의 프로퍼티가 강한참조를 하던 인스턴스의 참조 횟수를 1 감소시킨다**는 것을 알 수 있다. 

delma 변수가 참조하던 인스턴스가 메모리에서 해제되었다는 뜻은 room 변수가 참조하는 인스턴스의 프로퍼티인 host가 참조하는 인스턴스가 메모리에서 해제되었다는 의미이다. **host 프로퍼티는 약한 참조를 하기 때문에 자신이 참조하는 인스턴스가 메모리에서 해제되면 자동으로 nil을 할당한다** 는 것을 알 수 있다.





## 소유하지 않은 참조

참조 횟수를 증가시키지 않고 참조할 수 있는 방법에는 **소유하지 않은 참조(Unowned Reference)** 도 있다. 미소유참조는 약한참조와 다르게 자신이 참조하는 인스턴스가 항상 메모리에 존재할 것이라는 전제를 기반으로 동작한다. 즉, 자신이 참조하는 인스턴스가 메모리에서 해제되더라도 스스로 nil을 할당해주지 않는다는 뜻이다. 

미소유참조를 하면서 메모리에서 해제된 인스턴스에 접근하려 한다면 잘못된 접근으로 런타임 오류가 발생해 프로세스가 강제로 종료된다. 따라서 **미소유 참조는 참조하는 동안 해당 인스턴스가 메모리에서 해제되지 않으리라는 확신이 있을 때만 사용해야 한다.** 

참조 타입의 변수나 프로퍼티의 정의 앞에 **unowned** 키워드를 써주면 그 변수(상수)나 프로퍼티는 자신이 참조하는 인스턴스를 미소유참조하게 된다.





```swift
class Person{
  let name: String
  
  //카드를 소지할 수도, 소지하지 않을 수도 있기 때문에 옵셔널로 정의
  //카드를 한 번 가지면 참조를 잃지 않아야 하므로 강한참조를 해야 한다
  var card: CreditCard?
  
  init(name: String) {
    self.name = name
  }
  
  deinit{ print("\(name) is being deinitialized")}
}

class CreditCard {
  let number: UInt
  unowned let owner: Person		// 카드는 소유자가 분명히 존재해야 한다
  //CreditCard는 owner를 소유하지 않으면서 nil을 할당할 수 없는 미소유참조 상수 프로퍼티를 사용함
  
  init(number: UInt, owner: Person) {
    self.number = number
    self.owner = owner
  }
  
  deinit { print("Card \(number) is being deinitialized")}
  
}

var nana: Person? = Person(name: "nana")	//Person 인스턴스의 참조 횟수: 1

if let person: Person = nana {
  // CreditCard 인스턴스의 참조 횟수: 1
  person.card = CreditCard(number: 1111, owner: person)
  //Person 인스턴스의 참조 횟수: 1
}

nana = nil //Person 인스턴스의 참조 횟수: 0
//CreditCard 인스턴스의 참조 횟수: 0
//nana is being deinitialized
//Card 1111 is being deinitialized
```











<br>

<br>

<br>





### 더 알아볼 것

- 미소유 참조와 암시적 추출 옵셔널 프로퍼티
- 클로저의 강한참조 순환

### References

- https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html
- swift 프로그래밍, 한빛미디어















