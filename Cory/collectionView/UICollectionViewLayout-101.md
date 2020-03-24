# UICollectionViewLayout 톺아보기

UICollectionViewLayout은 collectionViewController를 만들 때 필수로 필요한 친구인데, 이 친구가 왜 필요한지 살펴보자.

#### contents

- [UICollectionViewLayout](https://github.com/corykim0829/iOS-squad/blob/master/Cory/collectionView/UICollectionViewLayout-101.md#uicollectionviewlayout)
- [UICollectionViewFlowLayout](https://github.com/corykim0829/iOS-squad/blob/master/Cory/collectionView/UICollectionViewLayout-101.md#uicollectionviewflowLlayout)
- [UICollectionViewController construction](https://github.com/corykim0829/iOS-squad/blob/master/Cory/collectionView/UICollectionViewLayout-101.md#construction)
- [Conclusion](https://github.com/corykim0829/iOS-squad/blob/master/Cory/collectionView/UICollectionViewLayout-101.md#coclusion)

<br>

## UICollectionViewLayout

collectionView의 layout 정보를 생성하는 추상적인 **'기반 클래스' (base class)**

공식문서에는 위와같이 설명이 나와있다.

<br>

#### Declaration

```swift
class UICollectionViewLayout : NSObject
```

#### Overview

layout 객체가 하는 일은 cell, supplementary view, 그리고 collection view의 바운드 내부에 있는 데코레이션 뷰(decoration view)의 위치를 결정하는 것이며 요청을 하면 위 정보들을 **collection view**에게 전달한다.

**사용하기 위해서는 `UICollectionViewLayout`을 서브클래싱 해야한다.** 그러나 서브클래싱을 하기 전, `UIColelctionViewFlowLayout` 클래스를 보고 레이아웃 요구에 맞게 조정할 수 있는지 확인해야한다.

<br>

#### Subclassing Notes

layout 객체의 주요 일은 collection view의 아이템들의 위치와 시각적 상태에 대한 정보를 제공하는 것이다. Layout 객체는 layout을 제공하는 뷰를 생성하지는 않는다. 뷰들은 collection view의 data source에서 만들어진다. 대신에 layout 객체는 보여지는 요소들을 **layout의 설계를 기반**으로 위치와 사이즈를 정의한다.

Collection view에는 layout이 필요한 3개의 보여지는 요소들의 타입을 가지고 있다.

- **Cell**은 layout에 의해 위치되는 주요 요소이다. 각 cell은 collection에 하나의 데이터를 표시한다. Collection view는 하나의 cell 그룹을 가지거나 cell을 여러개의 섹션으로 나눌 수 있다. layout 객체의 주업무는 cell을 collection view의 정보 영역(content area)에 배열하는 것이다.
- **Supplementary view**는 데이터를 의미하지만 cell과는 다르다. Supplementary view는 사용자에 의해 선택될 수 없다. 대신 supplementary view를 특정 섹션 또는 전체 collection view의 header와 footer 구현하기 위해 사용할 수 있다. Supplementary view는 선택적이며 사용하고 배치하는 일은 layout 객체가 처리한다.
- **Decoration view**는 선택될 수 없는 장식품이며 본질적으로 collection view의 데이터와 연관이 없다. Decoration view는 supplementary view의 다른 타입이다. supplementary view처럼, 선택적이며 사용하고 배치하는 일은 layout 객체가 처리한다.

Collection view는 시시각각 layout 객체에 layout 정보를 요청하여 위 3가지 정보를 제공하게 한다. 화면에 보여지는 모든 cell과 view는 layout 객체로부터 오는 정보를 사용해 위치하는 것이다. 비슷하게, Collection view에서 항목들이 추가되거나 삭제되면, 이에 따라 추가적인 layout이 발생하게 된다. 그러나 collection view는 layout을 화면에 보여지는 객체로 layout을 항상 제한한다.

<br>

#### Methods to Override

모든 layout 객체는 아래 메소드들을 구현해야한다.

- [`collectionViewContentSize`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617796-collectionviewcontentsize)
- [`layoutAttributesForElements(in:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617769-layoutattributesforelements)
- [`layoutAttributesForItem(at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617797-layoutattributesforitem)
- [`layoutAttributesForSupplementaryView(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617792-layoutattributesforsupplementary) (Layout이 supplementaryView를 지원한다면)
- [`layoutAttributesForDecorationView(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617809-layoutattributesfordecorationvie) (Layout이 decoration view를 지원한다면)
- [`shouldInvalidateLayout(forBoundsChange:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617781-shouldinvalidatelayout)

이 메소드들은 필수 collection view가 스크린에 정보들을 배치하는데 필요한 layout 정보를 제공한다. 당연히 layout이 supplementary나 decoration view를 지원하지 않는다면 해당 메소드는 구현할 필요는 없다.

collection view에 있는 데이터가 바뀌거나 항목이 추가 또는 삭제될 될 때, collection view는 layout 객체에 layout정보를 업데이트하라고 요청한다. 특히 이동, 추가 또는 삭제된 항목은 새 위치가 반영되어 업데이트된 layout 정보를 가지게된다. 이동된 아이템은 collection view가 아이템의 업데이트된 layout 속성(attributes)을 가져오기 위해 standard 메소드를 사용한다. 추가되거나 삭제된 항목에 대해서는 collection view가 다른 메소드를 사용하는데, 적절한 layout 정보를 제공하기 위해서는 우리가 오버라이드해서 사용해야한다.

- [`initialLayoutAttributesForAppearingItem(at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617789-initiallayoutattributesforappear)
- [`initialLayoutAttributesForAppearingSupplementaryElement(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617737-initiallayoutattributesforappear)
- [`initialLayoutAttributesForAppearingDecorationElement(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617726-initiallayoutattributesforappear)
- [`finalLayoutAttributesForDisappearingItem(at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617740-finallayoutattributesfordisappea)
- [`finalLayoutAttributesForDisappearingSupplementaryElement(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617742-finallayoutattributesfordisappea)
- [`finalLayoutAttributesForDisappearingDecorationElement(ofKind:at:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617762-finallayoutattributesfordisappea)

이 메소드들 외에도, 우리는 [`prepare(forCollectionViewUpdates:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617784-prepare) 메소드를 오버라이드하여 layout에 관련된 준비를 처리할 수 있다. 또한  [`finalizeCollectionViewUpdates()`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617787-finalizecollectionviewupdates) 를 오버라이드하여 전체 애니메이션 블록에 애니메이션을 추가하거나 최종 layout 관련 작업들을 구현할 수 있다.

#### Optimizing Layout Performance Using Invalidation Contexts

When designing your custom layouts, you can improve performance by invalidating only those parts of your layout that actually changed. When you change items, calling the `invalidateLayout()` method forces the collection view to recompute all of its layout information and reapply it. A better solution is to recompute only the layout information that changed, which is exactly what invalidation contexts allow you to do. An invalidation context lets you specify which parts of the layout changed. The layout object can then use that information to minimize the amount of data it recomputes.

To define a custom invalidation context for your layout, subclass the [`UICollectionViewLayoutInvalidationContext`](https://developer.apple.com/documentation/uikit/uicollectionviewlayoutinvalidationcontext) class. In your subclass, define custom properties that represent the parts of your layout data that can be recomputed independently. When you need to invalidate your layout at runtime, create an instance of your invalidation context subclass, configure the custom properties based on what layout information changed, and pass that object to your layout’s [`invalidateLayout(with:)`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617771-invalidatelayout) method. Your custom implementation of that method can use the information in the invalidation context to recompute only the portions of your layout that changed.

If you define a custom invalidation context class for your layout object, you should also override the [`invalidationContextClass`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617790-invalidationcontextclass) method and return your custom class. The collection view always creates an instance of the class you specify when it needs an invalidation context. Returning your custom subclass from this method ensures that your layout object always has the invalidation context it expects.

<br>

## UICollectionViewFlowLayout

A concrete layout object that organizes items into a grid with optional header and footer views for each section.

#### Declaration

```swift
class UICollectionViewFlowLayout : UICollectionViewLayout
```

#### Overview

The items in the collection view flow from one row or column (depending on the scrolling direction) to the next, with each row comprising as many cells as will fit. Cells can be the same sizes or different sizes.

collection view의 아이템들은 하나의 행 또는 열(스크롤 방향에 따라)에서 다음 아이템으로 흐르는데, 각 행(또는 열)에는 가능한 많은 cell이 들어간다. Cell들은 사이즈가 서로 같거나 다를 수도 있다.

**Flow layout**은 collection view의 **delegate 객체**를 통해 각 섹션과 그리드의 아이템, header 그리고 footer의 사이즈를 결정한다. 이 delegate 객체는 반드시 `UICollectionViewDelegateFlowLayout` 프로토콜을 채택해야한다. delegate의 사용은 layout 정보를 다이나믹하게 조절할 수 있게 해준다. 예를 들면, 그리드에 있는 아이템의 사이즈를 다르게 하고싶다면 delegate 객체를 사용해야한다. Delegate를 제공하지 않으면, flow layout은 이 클래스의 프로퍼티를 사용해 우리가 세팅해놓은 default 값을 사용한다.

Flow layout은 내용을 보여주기 위해 하나의 방향에서는 **고정된 거리(fixed distance)**를 사용하고, 다른 곳에서는 **스크롤 가능한 거리(scrollable distance)**를 사용한다. 예를 들어, 수직으로 스크롤하는 그리드에서는 그리드 컨텐츠의 가로길이가 collection view의 가로길이 의해서 제한되는데 반면에 컨텐츠의 높이는 그리드의 섹션과 아이템에 맞게 다이나믹하게 조절된다. Layout은 기본적으로 수직으로 스크롤되게 설정되지만 `scrollDirection` 프로퍼티를 사용하여 스크롤 방향을 설정할 수 있다.

Flow layout의 각 섹션은 자신의 커스텀 header와 footer를 가질 수 있다. View에 header와 footer를 설정하기 위해서는 header와 footer 사이즈가 0이 되지 않도록 설정해줘야한다. 적절한 **delegate 메소드**를 사용하거나 적당한 값을 [`headerReferenceSize`](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1617710-headerreferencesize)와 [`footerReferenceSize`](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1617703-footerreferencesize) 프로퍼티에 할당함으로써 사이즈를 설정해줄 수 있다. 만약 header와 footer의 사이즈가 0이라면, collection view에 추가되지 않을 것이다.

<br>

## Construction

UICollectionViewController와 UICollectionView 그리고 메인인 UICollectionViewLayout과 UICollectionViewFlowLayout의 실제 구현 구조를 살펴보자.

```swift
open class UICollectionViewController : UIViewController, UICollectionViewDelegate, UICollectionViewDataSource {

    public init(collectionViewLayout layout: UICollectionViewLayout)
    public init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?)
    public init?(coder: NSCoder)
    
    open var collectionView: UICollectionView!
    open var clearsSelectionOnViewWillAppear: Bool
    
    @available(iOS 7.0, *)
    open var collectionViewLayout: UICollectionViewLayout { get }
		// ... 생략
```

모든 `UICollectionViewController`는 `UICollectionView` 객체를 가지고 있다. 그리고 `UICollectionViewController`를 코드로 초기화할 때에는 `UICollectionViewLayout`을 인자로 넘겨줘야 한다.



```swift
open class UICollectionView : UIScrollView, UIDataSourceTranslating {

    public init(frame: CGRect, collectionViewLayout layout: UICollectionViewLayout)
    public init?(coder: NSCoder)
    
    open var collectionViewLayout: UICollectionViewLayout
    weak open var delegate: UICollectionViewDelegate?
    weak open var dataSource: UICollectionViewDataSource?
    // ... 생략
```

UICollectionView는 collectionViewLayout을 가지고 있다.

<br>

## Conclusion

UICollectionViewLayout는 사실 심플한 녀석이다. collectionview의 자식 뷰인 cell들의 위치만 계산하는 일반적인 방식만 겨우 제공할 뿐이다.

#### Difference Between **UICollectionViewLayout & UICollectionViewFlowLayout**

`UICollectionViewFlowLayout`는 `UICollectionViewLayout`의 서브클래스이다. 공식문서에 언급되어 있듯이 `UICollectionViewLayout`은 단순히 뼈대 역할만 하는 슈퍼클래스이며 배열을 위한 3가지 함수와 1개의 프로퍼티를 가지고있다. 

1. `Prepare()` — function
2. `CollectionViewContentSize` — Property
3. `layoutAttributesForElements(in rect)`
4. `layoutAttributesForItem(at indexPath)`

in **UICollectionViewLayout**, all these functions and property do nothing. For example, layoutAttributesForElements(in rect) return nils in UICollectionViewLayout. **UICollectionViewLayout** is waiting for someone to subclass it and provide the appropriate content.

`UICollectionViewLayout`에서는 이 함수들과 프로퍼티는 아무것도 하지 않는다. 예를들어 `layoutAttributesForElements(in rect)`는 nil값을 반환한다. `UICollectionViewLayout`은 **서브클래싱**을 하여 적절한 컨텐츠를 제공해주길 기다리고 있다.

`UICollectionViewFlowLayout`은 `UICollectionViewLayout`의 특정한 서브클래스이며 위 4가지가 모두 구현되어있기 때문에 그리드에 맞게 cell들이 배열될 것이다.

<br>

#### References

- [UICollectionViewLayout](https://developer.apple.com/documentation/uikit/uicollectionviewlayout)
- [UICollectionViewFlowLayout](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout)
- [Step by Step Guide to UICollectionViewLayout](https://medium.com/@linhairui19/step-by-step-guide-to-uicollectionviewlayout-d7e23b237539)

