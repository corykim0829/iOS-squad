# Modality



Modality는 사용자의 이전 컨텍스트와는 별개로 컨텐츠를 임시적인 모드로 표시하는 디자인 테크닉이다. 종료하려면 명시적인 조치가 필요하다. 

- 사람들이 독립적인 작업 또는 밀접하게 관련된 옵션에 집중하도록 돕는다.
- 사람들이 중요한 정보를 수신하고 필요한 경우 조치를 취하도록 보장한다. 

iOS는 앱의 특정 상황에서 사용하는 Alerts, Activity Views, Action Sheets를 제공한다. 앱에 사용자 정의 모달 컨텐츠를 표시하기 위해 iOS13 이상은 다음과 같은 프레젠테이션 스타일을 지원한다.


![](https://images.velog.io/images/delmasong/post/a9399c3e-4f0f-46b6-8f63-1b163ae11ab1/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-04-05%20%EC%98%A4%ED%9B%84%204.53.50.png)
### Sheet

sheet 프레젠테이션 스타일은 기본 내용을 부분적으로 커버하는 카드로 나타나고, 그것과의 상호작용을 방지하기 위해 모든 노출된 영역을 흐리게 한다. 현재 카드 뒤에는 카드를 열 때 중단한 작업을 기억하도록 하기 위해 상위 보기 또는 이전 카드의 위쪽 가장자리가 보인다. 아래와 같은 방법으로 카드를 지운다.

- 화면의 윗부분을 아래로 쓸어내린다
- 버튼을 누른다
- 카드 컨텐츠가 상단으로 스크롤되는 부분 어디든지 아래로 쓸어내린다

복잡한 작업을 실행하지 않는 일시적인 모달 컨텐츠에서는 sheet를 사용한다.



### Fullscreen

fullscreen 프레젠테이션 스타일은 전체 화면을 포괄한다. 이전의 view는 완전히 가려져 시각적 산만함이 최소화된다. 사용자는 버튼을 눌러 전체화면 모달보기를 dismiss한다. 

동영상, 사진 또는 카메라 뷰와 같은 몰입적인 컨텐츠에 대해 전체 화면 모달 뷰를 사용하거나, 문서 표시 또는 사진 편집과 같은 복잡한 작업 수행시 fullscreen 프레젠테이션을 사용하면 된다.



**modality 을 사용하는 것이 합리적일 때 사용** 현재의 업무와는 다른 선택을 하거나 업무를 수행하는데 사람들의 관심을 집중시키는 것이 중요한 경우에만 모달을 만든다. 모달 경험은 사용자를 현재의 맥락에서 벗어나게 하고, 멈출 것을 요구하기 때문에, 분명한 이익을 제공할때만 Modality를 사용하는 것이 효과적이다. 



**필수 정보를 제공하고 이상적으로 실행 가능한 정보를 제공하기 위해 경고를 예약합니다.** 일반적으로 문제가 발생했기 때문에 경고가 표시됩니다. 알림은 경험을 중단하고 탭을 눌러 제거해야 하기 때문에 사람들은 침입이 정당하다고 느끼는 것이 중요합니다. 자세한 내용은 [경고](https://developer.apple.com/design/human-interface-guidelines/ios/views/alerts/)를 참조하십시오.

**모달 작업을 단순하고 짧고 좁게 집중합니다.** 앱 내에서 앱을 만들지 마십시오. 모달 작업이 너무 복잡하면 모달 컨텍스트에 들어가면서 일시 중단한 작업을 보지 못할 수 있습니다. 사람들은 길을 잃을 수 있고 그들의 발자취를 되짚어 가는 법을 잊을 수 있기 때문에 관점의 계층을 포함하는 모달 작업을 만드는 것을 특히 조심하라. 모달 태스크에 하위 태스크가 포함되어야 하는 경우 계층을 통과하는 단일 경로와 완료할 수 있는 명확한 경로를 제공합니다. 작업을 완료하는 것 이외에는 완료 버튼을 사용하지 마십시오.

**모달 보기를 제거하는 버튼을 항상 포함합니다.** 예를 들어 완료 또는 취소를 사용할 수 있습니다. 버튼을 포함하면 모달 보기가 보조 기술에 액세스 할 수 있고 해고 제스처에 대한 대안을 제공합니다.

**필요한 경우 모달 보기를 닫기 전에 확인을 받아 데이터 손실을 피할 수 있습니다.** 사용자가 해제 제스처를 사용하든 아니면 버튼을 사용하여 뷰를 닫든 상관 없이 사용자 생성 컨텐츠가 손실될 수 있는 경우에는 상황을 설명하고 문제를 해결할 수 있는 방법을 제공하는 조치 시트를 제시하십시오.

**popover상단에 나타나는 카드를 표시하지 마십시오.** popover내에 카드를 표시할 수 있지만 popover상단에 아무 것도 표시되지 않아야 합니다(경고 가능 제외). 드물지만 사람이 popover에서 작업을 수행한 후 카드를 표시해야 하는 경우에는 카드를 표시하기 전에 popover를 닫습니다.

**일반적으로 모달 태스크를 식별하는 제목을 표시합니다.** 사람들이 모달 작업을 시작하면 이전의 맥락에서 벗어나 새로운 컨텍스트를 명확하게 만드는 것이 좋습니다. 보기의 다른 부분에 태스크를 더 자세히 설명하거나 지침을 제공하는 텍스트를 제공할 수도 있습니다.

**프로그램과 모달 뷰 모양을 조정합니다.** 예를 들어 모달 보기에 탐색 모음이 포함되어 있는 경우 앱의 탐색 모음과 모양이 동일해야 합니다.

**당신의 앱에 맞는 모달 전환 스타일을 선택합니다.** 앱과 조정되고 임시 컨텍스트 전환에 대한 인식을 높일 수 있는 전환 스타일을 사용합니다. 기본 전환은 화면 하단에서 모달 보기를 위쪽으로 이동한 다음 삭제되면 아래쪽으로 이동합니다. 앱 전체에서 일관된 모달 전환 스타일을 사용합니다.



개발자 지침은 [UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller)및 [UIPresentationController](https://developer.apple.com/documentation/uikit/uipresentationcontroller)를 참조하십시오.

**Reference**
https://developer.apple.com/design/human-interface-guidelines/ios/app-architecture/modality/