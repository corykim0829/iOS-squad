## 1. Capturing Values: 클로저 캡쳐란?

출처 : <https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID103>

> 클로져 캡쳐란? 

클로저 캡쳐란 **매개변수나 지역변수가 아닌 주변 외부의 context를 사용하기 위해 주변 외부의 context를 참조하는 것**이다. 
그래야 주변 외부 context가 없어질지라도 클로져가 주변 외부 context를 사용할 수 있다.

apple의 공식 예제로 클로져 캡쳐가 무엇인지 알아보자 

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

makeIncremeter라는 함수는 Int 타입을 매개변수로 갖고, () -> Int 의 타입인 클로저를 반환하는 형식을 가지고 있다.
makeIncremeter 함수의 내부를 보면 incrementer라는 함수가 클로저로서 반환되고 있다.

여기서 incrementer라는 클로저(함수)를 뚫어져라 쳐다보면, runningTotal와 amount을 캡쳐해서 사용하고 있다. 
즉 makeIncrementer가 호출이 되면 makeIncremeter의 매개변수인 amount와 지역변수인 runningTotal을 incrementer라는 클로져에서 캡쳐해서 사용하고 있다는 뜻이다.

```swift
func incrementer() -> Int {
    runningTotal += amount
    return runningTotal
}
```
위의 코드를 살펴보자. makeIncrementer에서 incrementer만 가지고 온 코드이다. 뭔가 이상해보이지 않는가? incrementer () 함수에는 매개 변수가 없지만 함수 본문 내에서 runningTotal 및 amount를 참조한다. 이는 surround 함수로부터 amount와 runningTotal에 대한 참조을 캡처하여 자체 함수 body 내에서 사용하여 수행한다. **참조로 캡처하면 makeIncrementer에 대한 호출이 종료 될 때 runningTotal과 amount가 사라지지 않으며 다음에 incrementer 함수가 호출 될 때 runningTotal을 사용할 수 있다.**

아래 코드를 보면서 더 이해해보자.
```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

=> incrementByTen은 makeIncrementer 라는 함수를 호출해서 반환된 클로져이다. makeIncrementer()을 호출했으므로 그 안에 있는 지역변수인 runningTotal이나 매개변수인 amount(== forIncrement)는 함수가 끝나면 사라지는게 정상이지만 **incrementByTen이 이 둘을 캡쳐함으로써 계속 살아있게 되고, 따라서 이 클로져를 호출함에 따라 값이 updating이 됨을 알 수 있다.**

**클로져는 또 reference 타입이라 클조의 reference count가 0이 되지 않는 한 캡쳐한 runningTotal이나 amount는 사라지지 않는다.**


## 2. Default capture semantics

출처 : <https://alisoftware.github.io/swift/closures/2016/07/25/closure-capture-1/#fn:block-modifier>

```swift

class Pokemon: CustomDebugStringConvertible {
  let name: String
  init(name: String) {
    self.name = name
  }
  var debugDescription: String { return "<Pokemon \(name)>" }
  deinit { print("\(self) escaped!") }
}

func delay(_ seconds: Int, closure: @escaping ()->()) {
  let time = DispatchTime.now() + .seconds(seconds)
  DispatchQueue.main.asyncAfter(deadline: time) {
    print("🕑")
    closure()
  }
}

func demo1() {
  let pokemon = Pokemon(name: "Mewtwo")
  print("before closure: \(pokemon)")
  delay(1) {
    print("inside closure: \(pokemon)")
  }
  print("bye")
}
// 실행 화면 
// before closure: <Pokemon Mewtwo>
// bye
// 🕑
// inside closure: <Pokemon Mewtwo>
// <Pokemon Mewtwo> escaped!

```
=> delay함수는 매개변수로 Int 타입인 seconds와 탈출 클로져인 closure를 가지고 있고, seconds 이후에 closure가 호출되는 함수이다. 
<br>demo1()을 보면 Mewtwo라는 이름을 가진 포켓몬 객체가 함수가 끝나면 없어져야 될 것 같지만 1초 후 클로져가 호출된 이후에 사라진다. **왜냐하면 클로져에서 해당 객체를 캡쳐함으로써 reference count가 1 올라가서 메모리에서 없어지지 않기 때문이다.**

## 3. Captured variables are evaluated on execution : 캡쳐된 변수는 클로져가 실행될 때 평가되어진다. 

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
// 실행 화면
// before closure: <Pokemon Pikachu>
// <Pokemon Pikachu> escaped!
// after closure: <Pokemon Mewtwo>
// 🕑
// inside closure: <Pokemon Mewtwo>
// <Pokemon Mewtwo> escaped! 
```
=> 클로져는 캡쳐할 변수를 실행되는 시점에 평가한다. 따라서 위의 예제에서 클로져는 1초 후에 실행이 되는데, 1초가 되기 전에 이미 pokemon이라는 변수는 "Mewtwo"라는 이름을 가진 포켓몬 인스턴스를 참조하므로, **클로져는 "Mewtwo"라는 이름을 가진 포켓몬 객체를 캡쳐한다.**
<br>따라서, 실행화면을 보면 Pickachu는 함수가 끝나자마자 메모리에서 deallocated 되고, Mewtwo는 1초후에 클로져가 실행되고 메모리에서 deallocated됨을 알 수 있다.  