출처 : <https://www.swiftbysundell.com/articles/caching-in-swift/>

# Caching in Swift

앱이 빠르고 반응적으로 느껴지는 것은 UI가 렌더링되는 방식을 조정하거나 UI렌더링 작업 및 알고리즘의 실행 속도를 향상시켜서도 가능하지만, **데이터를 효율적으로 관리하고 불필요한 작업을 피하는 것으로 가능합니다.**

이러한 불필요한 작업의 가장 일반적인 원인 중 하나는 **동일한 데이터를 여러 번 다시 로드 할 때(한 마디로 캐싱을 안해서 일어나는 일들)** 입니다. 
**동일한 데이터를 여러 번 다시 로드하는 경우는 일반적으로** 동일한 모델의 복제본을 로드하는 여러 기능이거나 화면에 다시 나타날 때마다 뷰의 데이터가 다시 로드되는 경우입니다.

이번 주에는 
1. 이러한 종류의 상황에서 캐싱이 얼마나 강력한 도구가 될 수 있는지 볼 것이고, 
2. Swift의 효율적이고 우아한 캐싱 API를 어떻게 빌드하는지 볼 것이고, 
3. 그리고 다양한 값들과 객체들을 전략적으로 캐싱하는 것이 얼마나 앱의 전반적인 성능에 큰 영향을 줄 수 있는지 볼 것입니다. 

## Part of the system

캐싱은 처음에는 실제보다 훨씬 단순해 보일 수 있는 작업 중 하나입니다. 하지만,,,
1. 효율적으로 값을 저장하고 로드해야 할뿐만 아니라 
2. 메모리 풋 프린트(memory footprint)를 낮게 유지하고 
3. 오래된 데이터를 무효화하는 등의 작업을 수행하기 위해 항목을 제거할 시기를 결정해야 합니다.
<br>(따라서 절대 단순한게 아니고 복잡한 작업입니다.)

* 메모리 풋 프린트(momory footprint)란?
<br>메모리 풋 프린트는 프로그램이 실행하는 동안 사용하거나 참조하는 기본 메모리의 양을 나타냅니다. 발자국이라는 단어는 일반적으로 물체가 차지하는 물리적 치수의 크기를 말하며 크기를 의미합니다.

고맙게도 Apple은 이미 내장 NSCache 클래스를 통해 많은 문제를 해결했습니다. 
그러나 **애플 자체 플랫폼의 Objective-C 클래스(오브젝티브 C기반 이었구나!)** 이므로 클래스 인스턴스만 저장할 수 있으며 **NSObject 기반의 키(NSObject-based keys)** 만 호환되므로 몇 가지 주의사항이 있습니다.

```swift
// To be able to use strings as caching keys, we have to use
// NSString here, since NSCache is only compatible with keys
// that are subclasses of NSObject:
let cache = NSCache<NSString, MyClass>()
```

그러나 NSCache를 중심으로 얇은 Wrapper를 작성하면, 훨씬 유연한(flexible) Swift 캐싱 API를 만들 수 있습니다. 
NSCache 기저의 로직과 파워의 모든 걸 다시 작성할 필요없이(즉 NSCache 같은거 다시 만들 필요없이 얇은 Wrapper를 만들면!) 이를 통해 구조체 및 기타 value type을 저장하고 해시 가능한 키 type을 사용할 수 있습니다. 

## It all starts with a declaration

가장 먼저 할 일은 새로운 캐시 type을 선언하는 것입니다. 
이걸 캐시라고 부르고, 그리고 해시가능한 키 type과 모든 값 type에 대해 제네릭하게 만들어 봅시다. 
우리는 그런 다음 NSCache 프로퍼티를 제공할 것입니다, WrappedKey type으로 키가 지정된 Entry 인스턴스를 저장하는.

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
}
```

우리의 WrappedKey 유형은 이름에서 알 수 있듯이 NSCache와 호환되도록 공개 키 값을 래핑합니다. 
이를 달성하기 위해 NSObject를 서브 클래싱하고 hash와 isEqual을 구현해 봅시다. 
왜 hash와 isEqual를 구현해야 하면 **Objective-C가 두 인스턴스가 동일한 지 여부를 결정하는 데 hash와 isEqual를 사용하기 때문입니다.**

```swift
private extension Cache {
    final class WrappedKey: NSObject {
        let key: Key

