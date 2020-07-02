출처: <https://developer.apple.com/design/human-interface-guidelines/ios/views/tables/>

# Tables - HIG

테이블은 데이터를 섹션 또는 그룹으로 나눌 수 있는 스크롤 식 단일 열 행 목록(single-column list of rows)으로 표시합니다. 표를 사용하여 대량 또는 소량의 정보를 목록 형식으로 깨끗하고 효율적으로 표시하십시오. 일반적으로 테이블은 텍스트 기반 컨텐츠에 이상적이며 종종 관련 컨텐츠가 반대쪽에 표시된 분할뷰의 한쪽에서 탐색 수단으로 나타납니다. 지침은 분할 뷰를 참조하십시오.

## 테이블 뷰의 세 가지 스타일 

* iOS는 일반(plain), 그룹화(grouped) 및 삽입 그룹화(inset grouped)의 **세 가지 스타일**의 테이블을 제공합니다.

**Plain.** 행은 레이블이 지정된 섹션으로 분리 될 수 있으며 선택적 인덱스는 테이블의 오른쪽 가장자리를 따라 세로로 나타날 수 있습니다. 섹션의 첫 번째 항목 앞에 머리글이 나타나고 마지막 항목 뒤에 바닥 글이 나타날 수 있습니다.

<img width="327" alt="Plain" src="https://user-images.githubusercontent.com/38216027/78471033-fe00f080-7768-11ea-9715-f9913b30f93c.png">

**Grouped.** 행은 그룹으로 표시되며 앞에 머리글(header)과 바닥글(footer)이 올 수 있습니다. 이 스타일의 테이블에는 **항상 하나 이상의 그룹이 포함**되고 **각 그룹에는 항상 하나 이상의 행**이 포함됩니다. 그룹화 된 테이블에는 index를 포함하지 않습니다.

<img width="329" alt="Grouped" src="https://user-images.githubusercontent.com/38216027/78471035-00fbe100-7769-11ea-9e00-97e9844aa65e.png">

**Inset grouped.** 행은 **둥근 모서리를 가진 그룹**으로 표시되며 상위 뷰의 가장자리(edge)에서 inset이 적용됩니다(아래 이미지의 오른쪽에 표시됨). 
이 스타일의 테이블에는 항상 하나 이상의 그룹이 포함되고 각 그룹에는 항상 하나 이상의 행이 포함되며 머리글과 바닥 글이 올 수 있습니다(header와 footer는 옵셔널). 
<br>삽입된 그룹화 된 테이블에는 index가 없습니다. 삽입된 그룹화 된 스타일은 일반적인 width 환경에서 가장 잘 작동합니다. 컴팩트(compact)한 환경에는 공간이 적기 때문에 특히 콘텐츠가 현지화(localized) 된 경우, 삽입 된 그룹화 된 테이블이 텍스트 줄바꿈(=>깨진다)을 일으킬 수 있습니다.

<img width="713" alt="Inset Grouped" src="https://user-images.githubusercontent.com/38216027/78471037-01947780-7769-11ea-9e7e-9ae550c5c5c4.png">


For developer guidance, see UITableView.

## Table Row (UITableViewCellStyle)

표준 표 셀 스타일을 사용하여 표 행에 내용이 표시되는 방식을 정의합니다.

> **Basic (Default).**

<img width="326" alt="Basics" src="https://user-images.githubusercontent.com/38216027/78470965-72875f80-7768-11ea-886b-cad3494dcc37.png">

An optional image on the left side of the row, followed by a left-aligned title. It’s a good option for displaying items that don’t require supplementary information. For developer guidance, see the UITableViewCellStyleDefault constant of UITableViewCell.

**행의 왼쪽에 있는 optional 이미지와 왼쪽에 정렬된 제목. 추가 정보가 필요없는 항목을 표시하는 데 좋은 옵션** 입니다. 
개발자 안내는 UITableViewCell의 UITableViewCellStyleDefault 상수를 참조하십시오.

> **Subtitle.**

<img width="327" alt="Subtitle" src="https://user-images.githubusercontent.com/38216027/78470967-73b88c80-7768-11ea-8825-097d71cddc66.png">

한 줄에 왼쪽 정렬된 제목과 다음 줄에 왼쪽 정렬된 **부제목**. 이 스타일은 행들이 시각적으로 유사한 테이블에서 잘 작동합니다. 추가 부제목은 행을 서로 구분하는데 도움이 됩니다. 
개발자 안내는 UITableViewCell의 UITableViewCellStyleSubtitle 상수를 참조하십시오.

> **Right Detail (Value 1).**

<img width="327" alt="Right Detail" src="https://user-images.githubusercontent.com/38216027/78470968-74e9b980-7768-11ea-9a46-5f5bfe3580e2.png">

같은 줄에 오른쪽으로 정렬된 부제목이 있는 왼쪽으로 정렬 된 제목. 
개발자 안내는 UITableViewCell의 UITableViewCellStyleValue1 상수를 참조하십시오.

> **Left Detail (Value 2).**

<img width="329" alt="Left Detail" src="https://user-images.githubusercontent.com/38216027/78470969-761ae680-7768-11ea-91a1-55b4bf1cbd80.png">

오른쪽 정렬된 제목 다음에 같은 줄에 왼쪽 정렬된 부제목이 옵니다. 개발자 안내는 UITableViewCell의 UITableViewCellStyleValue2 상수를 참조하십시오.