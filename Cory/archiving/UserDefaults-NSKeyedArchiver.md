## UserDefault와 NSKeyedArchiver로 데이터 저장하기

- [NSKeyedArchiver](https://github.com/corykim0829/iOS-squad/blob/master/Cory/archiving/NSKeyedArchiver.md)
- [UserDefaults](https://github.com/corykim0829/iOS-squad/blob/master/Cory/archiving/UserDefaults.md)

<br>

`NSKeyedArchiver`는 객체의 데이터를 저장하는 **인코더**이다.

그리고 `UserDefaults`는 key-value 쌍을 이용해서 사용자 기본 데이터베이스에 저장을 하는 **인터페이스**이다.

이 둘을 이용해서 앱 실행 중에 사용된 오브젝트들을 저장하고 불러올 수 있다.

<br>

`UserDefaults`를 사용해서 기본 데이터베이스에 저장하기 전에 오브젝트를 아카이브하여 데이터를 만들어야한다. 이때 사용되는 `NSKeyedArchiver`는 `NSCoder`를 상속한 서브클래스로 `NSCoding`을 채택한 오브젝트를 아카이브한다.

> #### 오브젝트를 아카이브하여 저장

`NSKeyedArchiver`의 타입 메소드인 `archivedData(withRootObject:requiringSecureCoding:)`는 `Data` 타입을 반환하며 `thorws` 메소드이기 때문에 try-catch문을 사용하여 구현해준다. `NSKeyedArchiver`를 사용해서 성공적으로 아카이브된 데이터가 반환됐다면, `UserDefaults`를 사용해 저장하면된다.

```swift
do {
	  let data = try NSKeyedArchiver.archivedData(withRootObject: vendingMachine, requiringSecureCoding: false)  
} catch {
    // 에러에 대응
}
```

<br>

`UserDefaults`는  `standard`라는 타입 프로퍼티를 사용해서 공유되는 기본 오브젝트를 반환한다. 이후 `set(_:forKey:)` 인스턴스 매소드를 사용해 저장한다. 이 때 키 값이 필요하며 이 키값은 `UserDefaults`를 사용해 다시 아카이브된 객체를 가져올 때에도 사용되기 때문에 상수로 선언해두는게 좋다.

```swift
let keySring = "keyString"
UserDefaults.standard.set(data, forKey: keyString)
```

<br>

> #### 데이터로 저장된 오브젝트를 불러와 언아카이브

저장했던 객체를 다시 불러올 때에는 `UserDefaults`의 `object(forKey:)` 인스턴스 메소드를 사용하여 `Any` 타입으로 반환할 수 있다. `NSKeyedArchiver`를 이용해서 아카이브된 데이터의 타입이 `Data`였기 때문에, `Data` 타입으로 캐스팅 해준다. 

```swift
guard let data = UserDefaults.standard.object(forKey: keyString) as? Data else { return }
```

<br>

이렇게 불러온 데이터를 `NSKeyedUnarchiver`를 사용하여 디코딩해주면 된다. 이 때 사용되는 메소드는 `unarchiveTopLevelObjectWithData(_:)` 라는 타입 메소드로 `Data` 타입을 인자로 받는다. 또한 `NSKeyedArchiver`와 마찬가지로 try-catch문을 사용해야 하는 `throws` 메소드다.

```swift
guard let pokemon = try NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(data) as? Pokemon else { return }
```