        init(_ key: Key) { self.key = key }

        override var hash: Int { return key.hashValue }

        override func isEqual(_ object: Any?) -> Bool {
            guard let value = object as? WrappedKey else {
                return false
            }

            return value.key == key
        }
    }
}
```

Entry 유형과 관련하여 유일한 요구 사항은 클래스 (NSObject를 서브 클래스화할 필요가 없음) 여야한다는 것입니다. 즉, 단순히 Value 인스턴스를 저장할 수 있습니다.

```swift
private extension Cache {
    final class Entry {
        let value: Value

        init(value: Value) {
            self.value = value
        }
    }
}
```

위의 내용을 적용하여 이제 캐시에 초기 API 세트를 제공 할 준비가 되었습니다. 주어진 키에 대한 값을 삽입하는 방법, 값을 검색하는 방법, 기존 값을 제거하는 방법의 세 가지 방법으로 시작해 보겠습니다.

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()

    func insert(_ value: Value, forKey key: Key) {
        let entry = Entry(value: value)
        wrapped.setObject(entry, forKey: WrappedKey(key))
    }

    func value(forKey key: Key) -> Value? {
        let entry = wrapped.object(forKey: WrappedKey(key))
        return entry?.value
    }

    func removeValue(forKey key: Key) {
        wrapped.removeObject(forKey: WrappedKey(key))
    }
}
```

캐시는 본질적으로 특수한 키-값 저장소이므로 subscrpting에 이상적인 사용 사례입니다. 따라서 다음과 같은 방법으로 값을 검색하고 삽입할 수 있습니다.

```swift
extension Cache {
    subscript(key: Key) -> Value? {}
        get { return value(forKey: key) }
        s et {
            guard let value = newValue else {
                // If nil was assigned using our subscript,
                // then we remove any value for that key:
                removeValue(forKey: key)
                return
            }

            insert(value, forKey: key)
        }
    }
}
```

초기 기능 세트가 구현되었습니다. 새로운 캐시를 사용해보십시오! 
Article를 읽기위한 앱을 개발 중이며 ArticleLoader를 사용하여 Article 모델을 로드한다고 가정해 보겠습니다. 새 캐시를 사용하여 로드한 Article를 저장하고 새 Article를 로드하기 전에 이전에 캐시된 Article를 확인하면 다음과 같이 각 Article를 한 번만 로드할 수 있습니다.

```swift
class ArticleLoader {
    typealias Handler = (Result<Article, Error>) -> Void

    private let cache = Cache<Article.ID, Article>()

    func loadArticle(withID id: Article.ID,
                     then handler: @escaping Handler) {
        if let cached = cache[id] {
            return handler(.success(cached))
        }

        performLoading { [weak self] result in
            let article = try? result.get()
            article.map { self?.cache[id] = $0 }
            handler(result)
        }
    }
}
```

위의 내용은 앱 성능에 큰 영향을 미치지 않는 것처럼 보이지만, 사용자가 이미 로드된 Article로 돌아갈 때 앱이 더 빠르게 보이도록 만들 수 있습니다. 즉시 거기 있는거지.
<br>또한, 사용자가 열 가능성이 높은 Article (예 : 사용자가 좋아하는 카테고리의 최신 Article)를 **prefetching하는 것**을 위의 캐시와 결합(combining)하면 앱을 훨씬 더 즐겁게 사용할 수 있습니다.
(prefetched data는 데이터를 사용하기 전에 미리 캐시에 데이터를 집어넣는 것이다.)

## Avoiding stale data

NSCache가 Swift 표준 라이브러리 (사전 등)에있는 컬렉션에 비해 값을 캐싱하는 데 더 적합한 이유는 시스템의 메모리가 부족할 때 자동으로 객체를 제거하여 앱 자체를 더 오래 메모리에 유지할 수 있다는 것입니다.

