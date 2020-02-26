## Closure

클로저는 기능을 가지고 있는 독립형태의 blocks(`{}`)이다. 클로저는 전달되거나 코드에 사용될 수 있다. 스위프트의 클로저는 C 와 Objective-C 의 blocks, 다른 프로그래밍 언어의 람다와 비슷하다.

클로저는 정의된 주변의 context로부터 어떤 상수나 변수의 참조를 캡쳐하거나 저장할 수있다. 이는 상수나 변수를 닫는 것으로 알려져 있다. 스위프트는 capturing의 모든 메모리 관리를 처리한다.



클로저는 1급 객체이다. 그렇기 때문에 중첩되거나 주위로 전달될 수 있다.
{: .notice}



Capturing Values에 관한 
{: .notice}

전역 함수와 중첩함수는 사실상 클로저의 특별한 케이스라고 할 수 있다. 클로저는 다음 3가지 형태로 표현될 수 있다.

- **Global function**은 이름을 가지고 어떤 변수도 캡쳐하지 않는 클로저다.
- **Nested function**은 이름을 가지고 enclosing function의 값을 캡쳐할 수 있는 클로저다.
- **Closure**의 표현은 이름이 없는 클로저로 주변 context의 값들을 캡쳐할 수 있는 가벼운 syntax로 쓰여진다. 

스위프트의 클로저 표현은 깨끗하고 명확한 스타일을 가지며 일반적인 시나리오에서 짧고 깔끔한 syntax를 권장하는 최적화 기능이 있다. 최적화에는 다음과 같은 기능이 포함되어있다.

- 파라미터를 추론하고 context로부터 값타입을 반환한다.
- 단일 표현 클로저에서 암시적인 반환
- 



## Capturing Values

클로저는 클로저가 **정의**된 context 주변의 상수나 변수를 **캡쳐**할 수 있다. 그 후 클로저는 상수와 변수를 정의한 scope가 존재하지 않더라도 본문 내 상수나 변수의 값을 **참조**하고 **수정**할 수 있다.

스위프트에서, 값을 캡쳐하는 가장 간단한 형식은 다른 함수의 본문 안에 쓰여진 nested function이다. nested function은 외부 함수의 인자를 캡쳐할 수 있고, 또 외부 함수 안에 정의된 상수나 변수를 캡쳐할 수 있다.

#### Example

- global function: `makeIncrementer`
- nested function: `incrementer`

> global function: makeIncrementer

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

`incrementer()`는 두개의 값, `runningTotal`과 `amount`를 캡쳐한다. 이 값들을 캡쳐한 후에, `incrementer()`는 `makeIncrementer()`에 의해 호출될 때마다, `amount`로 인해 `runningTotal`을 증가시키는 클로저로 리턴된다.

`makeIncrementer()`의 리턴타입은 `() -> Int`이다. 단순한 값이 아닌 **함수를 리턴**한다는 의미이다. 리턴되는 함수는 파라미터가 없으며, 호출될 때마다 Int 값을 리턴한다. 

`makeIncrementer(forIncrement:)`는 `runningTotal`이라는 Int 변수를 정의하며, runningTotal은 현재 실행중인 `incrementer`의 총합이 반환될 것이다. 이 변수는 0으로 초기화 된다.

`makeIncrementer(forIncrement:)` 함수는 하나의 Int 파라미터를 받아 amount라는 인자명을 사용한다. 파라미터로 전달된 인자값은 리턴된 `incrementer` 함수가 호출될 때마다 `runningTotal`을 증가해야하는지를 결정한다. `makeIncrementer` 함수는 실질적으로 덧셈 연산을 처리하는 nested function `incrementer`를 정의한다. 이 함수는 단순히 `amount`를 `runningTotal`에 더한 후에 결과를 반환한다.

> nested function: inrementer()

```swift
func incrementer() -> Int {
    runningTotal += amount
    return runningTotal
}
```

`incrementer()` 함수는 파라미터 없이, 함수 내에서 `runningTotal`과 `amount`를 참조하여 사용한다. 이것은 함수 주변의 `runningTotal`과 `amount`에 대한 **참조**를 **캡쳐**하여 함수 내에서 사용할 수 있다. **레퍼런스로 캡쳐하는 것(Capturing by reference)**은 `makeIncrementer` 함수의 호출이 끝나도 `runningTotal`과 `amount`가 사라지지 않게해주며, 다음에 `incrementer` 함수가 호출될 때, `runningTotal`을 사용할 수 있도록 해준다.

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

새로운 클로저를 만들면 레퍼런스도 변경된다.

```swift
let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()
// returns a value of 7
```

다시 `incrementByTen()`를 호출하면 `incrementByTen()` 함수 안에서 캡쳐된, `runningTotal` 레퍼런스의 값이 변경된다.

```swift
incrementByTen()
// returns a value of 40
```

<br>

## Closures Are Reference Types

위 예시에서 보면 알 수 있듯이, `incrementBySeven`과 `incrementByTen`은 상수이지만, `runningTotal` 변수 값을 변경시켰다. 이는 함수와 클로저는 모두 레퍼런스 타입이기 때문이다.

```swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
// returns a value of 50

incrementByTen()
// returns a value of 60
```



## Escaping closure

클로저가 함수에서 인자로 전달되지만 함수가 결과값을 반환 후에 호출이되면 우리는 그 클로저는 함수를 **탈출한다**라고 말한다. 파라미터를 클로저로 받는 함수를 선언했다고 하자, 우리는 `@escaping`를 파라미터 타입 앞에 작성하여 클로저가 탈출한다는 것을 명시할 수 있다.

클로저가 탈출할 수 있는 하나의 방법은 함수 밖에 선언된 변수로 저장되는 것이다. 예시로 많은 함수들이 클로저를 completion handler로 인자로 받아 비동기 동작을 시작한다. 함수는 동작후에 값을 리턴하지만, 클로저는 동작이 완료되기 전까지 호출되지 않는다-클로저는 탈출하여 나중에 호출이 되어야한다.

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

`someFunctionWithEscapingClosure(_:)` 함수는 클로저를 인자로 받아 함수 밖에 선언된 배열에 추가한다. 여기서 `@escaping`을 파라미터 앞에 작성하지 않으면, **컴파일 에러**가 발생한다.

클로저에 `@escaping` 표시를 한다는 것은 암시적으로 클로저를 `self`에게 참조한다는 것이다.

> 탈출 클로저와 비탈출클로저

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"

completionHandlers.first?()
print(instance.x)
// Prints "100"
```

<br>

> Tinder 예제

```swift
func fetchCurrentUser(completion: @escaping (User?, Error?) -> ()) {
    guard let uid = Auth.auth().currentUser?.uid else { return }
    Firestore.firestore().collection("users").document(uid).getDocument { (snapshot, err) in
        if let err = err {
            completion(nil, err)
            return
        }
        
        guard let dictionary = snapshot?.data() else { return }
        let user = User(dictionary: dictionary)
        completion(user, nil)
    }
}
```

`completion` 인자

Escaping closure captures non-escaping parameter 'completion'
{: .notice--danger}

---

```swift
// ViewController.swift
var num = 0

lazy var closure = { [weak self] in
    print("\(self?.num)")
}
```

```swift
// ViewController.swift
var num = 0

lazy var closure = { [unowned self] in
    print("\(self.num)")
}
```



