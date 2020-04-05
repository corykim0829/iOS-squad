## Codable

Codable은 Type Alias로, 외부 표현에서 또는 외부 표현으로 변환 할 수 있는 유형이다.

```swift
typealias Codable = Decodable & Encodable
```

<br>

#### Discussion

`Codable`은 `Encodable`과 `Decodable` 프로토콜의 type alias이다. `Codable`을 타입 또는 제너릭 제약조건으로 사용하면 `Encodable`, `Decodable` 프로토콜을 모두 채택하는 것과 동일하다.

<br>

---

## Encoding and Decoding Custom Types

JSON과 같은 외부 표현과 호환성을 위해 데이터 타입을 encodable, decodable하게 만들자.

<br>

#### Overview

많은 프로그래밍 작업에는 네트워크 연결을 통해 데이터를 보내기, disk에 데이터를 저장하기 또는 API와 서비스에 데이터를 제출하는 것들이 포함된다. 이 작업들에는 데이터가 전송되어지는 동안 데이터가 중간형식(intermediate format)으로 인코딩, 디코딩되는 것을 종종 필요로한다.

스위프트 표준 라이브러리는 데이터 인코딩, 디코딩 **표준 접근법**을 정의했다. 이 **접근법**은 [Encodable](https://developer.apple.com/documentation/swift/encodable), [Decodable](https://developer.apple.com/documentation/swift/decodable) 프로토콜을 커스텀 타입에 구현함으로써 사용할 수 있다. 이 프로토콜들을 채택하면 Encoder와 Decoder 프로토콜의 구현부분들이 우리의 데이터를 JSON 또는 property list와 같은 외부 표현으로 인코드, 디코드하게 한다. 인코딩과 디코딩을 모두 지원하기 위해서, `Encodable`과 `Decodable` 프로토콜을 합친 `Codable` 프로토콜 채택을 명시한다. 이 프로세스는 우리의 타입을 codable하게 해준다.

#### Encode and Decode Automatically

타입을 codable하게 하는 가장 쉬운 방법은 이미 `Codable`을 채택한 타입을 프로퍼티로 선언하는 것이다. 이러한 타입에는 `String`, `Int`, `Double`과 같은 표준 라이브러리 타입, 그리고 `Date`, `Data`, `URL`과 같은 Foundation 타입을 포함한다. 어느 타입이든 모든 프로퍼티가 codable이면 채택을 선언하면 자동적으로  `Codable`을 채택하게 된다.

이름과 설립년도를 저장하는 `Landmark` 구조체를 고려해보자:

```swift
struct Landmark {
    var name: String
    var foundingYear: Int
}
```

`Landmark`의 **상속목록(inheritance list)**에 `Codable`을 추가하면 모든 프로토콜 준수사항을 만족하는 **자동 채택(automatic conformance)**이 적용된다:

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    
    // Landmark now supports the Codable methods init(from:) and encode(to:), 
    // even though they aren't written as part of its declaration.
}
```

타입에 Codable을 상속하면 타입들을 내장된 데이터 형식, 그리고 커스텀 인코더, 디코더로부터 제공된 어느 형식으로 또는 그 형식으로부터 직렬화(serialize)할 수 있게 해준다. 예를들어, Landmark 구조체는 property list와 JSON을 처리하는 특별한 코드가 없어도 `PropertyListEncoder`와 `JSONEncoder` 클래스를 사용하여 인코드될 수 있다.

이러한 원칙은 다른 codable한 커스텀 타입으로 만들어진 커스텀 타입에도 적용된다. 타입의 프로퍼티가 모두 `Codable`하다면, 어느 커스텀 타입이든지 `Codable`이 될 수 있다.

아래 예제는 `location` 프로퍼티가 `Landmark` 구조체에 추가되었을 때, 어떻게 `Codable` 자동 채택이 적용되는지를 보여준다.

```swift
struct Coordinate: Codable {
    var latitude: Double
    var longitude: Double
}

struct Landmark: Codable {
    // Double, String, and Int all conform to Codable.
    var name: String
    var foundingYear: Int
    
