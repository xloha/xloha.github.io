---
layout: post
title:  "(iOS) Outlet의 Strong & Weak"
date:   2020-09-04 13:35:00 +0900
categories: iOS swift
---
<br>

```swift
@IBOutlet weak var listTableView: UITableView!
```

❓ Outlet을 연결하면 자동으로 선언되는 weak의 의미는 무엇일까,,,,,🤔  
<br>

## Strong과 Weak의 개념

**Strong** : (강한 참조를 의미함)소유 대상의 reference count를 1증가시켜서 dealloc되지 않도록 함

**Weak** : (객체를 소유하지 않고 단지 주소값만을 가지고 있는 포인터 개념)reference count는 증가시키지 않고, 소유함으로써 상호참조 발생을 막아 메모리 누수를 막기 위해 사용함

<br>
<br>

# if) outlet에 Strong 사용 시 

* viewController가 dealloc된다면, 소유하고 있던 outlet의 reference count를 감소함 => 결국 모든 outlet이 깔끔하게 dealloc됨!!

  > reference count가 0일 경우에 dealloc됨

<br>  

# Strong을 사용하는 경우

* 깊은 View Hierachy 구조에서 모든 connection이 weak일 경우 중간쯤 있는 view가 어떤 이유로 dealloc된다면 그 **하위 view들도 함께 dealloc되므로, 의도치 않게 subview가 nil일 수 있음**

  > 복잡한 View Hierachy를 가진 경우에는 Weak대신, Strong을 써야만 하는 경우가 있음

* viewController자체가 해제되지 않는 이상 해당 뷰는 화면에서 제거되더라도 여전히 메모리에 남아있음(뷰를 removeFromSuperview를 통해 화면에서 해당 뷰를 제거해도 메모리에는 여전히 남아서 해제되지 않음)

<br>

# Weak를 사용하는 경우

* 메모리 부족 (메모리 부족할 경우 didReceiveMemoryWarning에서 ***main view를 nil처리******하면서 main view를 포함한 subview들까지 모두 dealloc하여 메모리를 확보하는 동작 구현***)

  >  weak로 생성한 뷰는 해당 viewController의 소유가 아님을 의미하므로 removeFromSuperview메소드를 이용해서 화면에서 제거하면 메모리에서 해제됨(자동으로 메모리 해제)

* 뷰를 날리면 그 안에 속해있는 객체들도 날려야할 경우 weak를 사용함 

* weak사용 시 객체들을 따로 붙잡아 두고 있을 상위 객체가 없기 때문에 사용하지 않는 순간 dealloc됨

    > 만약 subview들을 outlet strong으로 가지고 있다면, ViewController가 Strong으로 갖고 있는 reference Count 1 때문에 절대 1아래로 내려가지 않기 때문에 subview의 parentview가 nil이 되더라도 subview는 dealloc되지 않으므로 보이지도 않는 뷰가 메모리를 차지 하게 됨

<br>

# Strong vs Weak

* Strong: **뷰를 scene에서 삭제했다가 다시 추가해야하는 상황일 경우** 

  > ***사용자가 직접 지정해주기때문에 lifecycle을 확실하게 지켜줄 수 있지만*** weak의 경우에는 그럴 수 없다는 문제점 존재

* Weak: 뷰를 scene에서 삭제했다가 다시 추가해야하는 상황이 아닐 경우, **메모리 관리 코드를 쓰지 않으므로 코드가 줄어들고, 가독성 good**

  > ***해당 메모리가 해제되면 자동으로 nil로 초기화가 되지만, 그 시점이 언제일지***,,언제 사라질 지 알 수 없다는 문제점!