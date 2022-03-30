---
layout: post
title:  "(iOS) RIBs - 복잡한 플로우1"
date: 2022-01-05 10:40:00 +0900
categories: ios
tags:
- iOS
- RIB
- architecture
---

## RIBs의 독특한 특징이자 장점
뷰없이도 로직을 태울 수 있음


##
`viewControllable`, `viewController` 는 UIViewController가 아니기 때문에 present 메소드가 없음

하지만 viewControllable에서 present를 하는 로직은 사실상 RIBs를 쓰는 모든 곳에서 사용되기 때문에 RIBs+Util에 선언 해놓았음


# `PageSheet`을 끌어서 당기면 원하는 생명주기로 돌지 않음
willResignActive

## FinanceHomeRouter

```swift
func attachAddPaymentMethod() {
    /// 여기 있는 방어로직 때문에 viewController가 다시 띄워지지 않음
    if addPaymentRouting != nil {
        return
    }

    let router = addPaymentMethodBuildable.build(withListener: interactor)

    /// navigation bar가 필요하기 때문에 navigation bar에 한번 싸서 present
    let navigation = NavigationControllerable(root: router.viewControllable)
    viewControllable.present(navigation, animated: true, completion: nil)

    addPaymentRouting = router
    attachChild(router)
}

func detachAddPaymentMethod() {

}
```

* ViewController를 띄웠던(present)를 했던 부모가 자식을 무조건 책임지고 닫아야 함
    * present와 dismiss가 쌍으로 같은 소스코드 위치에 있게 됨
    * 이 리블렛이나 viewController가 어디서든 재사용 가능하게 됨

이런 고민없이 코딩을 하게되면 viewController가 자기자신을 dismiss하는 코드를 자기자신 안에서 부르게 됨

➡️ 이렇게 되면 사실상 **해당 viewController는 재사용할 수 없음**
    * 해당 ***viewController를 사용하는 부모의 입장에서는 life cycle을 전적으로 관리할 수 없기 때문***
    * 따라서 닫는 행위는 부모에게 맡겨야함


* Interactor가 리블렛의 두뇌이기 때문에 Adapter는 Interactor에서 받을 수 있도록 함

```swift
final class FinanceHomeInteractor: PresentableInteractor<FinanceHomePresentable>, FinanceHomeInteractable, FinanceHomePresentableListener, UIAdaptivePresentationControllerDelegate {
    ...
}
```
* `UIAdaptivePresentationControllerDelegate` 여기에 추가하면 문제가 생김
    * `UIAdaptivePresentationControllerDelegate` delegate는 UIKit class임
    * interactor는 UIKit에 대해서 아무것도 모르고 있고, 계속 모르는 것이 좋을 것 같음 