    // Adding a property of a custom Codable type maintains overall Codable conformance.
    var location: Coordinate
}
```

<br>

Built-in types such as [`Array`](https://developer.apple.com/documentation/swift/array), [`Dictionary`](https://developer.apple.com/documentation/swift/dictionary), and [`Optional`](https://developer.apple.com/documentation/swift/optional) also conform to `Codable` whenever they contain codable types. You can add an array of `Coordinate` instances to `Landmark`, and the entire structure will still satisfy `Codable`.

The example below shows how automatic conformance still applies when adding multiple properties using built-in codable types within `Landmark`:

```
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    
    // Landmark is still codable after adding these properties.
    var vantagePoints: [Coordinate]
    var metadata: [String: String]
    var website: URL?
}
```

<br>

#### Encode or Decode Exclusively

In some cases, you may not need `Codable`'s support for bidirectional encoding and decoding. For example, some apps only need to make calls to a remote network API and do not need to decode a response containing the same type. Declare conformance to `Encodable` if you only need to support the encoding of data. Conversely, declare conformance to `Decodable` if you only need to read data of a given type.

The examples below show alternative declarations of the `Landmark` structure that only encode or decode data:

```
struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}
struct Landmark: Decodable {
    var name: String
    var foundingYear: Int
}
```

<br>

#### Choose Properties to Encode and Decode Using Coding Keys

Codable 타입은 `CodingKeys`라는 [`CodingKey`](https://developer.apple.com/documentation/swift/codingkey) 프로토콜을 준수하는 특별한 중첩 열거형을 선언할 수 있다. 이 열거형이 존재하면, case는 codable 타입의 인스턴스가 인코드, 디코드 될 때 꼭 포함되어야 하는 프로퍼티의 권위있는(authoritative) 리스트로 제공된다. 열거형 case의 이름은 해당 타입에서 case와 매치되는 프로퍼티의 이름과 동일해야한다.

만약 인스턴스를 디코딩할 때 존재하지 않는 프로퍼티나, 특정 프로퍼티를 인코드된 표현으로 포함해야하지 않을 때, `CodingKeys` 열거형에서 프로퍼티를 생략해라. `CodingKeys`에서 생략된 프로퍼티는 포함하는 타입이 `Decodable`과 `Codable`을 자동 채택 받기 위해 default 값이 있어야한다.

만약 직렬화된 데이터 형식에서 사용되는 키가 데이터 타입에 있는 프로퍼티 이름과 매치가 되지 않는다면, `CodingKeys` 열거형에 raw-value 타입을 `String`으로 명시함으로써 대체 키를 제공할 수 있다. 각 열거형 case에서 raw-value로 사용되는 문자열은 인코딩 및 디코딩에 사용되는 키 이름이 된다. case 이름과 raw value의 연관성을 통해 모델링 중인 직렬화 형식의 이름, 문장 부호 및 대문자를 일치시키지 않고 [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)에 따라 데이터 구조의 이름을 지정할 수 있다.

아래 예제는 인코딩, 디코딩 시, `Landmark` 구조체의 `name`과 `foundingYear` 프로퍼티에 대체 키를 사용한 것이다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    var vantagePoints: [Coordinate]
    
    enum CodingKeys: String, CodingKey {
        case name = "title"
        case foundingYear = "founding_date"
        
        case location
        case vantagePoints
    }
}
```

<br>

#### Encode and Decode Manually

만약 스위프트 타입의 구조체가 인코드된 형태와 다르다면, 인코딩 및 디코딩 로직을 정의하여 `Encodable`과 `Decodable`의 커스텀 구현을 제공할 수 있다.

아래 예제는 `Coordinate` 구조체는 `additionalInfo` 컨테이너 안에 중첩된 `elevation` 프로퍼티를 지원하기위해 확장되었다.

```swift
struct Coordinate {
    var latitude: Double
    var longitude: Double
    var elevation: Double

    enum CodingKeys: String, CodingKey {
        case latitude
        case longitude
        case additionalInfo
    }
    
    enum AdditionalInfoKeys: String, CodingKey {
        case elevation
    }
}
```

`Coordinate` 타입의 인코딩 형식은 두번째 수준(second level)의 중첩된 정보를 포함하기 때문에, `Encodable` 과 `Decodable` 프로토콜을 채택할 때 각 타입 특정 수준에서 사용되는 전체 coding key 세트를 나열하는 두 개의 열거형을 사용한다.

아래 예제는 `Decodable` 프로토콜을 준수하기 위해서 `Coordinate` 필수 생성자 [`init(from:)`](https://developer.apple.com/documentation/swift/decodable/2894081-init)를 구현하여 구조체를 확장한것이다.

```swift
extension Coordinate: Decodable {
    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        latitude = try values.decode(Double.self, forKey: .latitude)
        longitude = try values.decode(Double.self, forKey: .longitude)
        
        let additionalInfo = try values.nestedContainer(keyedBy: AdditionalInfoKeys.self, forKey: .additionalInfo)
        elevation = try additionalInfo.decode(Double.self, forKey: .elevation)
    }
}
```

<br>

생성자는 파라미터로 받은 `Decoder` 인스턴스의 메소드를 사용하여 `Coordinate` 인스턴스를 채운다. `Coordinate` 인스턴스의 두 프로퍼티는 스위프트 표준 라이브러리가 제공하는 **keyed container API**를 사용하여 초기화된다.

아래 예제는 `Coordinate` 구조체가 `Encodable` 프로토콜을 준수하기 위해 필수 메소드인 [`encode(to:)`](https://developer.apple.com/documentation/swift/encodable/2893603-encode)를 구현하여 확장될 수 있는 것을 보여준다.

```swift
extension Coordinate: Encodable {
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(latitude, forKey: .latitude)
        try container.encode(longitude, forKey: .longitude)
        
        var additionalInfo = container.nestedContainer(keyedBy: AdditionalInfoKeys.self, forKey: .additionalInfo)
        try additionalInfo.encode(elevation, forKey: .elevation)
    }
}
```

이  `encode(to:)` method 구현은 이전 예제의 디코딩 작업을 반대로 한 것이다.

인코딩 및 디코딩 과정을 커스터마이즈 할 때 사용되는 container 타입 더 많은 정보가 필요하면 [`KeyedEncodingContainerProtocol`](https://developer.apple.com/documentation/swift/keyedencodingcontainerprotocol)와 [`UnkeyedEncodingContainer`](https://developer.apple.com/documentation/swift/unkeyedencodingcontainer)를 참고하자.

<br>

#### References

- [Codable](https://developer.apple.com/documentation/swift/codable)
- [Encoding and Decoding Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)