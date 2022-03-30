---
layout: post
title:  "(iOS) RIBs - 아키텍쳐와 Composition"
date: 2022-01-03 10:40:00 +0900
categories: ios
tags:
- iOS
- RIB
- architecture
---

### ❓ `Composition(합성, 조립)`

### ✅  객체와 데이터 타입을 조합해서 더 복잡하고 어려운 일을 객체로 만드는 방식

* 작은 객체로 이루어진 코드는 재사용성, 유지보수, 테스트 용이
    * 작은 객체는 public api도 적고, 파라미터도 적기 때문에 테스트해야되는 로직의 순열이 적기 때문

<br>

> Favor Object Composition Over class inheritance (상속 대신 객체의 합성을 선호해라)
> 

>  상속을 가장 잘 쓰는 방법은 상속을 쓰지 않는 것


* 상속은 필요할때는 유용하지만, 유연성이 떨어짐
    * 상속은 코드의 가장 강한 결합형태로, 원치않는 기능도 딸려옴
    * 부모의 행동을 거부하기 위해서 빈 override를 만든다던지, super를 호출하지 않고 기능을 바꾼다던지 방법을 사용하여 부모의 행위를 덮어씌움
        * 부모의 행위를 거부하는 것은 rescope원칙에 위배되어 코드를 작성하는 사람이 예상치 못한 동작을 마주치게 됨

<br>

> UIView에 존재하는 frame 프로퍼티로 위치와 사이즈 변경가능 (UIImageView, UILabel,,)
>
> UISwitch는 사이즈를 바꿔도 적용이 되지 않음
>
> 원칙을 100프로 지켜야하는건 아니지만, 원칙을 어겼을 때 코드의 부작용을 이해하고 제한적으로 쓰는 것이 중요 
>
> 코드를 재사용해야할 때 상속부터 시작한다면, 요구사항이 변할 때 민첩하게 대처하기 힘들어짐
>
> ex. SwiftUI의 View들은 `Value Type`으로 SwiftUI View의 기능을 확장하기 위해서는 `view modifier`를 활용해서 상속없이 기능을 확장할 수 있게 됨
>
> Type, interface, module, function 까지도 조합을 할 수 있음 

<br>

# 객체의 Composition
EX] A View안에 B subview 예제

## 상속 이용

```swift
class A: UIViewController {
    func setupViews() {
        // add subviews
    }
}

class B: A {
    override func setupViews() {
        super.setupViews()
        // add B View
    }
}
```
* A와 B가 끈끈하게 묶여있어서 유지보수가 쉽지 않음

<br>

# 객체 Composition 이용

```swift
class A: UIViewController {

}

class B: UIViewController {

}

/// A와 B를 조합해서 새로운 화면 생성하는 C ViewController
/// 조립하는 역할만 하는 C
class C: UIViewController {
    private let a: UIViewController
    private let b: UIViewController

    // add a,b to child viewController
}
```


* Composition : BlackBox
* Inheritance : WhiteBox
    * 상속된 B는 A의 모든 것을 알고 있기 때문
    * 하나의 객체가 너무 많은 것을 알고있음
        * (많은 로직과 데이터를 가지고 있는 것이 좋을때는 거의 없음)
        * 더 작고, 적은 정보를 아는 객체를 만드는 것이 유지보수 관점, 코드를 이해하기 더 쉬움

# UIKit에 두가지 컨테이너 ViewController
* UINavigationController
* UITabBarController

화면 전체의 UI를 표시하는 것이 아니라, Child ViewController들을 화면에 배치하고, 이동시키는 역할
> 상당히 복잡한 로직을 가지고 있고, 여러 ViewController들을 관리하는 역할
>
> Navigation Controller,Tabbar Controller는 상속이 필요하지 않음
>
> 표시되는 UI들은 navigation item, tabbar item의 값을 세팅해놓으면,
> Container viewController가 그 값을 가지고 composition을 해서
> 화면에 보이는 navigationbar와 tabbar를 만드는 것임

상속없이도, 복잡해보이는 기능을 단순한 API로 만들 수 있음

<br>

# 함수의 Compsition

```swift
let result = [1, 2, 3].map { $0 + 1 }.map { "만 \($0) 살" }
print(result)

/// Optional에 값이 있으면 Unwrapping을 해서 함수를 실행한 값을 다시 Optional에 Wrapping을 해서 return해줌
/// num = nil 이면 nil이 나옴
let num: Int? = 1
let result2 = num.map { $0 + 1 }
print(result2)

/// success(3)
/// mapError 라는 함수로, Error를 Mapping하는 함수도 존재
let myResult: Result<Int, Error> = .success(2)
let result3 = myResult.map { $0 + 2 }
print(result3)
```

1. 모두 Generic 타입
> enum Optional<Wrapped>
>
> associatedtype Element
>
> enum Result<Success, Failure> where Failure : Error
>
> associatedtype Output

2. transform 함수를 인자로 받음

> transform: A -> B
>
> F\<A> -(map)-> F\<B>
>
> F에는 Optional이 들어갈수도, Sequence,,,,가 들어갈 수도 있음

<br>

```swift
let ageString: String? = "10"
let result4 = ageString.map { Int($0) }
```

> transform: A -> B
>
> Optional\<A> -(map)-> Optional\<B>

* Int의 initializer Failable initializer이므로 Optional Int를 return
> transform: A -> Optional\<B>
>
> Optional\<A> -(map)-> Optional\<Optional\<B>>

<br>

### 해결방법

```swift
if let x = ageString.map(Int.init), let y = x {
    print(y)
}

if case let .some(.some(x)) = ageString.map(Int.init) {
    print(x)
}

if case let x?? = ageString.map(Int.init) {
    print(x)
}
```
문법이 알아보기 힘들고, unwrapping을 2번 해야하는 것이 이상함

``` swift
let result5 = ageString.flatMap(Int.int)
```
> transform: A -> Optional\<B>
>
> Optional\<A> -(flatmap)-> Optional\<B>

<br>

# 프로그래밍은 데이터 타입 변환의 일련의 과정
UIEvent -> (tableView)IndexPath -> Model -> URL -> Data -> Model -> ViewModel -> View
* 앱에서 일어나는 작업들은 타입을 변환하는 작업이라고 볼 수 있음

```swift
struct MyModel: Decodable {
    let name: String
}

let myLabel = UILabel()

if let data = UserDefaults.standard.data(forKey: "my_data_key") {
    if let model = try? JSONDecoder().decode(MyModel.self, from: data) {
        let welcomeMessage = "Hello \(model.name)"
        myLabel.text = welcomeMessage
    }
}
```

```swift
let welcomeMessage = UserDefaults.standard.data(forKey: "my_data_key")
    .flatMap { try? JSONDecoder().decode(MyModel.self, from: $0) }
    .map(\.name)
    .map { "Hello \($0)" }

myLabel.text = welcomeMessage
```

<br>

# Interactor, Router기반의 아키텍쳐 Composition
Ex) RIBs, VIPER

### 리블렛
Builder, Router, Interactor, Presenter, View

* 리블렛은 한 화면 전체를 담당할 수 있고, 일부를 담당할 수도 있고, 화면이 아예 없을 수도 있음
    * ViewController가 ChildViewController를 가질 수 있듯이 리블렛도 child리블렛으로 분리 가능

