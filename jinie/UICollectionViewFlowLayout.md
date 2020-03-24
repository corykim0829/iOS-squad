# UICollectionViewFlowLayout

UICollectionViewFlowLayout 클래스를 사용하면 컬렉션 뷰의 셀을 원하는 형태로 정렬할 수 있다. 플로우 레이아웃은 레이아웃 객체가 셀을 그리드 혹은 라인 기반으로 배치하고 이 행을 따라 최대한 많은 셀을 채우는 것을 의미한다. 현재 행에서 공간이 부족하면 새로운 행을 생성하고 그 행에서 레이아웃 프로세스를 계속 진행한다.



**구성 단계**

1. 플로우 레이아웃 객체를 컬렉션 뷰의 레이아웃 객체로 지정한다.
2. 셀의 너비와 높이를 구성한다.
3. 필요한 경우 셀의 간격을 조절한다.
4. 필요한 경우 섹션 헤더 및 푸터의 크기를 지정한다.
5. 레이아웃의 스크롤 방향을 설정한다.

플로우 레이아웃은 대부분 프로퍼티의 기본값을 가지고 있다. 셀의 너비와 높이는 기본값이 0으로 지정되어 있기 때문에 셀의 크기를 지정하지 않을 경우 셀이 화면에 보이지 않을 수 있다.



**속성**

플로우 레이아웃 객체는 컨텐츠의 모양을 구성하기 위해 여러 프로퍼티를 제공한다. 프로퍼티에 적절한 값을 설정하면 모든 셀에 동일한 레이아웃이 적용된다. 예를 들어, 플로우 레이아웃 객체의 itemSize 프로퍼티를 사용하여 셀의 크기를 설정할 경우 모든 셀의 크기가 동일하게 적용된다.



**셀 크기**

컬렉션 뷰의 모든 셀이 같은 크기인 경우 플로우 레이아웃 객체의 itemSize 프로퍼티에 적절한 너비와 높이 값을 할당한다. 각 셀마다 다른 크기를 지정하려면 델리게이트에서 `collectionView(_:layout:sizeForItemAt:)` 메서드를 구현해야 한다. 메서드의 매개변수로 제공하는 indexPath를 사용하여 해당 셀의 크기를 반환할 수 있다.



**셀 및 행 간격**

플로우 레이아웃을 사용하여 같은 행의 셀 사이의 최소 간격과 연속하는 행 사이의 최소 간격을 지정할 수 있다. 명심해야 할 것은 설정하는 간격이 최소 간격이라는 점이다. 모든 셀의 크기가 같다면 플로우 레이아웃은 행 간격의 최솟값을 절대적으로 수용하며, 하나의 행에 있는 모든 셀이 다음 행의 셀과 균등한 간격을 유지할 수 있다.



**컨텐츠 여백**

섹션 인셋은 셀을 배치할 때 여백을 조절하는 방법의 하나이다. 인셋을 사용하여 섹션 헤더 다음과 푸터 앞에 공간을 삽입할 수 있다. 또한 컨텐츠의 면 주위에 공간을 삽입할 수도 있다. 인셋은 셀 배치에 있어서 사용 가능한 공간을 줄이기 때문에 이를 활용하여 주어진 행의 셀 개수를 제한할 수 있다.



**셀 예상 크기**

셀에 오토레이아웃을 적용하고 셀 스스로 크기를 결정한 후 이를 UICollectionViewLayout 객체에 알린다. 이 방법을 사용하기 위해 estimatedItemSize 프로퍼티를 사용하여 대략적인 셀의 최소 크기를 미리 지정한다.



# UICollectionViewDelegateFlowLayout

UICollectionViewDelegateFlowLayout 프로토콜은 UICollectionViewFlowLayout 객체와 상호작용하여 레이아웃을 조정할 수 있는 메서드를 정의한다. 이 프로토콜의 메서드는 셀의 크기와 셀 사이의 간격을 정의하며, 모두 선택사항이다.



**주요 선택 메서드**

- collectionView(_:layout:sizeForItemAt:): 지정된 셀의 크기를 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               sizeForItemAt indexPath: IndexPath) -> CGSize
  ```

- collectionView(_:layout:insetForSectionAt:): 지정된 섹션의 여백을 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               insetForSectionAt section: Int) -> UIEdgeInsets
  ```

- collectionView(_:layout:minimumLineSpacingForSectionAt:): 지정된 섹션의 행 사이 최소 간격을 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               minimumLineSpacingForSectionAt section: Int) -> CGFloat
  ```

- collectionView(_:layout:minimumInteritemSpacingForSectionAt:): 지정된 섹션의 셀 사이 최소 간격을 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               minimumInteritemSpacingForSectionAt section: Int) -> CGFloat
  ```

- collectionView(_:layout:referenceSizeForHeaderInSection:): 지정된 섹션의 헤더 뷰의 크기를 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               referenceSizeForHeaderInSection section: Int) -> CGSize
  ```

- collectionView(_:layout:referenceSizeForFooterInSection:): 지정된 섹션의 푸터 뷰의 크기를 반환

  ```swift
  optional func collectionView(_ collectionView: UICollectionView,
                               layout collectionViewLayout: UICollectionViewLayout,
                               referenceSizeForFooterInSection section: Int) -> CGSize
  ```