그러나 우리는 직접 캐시 무효화 조건을 추가하고 싶을 수 있습니다. 그렇지 않으면 NSCache더라도 오래된 데이터가 유지 될 수 있습니다. 이미 로드한 데이터를 재사용 할수 있는 것은 좋은 일이지만, **사용자에게 오래된 데이터를 표시하는 것은 분명하지 좋지 않죠.**

이 문제를 완화하는 한 가지 방법은 특정 시간 간격 후에 캐시 항목을 제거하여 캐시 항목의 수명을 제한하는 것입니다. **이를 위해 Entry 클래스에 expirationDate 속성을 추가하여 각 항목의 남은 수명을 추적(keep track of)할 수 있습니다.**

```swift
final class Entry {
    let value: Value
    let expirationDate: Date

    init(value: Value, expirationDate: Date) {
        self.value = value
        self.expirationDate = expirationDate
    }
}
```

다음으로 주어진 항목(entry)이 여전히 유효한지 확인하기 위해 캐시가 현재 날짜를 보유할 수 있는 방법이 필요합니다. 
필요할 때마다 Date() 인라인(inline)을 호출 할수는 있지만 **단위 테스트가 매우 어려워집니다.** 대신 이니셜 라이저의 일부로 Date-producing 함수를 삽입하십시오. 
또한 기본값 12시간으로 entryLifetime 속성을 추가합니다.

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
    private let dateProvider: () -> Date
    private let entryLifetime: TimeInterval

    init(dateProvider: @escaping () -> Date = Date.init,
         entryLifetime: TimeInterval = 12 * 60 * 60) {
        self.dateProvider = dateProvider
        self.entryLifetime = entryLifetime
    }
    
    ...
}
```
위의 내용을 적용하여 현재 날짜와 지정된 entryLifetime을 고려하여 값을 삽입하고 검색하는 방법을 업데이트하겠습니다.

```swift
func insert(_ value: Value, forKey key: Key) {
    let date = dateProvider().addingTimeInterval(entryLifetime)
    let entry = Entry(value: value, expirationDate: date)
    wrapped.setObject(entry, forKey: WrappedKey(key))
}

func value(forKey key: Key) -> Value? {
    guard let entry = wrapped.object(forKey: WrappedKey(key)) else {
        return nil
    }

    guard dateProvider() < entry.expirationDate else {
        // Discard values that have expired
        removeValue(forKey: key)
        return nil
    }

    return entry.value
}
```

오래된 항목을 정확하게 무효화하는 것은 모든 종류의 캐싱을 구현하는 데 있어 가장 어려운 부분일 수 있습니다.
위의 만료 날짜를 특정 이벤트(예 : 사용자가 기사를 삭제하는 경우)에 따라 값을 제거하기 위한 모델별 논리와 **결합**함으로써 **중복 작업과 유효하지 않은 데이터를 모두** 피할 수 있습니다.

## Persistent Caching (캐시된 값을 디스크에 유지하기)

지금까지 우리는 메모리에 값을 캐싱했습니다. 즉, 앱이 종료되자마자 데이터가 사라질 것입니다.
<br>이것이 실제로 원하는 것일 수도 있지만 때로는 캐시된 값을 디스크에 유지하는 것이 정말로 가치가 있을 수 있으며 우리 앱을 사용하는 것의 새로운 방법을 열수 있습니다. **가령 offline 중에 앱을 런칭할 때 network를 통해 다운로드된 데이터에 여전히 접근하는 것입니다.**  

디스크에 특정 캐시만 선택적으로 유지하고 싶을 수도 있으므로 완전히 선택적 기능으로 만들어 보겠습니다. 
시작하려면 Entry을 업데이트하고 또한 관련 키를 저장하여, 우리가 각 entry을 직접 유지(디스크에 유지한다는 뜻)하고 사용하지 않는 키를 제거할 수 있도록 합니다.
(디스크에서도 사용성이 많은 데이터들만 골라 저장하고 사용하지 않는 것은 제거한다는 뜻이다.)

```swift
final class Entry {
    let key: Key
    let value: Value
    let expirationDate: Date

