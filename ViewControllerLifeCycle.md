# Understand the View Controller LifeCycle



## Why do we need to know?

대부분의 어플리케이션은 단일 뷰보다는 여러 뷰가 존재하며, 그에 따른 뷰 컨트롤러도 다수개 있다.

그렇게 복잡한 앱일수록 사용자와의 상호작용이나, 화면이 load/unload 되는 게 많아지므로

뷰가 보여지는 상황이 변화하는 시점에 잘 맞춰 UI의 변화나 데이터 변화를 잘 다룰 수 있어야 한다.



## States of View

뷰가 보여지는 상황은 크게 4가지로 분류할 수 있다.

- Appearing - 뷰가 화면에 나타나는 중
- Appeard - 뷰가 화면에 나타나는게 완료 된 상태
- Disappearing - 뷰가 화면에서 사라지는 중
- Disappeared - 뷰가 화면에서 사라진 상태 



![스크린샷 2020-02-10 오전 11 32 52](https://user-images.githubusercontent.com/40784518/74116941-19cc9800-4bf9-11ea-84ab-d830cdd197d1.png)



## Methods be called

 `UIViewController` 는 뷰가 보여진 상태에 따라 호출하는 메소드가 다르다.

그럼 각각의 시점에 어떤 메소드가 호출되는지 알아보자.

- `viewDidLoad()`

  -  일반적으로, content view가 처음 생성될 때만 호출된다.
  - viewController의 추가적인 setup을 이곳에서 작성하면 된다.

- `viewWillAppear()`

  - viewController의 content view가 뷰 계층에 추가되기 직전(화면에 보여지기 시작하면서)에 호출된다.
  - 화면에 표시되기 전에 발생해야 하는 작업을 실행하려면 여기에 작성하면 된다.(화면이 보여지지 않았던 시간동안 변경되었을 데이터를 업데이트 하기 위한 코드 등)
  - viewController가 화면에 나타날 때마다 반복 실행된다.

- `viewDidAppear()`

  - viewController의 content view가 뷰 계층에 추가되기 직후(화면에 완전히 보여지고 난 후)에 호출된다.
  - 데이터를 가져오거나 애니메이션 처럼 뷰가 화면에 나타난 즉시 발생해야 하는 작업들은 이 메소드를 이용한다.

- `viewWillDisappear()`

  - viewController의 content view가 뷰 계층에 제거되기 직전(화면에서 사라지기 시작하면서)에 호출된다.(다음 뷰 컨트롤러의 전환이 발생하고 원본 뷰 컨트롤러가 화면에서 제거되기 전)

  - 일반적으로 이 시점에서 수행해야 하는 작업이 거의 없다.

    

- `viewDidDisappear()`

  - viewController의 content view가 뷰 계층에 제거되기 직후(화면에서 완전히 사라지고 난 후)에 호출된다.

  - 뷰 컨트롤러가 화면에 없는 동안 실행해서는 안되는 작업을 중지하려면 이 메소드를 이용한다. 

    







### References

- https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/WorkWithViewControllers.html#//apple_ref/doc/uid/TP40015214-CH6-SW3
- https://www.codementor.io/@hemantkumar434/view-controller-lifecycle-ios-applications-7oyju9lp6
- https://theswiftpost.co/view-lifecycle/

