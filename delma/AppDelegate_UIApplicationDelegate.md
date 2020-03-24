# AppDelegate.swift 와 UIApplicationDelegate



<br>


> App Delegate 에 대해 이해하고자 공식 문서를 번역, 정리한 글입니다 


<br>

프로젝트를 만들면 반드시 있는 AppDelegate.swift 파일은 두가지의 주요한 기능을 한다
## AppDelegate.swift 의 역할 2가지
<br>

### 1. AppDelegate 클래스 정의

app delegate(AppDelegate클래스의 인스턴스)는 앱 안에서 상태 변화에 응답하고, 앱의 컨텐츠가 그려지는 창(window)를 만든다.  * -> 창을 만드는 부분은 SceneDelegate가 생기면서 그쪽으로 넘어간 듯 보임. *

### 2. entry point(진입 지점)과 입력 이벤트(input events)를 앱에 전달하는 run loop 생성

이 작업은 `UIApplicationMain` 속성에 의해 완료된다. (상단에 있는 `@UIApplicationMain` 파일)
`UIApplicationMain` 속성을 사용하는 것은 `UIApplicationMain` 함수를 호출하고 AppDelegate 클래스의 이름을 delegate 클래스의 이름으로 전달하는 것과 같다. (UIApplicationMain 메서드의 인자로 들어갈 delegate 클래스 이름)

이에 대한 응답으로 시스템은 응용 프로그램 객체(application object)를 생성한다.
이 응용프로그램 객체는 앱의 생명주기를 관리한다. 시스템은 또한 AppDelegate 클래스의 인스턴스를 생성하고 응용프로그램 객체에 할당한다.
마침내 시스템이 앱을 시작한다.!

<img src="https://images.velog.io/images/delmasong/post/a54e0e1b-3317-45fa-abcb-60eae5f89be7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-03-11%20%EC%98%A4%ED%9B%84%207.21.01.png" width = "80%"/>

AppDelegate 클래스는 프로젝트를 생성하면 자동적으로 만들어진다. 아주 예외적인(highly unusual) 일을 하지 않는 한, Xcode가 제공하는 이 AppDelegate 클래스를 사용해 앱을 초기화하고 앱 수준 이벤트에 응답해야 한다. AppDelegate 클래스는 UIApplicationDelegate 프로토콜을 채택한다. 이 프로토콜은 앱을 설정하고 앱의 상태변화에 응답하고 다른 앱 수준 이벤트를 다루는 방법을 정의한다. 

공식 문서에는 AppDelegate 클래스의 단일 프로퍼티인 window에 대해 설명이 있는데, 현재 프로젝트를 생성하면 AppDelegate가 아닌 SceneDelegate 클래스에 window 프로퍼티가 있습니다.! 아직 공식 문서가 업데이트 되지 않았나봐요. 그래도 해당 프로퍼티가 어떤 역할을 맡고있는지는 알아야 하니까 프로젝트 상에서는 업데이트 되었다는 걸 인지하고 마저 알아봅시다.!

```swift
var window: UIWindow?
```

이 프로퍼티는 앱의 창(window)의 참조를 저장한다. 이 창은 **앱의 뷰 구조의 루트**를 나타낸다. 이 윈도우는 앱 컨텐츠가 그려지는 장소다. window 프로퍼티가 옵셔널인것에 주목하자. 이것은 어떤 부분에서 값이 없을수도(nil) 있음을 의미한다.

AppDelegate 클래스는 아래의 델리게이트 메소드 구현을 포함한다.
```swift
//애플리케이션이 실행된 직후 사용자의 화면에 보여지기 직전에 호출 
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool	

//애플리케이션이 최초 실행될 때 호출되는 메소드 
func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool		
//애플리케이션이 InActive 상태로 전환되기 직전에 호출 
func applicationWillResignActive(_ application: UIApplication)	

//애플리케이션이 백그라운드 상태로 전환된 직후 호출
func applicationDidEnterBackground(_ application: UIApplication)	

//애플리케이션이 Active 상태가 되기 직전, 화면에 보여지기 직전에 호출 
func applicationWillEnterForeground(_ application: UIApplication)	

//애플리케이션이 Active 상태로 전환된 직후 호출
func applicationDidBecomeActive(_ application: UIApplication)

//애플리케이션이 종료되기 직전에 호출 
func applicationWillTerminate(_ application: UIApplication)	
```

