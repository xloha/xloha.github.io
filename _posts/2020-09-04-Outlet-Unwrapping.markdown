---
layout: post
title:  "(iOS) Outlet의 Implicitly Unwrapped Optional"
date:   2020-09-04 15:10:00 +0900
categories: iOS swift
---
<br>

❓왜 outlets은 Optional로 정의되어있고 Forced Unwrapping을 하고 있을까?  

✅ @IBOutlet과 같은 변수는 연결했다는 것을 확실히 할 수 있기 때문에 !을 붙이는 것임!

> ```swift
> @IBOutlet weak var movieTableView: UITableView! 
> ```
>
> 

<br>

# @IBOutlet의 Forced Unwrapping

* **Implicitly unwrapped optional** : 데이터 구조상 '**항상**' 값을 갖는 옵셔널이 있을 수 있음. 이 경우에는 옵셔널을 언래핑할 때 굳이 nil을 체크할 필요가 없음! 항상 값을 가지고 있으니 unwrapping이 안전하게 될 것임 (초기화된 이후로 항상 값을 갖고 있다는 것이 보장될 때 유용하게 사용할 수 있음)

* **Implicitly unwrapped optional** 은 주로 클래스 초기화 과정에 사용됨

  ```swift
  class ViewContrller: UIViewController {
    @IBOutlet weak var leftButton: UIButton!
  }
  ```

  * ***@IBOutlet과 같은 변수는 연결했다는 것을 확실히 할 수 있기 때문에 !을 붙이는 것임!***
  * 하지만 만약 **@IBOutlet을 선언했음에도 불구하고 이 버튼을 사용하지 않을 수 도 있을 경우**에는 **?**를 사용하면 됨 (지금 당장에는 이 버튼을 사용한다는 확신이 있지만, 연결되지 않은 경우도 생길 수 있기 때문에 만약을 대비하여 `UIButton?` 을 사용하는 것이 훨씬 좋은 방법임)

* **class나 structure의 인스턴스가 initialization(초기화)되기 전에 저장된 모든 property(속성에)는 valid value를 가지고 있어야 하기 때문임!**

    > Classes and structures must set all of their stored properties to an appropriate initial value by the time an instance of that class or structure is created. Stored properties cannot be left in an indeterminate(불확실한) state. 

* 초기화시 속성에 어떠한 값을 저장할지 모를 경우에 default value를 할당할 수 있고, 또는 속성을 implicitly unwrapped로 선언할 수 있음
  * outlet의 값은 초기화시에 셋팅할 수 없다!!
  * viewController가 초기화될 때 view는 아직 load되지 않았고, viewController가 초기화 되고 난 후에 view가 load됨 ==> 즉  viewController에 있는 어떠한 outlet이던지 viewController의 초기화 후에 바로 값을 가지고 있지 않기 때문에 outlet이 (implicitly unwrapped) optional로 선언된 것임

  
<br>
<br>


# Optional(!) vs Optional(?)

* 만약 viewDidLoad가 된 상태가 아니라면 연결된 UI속성은 아직 객체가 생성되지 않은 상태이므로 UI속성에 객체가 당연히 있다고 가정하고 접근한다면 `EXC_BAD_INSTRUCTION` 런타임 에러가 발생함

* 이를 방지하기 위해 `forced unwrapping optional`이 아니라 `optional`타입으로 사용하거나 `optional chaining`을 이용하거나` isViewLoaded함수`로 view가 메모리에 적재되었는지 확인해야 함 



```swift
// forced unwrapping optional을 optional 타입으로 바꾼 경우
	class CustomViewController: UIViewController {
		@IBOutlet weak var label: UILabel?

		func setText(str: String) { self.label?.text = str }
	}

	// isViewLoaded 함수를 사용한 경우
	class CustomViewController: UIViewController {
		@IBOutlet weak var label: UILabel!

		func setText(str: String) { 
			guard self.isViewLoaded == true else { return }
			self.label.text = str
		}
	}

	// optional chaining을 이용한 경우
	class CustomViewController: UIViewController {
		@IBOutlet weak var label: UILabel!

		func setText(str: String) {
			self.label?.text = str
		}
	}
```


