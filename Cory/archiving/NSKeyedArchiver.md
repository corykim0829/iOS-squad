## NSKeyedArchiver

키로 참조되는 아카이브에 객체의 데이터를 저장하는 **인코더**

<br>

#### Declaration

```swift
class NSKeyedArchiver : NSCoder
```

<br>

#### Overview

`NSKeyedArchiver`는 `NSCoder`를 상속한 서브클래스로 객체(및 스칼라 값)을 **파일에 저장**하기에 적합한 방법을 제공하는데, 이는 **아키텍처 독립적인 형태(architecture-independent)**로 인코딩하는 방법이다. 우리가 객체 세트(set)를 아카이브할 때, 아카이버(archiver)는 각 객체에 대한 클래스 정보와 인스턴스 변수를 아카이브에 쓴다. `NSKeyedUnarchiver`는 아카이브에 있는 데이터를 디코드하고 기존 오브젝트 세트와 동일한 ㅇ세트를 생성한다.

**keyed archive**는 키 기반 아카이브로 키 기반이 아닌 아카이브인 **non-keyed archive**라는 두 개 타입의 아카이브가 있는데, keyed archive는 인코딩되는 모든 객체와 값은 **이름 또는 키**를 가지고 있다. non-keyed archive를 디코딩 할 때에는 디코더는 기존의 인코더가 사용했던 같은 순서대로 디코딩해야한다. keyed archive를 디코딩할 때에는 디코더는 **이름**으로 값을 요청하기 때문에 인코딩 한 값이 있다면 그대로 디코딩한 값을 디코드하거나, 아무 것도 디코드할 수 없다. 따라서 keyed archive는 순반향 및 역방향 호환성을 더 잘 지원한다.

인코드된 값에 **주어진 키**는 현재 인코딩되는 오브젝트 스코프 안에서만 유일해야 한다. keyed archive는 **계층적**이기 때문에 `오브젝트 A`에서 인스턴스 변수를 인코드하기 위해 **사용된 키**는 `오브젝트 B`에 사용되어도 충돌이 나지 않는다. A와 B가 같은 클래스의 인스턴스여도 이는 성립된다. 하지만 단일 객체 안에서는 서브클래스에 사용된 키는 상위 클래스와 **충돌**이 날 수 있다.

`NSArchiver` 오브젝트는 아카이브 데이터를 파일 또는 mutable 데이터 오브젝트(`NSMutableData` 인스턴스)로 쓸(write) 수 있다.
<br>

---

> Type Method

### archivedData(withRootObject:requiringSecureCoding:)

주어진 root 오브젝트의 오브젝트 그래프를 데이터 표현식으로 인코딩한다. 선택적으로 secure coding을 요구한다.

<br>

#### Declaration

```swift
class func archivedData(withRootObject object: Any, 
  requiringSecureCoding requiresSecureCoding: Bool) throws -> Data
```

#### Parameters

- `object`

  The root of the object graph to archive.

- `requiresSecureCoding`

  A Boolean value indicating whether all encoded objects must conform to [`NSSecureCoding`](https://developer.apple.com/documentation/foundation/nssecurecoding).

<br>

#### Discussion

`NSKeyedUnarchiver`가 디코드할 수 없는 오브젝트를 인코딩하는 가능성을 막기 위해서, `requiresSecureCoding`을 가능하면 true로 설정한다. 이로 인해 모든 인코딩된 객체는 `NSSecureCoding`을 채택한다.

> **Note**

secure coding을 가능하게 해도 아카이브의 출력 포맷은 변하지 않는다. 이는 우리가 secure coding을 가능하게 아카이브를 인코딩하게 하는 것이며, 나중에 secure coding이 불가능한 상태로 디코딩하게 하는 것이다.

<br>


#### References

- [NSKeyedArchiver Document](https://developer.apple.com/documentation/foundation/nskeyedarchiver)
- [archivedData(withRootObject:requiringSecureCoding:) Document](https://developer.apple.com/documentation/foundation/nskeyedarchiver/2962880-archiveddata)

