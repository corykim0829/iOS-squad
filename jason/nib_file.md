# Nib Files 
(Nib 파일은 쉽게 말해 storyboard 관련 파일들이다.)
<br>Nib 파일은 OS X 및 iOS에서 응용 프로그램을 만드는 데 중요한 역할을 합니다.
**Nib 파일을 사용하면 프로그래밍 방식이 아닌 Xcode를 사용하여 그래픽으로 사용자 인터페이스를 만들고 조작 할 수 있습니다.** 
변경 결과를 즉시 볼 수 있기 때문에 다양한 레이아웃과 설정(configure)을 매우 빠르게 실험할 수 있습니다.
당신은 또한 이후에 당신의 유저 인터페이스의 많은 부분들을 코드를 변경할 필요없이 고칠필요 없이 바꿀 수 있습니다.**(nib file을 통해서!!)**

AppKit 또는 UIKit 프레임 워크를 사용하여 빌드된 응용 프로그램의 경우 nib 파일이 추가로 중요합니다.
이 두 프레임 워크는 windows, views 및 controls을 배치하고 해당 항목을 응용 프로그램의 이벤트 처리 코드와 통합하기 위해 nib 파일 사용을 지원합니다. 
Xcode는 이러한 프레임 워크와 함께 작동하여 사용자 인터페이스의 컨트롤을 해당 컨트롤에 응답하는 프로젝트의 객체에 연결합니다. 
이 통합으로 nib 파일이 로드된 후 필요한 설정 양이 크게 줄어들고 나중에 코드와 사용자 인터페이스 간의 관계를 쉽게 변경할 수 있습니다.

> Note : nib 파일을 사용하지 않고 Objective-C 애플리케이션을 작성할 수 있지만 그렇게하는 것은 매우 드물며 권장되지 않습니다. 응용 프로그램에 의존하면서, nib 파일 사용을 피하는 것은 많은 양의 프레임 워크 동작을 교체해야 nib 파일을 사용할 때와 동일한 결과를 얻을 수 있습니다. ( 한 마디로 nib 파일 사용을 피하고 코드로 작성하는 것은 프레임 워크와 관련된 코드를 많이 작성해야 한다는 것이다. )

# The Nib Object Life Cycle

'nib' 파일이 메모리에 로드될 때, nib-loading 코드는 nib 파일의 객체가 올바르게 생성되고 초기화되도록 여러 단계를 수행합니다. 이러한 단계를 이해하면 사용자 인터페이스를 관리하기 위해 더 나은 컨트롤러 코드를 작성할 수 있습니다.

## The Object Loading Process

NSNib 또는 NSBundle의 메소드를 사용하여 nib 파일에서 객체를 로드하고 인스턴스화 할 때 기본 nib-loading 코드는 다음을 수행합니다. 

1. nib 파일의 내용과 참조 된 리소스 파일을 메모리에 로드합니다.
    * 전체 'nib'객체 그래프(object graph)의 원시 데이터는 메모리에 로드되지만 언아카이브(unarchived)는 아직 되지 않습니다.
    * nib 파일과 관련된 모든 custom image resources가 로드되어 Cocoa 이미지 캐시에 추가됩니다.

