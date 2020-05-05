# Core Animation Programming Guide



## Core Animation Basics

CoreAnimation은 앱의 뷰 및 기타 시각적 요소를 애니메이션 하기 위한 일반적인 시스템을 제공함. 앱의 뷰를 대체하는 것이 아니라 컨텐츠의 애니메이션을 지원하고 더 나은 성능을 지원하기 위한 통합된 기술임.

그래픽 하드웨어에서 직접 조작할 수 있는 비트맵에 뷰의 컨텐츠를 캐싱해 이 동작들을 수행. 

<br>

## Layers Provide the Basics for Drawing and Animations

Layer Object는 3D공간에 구성된 2D로, CoreAnimation으로 수행하는 모든 작업의 중심에 있다. 뷰와 마찬가지로 레이어는 표면의 기하학적 요소, 컨텐츠 및 시각적 특성에 대한 정보를 관리함. 뷰와 달리 레이어는 자체 모양을 정의하지 않음. 단지 비트맵을 둘러싼 상태정보를 관리할 뿐. 

비트맵 자체는 사용자가 지정한(그린) 뷰 자체 또는 고정 이미지일 수 있음. 그래서 앱에서 사용되는 기본 레이어는 주로 데이터를 관리하기 때문에 모델 객체로 간주됨.



<br>

### The Layer-Based Drawing Model

대부분의 레이어는 앱에서 실제로 그리는 행동을 하지는 않는다. 대신 레이어는 앱에서 제공하는 컨텐츠를 캡처해 비트맵(backing store라고도 함)으로 캐시함. 이후에 레이어의 속성을 변경할 때는 레이어와 관련된 상태 정보만 변경하면 된다. 

변경으로 인해 애니메이션이 시작되면, 코어 애니메이션은 레이어의 비트맵 및 상태 정보를 그래픽 하드웨어에 전달함. 그래픽 하드웨어는 (전달받은)새로운 정보를 사용해 비트맵을 렌더링 함. 하드웨어에서 비트맵을 관리하면 소프트웨어에서 하는 것보다 훨씬 빠른 애니메이션을 만들 수 있음.

<img src = "https://user-images.githubusercontent.com/40784518/81037466-d3c45f00-8edd-11ea-9603-e39dfd2d017d.png"/>

정적 비트맵을 조작하기 때문에 레이어 기반 드로잉은 기존 뷰 기반 드로잉과 크게 다르다. 뷰 기반으로 화면을 그리는 경우 뷰 자체를 변경하면 종종 새롭게 그려진 컨텐츠를 새 파라미터로 `drawRect:` 메소드를 다시 호출한다. 하지만 이렇게 그리는 건 메인 스레드의 CPU를 사용하기 때문에 비싸다. Core Animation은 동일하거나 유사한 효과를 얻기 위해 가능한 한 하드웨어에서 캐시된 비트맵을 조작함으로 이런 비용을 방지한다.



