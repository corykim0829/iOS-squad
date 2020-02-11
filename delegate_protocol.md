# Delegate Pattern vs Protocol

출처 : <https://docs.swift.org/swift-book/LanguageGuide/Protocols.html>

# Protocol

프로토콜은 특정 작업이나 기능에 적합한 방법, 속성 및 기타 요구 사항의 청사진을 정의합니다.
그런 다음 이러한 요구 사항을 실제로 구현하기 위해 Class, Struct 또는 Enum로 프로토콜을 채택 할수 있습니다.

해당 프로토콜 요구 사항을 충족하는 모든 유형은 해당 프로토콜을 준수한다고 합니다. 
<br>=> 프로토콜을 채택해서 프로토콜의 모든 요구사항을 충족하는 유형은 해당 프로토콜을 준수한다고 말한다. 

```swift
import Foundation

@objc
protocol Assistable {
    func manageSchedule(topic : String, date : Date)
    func drive(to destination: String)
    @objc optional func takeCareMypet(name : String)
    func bossAskedSchedule(when: Date) -> String?
    func bossAskedCanGoHomed(when: Date) -> Bool
    
    var scheduleLicense : Any {get set}
    var driverLicense : Any { get }
    @objc optional var petLicense : Any { get }
}

class Assistance : Assistable {
    func bossAskedSchedule(when: Date) -> String? {
        return nil
    }
    
    func bossAskedCanGoHomed(when: Date) -> Bool {
        return false
    }
    
    func manageSchedule(topic : String, date : Date) {
        
    }
    
    func drive(to destination : String) {
        print("대리운전 가능합니다. 새벽 4시에도 호출가능합니다. \(destination)까지 끄떡없습니다.")
    }
    
    func makeACall(to someone : String){
        print("\(someone)에게 전화합니다")
    }
    
    var scheduleLicense : Any = ""
    var driverLicense : Any = ""
    var toiecGrade : Int = 999
    var programmingSkill : Any = ""
    var certifications : [Any] = [""]
}

class Secretary : Assistable {
    func bossAskedSchedule(when: Date) -> String? {
        return nil
    }
    
    func bossAskedCanGoHomed(when: Date) -> Bool {
        return false
    }
    
    func manageSchedule(topic : String, date : Date) {
        print("2시 ~ 3시에 가능합니다. 근데 대충할거임")
    }
    
    func drive(to destination : String) {
        print("2시 ~ 3시만 \(destination)에 갈 수 있습니다.")
    }
    
    func makeACall(to someone : String){
        print("2시 ~ 3시만 \(someone)에게 전화합니다")
    }
    
    var scheduleLicense : Any = ""
    var driverLicense : Any = ""
    var toiecGrade : Int = 999
    var programmingSkill : Any = ""
    var certifications : [Any] = [""]
}
```

준수 유형이 구현해야하는 요구 사항을 지정하는 것 외에도 프로토콜을 확장하여 이러한 요구 사항 중 일부를 구현하거나 준수 유형이 활용할 수 있는 추가 기능을 구현할 수 있습니다.
<br>=> 프로토콜을 확장하는 것

```swift
protocol Personable {
    var name: String { get }
    var age: Int { get }
    
    func getAge() -> Int
}

extension Personable {
    func getAge() -> Int {
        return self.age
    }
}

struct FireFighter: Personable {
    var name: String
    var age: Int
    
    func extinguish() {
    }
}
```
=> FireFighter 클래스는 Personable 프로토콜을 준수하지만 getAge()를 따로 작성할 필요가 없습니다. 
<br>=> 변형해서 사용하고 싶다면 함수를 작성하시면 됩니다. 
<br>=> 메서드 중복 작성을 피할 수 있습니다.

# Delegation