2. nib object graph를 언아카이브하고 그 객체들을 초기화합니다. 각각의 새로운 객체를 어떻게 초기화할지는 객체의 타입에 따라 다르고 그리고 아카이브에서 어떻게 인코딩 되었느냐에 따라 다릅니다. nib-loading 코드는 다음의 규칙들을 순서대로 사용하여 사용할 초기화 방법들을 결정합니다.  
    1. 기본적으로, 객체들은 initWithCoder: 라는 메시지를 받습니다. 
    <br><br>OS X에서, 표준 객체(standard object)의 목록은 시스템에서 제공하고 Xcode 라이브러리에서 사용할 수 있는 view, cells, menus, view controllers 를 포함합니다.
    표준 객체 목록은 또한 사용자 플러그인을 사용하여 라이브러리에 추가된 외부의 객체들도 포함합니다. 설령 당신이 이러한 객체들의 클래스를 변경할지라도(표준 객체의 클래스를 custom하더라도), Xcode는 표준 객체(standard object)를 nib 파일로 인코딩하고 이 객체가 언카이브될 때 아카이버가 custom class로 스왑하도록 지시합니다.
    <br><br>iOS에서 NSCoding 프로토콜을 준수하는 모든 객체는 initWithCoder : 메소드를 사용하여 초기화됩니다.    
    여기에는 기본 Xcode 라이브러리 또는 custom 클래스의 일부인지에 관계없이 UIView 및 UIViewController의 모든 subclass가 포함됩니다.

    2. OS X의 Custom views는 initWithFrame : 메시지를받습니다.
    custom view는 Xcode에 사용 가능한 구현이 없는 NSView의 서브 클래스입니다. 일반적으로 이들은 앱에서 정의하고 custom 시각적 콘텐츠를 제공하는 데 사용하는 view입니다. custom view에는 기본 라이브러리 또는 통합된 타사(third party) 플러그인의 일부인 표준 시스템 view(NSSlider와 같은)가 포함되지 않습니다.
    <br><br>커스텀 뷰를 만나면 Xcode는 특별한 NSCustomView 객체를 nib 파일로 인코딩합니다. custom view 객체에는 당신이 지정한 실제 view 하위 클래스를 빌드하는 데 필요한 정보가 포함됩니다.
    로드시 NSCustomView 객체는 alloc 및 initWithFrame : 메시지를 실제 뷰 클래스로 보낸 다음 결과 뷰 객체를 교체합니다.
    실제 효과는 실제 뷰 객체가 nib-loading 프로세스 동안 이후의 상호 작용을 처리한다는 것입니다.
    <br><br>iOS의 custom view는 초기화에 initWithFrame : 메소드를 사용하지 않습니다.

    3. 이전 단계에서 설명한 것 이외의 custom 객체는 초기화 메시지를 받습니다.

3. nib 파일의 객체 사이의 모든 연결 (액션, 콘센트 및 바인딩)을 다시 설정합니다. (왜냐하면 표준에서 custom으로 변경되었으니깐) 
여기에는 파일 소유자 그리고 placeholder 객체에 대한 연결을 포함합니다. 연결 설정 방법들은 platform마다 다릅니다. 

    * Outlet connections
      * OS X에서, nib-loading 코드는 해당 outlet 객체의 메서드를 먼저 사용하여 outlet과 재연결하려고 노력합니다. 
      각각의 outlet에 대해, Cocoa는 setOutletName: 형식의 메서드를 먼저 찾고, 이러한 메서드가 있으면 호출합니다. 
      만약 이러한 메서드를 찾을 수 없으면, Cocoa는 해당 outlet 이름과 일치하는 인스턴스 변수를 찾기 위해 객체를 검색하고 값을 직접 설정하려고 합니다. 
      만약에 인스턴스 변수를 찾을 수 없다면, outlet 과의 연결은 생기지 않습니다.
      <br><br>outlet을 설정하는 것은 또한 등록된 모든 관찰자에 대한 키-값 관찰(KVO) notifications을 생성합니다. 
      이러한 notifications 는 모든 inter-object 연결이 재성립되기 전에 일어날 것이고 확실히 객체들의 awakeFromNib 메서드를 호출하기 전에 일어날 것입니다. 

      * iOS에서, nib-loading 코드는 각각의 outlet을 재연결하기 위하여 setValue:forKey: 메소드를 사용합니다. 이 방법은 마찬가지로 적절한 accessor 메소드를 찾고 만약 찾는 것이 실패하면 다른 방법으로 대체합니다. 어떻게 이 메소드가 값을 set하는지에 대한 더 많은 정보는, NSKeyValueCoding Protocol Reference 의 설명을 보세요.
      <br><br>iOS에서 outlet을 설정하면 등록된 모든 관찰자에게 KVO 알림이 생성됩니다. 이러한 알림은 모든 객체 간 연결이 재설정되기 전에 발생할 수 있으며 객체의 awakeFromNib 메소드가 호출되기 전에 확실히 발생합니다.

    * Action connecitons



    * Bindings

  4. 일치하는 selector를 정의하는 nib 파일안의 해당 객체로 awakeFromNib 메시지를 보냅니다.
  * OS X에서 이 메시지는 메소드를 정의하는 인터페이스 객체로 전송됩니다. 또한 파일의 소유자 및 파일을 정의하는 모든 placeholder 객체로 전송됩니다.
  * **iOS에서 , 이 메시지는 nib-loading 코드에 의해 초기화된 인터페이스 객체에게만 전달됩니다.** 

  5. nib 파일에서 "Visible at launch time" 속성이 활성화 된 모든 windows들을 표시합니다.