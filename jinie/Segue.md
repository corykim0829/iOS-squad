# UIStoryboardSegue

두 개의 뷰 컨트롤러 사이의 시각적 전환을 준비하고 수행하는 객체

<br>

# Overview

Segue 객체는 전환과 관련된 뷰 컨트롤러에 대한 정보를 포함한다. Segue가 트리거되었지만 시각적 전환이 발생하기 전에 스토리보드 런타임은 현재 뷰 컨트롤러의 prepare(for:sender:) 메서드를 호출하여 필요한 데이터를 표시될 뷰 컨트롤러에 전달할 수 있다.

뷰 컨트롤러 사이의 segue를 수행해야 할 때 스토리보드 런타임이 segue 객체를 생성한다. 원한다면 performSegue(withIdentifier:sender:) 메서드를 통해 프로그래밍 방식으로 segue를 시작할 수 있다.

<br>

# Subclassing

뷰 컨트롤러 사이의 커스텀 전환을 제공할 경우 UIStoryboardSegue를 서브클래싱할 수 있다. 커스텀 segue를 사용하려면 인터페이스 빌더의 뷰 컨트롤러 사이에 segue 라인을 만들고, inspector에서 타입을 Custom으로 설정한다. 또한 inspector에서 segue의 클래스명을 지정해야 한다.

스토리보드 런타임이 커스텀 Segue를 감지하면 클래스의 새 인스턴스를 생성하고, 뷰 컨트롤러 객체를 구성하고, 뷰 컨트롤러 소스에 segue를 준비할 것을 요청한 후 segue를 수행한다.

커스텀 segue의 경우 재정의해야 할 주요 메서드는 perform() 메서드이다. 스토리보드 런타임은 source 뷰 컨트롤러에서 destination 뷰 컨트롤러로 시각적 전환을 수행할 경우 이 메서드를 호출한다. 커스텀 segue 서브클래스에서 변수를 초기화해야 할 경우 init(identifier:source:destination:) 메서드를 재정의할 수 있다.

<br>

# Alternatives to Subclassing

Segue가 추가 정보를 저장하거나 perform() 메서드 이외의 것을 제공할 필요가 없을 경우 서브클래싱 대신 init(identifier:source:destination:performHandler:) 메서드를 사용해도 된다.

<br>

# Pass Data to the Presented View Controller

UIKit은 전환 중에 자동으로 뷰 컨트롤러를 생성하고 표시하므로 segue가 발생하기 전에 prepare(for:sender:) 메서드를 사용하여 데이터를 해당 뷰 컨트롤러에 전달해야 한다. Segue를 시작한 객체가 포함된 뷰 컨트롤러에서 메서드를 구현하면 된다.

```
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let subViewController = segue.destination as? SubViewController {
        subViewController.data = self.data
    }
}
```

<br>

델리게이트 디자인 패턴을 사용하여 서브 뷰 컨트롤러에서 메인 뷰 컨트롤러로 데이터를 다시 전달할 수 있다. prepare(for:sender:) 메서드에서 해당 관계를 구성하며, 메인 뷰 컨트롤러가 서브 뷰 컨트롤러의 델리게이트가 되어야 한다.

```
protocol MyDataDelegate {
    func passData(_ data: MyData)
}

class MainViewController: UIViewController, MyDataDelegate {
    var data: MyData?

    func passData(_ data: MyData) {
        self.data = data
    }

    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "SubSegue", let subViewController = segue.destination as? SubViewController {
            subViewController.data = self.data
            subViewController.delegate = self
        }
    }
}

class SubViewController: UIViewController {
    var data: MyData?
    var delegate: MyDataDelegate?

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        delegate?.passData(data)
    }
}
```

<br>

# References

- [UIStoryboardSegue](https://developer.apple.com/documentation/uikit/uistoryboardsegue)
- [Customizing the Behavior of Segue-Based Presentations](https://developer.apple.com/documentation/uikit/resource_management/customizing_the_behavior_of_segue-based_presentations)