    init(key: Key, value: Value, expirationDate: Date) {
        self.key = key
        self.value = value
        self.expirationDate = expirationDate
    }
}
```

다음으로 NSCache는 해당 정보를 공개하지 않기 때문에 캐시에 entry가 포함 된 키를 추적(track)할 수 있는 방법이 필요합니다. 
이를 위해 entry가 삭제될 때마다 알림을 받기 위해 기본 NSCache의 delegate가 될 전용 KeyTracker 타입을 추가합니다.

```swift
private extension Cache {
    final class KeyTracker: NSObject, NSCacheDelegate {
        var keys = Set<Key>()

        func cache(_ cache: NSCache<AnyObject, AnyObject>,
                   willEvictObject object: Any) {
            guard let entry = object as? Entry else {
                return
            }

            keys.remove(entry.key)
        }
    }
}
```

캐시를 초기화하는 동안 KeyTracker를 설정하고 최대 항목 수를 설정하여 다음과 같이 디스크에 너무 많은 데이터를 쓰지 않도록합니다.

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
    private let dateProvider: () -> Date
    private let entryLifetime: TimeInterval
    private let keyTracker = KeyTracker()

    init(dateProvider: @escaping () -> Date = Date.init,
         entryLifetime: TimeInterval = 12 * 60 * 60,
         maximumEntryCount: Int = 50) {
        self.dateProvider = dateProvider
        self.entryLifetime = entryLifetime
        wrapped.countLimit = maximumEntryCount
        wrapped.delegate = keyTracker
    }
    
    ...
}
```

캐시에서 항목을 제거 할 때마다 KeyTracker에 이미 알림이 표시되므로 통합을 완료하기 위해 수행해야 할 모든 작업은 키가 추가 될 때마다 이를 알리는 것입니다, 키가 추가되는 코드는 insert 메소드의 한 부분입니다. 

```swift
func insert(_ value: Value, forKey key: Key) {
    ...
    keyTracker.keys.insert(key)
}
```

캐시의 내용을 실제로 디스크에 유지하려면 먼저 캐시를 직렬화해야합니다. NSCache를 활용하여 시스템 위에 직접 캐싱 API를 구축한 것과 마찬가지로 Codable을 사용하여 호환 가능한 형식 (예 : JSON)을 사용하여 캐시를 인코딩 및 디코딩 할 수 있습니다.

Entry 타입을 Codable을 적용하는 것으로 시작하겠습니다. 
그러나 모든 캐시 entry를 codable 할 필요는 없습니다. 
따라서 다음과 같이 Codable 가능 키와 값이 있는 entry에 대해서만 Codable이 가능하도록 다음과 같이 조건부 적합성(conditional conformance)을 사용합시다
```swiftsed
extension Cache.Entry: Codable where Key: Codable, Value: Codable {}
```

인코딩 및 디코딩 프로세스하는 동안에 Entry들을 검색하고 삽입하므로, 따라서 이전 `insert` 및 `value` 메소드에서 코드가 중복되는 것을 방지해야 합니다.
Entry 인스턴스를 처리하는 모든 논리를 두 가지 새로운 개인 유틸리티 메소드로 옮깁시다.

```swift
private extension Cache {
    func entry(forKey key: Key) -> Entry? {
        guard let entry = wrapped.object(forKey: WrappedKey(key)) else {
            return nil
        }

        guard dateProvider() < entry.expirationDate else {
            removeValue(forKey: key)
            return nil
        }

        return entry
    }

    func insert(_ entry: Entry) {
        wrapped.setObject(entry, forKey: WrappedKey(entry.key))
        keyTracker.keys.insert(entry.key)
    }
}
```

퍼즐의 마지막 부분은 이전에 사용했던 것과 동일한 조건에서 캐시 자체를 코딩 가능하게 만드는 것입니다. 위의 두 가지 유틸리티 방법을 사용하여 이제 모든 항목을 매우 쉽게 인코딩하고 디코딩 할 수 있습니다.

