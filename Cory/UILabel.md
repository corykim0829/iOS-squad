## UILabel

하나 이상의 read-only 텍스트를 보여주는 뷰이다. 종종 의도된 목적을 설명하기 위해 컨트롤과 함께 사용된다.

UIControl

### Overview

- 컨텐츠를 표현할 string 또는 attributed string이 필요하다.
- 만약 nonattributed string이라면, UILabel의 모양(appearance)을 구성해야한다.
- 인터페이스에서 label의 사이즈와 좌표를 제어하기 위해 Auto Layout 규칙을 설정해야한다.
- 접근성 정보와 현지화 된 문자열을 제공한다.

<br>

### Topics

#### Accessing the Text Attributes

- `var text: String? { get set }`
  - label에서 보여지는 현재 텍스트
- `var attributedText: NSAttributedString? { get set }`
  - label에 보여지는 현재 스타일 텍스트(styled text)
- `var font: UIFont! { get set }`
- `var textColor: UIColor! { get set }`
- `var textAlignment: NSTextAlignment { get set }`
- `var lineBreakMode: NSLineBreakMode { get set }`
  - label 텍스트를 줄바꿈하고 잘라내는 데 사용되는 기술, 스타일 텍스트를 사용하지 않고있다면, 이 프로퍼티는 `text` 프로퍼티에 있는 전체 텍스트 string에 적용된다.
- `var isEnabled: Bool { get set }`
  - 이 프로퍼티는 label을 그리는 방법을 결정한다. Disabled된 텍스트는 활성화 여부를 나타내기 위해 흐리게 표시된다. 이 프로퍼티는 default로 true이다.

#### Sizing the Label’s Text

- `var adjustsFontSizeToFitWidth: Bool { get set }`

  - dufault 값은 false이며 label의 바운딩에 맞추어 폰트 사이즈가 줄어들어야하는지를 담고 있는 Boolean 값이다. true로 바꾸면 `minimumScaleFactor` 프로퍼티를 수정하여 적절한 최소 폰트 배율을 설정해야한다.

- `var baselineAdjustment: UIBaselineAdjustment { get set }`

  - 텍스트가 label에 맞게 줄어들어야할 때, 텍스트 **baseline**이 어떻게 조정되는지 제어한다. baseline이 뭔지 모르겠으면 아래 그림을 참고

  - ` adjustsFontSizeToFitWidth` 프로퍼티가 true이면 이 프로퍼티는 폰트사이즈 조정이 필요할 때, text baseline의 행동을 제어한다. default 값은 `UIBaselineAdjustment.alignBaseline` 이다. 이 프로퍼티는 `numberOfLines` 프로퍼티가 1일 때 가장 효과적이다.

    **UIBaselineAdjustment**

    - `.alignBaselines` : default값으로, 텍스트를 label의 baseline에 맞추어 조정한다.
    - `.alignCenters` : Label의 bounding box의 중심에 맞추어 텍스트를 조정한다.
    - `.none` : Label bounding box의 top-left 코너에 맞추어 텍스트를 조정한다. 

  <img src="https://irekasoft.com/_main/wp-content/uploads/Typography-Basics.jpg" width="480px">

- `var minimumScaleFactor: CGFloat { get set }`

  - `adjustsFontSizeToFitWidth`가 true일 때, 폰트 크기 value가 아닌 현재 폰트에서 작아져야할 때 최소 배율을 작성한다. default는 0이며 0으로하면 현재 폰트가 그대로 반영된다.

- `var numberOfLines: Int { get set }`

  - 텍스트를 렌더링하는 최대 줄 개수이다. default 값은 1이며, 0으로 설정하면 필요한 만큼 줄을 사용할 수 있다.

- `var allowsDefaultTighteningForTruncation: Bool { get set }`

  - 텍스트 자르기가 발생하기 전 텍스트 간격을 좁힌다. label은 폰트, 현재 줄 넓이, 줄 바꿈 모드와 다른 관련된 정보를 기반으로 좁힐 수 있는 최대 값을 결정한다.

#### Managing Highlight Values

- `var highlightedTextColor: UIColor? { get set }`
  - 강조 색깔이 적용된 label의 텍스트
  - Text 버튼 타입을 구현하기 위해서 label을 사용하는 서브클래스들은 버튼의 눌린 상태를 그릴 때, 이 프로퍼티를 사용할 수 있다.
  - 지정된 UIColor는 `isHighlighted` 프로퍼티가 true일 때, 자동적으로 적용된다.
- `var isHighlighted: Bool { get set }`
  - 강조표시로 Label을 그려야하는지 여부를 나타내는 Boolean 값
  - default값은 false이다.

#### Drawing a Shadow

- `var shadowColor: UIColor? { get set }`
  - 그림자의 색깔로, default 값은 nil이며 그려진 그림자가 없다는 것을 의미한다.
- `var shadowOffset: CGSize { get set }`
  - 이 프로퍼티에서 다른 효과를 가지기 위해서는 그림자 색깔은 nil이 아니여야한다.
  - default offset 사이즈는 `(0, 1)` 이며, 이는 그림자가 텍스트 1포인트 위에 있다는 의미이다.
  - 텍스트 그림자는 명시된 offset과 color 함께 그리고 blurring 없이 그려진다.

#### Drawing and Positioning Overrides

- `func textRect(forBounds bounds: CGRect, limitedToNumberOfLines numberOfLines: Int) -> CGRect`
  - Returns the drawing rectangle for the label’s text.

- `func drawText(in rect: CGRect)`
  - Draws the label's text (or its shadow) in the specified rectangle.

#### Getting the Layout Constraints

- `var preferredMaxLayoutWidth: CGFloat { get set }`
  - The preferred maximum width (in points) for a multiline label.

#### Setting and Getting Attributes

- `var isUserInteractionEnabled: Bool { get set }`
  - A boolean value that determines whether user events are ignored and removed from the event queue.