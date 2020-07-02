출처: <https://medium.com/ios-os-x-development/isolating-tasks-in-swift-or-how-to-create-a-testable-networking-layer-d0380e69f7e3>

# Isolating tasks in Swift, or how to create a testable networking layer.

지난 몇 년 동안 점점 더 빛나는 iOS 아키텍처가 많이 있습니다. 
그것들은 모두 유효하고 좋은 부분과 나쁜 부분을 가지고 있지만, **그들의 공통점(좋은 점)은 바로 비즈니스 로직과 프레젠테이션 로직을 분리한다는 것입니다.** 오늘은 당신의 다음 프로젝트 아키텍처에 적용할 수 있는 간단한 개념에 대해 설명하겠습니다.

## A pretty common networking layer

저의 요점을 설명하기 위해, 먼저 네트워크 계층이 종종 어떻게 구현되는지에 대해 조금 이야기하겠습니다.

저는 서로 다른 수많은 네트워크 계층을 보았습니다. 대부분에는 NetworkManager, ConnectionManager 또는 이와 유사한 것이 있습니다. 이것들은 앱에서 모든 단일 API들 호출을 포함하는 단일 클래스입니다. 이것들은 유효하고 잘 작동하지만 **소프트웨어 디자인의 핵심 개념인 단일 책임 (Single Responsibility)에서는 실패합니다.**

아래 그림의 `ConnectionManager`은 너무 많은 책임이 포함되어있어 우수 사례로 간주되지 않습니다. 또한 싱글톤으로 구현되는 경우가 많은데, 싱글톤이 반드시 나쁜 것은 아니지만, 의존성으로 주입될 수 없으므로, 테스트 할 때 쉽게 mock 할 수 없습니다.

* Networking Layer is commonly implemented as a singleton