```swift
extension Cache: Codable where Key: Codable, Value: Codable {
    convenience init(from decoder: Decoder) throws {
        self.init()

        let container = try decoder.singleValueContainer()
        let entries = try container.decode([Entry].self)
        entries.forEach(insert)
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        try container.encode(keyTracker.keys.compactMap(entry))
    }
}
```

위의 코드를 사용하면 Codable한 키와 값이 포함된 캐시를 데이터로 인코딩한 다음 데이터를 인코딩한 다음 앱의 전용 캐싱 디렉토리에있는 파일에 다음과 같이 파일에 기록하면 됩니다.

```swift
extension Cache where Key: Codable, Value: Codable {
    func saveToDisk(
        withName name: String,
        using fileManager: FileManager = .default
    ) throws {
        let folderURLs = fileManager.urls(
            for: .cachesDirectory,
            in: .userDomainMask
        )

        let fileURL = folderURLs[0].appendingPathComponent(name + ".cache")
        let data = try JSONEncoder().encode(self)
        try data.write(to: fileURL)
    }
}
```

마찬가지로 NSCache 및 NSCache와 같은 시스템 API를 활용하여 시간 기반 무효화, 디스크 내 지속성 및 포함된 항목 수에 대한 제한을 지원하는 Swift와 완벽하게 호환되는 매우 동적 인 캐시를 구축했습니다. 휠을 다시 발명하지 않아도되도록 코딩 할 수 있습니다.

## Conclusion

동일한 데이터를 여러 번 다시 로드하지 않도록 전략적으로 캐싱을 배포하면 앱 성능에 큰 **긍정적인 영향**을 줄 수 있습니다. 
결국 앱 내에서 데이터를 로드하는 방식을 최적화하더라도 데이터를 전혀 로드하지 않아도 항상 빨라질 수 있으며 **캐싱은 이를 달성하기위한 좋은 방법이 될 수 있습니다.**

그러나 데이터로딩 파이프라인에 캐싱을 추가 할 때 명심해야 할 여러 가지 사항이 있습니다. 
예를 들어 오래된 데이터를 너무 오래 보관하지 않고 앱 환경이 변경될 때 캐시 항목을 무효화하는 경우 (예 : 사용자가 원하는 로케일을 변경하는 경우), 삭제 된 항목이 올바르게 제거되었는지 확인하십시오.

캐싱을 배포 할때 고려해야 할 또 다른 사항은 캐시 할 데이터와 수행할 위치입니다. 
이 기사에서 NSCache 기반 접근 방식을 살펴 보았지만 네트워킹 계층 내에서 캐싱을 수행하기 위해 다른 시스템 API (URLCache)를 사용하는 것과 같이 탐색 할 수있는 다른 여러 경로가 있습니다. 다음 기사에서 이 내용과 다른 유형의 캐싱에 대해 자세히 살펴 보겠습니다.

> 로케일(locale)이란

```
세계 여러 나라들은 각자 다른 문화(언어, 날짜, 시간 등)을 갖고 있다. 프로그램의 국제화(Internationalization, 줄여서 i18n)는 사용자로 하여금 프로그램 수행시 로케일이란 것에 의해 입맛에 맞는 환경을 선택할 수 있도록 만든 것을 말한다. 예를 들어 어떤 프로그램의 메시지가 여러가지 언어로 주어져 있는 경우 이중에 어떤 언어의 것을 출력할 것인가를 사용자가 결정할 수 있는 것이다. 그것을 가능하게 해 주는 수단이 바로 로케일이다. 이것은 단순히 메시지 뿐만이 아니고 숫자표현법, 날짜 또는 시간표현법 등 여러가지에 사용될 수 있다. 그것 각각을 우리는 카테고리(category)라고 부른다. 카테고리에는 LC_COLLATE, LC_CTYPE, LC_MESSAGES, LC_MONETARY, LC_NUMERIC, LC_TIME 가 있다.
```