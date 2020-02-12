## UILabel

하나 이상의 read-only 텍스트를 보여주는 뷰이다. 종종 의도된 목적을 설명하기 위해 컨트롤과 함께 사용된다.

UIControl

### Overview

- 컨텐츠를 표현할 string 또는 attributed string이 필요하다.
- 만약 nonattributed string이라면, UILabel의 모양(appearance)을 구성해야한다.
- 인터페이스에서 label의 사이즈와 좌표를 제어하기 위해 Auto Layout 규칙을 설정해야한다.
- 접근성 정보와 현지화 된 문자열을 제공한다.



### Accessing the Text Attributes

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

### Sizing the Label’s Text

- `var adjustsFontSizeToFitWidth: Bool { get set }`
  - dufault 값은 false이며 label의 바운딩에 맞추어 폰트 사이즈가 줄어들어야하는지를 담고 있는 Boolean 값이다. true로 바꾸면 `minimumScaleFactor` 프로퍼티를 수정하여 적절한 최소 폰트 배율을 설정해야한다.
- `var baselineAdjustment: UIBaselineAdjustment { get set }`
  - 텍스트가 label에 맞게 줄어들어야할 때, 텍스트 baseline이 어떻게 조정되는지 제어한다.
  - ` adjustsFontSizeToFitWidth` 프로퍼티가 true이면 이 프로퍼티는 
- `var minimumScaleFactor: CGFloat { get set }`
  - `adjustsFontSizeToFitWidth`가 true일 때, 폰트 크기 value가 아닌 현재 폰트에서 작아져야할 때 최소 배율을 작성한다. default는 0이며 0으로하면 현재 폰트가 그대로 반영된다.
- `var numberOfLines: Int { get set }`
  - 텍스트를 렌더링하는 최대 줄 개수이다. default 값은 1이며, 0으로 설정하면 필요한 만큼 줄을 사용할 수 있다.
- `var allowsDefaultTighteningForTruncation: Bool { get set }`
  - 텍스트 자르기가 발생하기 전 텍스트 간격을 좁힌다. label은 폰트, 현재 줄 넓이, 줄 바꿈 모드와 다른 관련된 정보를 기반으로 좁힐 수 있는 최대 값을 결정한다.