이 메소드들은 응용프로그램 객체가 app delegate(AppDelegate 클래스의 인스턴스)와 상호작용 하게 한다. 앱의 상태가 변하는 동안(앱 시작,종료, 백그라운드로 전환 등) 응용프로그램 앱이 응답할 기회를 주면서 객체는 대응되는 델리게이트의 메소드를 호출한다. 응용프로그램 객체가 해당 작업을 처리하므로, 올바른 시간에 이러한 메소드를 호출하기 위해 별다른 작업을 수행하지 않아도 된다. 

각각의 델리게이트 메소드는 기본 동작(default behavior)을 가지고 있다. 상세 내용을 구현하지 않거나 AppDelegate 클래스에서 해당 메소드를 삭제하면 해당 메소드가 호출될 때 기본 동작을 한다. 
아니면 커스텀 코드를 그 안에 작성할 수도 있다. 

<br>
<br>
<br>

<img src = "https://images.velog.io/images/delmasong/post/e85dfa18-7974-487d-b3e2-dcfbc11b07cc/xmZbnbE.png" width = "70%"/>




### App State

- **Not Running** : 실행되지 않았거나, 시스템에 의해 종료된 상태
- **Inactive** : 실행 중이지만 이벤트를 받고있지 않은 상태. 예를들어, 앱 실행 중 미리알림 또는 일정 얼럿이 화면에 덮여서 앱이 실질적으로 이벤트는 받지 못하는 상태 등
- **Active** : 어플리케이션이 실질적으로 활동하고 있는 상태.
- **Background** : 백그라운드 상태에서 동작을 하고 있는 상태. 예를 들어 백그라운드에서 음악을 실행하거나, 걸어온 길을 트래킹 하는 등의 행동이 돌아감.
- **suspend** : 백그라운드 상태에서 활동을 멈춘 상태. 빠른 재실행을 위하여 메모리에 적재된 상태지만 실질적으로 동작하고 있지는 않음. 메모리가 부족할때 시스템이 강제종료 함

<br>
<br>
<br>

<hr>

<br>
<br>


AppDelegate 클래스가 채택한 UIApplicationDelegate에 대해 공식문서를 기반으로 좀 더 자세히 알아보자.!

<br>

## UIApplicationDelegate
앱의 공유된 동작을 관리하는데 사용하는 일련의 방법.

<br>


### Declaration
```swift
protocol UIApplicationDelegate
```

<br>

### Overview
앱 델리게이트 객체는 앱의 공유된 행동을 관리한다. 앱 델리게이트는 사실상 앱의 루트 객체이고 시스템과의 상호작용을 관리하기 위해 `UIApplication` 과 함께 작동한다. UIApplication 객체와 마찬가지로, UIKit은 앱 실행주기 초기에 앱 델리게이트 객체를 생성해서 항상 존재한다. 

아래의 작업을 처리하기 위해 앱 델리게이트 객체를 사용한다
- 앱의 중앙 데이터 구조를 초기화함
- 앱의 화면을 구성함
- 메모리 부족 경고, 다운로드 완료 알림 같은 앱 외부에서 발생한 알림에 응답함
- 앱 자체를 타겟으로 해 앱의 scene, 뷰나 뷰 컨트롤러와 관련이 없는 이벤트에 응답함
- Apple Push Notification 서비스와 같이 시작시 필요한 서비스를 등록함


<br>

### Life Cycle Management in iOS 12 and Earlier
iOS 12 및 이번 버전에서는 앱 델리게이트를 이용해 앱의 주요 생명 주기를 관리한다. 특히, 앱이 포어그라운드나 백그라운드로 이동할 때 앱 델리게이트의 메서드를 사용해 앱의 상태를 업데이트 한다.




<br>
<br>
<br>


**References**
- https://developer.apple.com/documentation/uikit/uiapplicationdelegate
- https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/BuildABasicUI.html#//apple_ref/doc/uid/TP40015214-CH5-SW1