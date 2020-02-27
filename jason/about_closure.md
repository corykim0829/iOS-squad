## 1. Capturing Values: 클로저 캡쳐란?

출처 : <https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID103>

> 클로저 캡쳐란? 

클로저 캡쳐란 **매개변수나 지역변수가 아닌 주변 외부의 context를 사용하기 위해 주변 외부의 context를 참조하는 것(Capturing by reference)** 입니다. 
그래야 주변 외부 context가 없어질지라도 클로저가 주변 외부 context를 사용할 수 있습니다.

apple의 공식 예제로 클로저 캡쳐가 무엇인지 알아봅시다 

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
> 코드 설명 

makeIncremeter라는 함수는 Int 타입을 매개변수로 갖고, () -> Int 의 타입인 클로저를 반환하는 형식을 가지고 있습니다.
makeIncremeter 함수의 내부를 보면 incrementer라는 함수가 클로저로서 반환되고 있습니다.

여기서 incrementer라는 클로저(함수)를 뚫어져라 쳐다보면, runningTotal와 amount을 캡쳐해서 사용하고 있습니다. 
즉 makeIncrementer가 호출이 되면 makeIncremeter의 매개변수인 amount와 지역변수인 runningTotal을 incrementer라는 클로저에서 캡쳐해서 사용하고 있다는 뜻입니다.

```swift
func incrementer() -> Int {
    runningTotal += amount
    return runningTotal
}
```
> 위의 코드를 살펴봅시다. makeIncrementer에서 incrementer만 가지고 온 코드입니다. 뭔가 이상해보이지 않나요? 

incrementer () 함수에는 매개 변수가 없지만 함수 본문 내에서 runningTotal 및 amount를 참조합니다. 이는 surround 함수로부터 amount와 runningTotal에 대한 참조를 캡처(Cpaturing by reference)하여 자체 함수 body 내에서 사용하여 수행합니다. **참조로 캡처(Capturing by reference)하면 makeIncrementer에 대한 호출이 종료 될 때 runningTotal과 amount가 사라지지 않으며 다음에 incrementer 함수가 호출 될 때 runningTotal을 사용할 수 있습니다.**

아래 코드를 보면서 더 이해해봅시다.
```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

=> incrementByTen은 makeIncrementer 라는 함수를 호출해서 반환된 클로저입니다. makeIncrementer()을 호출했으므로 그 안에 있는 지역변수인 runningTotal이나 매개변수인 amount(== forIncrement)는 함수가 끝나면 사라지는게 정상이지만 **incrementByTen이 이 둘을 캡쳐함으로써 계속 살아있게 되고, 따라서 이 클로저를 호출함에 따라 값이 updating이 됨을 알 수 있습니다.**

**클로저는 또 reference 타입이라 클로저의 reference count가 0이 되지 않는 한 캡쳐한 runningTotal이나 amount는 사라지지 않습니다.**

# 1-2 Capturing by reference

하지만  `Capturing by reference` 라는 말이 선뜻 이해가 안가네요. 

무엇이 이해가 안가냐면, 
<br>클로저가 surrounding context가 value type이든, reference type이든 context를 Capturing by reference를 합니다는데, 
<br>결론적으로 **value type인 context를 어떻게 참조로 캡쳐합니다는 것(Capturing by reference)일까요?**

이것이 매우 궁금하여 구글링도 해보고, 아는 선생님께 질문을 해본 뒤에, 다음과 같은 사실을 알아냈습니다. 

* value type도 만들어질 때 내부에 레퍼런스 카운터가 생긴다.
* value type이더라도 클로저가 캡쳐하면 reference type으로 결정해서 heap 영역으로 옮길 것 같다.  

위의 항목을 그대로 반영하면, context가 value type이더라도 클로저 캡쳐하면 reference count 가 1 증가하기 때문에 context를 계속 사용할 수 있는 것입니다.



## 2. Default capture semantics

출처 : <https://alisoftware.github.io/swift/closures/2016/07/25/closure-capture-1/#fn:block-modifier>

```swift

class Pokemon: CustomDebugStringConvertible {
  let name: String
  init(name: String) {
    self.name = name
  }
  deinit { print("\(self) escaped!") }
}

func delay(_ seconds: Int, closure: @escaping ()->()) {
  let time = DispatchTime.now() + .seconds(seconds)
  DispatchQueue.main.asyncAfter(deadline: time) {
    print("🕑")
    closure()
  }
}
```

```swift
func demo1() {
  let pokemon = Pokemon(name: "Mewtwo")
  print("before closure: \(pokemon)")
  delay(1) {
    print("inside closure: \(pokemon)")
  }
  print("bye")
}
demo1()

// 실행 화면 
// before closure: Mewtwo
// bye
// 🕑
// inside closure: Mewtwo
// Mewtwo escaped!

