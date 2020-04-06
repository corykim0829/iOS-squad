출처: <https://developer.apple.com/documentation/uikit/views_and_controls/table_views/configuring_the_cells_for_your_table>
# Configuring the Cells for Your Table

스토리 보드에서 하나 이상의 프로토 타입 셀을 정의하여 테이블 행의 모양과 내용을 지정하십시오.
<br>(하나 이상의 커스템 셀 만들어서 적용해라)

## 1. Overview

셀은 테이블 행의 시각적 표현을 제공합니다. 
대부분의 테이블에는 **하나 또는 두 개의 다른 유형의 셀**만 제공합니다. 
**가장 중요한 정보가 눈에 띄도록 셀을 설계**하십시오. 
셀에서 **뷰 및 뷰 구성(configuration)** 을 신중하게 선택하여 수행하십시오.

스토리 보드 파일에서 디자인 타임에 셀 모양을 지정합니다. 
Xcode는 각 테이블마다 하나의 프로토 타입 셀을 제공하며 필요에 따라 **더 많은 프로토 타입 셀을 추가**할 수 있습니다. 
프로토 타입 셀은 **셀 모양을 위한 템플릿 역할**을합니다. 
프로토 타입 셸은 당신이 표현하고 싶은 뷰들을 포함하고 그리고 셀의 콘텐츠 영역 안에서 뷰들의 정렬을 포함합니다. (셀이 뷰들을 하위 뷰로 갖고 뷰들이 셀안에 위치해 있다.)  
런타임시 **테이블의 데이터 소스 객체(table's data source object)** 는 프로토 타입에서 실제 셀을 만들어 앱의 데이터로 구성합니다.

<img width="200" alt="cell1" src="https://user-images.githubusercontent.com/38216027/78468144-b326af00-774f-11ea-93ed-dfbaafe62b75.png">

> 셀 모양을 디자인하는 방법에 대한 팁은 HIG의 테이블을 참조하십시오.

## 2. Assign a Reuse Identifier to Each Cell

재사용 식별자는 테이블 셀의 생성 및 재활용을 용이하게 합니다. 
재사용 식별자는 테이블의 **각 프로토 타입 셀에 할당하는 문자열(String)** 입니다. 
스토리 보드에서 프로토 타입 셀을 선택하고 식별자 속성에 비어 있지 않은 값을 지정하십시오.(식별자 속성에 값을 지정해라) 
<br>**테이블 뷰의 각 셀에는 고유한 재사용 식별자가 있어야 합니다.(must!)**

런타임에 셀 객체가 필요하면 **테이블 뷰의 dequeueReusableCell (withIdentifier : for :) 메소드를 호출하여 원하는 셀의 재사용 식별자를 전달**하십시오. 
테이블뷰는 이미 작성된 셀의 내부 큐를 유지보수합니다. 
큐에 요청 된 유형의 셀이 포함된 경우 테이블뷰는 해당 셀을 리턴합니다. 
그렇지 않은 경우 스토리 보드의 프로토 타입 셀을 사용하여 새 셀을 만듭니다. 
**셀을 재사용하면 스크롤과 같은 중요한 시간동안 메모리 할당이 최소화되어 성능이 향상됩니다.**

```swift
var cell = tableView.dequeueReusableCell(withIdentifier: “myCellType”, for: indexPath)
```

## 3. Configure a Cell with a Built-In Style

셀을 구성하는 가장 간단한 방법은 **UITableViewCell에서 제공하는 내장 스타일 중 하나를 사용하는 것**입니다. 이 스타일을 그대로 사용하십시오. 셀을 관리하기 위해 커스텀 서브 클래스를 제공 할 필요는 없습니다. 각 스타일에는 하나 또는 두 개의 레이블이 통합되어 있으며 스타일에 따라 셀의 컨텐츠 영역 내에서 레이블의 위치가 결정됩니다. 대부분의 스타일은 셀 내용의 앞 부분에 이미지를 통합(incorporate)합니다.

표준 스타일 중 하나를 사용하여 프로토 타입 셀을 구성하려면 스토리 보드에서 셀을 선택하고 셀의 스타일 특성을 커스텀(custom) 이외의 값으로 설정하십시오.

<img width="200" alt="cell2" src="https://user-images.githubusercontent.com/38216027/78468170-fd0f9500-774f-11ea-97dd-5b66c824934a.png">


**tableView (_ : cellForRowAt :) 메소드**에서 UITableViewCell의 textLabel, detailTextLabel 및 imageView 특성을 사용하여 셀의 컨텐츠를 구성하십시오.
이러한 속성에는 뷰가 포함되어 있지만 **스타일이 일치하는 콘텐츠를 지원하는 경우**에만 셀 객체가 뷰를 할당합니다. 
예를 들어, 기본 셀 스타일은 세부 사항 문자열을 지원하지 않으므로, **해당 스타일의 detailTextLabel 특성은 nil**입니다. 
다음 예제 코드는 기본 셀 스타일을 사용하는 셀을 구성하는 방법을 보여줍니다.

```swift
override func tableView(_ tableView: UITableView, 
             cellForRowAt indexPath: IndexPath) -> UITableViewCell {
   // Reuse or create a cell. 
   let cell = tableView.dequeueReusableCell(withIdentifier: "basicStyle", for: indexPath)

   // For a standard cell, use the UITableViewCell properties.
   cell.textLabel!.text = "Title text"
   cell.imageView!.image = UIImage(named: "bunny")
   return cell
}
```

## 4. Configure a Cell with Custom Views

표준 스타일 이외의 모양에는 **커스텀 셀 스타일**을 사용하십시오. 
커스텀 셀을 사용하면 셀에서 원하는뷰, 해당 구성 및 크기 및 위치를 지정할 수 있습니다. 
레이블 및 이미지와 같은 정적 뷰는 셀에 가장 적합한 컨텐츠를 만듭니다. 
**컨트롤과 같은 사용자 상호 작용(user interactions such as controls)이 필요한 뷰는 피하십시오.** 스크롤뷰, 테이블뷰, 컬렉션뷰 또는 기타 복잡한 컨테이너뷰를 셀에 포함하지 마십시오. **셀에 스택 뷰(stack view)를 포함 할 수 있지만 성능을 향상 시키려면 스택뷰에서 항목 수를 최소화하십시오.**

커스텀 셀을 구성하려면 테이블의 프로토 타입 셀로 뷰를 끌어 오십시오.(IBOutlet 속성을 끌어오라는 뜻인 것 같다.) 다음 그림은 뷰에 대한 **커스텀 레이아웃** 및 포맷이 있는 셀을 보여줍니다. constraints를 사용하여 **셀의 컨텐츠 영역 내에 뷰를 배치**합니다. constraints를 설정할 때 "contrain to margin"옵션(constant를 두라는 뜻인듯)을 사용하여 **셀의 컨텐츠 영역 사이의 간격(gap)을 유지**하십시오.

<img width="200" alt="cell3" src="https://user-images.githubusercontent.com/38216027/78468189-46f87b00-7750-11ea-880e-8922bc4e67e2.png">

커스텀 셀의 경우 셀의 뷰에 접근하려면 **UITableViewCell 서브 클래스를 정의**해야합니다. 서브 클래스에 아울렛을 추가하고 해당 아울렛을 프로토 타입 셀의 해당 뷰에 연결하십시오.

```swift
class FoodCell: UITableViewCell {
    @IBOutlet var name : UILabel?
    @IBOutlet var plantDescription : UILabel?
    @IBOutlet var picture : UIImageView?
}
```

데이터 소스의 tableView (_ : cellForRowAt :) 메소드에서 셀의 콘센트를 사용하여 모든 뷰에 값을 지정하십시오.

```swift
override func tableView(_ tableView: UITableView, 
             cellForRowAt indexPath: IndexPath) -> UITableViewCell {

   // Reuse or create a cell of the appropriate type.
   let cell = tableView.dequeueReusableCell(withIdentifier: "foodCellType", 
                         for: indexPath) as! FoodCell

   // Fetch the data for the row.
   let theFood = foods[indexPath.row]
        
   // Configure the cell’s contents with data from the fetched object.
   cell.name?.text = theFood.name
   cell.plantDescription?.text = theFood.description
   cell.picture?.image = theFood.picture
        
   return cell
}
```

## 5. Change the Height of Rows

**테이블뷰는 행을 나타내는 셀과 별도로 행의 높이를 추적합니다.** 
UITableView는 행의 기본 크기를 제공하지만 테이블 뷰의 rowHeight 속성에 커스텀 지정 값을 할당하여 기본 높이를 재정의(override) 할 수 있습니다. 
**모든 행의 높이가 동일한 경우 항상 이 속성을 사용하십시오. 
이렇게하면 델리게이트 객체에서 높이 값을 반환하는 것보다 더 효율적입니다.(엄청 좋은 내용이다 짱짱)**

**행 높이가 모두 같지 않거나 동적으로 변경 될 수 있으면 델리게이트 객체의 tableView (_ : heightForRowAt :) 메서드를 사용하여 높이를 제공하십시오.** 이 메소드를 구현할 때 테이블의 모든 행에 값을 제공해야합니다. 다음 예제 코드는 각 섹션의 첫 번째 행에 대한 커스텀 높이를 반환하고 다른 모든 행에 기본 높이를 사용하는 방법을 보여줍니다.

```swift
override func tableView(_ tableView: UITableView, 
           heightForRowAt indexPath: IndexPath) -> CGFloat {
   // Make the first row larger to accommodate a custom cell.
  if indexPath.row == 0 {
      return 80
   }

   // Use the default size for all other rows.
   return UITableView.automaticDimension
}
```
테이블 뷰는 보이는 행의 높이만 묻습니다. 사용자가 스크롤 할때 테이블뷰는 화면 밖으로 이동한 후 다시 화면으로 돌아올 때를 포함하여 각 행의 높이를 제공하도록 요청합니다.

## Restore Your Cell's Original Appearance Before Reuse (재사용 하기 전에 원래 셀의 모습 복원시키기)

셀이 화면의 외부로 이동하면, 테이블 뷰는 뷰 계층에서 **해당 셀을 제거하고 해당 셀을 내부적으로 관리되는 재활용 큐에 배치**합니다. 
테이블 뷰의 dequeueReusableCell (withIdentifier : for :) 메소드를 사용하여 새 셀을 요청하면 테이블 뷰는 **먼저 재활용 큐에서 셀을 반환**합니다.(셸을 갖다줍니다.) 
<br>**대기열이 비어 있으면 테이블뷰가 스토리 보드에서 새 셀을 인스턴스화합니다. (비어있으면 객체를 새로 만들어 줍니다.)**

## 6. Add an Accessory View to Your Cell

액세서리 뷰는 셀의 끝 가장자리(the trailing edge of cell)에 나타나는 optional 시스템 정의 뷰입니다.(optional이므로 설정을 안해줘도 된다.) 
<br>액세서리 뷰를 사용하여 표준 셀 동작을 사용자에게 전달합니다. 
**예를 들어, 행을 누르면 행에 대한 자세한 정보가 표시됨을 사용자에게 알리기 위해 세부 정보 버튼을 추가합니다.(이런게 모두 액세서리 뷰!)**

액세서리 뷰를 구성하기 위해서는:

* 스토리보드에서는, 셀의 Accessory attribute을 사용하세요, 당신이 원하는 액세서리 뷰를 고르는데.

* 당신의 코드에서는, 셀의 accessoryType 속성 값을 바꾸세요.

사용자는 **탭할 때 액세서리뷰에 특정 동작이있을 것으로 기대**합니다. 이러한 동작을 구현하는 방법에 대한 자세한 내용은 **UITableViewCell.AccessoryType**을 참조하십시오.

### 6-1. UITableViewCell.AccessoryType => enum 타입이다.

셀에서 사용하는 표준 액세서리 제어 유형입니다.

* Declaration

```swift
enum AccessoryType : Int
```

이 상수를 사용하여 accessoriesType 속성 값을 설정하십시오.

여러 액세서리뷰는 추가 상호 작용을 지원합니다. 
예를 들어, detail button는 사용자가 볼 수 있는 추가 정보가 있음을 알려줍니다. 
특정 액세서리 뷰와의 상호 작용에 응답하는 방법에 대한 자세한 내용은 해당 유형에 대한 설명을 참조하십시오.

* Accessory Views

  * case **none** 
    <br>No accessory view.

  * case **disclosureIndcator** 
    <br>새로운 컨텐츠를 표현하기위한 셰브론 모양의 컨트롤입니다.

  * case **detailDisclosureButton**
    <br>정보 버튼 및 disclosure (쉐브론) 컨트롤.
 
  * case **checkMark**
    <br>체크마크 이미지

  * case **detailButton**  
    <br>정보 버튼 


> dislosure 뜻 : 폭로, 공개

> chevron-shaped(셰브론) 
<br><img width="20" alt="셰브론" src="https://user-images.githubusercontent.com/38216027/78466514-db58e280-773c-11ea-8d57-80446804f94e.jpg">

> checkmark

<img width="400" alt="checkmark" src="https://user-images.githubusercontent.com/38216027/78468317-d94d4e80-7751-11ea-84ba-d2f3452fc0a0.png">


> detailDisclosureButton

<img width="400" alt="detailDisclosureButton" src="https://user-images.githubusercontent.com/38216027/78469116-1b2dc300-7759-11ea-86a8-193204ef4889.png">
