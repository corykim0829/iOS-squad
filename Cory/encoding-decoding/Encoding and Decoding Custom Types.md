## Encoding and Decoding Custom Types

JSON과 같은 외부 표현과의 호환성을 위해 데이터 타입을 인코딩, 디코딩 가능하게 만든다.

<br>

---

#### Overview

많은 프로그래밍 작업에는 데이터를 네트워크 통신을 통해 보내는 것, 디스크에 데이터 저장, 또는 API와 서비스에 데이터 제출하는 것을 포함한다. 이 작업들을 데이터가 전송되는 과정에서 중간 포맷으로 데이터를 인코딩 또는 디코딩이 필요할 때가 있다.

Swift standard 라이브러리는 데이터 인코딩과 디코딩에 표준화된 접근방식을 정의한다. 우리는 `Encodable` 과 `Decodable`을 우리의 커스텀 타입에 구현하여 이 접근방식을 채택할 수 있다. 이 프로토콜을 채택하는 것은 `Encoder`와 `Decoder` 프로토콜의 구현에서 우리의 데이터를 가져와 JSON 또는 property list와 같은 외부 표현으로 데이터를 인코딩하거나 디코딩할 수 있다. 인코딩과 디코딩을 모두 사용하기 위해서는 `Codable` 프로토콜을 채택을 선언하면 되는데, 이 프로토콜은 `Encodable`과 `Decodable` 프로토콜을 합친 것이다. 이 프로세스는 우리의 타입을 codable하게 만드는 과정이다.

<br>

#### Encode and Decode Automatically

타입을 codable로 만드는 가장 간단한 방법은 이미 Codable한 타입을 사용하는 것이다. 이러한 타입에는 standard 라이브러리 타입에는 `String`, `Int` 그리고 `Double`이 있고, Foundation 타입에는 `Date`, `Data` 그리고 `URL`이 있다. 어떤 타입의 프로터피가 codable이면 `Codable` 채택을 선언하기만 하면 자동적으로 `Codable`이 채택된다.

`name`과 `foundingYear` 프로퍼티를 가지고 있는 `Landmark` 구조체를 고려해보자.

```swift
struct Landmark {
    var name: String
    var foundingYear: Int
}
```

타입에 `Codable`을 채택하면 내장된 데이터 포맷 및 커스텀 인코더와 디코더에서 제공하는 포맷과 **직렬화(serialize)** 할 수 있다. 예를들어, `Landmark` 구조체는 `Landmark` 자체에서 property list 또는 JSON을 특별히 처리하는 코드가 포함되어있지 않지만, `PropertyListEncoder`와 `JSONEncoder` 클래스를 사용하여 인코딩 될 수 있다.

같은 원칙은 codable한 커스텀 타입으로 만들어진 커스텀 타입에도 적용된다. 모든 프로퍼티가 `Codable`하다면, 어떠한 커스텀 타입이라도 `Codable`이 될 수있다.

아래 예제는 `location` 프로퍼티가 `Landmark` 구조체에 추가될 때, 어떻게 자동 `Codable` 채택이 적용되는지를 보여준다.

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

내장 타입인 `Array`, `Dictionary`, 그리고 `Optional`은 codable 타입들을 가지고 있다면, `Codable`을 채택한다. `Landmark`에 `Coordinate` 인스턴스 배열을 추가해도 되며, 전체 구조체는 `Codable`을 여전히 만족한다.

아래 예제는 `Landmark`에 내장된 codable 타입을 사용하여 여러가지 프로퍼티를 추가하면 여전히 자동 채택이 어떻게 적용되는지를 보여준다.

```swift
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

#### Encode or Decode Exclusively

몇가지 경우에는, 양방향 인코딩과 디코딩을 위해서 `Codable` 지원이 필요없을 수 있다. 예를 들면 일부 앱은 원격 네트워크 API를 호출하기만 하면되고 동일한 타입을 포함하는 응답을 디코딩할 필요가 없다. 데이터 인코딩만을 필요로한다면, `Encodable` 채택만 선언하면 된다. 거꾸로 주어진 타입의 데이터를 읽기만 한다면 `Decodable` 채택만 선언하면 된다.

아래 에제는 Landmark 구조체에서 데이터를 인코딩, 디코딩만 하는 다른 선언을 보여준다.

```swift
struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}
```

```swift
struct Landmark: Decodable {
    var name: String
    var foundingYear: Int
}
```

#### Choose Properties to Encode and Decode Using Coding Keys

`Codable` 타입은 `CodingKey` 프로토콜을 채택하는 `CodingKeys`라는 특별한 nested enumeration을 선언할 수 있다. 이 enumeration이 존재하면, codable한 타입의 인스턴스가 인코딩 또는 디코딩 될 때 포함되어야하는 권한있는 속성 목록으로 사용된다. enumeration의 케이스 이름들은 타입에 있는 프로퍼티에 지정한 이름과 일치해야한다.

인스턴스를 디코딩 할 때 존재하지 않거나 인코딩 된 표현에 특정 속성을 표함하지 않아야하는 경우에는 `CodingKeys` enumeration에서 속성을 생략한다. `CodingKeys`에서 생략된 프로퍼티에는 포함 타입이 Decodable 또는 Codable에 대한 자동 채택을 받기 위해서는 default 값이 필요하다.

만약 직렬화된 데이터 포맷에 사용된 키가 데이터 타입과 이름이 일치하지 않으면, `CodingKeys` enumeration의 raw-value로 `String` 값을 명시함으로써 대체 키를 제공해야한다. enumeration의 각 case의 raw value로 사용된 string은 인코딩과 디코딩에 사용되는 키 이름이다. case 이름과 raw value 사이의 연관성을 통해 모델링 중인 직렬화 포맷의 이름, 문장 부호 및 capitalization를 일치시키지 않고 Swif API [Design Guidelines](https://swift.org/documentation/api-design-guidelines/)에 따라 데이터 구조의 이름을 지정할 수 있다.

아래 예제는 인코딩 및 디코딩 할 때, `Landmark` 구조체의 프로퍼티인 `name`과 `foundingYear` 대체 키를 사용한 것이다.

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



#### Encode and Decode Manually

만약 Swift 타입의 구조체가 인코딩한 형태와 다르다면, 인코딩, 디코딩 로직을 정의하기 위해 Encodable과 Decodable의 커스텀 구현을 제공할 수 있다. 

아래 예제는 `additionalInfo` container 안에 중첩된 `elevation` 프로퍼티를 지원하기 위해  `Coordinate` 구조체를 확장한다.

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

`Coordinate` 타입의 인코딩된 형태에는 두 번째 수준의 nested 정보가 포함되어 있으므로 `Encodable` 및 `Decodable` 프로토콜을 채택할 때 각 타입이 특정 수준에서 사용되는 전체 coding key 세트를 나열하는 두 개의 enumeration을 사용한다.

아래 예제는 `Coordinate` structure를 필수 initializer `init(from:)`을 구현하여 `Decodable` 프로토콜에 맞게 확장한다.

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

initializer는 매개 변수로 수신한 `Decoder` 인스턴스의 메소드를 사용하여 `Coordinate` 인스턴스를 채웁니다. `Coordinate` 인스턴스의 두개의 프로퍼티는 Swift standard 라이브러리에서 제공하는 키 컨테이너 API를 사용하여 초기화됩니다.

아래 예제는 `Coordinate` 구조체가 `Encodable` 프로토콜을 채택하기 위해 필수 메소드 `encode(to:)`를 구현하여 확장되는 것이다.

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

이 `encode(to:)` 메소드 구현은 이전 예제인 디코딩 작업과 반대되는 작업이다.

