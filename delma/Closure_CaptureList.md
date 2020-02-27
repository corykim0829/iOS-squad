# Closure - Capture List

## What is Capture List ?

주변 환경의 범위에서 참조한 변수들을 얼마나 강하게 캡쳐해야하는지를 명시하는 것으로, 캡쳐리스트를 사용하여 메모리 누수를 일으키는 강한 참조 순환을 피할 수 있게 된다.



<br>



캡쳐리스트는 참조 방식과 참조할 대상을 대괄호([])로 둘러싼 목록 형식으로 작성하고, 캡쳐리스트 뒤에는 in 키워드를 써준다.  **캡쳐리스트에 명시한 요소가 참조타입이 아니라면 해당 요소들은 클로저가 생성될때 초기화된다.**



**참고로 얼마나 강하게 캡쳐할지 지정하는 옵션 키워드(weak, unowned)는 참조타입을 캡쳐할 때만 사용 가능하다.**



<br>

## When do we use it?

1. 클로저의 강한참조 순환 문제를 해결하기 위해 사용(**참조 타입**)
2. 클로저 내부에서 클로저 외부의 값을 참조할 때 참조하는 값이 변경되면 클로저 내부에서도 참조하는 값 또한 바뀌게 되므로 이를 방지하고자 사용(**값 타입**)



<br>



### 클로저의 강한참조 순환 문제 

강한 참조 순환이 발생하는 이유는 클로저가 클래스와 같은 참조타입이기 때문이다. 클로저를 클래스 인스턴스의 프로퍼티로 할당하면 클로저의 참조가 할당된다. 이때 참조타입과 참조타입이 서로 강한 참조를 하기 때문에 강한참조 순환 문제가 발생한다. 이때 캡쳐리스트에 옵션을 지정해주면, 참조 횟수(Reference Counting)을 증가할지 여부를 결정할 수 있다.



<br>



```swift
//참조 타입인 경우 캡쳐리스트의 옵션 지정
class SimpleClass{
	var value: Int = 0
}

var x: SimpleClass? = SimpleClass()
var y = SimpleClass()

let closure = { [weak x, unowned y] in
	print(x?.value, y.value)
}

x = nil
y.value = 10

closure()	//nil 10
```



**참조 타입은 캡쳐리스트 시 캡쳐 시점이 클로저가 호출될 때** 이므로 0이 아닌 변경 된 이후의 값으로 결과가 나오게 된다.



<br>



캡쳐리스트에서 x를 weak로, y를 unowned로 지정했다.

 x가 weak 참조를 하게 되므로 클로저 내부에서 사용하더라도 클로저는 x가 참조하는 인스턴스의 참조 횟수를 증가시키지 않는다. 그렇게 되면 변수 x가 참조하는 인스턴스가 메모리에서 해제되어 클로저 내부에서도 더 이상 참조가 불가능한 것을 볼 수 있다. 

y는 unowned 참조를 했기 때문에 클로저가 참조 횟수를 증가시키지 않지만, 만약 메모리에서 해제된 상태에서 사용하려 한다면 실행 중에 오류로 애플리케이션이 강제로 종료될 가능성이 있다.



<br>



### 클로저 내부에서 외부의 값 참조하기



<br>



아래의 출력 값을 예상해보자

```swift
var index = 0
var closureArr: [() -> ()] = []

for _ in 1...5 {
  closureArr.append({print(index)})
  index += 1
}

for i in 0..<closureArr.count {
    closureArr[i]()
}
```



위 코드의 결과로 5 5 5 5 5가 출력되게 된다.

왜냐하면 closureArr.append() 시 변수 index 가 반복문을 돌면서 최종값으로 5가 되는데,

클로저가 이미 변경된 index 값인 5를 참조하기 때문이다. 



<br>



이렇게 예상치 못한 결과가 나오는 것을 방지하고자 Capture List를 사용한다.



```swift
var index = 0
var closureArr: [() -> ()] = []

for _ in 1...5 {
  closureArr.append({[index] in 
   		print(index)       
   })
  index += 1
}

for i in 0..<closureArr.count {
    closureArr[i]()
}
```



이 코드의 실행 결과로는 0 1 2 3 4 가 나올 것이다.

이전 코드와 다른 점은, 클로저 안에 참조하는 변수에 대괄호[]를 붙여줌으로 캡쳐 리스트를 사용한다고 명시했다.

그래서 capture list를 이용해서 값이 변경되기 이전의 값을 참조 해 우리가 예상하는 0 1 2 3 4 의 결과가 나오게 된다.

**캡쳐 리스트시 값타입의 캡쳐 시점은 클로저가 생성될 때 이기 때문이다.**



<br>



요약하자면 캡쳐리스트에서 캡쳐 시점은 값타입 / 참조타입에 따라 나뉘게 되는데, 

**값타입** => 클로져가 생성될 때 캡쳐

**참조타입** =>  클로져가 호출될 때 캡쳐

를 잘 기억해서 의도하는대로 사용하면 되겠다.





<br>

<br>





### References

- https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html
- swift programming 5, 한빛미디어
- https://baked-corn.tistory.com/25