![connectionManager](https://user-images.githubusercontent.com/38216027/81043176-bba80c00-8eec-11ea-9a4d-ac68ec68cfe2.png)

This is a very frequent practice. I’ve also seen this in MVVM, or MVP architectures.

이것은 매우 흔한 나쁜 관습입니다. 저는 심지어 MVVM 또는 MVP 아키텍처에서도 이 객체를 보았습니다.

## A different approach

데이터 접근 계층은 다른 방식으로 구현 될 수 있습니다. network fetching의 프로세스를 설명하겠습니다.


* Steps involved in a network call

![differentApproach](https://user-images.githubusercontent.com/38216027/81043205-cc588200-8eec-11ea-9270-09d6ff876804.png)

위와 같은 방식이면, network call은 적어도 세 단계를 의미합니다.

1. **Create a request**: this means setting the URL, method, parameters (either in URL path or http body) and HTTP headers.

2. **Dispatch request**: This is a very, very important step. The request that was created and configured in the previous step must be dispatched using URLSession, or a layer over that (for example, Alamofire).

3. **Get and parse response**: It’s important that this step should be implemented separated from the previous two. This is where you should validate JSON or XML response and parse that into a valid Entity (or Model if you prefer to say).


**당신이 정말로 클린하고 테스트 가능한 아키텍처를 갖고 싶다면, 위의 세 단계를 각각 서로 다른 객체로 구현해야합니다.**


* Network layer should be implemented using, at least, three different components.

![differentApproach2](https://user-images.githubusercontent.com/38216027/81043231-da0e0780-8eec-11ea-84ee-d852099e63d4.png)

각각의 서로 다른 객체는 위와 같습니다.

1. **Request**: A `Request` object has everything needed for configuring a network request. It is a struct, or class that is responsible for configuring a single network request. And this is very important: One network request, one `Request` object.

2. **NetworkDispatcher**: A `NetworkDispatcher` is an object that is responsible of getting a `Request` and return a `Response`. It should be implemented as a protocol. You should code against that protocol and not against a concrete class (or struct), and it should never, ever, be implemented as a singleton. If you do that, this `NetworkDispatcher` can be replaced for a `MockNetworkDispatcher` that doesn’t actually make any network request, and instead getting the response from a JSON file, leading to a naturally testable architecture.

3. **NetworkTask**: A `NetworkTask` is a subclass of the generic class Task. A task, as I will explain better in a moment, is a generic class that is responsible of getting an `Input` type and returning a `Output` type, either in a synchronous, or asynchronous way. You can implement a `Task` using RxSwift, ReactiveCocoa, Hydra, Microfutures, FOTask, or simply using closures. It’s up to you. The important part here is the concept, not the implementation details.

## Implementing a Request

`Request` 객체는 `URLRequest`를 작성하는 데 필요한 모든 것을 구성하는 객체입니다.


* 네트워크 Request의 예는 다음과 같습니다.

```swift
//
//  Request.swift
//
//  Created by Fernando Ortiz on 2/12/17.
//
import Foundation

enum HTTPMethod: String {
    case get, post, put, patch, delete
}

protocol Request {
    var path        : String            { get }
    var method      : HTTPMethod        { get }
    var bodyParams  : [String: Any]?    { get }
    var headers     : [String: String]? { get }
}

extension Request {
    var method      : HTTPMethod        { return .get }
    var bodyParams  : [String: Any]?    { return nil }
    var headers     : [String: String]? { return nil }
}
```

보이는 것처럼 간단합니다. **여기서 중요한 부분은 모든 Request객체가 별도의 객체로 구현된다는 것입니다.** 물론 이것은 Moya처럼 열거형으로 구현될 수도 있는데, 당신이 더 좋아하는 것으로 구현하십시오. 개인적으로 저는 객체 지향적 스타일을 사용하므로, BaseRequest 클래스와 AuthenticatedRequest와 같은 하위 클래스 및 GetAllUsersRequest 또는 LoginRequest와 같은 특정 Request 객체를 구현하는 것을 선호합니다.

## Implementing a NetworkDispatcher

`NetworkDispatcher`는 네트워크 요청을 발송(dispatch)하는 구성 요소입니다.

Note : 이 기사에서 필자의 예제는 RxSwift를 사용하지만 당신은 원하는 방식으로 자유롭게 구현할 수 있습니다.

```swift
//
//  NetworkDispatcher.swift
//
//  Created by Fernando Ortiz on 2/11/17.
//  Copyright © 2017 Fernando Martín Ortiz. All rights reserved.
//
import Foundation
import RxSwift

protocol NetworkDispatcher {
    func execute(request: Request) -> Observable<Any>
}
```

`NetworkDispatcher`는 Request 객체를 디스패치하고 Response를 리턴받는 단일 책임을 담당합니다.

여기서 **특정 구현(ex: 클래스) 대신 프로토콜을 사용하는 것에 대한 멋진 점은 프로토콜 기반 구현이 쉽게 상호 교환 가능(interchangable) 하다는 것**입니다. 실제로 당신은 프로토콜 덕분에 "네트워크" 작업을 수행하지 않고 대신 JSON 파일에서 응답을 반환하여 테스트를 훨씬 쉽게하는 `MockNetworkDispatcher` 객체를 사용할 수 있습니다.

## Isolating Tasks

작업들(Tasks)은 단일 로직 opration을 수행하는 간단한 객체입니다. 예를 들어 백엔드로부터 사용자들 정보를 가져오거나, 로그인하거나 또는 사용자를 등록할 수 있습니다(다 따로따로). 작업들(Tasks)은 동기식이거나 비동기식일 수 있지만, 클라이언트 쪽에서는 투명해야합니다. 저는 RxSwift의 Observable 인 편리한 추상화를 사용하는 것을 좋아하지만 Promise, Signal, Future 또는 간단한 콜백 함수이면 충분합니다.

* `Task`의 단순한 구현은 아래와 같습니다.
```swift
//
//  Task.swift
//
//  Created by Fernando Ortiz on 2/11/17.
//
import Foundation
import RxSwift

class Task<Input, Output> {
    func perform(_ element: Input) -> Observable<Output> {
        fatalError("This must be implemented in subclasses")
    }
}
```

나는 객체 지향 스타일을 사용하지만, 당신은 associated types 및 type erasure와 같은 멋진 도구로도 Task를 작성할 수 있습니다. 나는 hacky가 덜하고(단순하게 보이고) 구현이 더 간단하기 때문에 객체 지향 스타일을 선호합니다.

Every Task needs two generic parameters: a Input type, and an Output one. A Task performs an operation that includes receiving an Input object and returning an Output, maybe using an abstraction over it, like Observable.

모든 `Task`에는 `Input` type과 `Output` type의 두 가지 일반 매개 변수가 필요합니다. `Task` 객체는 `Input` 객체를 수신(receiving)하고 Observable과 같은 추상화를 사용하여 `Output`을 반환하는 작업을 수행합니다.

네트워킹 operation들을 수행(perform)하려면 `Task`를 전문화(specailize)해야 합니다.

```swift
//
//  NetworkTask.swift
//
//  Created by Fernando Ortiz on 2/11/17.
//
import Foundation
import RxSwift

class NetworkTask<Input: Request, Output>: Task<Input, Output> {
    let dispatcher: NetworkDispatcher
    
    init(dispatcher: NetworkDispatcher) {
        self.dispatcher = dispatcher
    }
    
    override func perform(_ element: Input) -> Observable<Output> {
        fatalError("This must be implemented in subclasses")
    }
}
```

보시다시피, `NetworkTask`에는 두 가지 generic type인 `Input`과 `Output`이 필요합니다. `Input`은 `Request` 객체여야 합니다. `NetworkTask`는 초기화할 때 `NetworkDispatcher`를 객체를 주입해야만 테스트 할 때 `MockNetworkDispatcher`를 쉽게 전달할 수 있습니다.

## Reviewing architecture

이러한 방식으로 비즈니스 로직을 구현하면 복잡성(complexity)을 피하고 테스트 가능성을 높이려고 다른 수단을 찾지 않고도 커플링(결합도)을 줄일 수 있습니다.

* 이 아이디어는 아래의 다이어그램으로 보여질 수 있습니다:
![totalArchitecture](https://user-images.githubusercontent.com/38216027/81043295-f90c9980-8eec-11ea-9989-192345c7bf01.png)

## Conclusion

**보다 테스트 가능한 아키텍처를 만들 수 있으므로** 비즈니스 로직 작업을 여러 객체로 분리하여 격리시키는 것(SRP)은 좋습니다. 이러한 구조는 복잡성(Complexity)을 줄이고 사용 중인 아키텍처와 완전히 독립적입니다. ViewModel, Presenter, Interactor, Store 또는 비즈니스 로직과 프리젠 테이션 로직을 분리하는 데 당신이 사용하는 모든 item 뒤에 사용할 수 있습니다.