```
=> delay함수는 매개변수로 Int 타입인 seconds와 탈출 클로저인 closure를 가지고 있고, seconds 이후에 closure가 호출되는 함수입니다. 
<br>=> demo1()을 보면 Mewtwo라는 이름을 가진 포켓몬 객체가 함수가 끝나면 없어져야 될 것 같지만 1초 후 클로저가 호출된 이후에 사라집니다. **왜냐하면 클로저에서 해당 객체를 캡쳐함으로써 reference count가 1 올라가서 메모리에서 없어지지 않기 때문입니다.**

## 3. Captured variables are evaluated on execution : 캡쳐된 변수는 클로저가 실행될 때 평가되어집니다. 

```swift
func demo2() {
  var pokemon = Pokemon(name: "Pikachu")
  print("before closure: \(pokemon)")
  delay(1) {
    print("inside closure: \(pokemon)")
  }
  pokemon = Pokemon(name: "Mewtwo")
  print("after closure: \(pokemon)")
}
demo2()

// 실행 화면
// before closure: Pikachu
// Pikachu escaped!
// after closure: Mewtwo
// 🕑
// inside closure: Mewtwo
// Mewtwo escaped! 
```
=> 클로저는 캡쳐할 변수를 실행되는 시점에 평가합니다. 따라서 위의 예제에서 클로저는 1초 후에 실행이 되는데, 1초가 되기 전에 이미 pokemon이라는 변수는 "Mewtwo"라는 이름을 가진 포켓몬 인스턴스를 참조하므로, **클로져는 "Mewtwo"라는 이름을 가진 포켓몬 객체를 캡쳐합니다.**
<br>따라서, 실행화면을 보면 Pickachu는 함수가 끝나자마자 메모리에서 deallocated 되고, Mewtwo는 1초후에 클로저가 실행되고 메모리에서 deallocated됨을 알 수 있습니다.  

> 그러면 위의 코드에서 클로저가 호출될때 Mewtwo를 출력하지 않고 Pikachu를 출력하고 싶을면 어떻게 해야 할까? 답은 capture list에 있습니다. 

## 4. Capture List을 사용하면 캡쳐된 변수는 클로저가 생성되는 시점에 평가되어집니다. 

* Capture list는 클로저의 `{` 바로 다음과 `in` 바로 뒤에 기록됩니다.

```swift 
func demo3() {
    var pokemon = Pokemon(name: "Pikachu")
    print("before closure: \(pokemon.name)")
    delay(1) { [pokemon] in
        print("inside closure: \(pokemon.name)")
    }
    pokemon = Pokemon(name: "Mewtwo")
    print("after closure: \(pokemon.name)")
}
demo3()

// 실행화면 
// before closure: Pikachu
// after closure: Mewtwo
// Mewtwo escaped!
// 🕑
// inside closure: Pikachu
// Pikachu escaped!
```

=> capture list을 사용하면 클로저 생성 시점에 캡쳐할 변수를 평가하기 때문에 클로저 생성 되기 이전의 "Pikachu" 이름을 가진 포켓몬 인스턴스를 캡쳐합니다. 
<br> 따라서 클로저가 호출될때 Pickachu를 출력하는 것입니다. 

## 4-2. warning!: 클로저 캡쳐할 때 캡쳐한 변수가 value type인지 reference type인지를 구별하세요.

* 말 그대로 값 타입은 값 타입으로서의 특성을 띄고 있고, 참조 타입은 참조 타입은 참조 타입으로서의 특성을 띄고 있기 때문에 조심하라는 것입니다. 

* 아래 코드를 보면 demo1()에서 struct 타입인 StructPokemon 객체인 경우에는 클로저가 호출할 때 이전의 이름이 Pikachu를 출력하고, demo2()에서 class 타입인 Pokemon 객체의 경우에는 클로저가 호출할 때 현재의 이름인 Mewtwo가 호출됨을 알 수 있습니다. 
<br>왜냐하면 캡쳐리스트로 캡쳐할 때 value type인 경우에는 값을 복사하고, reference type인 경우에는 주소값을 복사하기 때문입니다.    

```swift
class Pokemon: CustomDebugStringConvertible {
    var name: String
    init(name: String) {
        self.name = name
    }
    deinit { print("\(self) escaped!") }
}

struct StructPokemon: CustomDebugStringConvertible {
    var name: String
    init(name: String) {
        self.name = name
    }
}

func delay(_ seconds: Int, closure: @escaping ()->()) {
    let time = DispatchTime.now() + .seconds(seconds)
    DispatchQueue.main.asyncAfter(deadline: time) {
        print("🕑")
        closure()
    }
}

func demo1() {
    var pokemon = StructPokemon(name: "Pikachu")
    print("before closure: \(pokemon.name)")
    delay(1) { [pokemon] in
        print("inside closure: \(pokemon.name)")
    }
    pokemon.name = "Mewtwo"
    print("after closure: \(pokemon.name)")
}

demo1()
// 실행화면 
// before closure: Pikachu
// after closure: Mewtwo
// 🕑
// inside closure: Pikachu

func demo2() {
    let pokemon = Pokemon(name: "Pikachu")
    print("before closure: \(pokemon.name)")
    delay(1) { [pokemon] in
        print("inside closure: \(pokemon.name)")
    }
    pokemon.name = "Mewtwo"
    print("after closure: \(pokemon.name)")
}
demo2()
// 실행화면 
// before closure: Pikachu
// after closure: Mewtwo
// 🕑
// inside closure: Mewtwo
// <Pokemon Mewtwo> escaped!
```



## 다음에는 클로져의 강한 참조 순환과 대안책인 capture list & weak type 에 대해 알아보겠습니다. 그럼 빠이~!