CoreAnimation은 가능한 많이 캐시된 컨텐츠를 사용하지만 앱은 여전히 이니셜 컨텐츠를 제공하고 수시로 업데이트 해야 한다. [더 자세한 내용(Providing a Layer's Contents)](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW4)



<br>

### Layer-Based Animations

레이어의 데이터 및 상태 정보는 화면에서 해당 레이어 컨텐츠의 시각적 표시로부터 분리된다. (레이어 객체가 가지고 있는 정보랑 화면에 보여지는 거랑 별개라는 의미?) 이런 분리를 통해 CoreAnimation은 이전 상태 값에서 새 상태값으로의 변경을 애니메이션화 할 수 있다. 예를들어, 레이어의 위치 속성을 변경하면 CoreAnimation은 레이어를 현재 위치에서 새로 지정된 위치로 이동함. 애니메이션을 트리거하는 레이어 속성 목록은 [여기(Animatable Properties)](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW1) 를 참조



<img src = "https://user-images.githubusercontent.com/40784518/81037509-f0609700-8edd-11ea-8b76-b8c82d844d0b.png"/>



애니메이션 도중 코어 애니메이션은 모든 프레임별 drawing을 하드웨어에서 사용할 수 있다. 애니메이션의 시작점과 끝점만 지정하고 나머지는 CoreAnimation에서 처리하면 됨. 필요하면 사용자 지정 타이밍 정보나 애니메이션 매개변수를 지정할 수 있지만 아니라면 CoreAnimation이 적절한 기본 값을 제공함.

애니메이션을 시작하고 매개변수를 구성하는 방법은 [여기(Animating Layer Content)](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1) 를  참조



<br>



## Layer Objects Define Their Own Geometry

레이어의 작업 중 하나는 컨텐츠의 시각적 형상(visual geometry)를 관리하는 것이다. visual geometry는 컨텐츠의 경계, 화면에서의 위치, 레이어가 어떤 방식으로 변환(조정/회전 등)되었는지 여부에 대한 정보를 포함 함. 뷰와 마찬가지로 레이어에는 프레임 및 바운드가 있어 내용을 배치하는 데 사용할 수 있다. 레이어는 뷰에는 없는 anchor point 같은 다른 특성도 있다. 뷰에 대한 정보를 지정하는 방식과 레이어의 기하학적인 측면을 지정하는 방법이 다르다.



### Layer Use Two Types of Coordinate Systems

레이어는 point-based coordinate systems(점 기반 좌표계)와 unit coordinate systems(단위 기반 좌표계)를 모두 사용해 컨텐츠를 배치함. 사용되는 좌표계는 전달되는 정보의 유형에 따라 달라지는데,

- 점 기반 좌표는 화면 좌표에 직접 매핑되거나 레이어와 같이 다른 레이어에 의해 상대적으로 지정되어야 하는 값을 지정할 때 사용됨.
- 단위 좌표는 다른 값에 상대적이기 때문에 값을 화면 좌표에 묶어두지 않아야 할 때 사용됨. 예를 들어 레이어의 anchorPoint 특성은 변경될 수 있는 레이어 자체의 바운드에 상대적인 점을 지정함. 

점 기반 좌표에 대한 가장 일반적인 사용은 레이어의 bounds와 position 속성을 이용해 레이어의 크기와 위치를 지정하는 것. bounds는 레이어 자체의 좌표계를 정의하고 화면에서 레이어 크기를 포함함. position은 상위 좌표계에 상대적인 레이어의 위치를 정의함. 레이어에 bounds와 position에 의해 파생되는 frame 속성이 있지만, bounds와 position에 비해 덜 사용됨. 



bounds와 frame의 방향은 항상 기본 플랫폼의 기본 방향과 일치한다. iOS는 시작점이  왼쪽 상단, OS X는 왼쪽 하단에 있다.



<img src = "https://user-images.githubusercontent.com/40784518/81039301-9a8eed80-8ee3-11ea-85ea-8e52d8f8acf4.png"/>





1-3에서 기억해야 할 한가지는, position 특성은 레이어의 중간에 위치한다. position은 레이어의 anchorPoint 속성을 기반으로 정의가 변경되는 속성 중 하나임. 고정점(anchor point)은 특정 좌표가 시작되는 지점을 나타낸다. 자세한 설명은 [Anchor Points Affect Geometric Manipulations](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW17)





## Layer Trees Reflect Different Aspect of the Animation State

CoreAnimation을 사용하는 앱에는 3가지의 레이어 객체가 있다. 각각의 레이어는 앱의 컨텐츠를 화면에 표시하는 데 있어 서로 다른 역할을 한다.

- model layer tree : 애플리케이션과 가장 많이 상호작용 하는 객체. 이 트리의 객체는 애니메이션의 대상 값(target value)을 저장하는 모델 객체임. 레이어의 특성을 변경할 때마다 이 객체 중 하나를 사용함.
- presentation tree : 실행중인 애니메이션의 실행중인 값을 포함하고 있음. 레이어 트리 객체는 애니메이션의 대상 값을 포함하는 반면,  **프레젠테이션 트리의 객체는 화면에 표시되는 현재 값을 반영함. ** 이 트리의 객체는 수정하면 안됨. 대신 이 객체를 사용해 현재 애니메이션 값을 읽거나 해당 값에서 시작하는 새 애니메이션을 만들 수 있음.
- render tree: 실제 애니메이션을 수행하고 Core Animation 전용으로 사용됨

각각의 레이어는 앱의 뷰와 같은 계층 구조로 구성됨. 실제로 모든 뷰에 대해 레이어를 사용할 수 있도록 하는 애플리케이션의 경우 각 트리의 초기 구조가 뷰의 계층구조와 정확히 일치한다. 애플리케이션은 필요에 따라 추가적인 레이어 객체를 추가할 수 있다. (뷰와 관련되지 않은 레이어)  이렇게 하면 뷰의 오버헤드가 전혀 필요하지 않은 컨텐츠에 대해 앱의 성능을 최적화 할 수 있다. 그림 1-9는 단순한 iOS 앱에서 발견되는 레이어의 분류를 보여준다. 

<img src = "https://user-images.githubusercontent.com/40784518/81041830-c3b27c80-8ee9-11ea-9f8d-83574ae647a2.png"/>



레이어 트리의 모든 객체에 대해 그림 1-10 처럼 presentation tree와 render tree에 일치하는 객체가 있다. 앞서 말했듯이 애플리케이션은 주로 layer tree에서 작동하지만 때로는 presentation tree에 접근할 수도 있다. 구체적으로 layer tree의 presentationLayer 속성에 접근하면 presentation tree에서 일치하는 객체를 반환한다. 



<img src = "https://user-images.githubusercontent.com/40784518/81042498-3a03ae80-8eeb-11ea-8042-49145707fadc.png"/>



‼️ 애니메이션이 이동 중인 동안에만 presentation tree 객체에 액세스 해야 함. 애니메이션이 진행중인 동안 presentation tree 는 해당 순간에 화면에 표시되는 레이어 값을 포함함. 이 동작은 항상 코드에 의해 설정된 마지막 값을 반영하며 애니메이션의 최종 상태와 동일함.

