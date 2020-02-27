# Swift’s closure capturing mechanics

### Implicit capturing

프로퍼티에 저장되거나 다른 탈출 클로저에 의해 캡쳐되는 **'탈출 클로저'**를 정의할때마다, 암시적으로 다른 **[객체, 값, 함수]**들을 캡쳐할 것이다. 이와 같은 클로저들은 조금 시간이 지난 뒤에 실행될 수도 있기 때문에, 그동안 할당이 해제되는 것을 방지하기 위해 값들은 **[객체, 값, 함수]**의 의존성에 강한 참조를 유지할 필요가 있다

GCD를 사용하여 `UIAlertController`를 3초 뒤에 표시하기 위해서는, **view controller 인스턴스**인 `presenter`를 **캡쳐**하기 위해 `asyncAfter` 호출로 전달된 클로저가 필요하다.

```swift
func presentDelayedConfirmation(in presenter: UIViewController) {
    let queue = DispatchQueue.main

    queue.asyncAfter(deadline: .now() + 3) {
        let alert = UIAlertController(
            title: "...",
            message: "...",
            preferredStyle: .alert
        )

        // By simply refering to 'presenter' here, our closure
        // will automatically capture that instance, and retain
        // it until the closure itself gets released from memory:
        presenter.present(alert, animated: true)
    }
}
```

위 과정은 굉장히 편리한하지만, 조심하지 않으면 까다로운 버그나 메모리 관련 이슈를 발생시킬 수 있다.

예를 들어, 위와 같이 코드의 실행을 몇초 가량을 지연시켰기 때문에, `presenter` view controller는 클로저가 실제로 실행됐을 때 앱의 view 계층에서 제거됐을 수가 있다. view controller가 여전히 다른 객체에 의해 retain되고 있다면 present를 하는게 좋다.

### Using capture lists

주어진 클로저가 어떻게 객체나 값들을 캡쳐하는지를 커스터마이즈 할 수 있게 해준다. 캡쳐 리스트를 사용하면, 위 클로저에게 `presenter` view controller를 약하게 캡쳐하라고 지시할 수 있다. 이러한 방식은 view controller는 참조되는 코드가 없다면 메모리에서 할당 해제되며 메모리는 더 빠르게 공간을 확보하고 불필요한 작업을 줄일 수 있다.

```swift
func presentDelayedConfirmation(in presenter: UIViewController) {
    let queue = DispatchQueue.main

    // A capture list is defined using a set of square brackets
    // directly following a closure's opening curly bracket:
    queue.asyncAfter(deadline: .now() + 3) { [weak presenter] in
        // Here we verify that our presenter is still in memory,
        // otherwise we can return early:
        guard let presenter = presenter else { return }

        let alert = UIAlertController(
            title: "...",
            message: "...",
            preferredStyle: .alert
        )

        presenter.present(alert, animated: true)
    }
}
```

<br>

**캡쳐 리스트**는 `self`를 참조할 때 더욱 유용한데, 특히 두개의 객체나 클로저가 서로를 참조하여 생기는 retain cycle이 발생할때 유용하다.

캡쳐 리스트를 사용해 `self`가 강하게 참조되는 것을 피했다. 여기서 클로저는 `self`에 의해 참조가 유지될 수 있었다.

```swift
class UserModelController {
    let storage: UserStorage
    private var user: User { didSet { userDidChange() } }

    init(user: User, storage: UserStorage) {
        self.storage = storage
        self.user = user

        storage.addObserver(forID: user.id) { [weak self] user in
            self?.user = user
        }
    }
}
```

`self`를 약하게 캡쳐하지 않았다면 `UserStorage`가 클로저에 참조를 유지하기 때문에 결국 retain cycle이 발생했을 것이다. 그리고 `self`는 이미 `storage` 프로퍼티를 통해 참조를 유지하고 있다.

<br>

### Weak references are not always the answer

두개의 샘플 코드를 보면 self를 약하게 참조하는 방식이 언제나 맞는 방법인 것 같지만, 꼭 그런 것은 아니다. 다른 종류의 메모리 관리와 마찬가지로, 우리는 `self`가 각 상황에서 어떻게 사용되고 있는지, 그리고 캡쳐 클로저가 얼마나 메모리에 남아있어야하는지를 신중하게 고려해야한다.

예를 들어, UIView.animate API와 같이 빨리 끝나는 클로저를 다룰 때에는, `self`를 캡쳐하는 것은 큰 문제가 되지 않으며 코드를 더 읽기 쉽게 도와준다.

```swift
extension ProductViewController {
    func expandImageView() {
        UIView.animate(withDuration: 0.3) {
            self.imageView.frame = self.view.bounds
            self.showImageCloseButton()
        }
    }
}
```

탈출 클로저 내에서 인스턴스 메소드와 속성에 접근할 때 헝상 명시적으로 `self`를 참조해야한다. That’s a good thing, as it requires us to make an explicit decision to capture `self`, given the consequences that doing so might have.

때로는 `self`를 조금 더 오래 참조하여 사용하고 싶을 상황이 있을 수 있다. 예를들어 만약 클로저의 작업을 위해 현재 객체가 필요할 때이다.

```swift
extension NetworkingController {
    func makeImageUploadingTask(for image: Image) -> Task {
        Task { handler in
            let request = Request(
                endpoint: .imageUpload,
                payload: image
            )

            // The current NetworkingController is required here,
            // so for as long the returned task is retained,
            // we'll also retain its underlying controller:
            self.perform(request, then: handler)
        }
    }
}
```

위 코드는 `NetworkingController`가 생선한 task에 대한 참조를 유지하지 않기 때문에 retan cycle이 발생하지 않는다. 

우리는 capture list를 사용하여 `self`를 참조하는 것이 아니라, 각 클로저의 의존성들을 직접적으로 캡쳐할 수 있다. 예를 들어 여기서 우리는 image loder의 `cache` 프로퍼티를 **캡쳐**하여 image가 성공적으로 다운로드 되면 이를 사용하려고 한다.

```swift
class ImageLoader {
    private let cache = Cache<URL, Image>()

    func loadImage(
        from url: URL,
        then handler: @escaping (Result<Image, Error>) -> Void
    ) {
        // Here we capture our image loader's cache without
        // capturing 'self', and without having to deal with
        // any optionals or weak references:
        request(url) { [cache] result in
            do {
                let image = try result.decodedAsImage()
                cache.insert(image, forKey: url)
                handler(.success(image))
            } catch {
                handler(.failure(error))
            }
        }
    }
}
```

위 방식은 `self`로 전체로 접근하는 것보다 이렇게 몇개의 프로퍼티에만 접근하려고 할 때 유용하다. 단, 캡쳐되는 프로퍼티들이 클래스 인스턴스와 같은 **레퍼런스 타입**이거나 **immutable value 타입**을 가지고 있는 한에서 적용된다.



#### references

- [원문 - https://www.swiftbysundell.com/articles/capturing-objects-in-swift-closures/#capturing-the-context-instead-of-self](https://www.swiftbysundell.com/articles/swifts-closure-capturing-mechanics/) 