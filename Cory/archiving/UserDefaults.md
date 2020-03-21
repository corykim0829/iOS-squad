## UserDefaults

앱 실행시 지속적으로 **key-value** 쌍을 저장하는 사용자의 **기본 데이터베이스**에 대한 인터페이스.

<br>

#### Declaration

```swift
class UserDefaults: NSObject
```

<br>

#### Overview

`UserDefault` 클래스는 defaults system과 상호작용에 프로그래밍 인터페이스를 제공한다. defaults system을 통해 앱은 사용자의 설정에 맞게 동작을 커스터마이즈 할 수 있다. 예를 들어, 사용자가 선호하는 측정 단위나 미디어 재생 속도를 명시할 수 있게 할 수 있다. 앱은 값들을 사용자의 기본 데이터베이스에 있는 파라미터 세트에 할당하여 이러한 설정들을 저장한다. 파라미터들은 기본값이라고 하는데 그 이유는 파라미터들은 보통 시작시 앱의 기본 상태 또는 기본적으로 동작되는 방법을 결정하기 때문이다.

런타임에는 `UserDefaults` 객체를 사용하여 사용자의 기본 데이터베이스에서 사용하는 defaults를 읽는다. `UserDefaults`는 기본값(default value)이 필요할 때마다 사용자의 기본 데이터베이스를 여는 것을 피하기 위해 정보를 캐시한다. 기본값을 설정하면, 프로세스 내에서 동기적으로 변경되고, 영구 저장소 및 다른 프로세스와 비동기적으로 변경된다.

...

<br>

#### Storing Default Objets

`UserDefaults` 클래스는 floats, doubles, integers, Boolean, values 그리고 URLs와 같은 기본 타입을 위한 convenience 메소드를 제공한다. 이러한 메소드는 [Setting Default Values](https://developer.apple.com/documentation/foundation/userdefaults#1664798)에 설명되어있다.

기본 객체는 property list 여야만 한다. 즉, [`NSData`](https://developer.apple.com/documentation/foundation/nsdata), [`NSString`](https://developer.apple.com/documentation/foundation/nsstring), [`NSNumber`](https://developer.apple.com/documentation/foundation/nsnumber), [`NSDate`](https://developer.apple.com/documentation/foundation/nsdate), [`NSArray`](https://developer.apple.com/documentation/foundation/nsarray), [`NSDictionary`](https://developer.apple.com/documentation/foundation/nsdictionary)의 인스턴스(또는 컬렉션, 조합)여야 한다. **다른 타입의 객체를 저장하고 싶다면, NSData 인스턴스를 만들기 위해 객체를 아카이브 해야한다.**

mutable 객체를 값으로 설정했어도, UserDefaults에서 반환되는 값은 immutable하다. 예를들어 만약 mutable 문자열을 "MyStringDefault"의 값으로 설정했다면, `string(forKey:)` 메소드를 사용해서 가져온 값은 immuatble하다. 만약 mutable 문자열을 기본 값으로 설정하고 값이 변경되면 `set(_:forKey:)`메소드를 다시 호출하지 않으면 변경된 문자열 값은 기본값에 반영되지 않는다.

<br>

> Type Property

### standard

공유된 기본 객체를 반환한다.

#### Declaration

```swift
class var standard: UserDefaults { get }
```

<br>

#### Reference

- [UserDefaults Document](https://developer.apple.com/documentation/foundation/userdefaults)
- [standard Document](https://developer.apple.com/documentation/foundation/userdefaults/1416603-standard)