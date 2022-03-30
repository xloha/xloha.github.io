---
layout: post
title:  "(iOS) RIBs - 복잡한 플로우2"
date: 2022-01-07 10:40:00 +0900
categories: ios
tags:
- iOS
- RIB
- architecture
---

# 충전하기

## 카드 개수에 따라서 화면이 달라져야하는 로직의 위치?
* `FinanceHome` : cardsOnFileRepository를 받아와서 카드 개수를 확인하고 충전화면을 띄울지,,카드추가화면을 띄울지,,
=> 충전하기 버튼을 눌렀을 때 떠야하는 플로우를 사용하는 쪽에서 해야할 일이 너무 많아짐
=> 재사용 불편해짐

<br>

> RIBs
> ViewController가 앱을 주도하는 것이 아니라 리블렛의 Interactor가 앱의 로직을 주도하기 때문에 화면에 얽매이지 않고 비즈니스 로직을 마음껏 구현할 수 있음
> View없이 로직만 있는 리블렛을 만들 수 있음 => viewless RIB

* `충전하기` 같은 경우는 viewless riblet을 최상위에 두면 카드의 개수를 확인해서 각기 다른 첫 화면을 띄워주는 로직을 손쉽게, 재사용가능하게 만들 수 있음

<br>

# TopupDependency

```swift
protocol TopupDependency: Dependency {
    // TODO: Make sure to convert the variable into lower-camelcase.
    var topupBaseViewController: ViewControllable { get }
    // TODO: Declare the set of dependencies required by this RIB, but won't be
    // created by this RIB.
}
```
* Topup riblet이 동작하기 위해 필요한 것들을 선언해 놓는 곳
Topup riblet은 부모riblet이 viewController를 하나 지정을 해줘야함 
    * 이 viewController는 router한테 넘겨줘서, router에서 이 viewController를 접근할 수 있음
    * Topup riblet에서 어떤 화면을 새로 띄우고싶을 때 이 viewController에다가 present나 dismiss를 해주면 됨
    * 따라서 해당 viewController는 Topup riblet이 소유하고 있는 view가 아니라 Topup riblet을 띄운 부모가 지정해준 view임


* topup riblet은 viewless riblet임
    * view가 있는 riblet일 경우 부모에게 "나 끝났다" -> didClose를 알려주면 떠있던 화면을 dismiss 시키는건 부모의 역할이였음
    * viewless riblet은 부모가 직접 present한 view가 없기 때문에 dismiss 하지 않음
        * viewless riblet은 자신의 역할이 끝났을 때 본인이 띄웠던 화면을 본인이 다 닫아줘야할 책임이 있음

## cleanupViews()
* cleanupViews 는 TopupInteractor의 `willResignActive`에서 호출됨 
    * `willResignActive`는 부모가 detachChild를 호출할 때 불리게 됨

```swift
func cleanupViews() {
    if viewController.uiviewController.presentedViewController != nil,
        navigationControllable != nil {
        navigationControllable?.dismiss(completion: nil)
    }
}
```

* viewController에 띄웠던 view가 있다면, 우리가 띄운 navigationControllable이 있다면 navigationControllable을 dismiss하는 것을 topuprouter가 책임지고 해야됨


## 생각해 볼 점
### 충전하기 화면에서 `카드선택 목록`, `카드추가 화면` 이 노출될 수 있는데, 이 화면들이 어떻게 구성되어야 할까?
> 충전하기 화면의 다음화면(이어지는 화면)이라고 해서 `EnterAmount` 리블렛에서 다음 리블렛(카드선택목록)을 붙이고, 카드선택목록 리블렛에서 다른 리블렛(카드추가화면)을 붙이면 `EnterAmount`리블렛이 점점 커짐
> 
> EnterAmount Builder에 점점 child builder들이 많아지게 되고, 그러면 줄줄이 연관이 생겨서 ***riblet이나 객체의 재사용성이 떨어지게 됨***
>
> 크고 복잡하고, 로직이 많은 리블렛으로 변하게 됨
>
> 충전하기 화면, 카드선택목록 화면, 카드추가 화면이 서로를 모르게 하는 것이 가장 좋음

### `Topup 리블렛`의 역할 
자신은 View가 없으면서 자식 리블렛들의 navigation만 담당하고 있음

따라서 Topup 리블렛이 비즈니스 로직에 맞춰서 이 화면을 띄웠다, 저 화면을 띄웠다 유연하게 화면간 이동을 해줄 수 있음


부모가 자식한테 데이터를 전달할때는 Stream으로 전달함


# 총정리

## TopupDependency
topup riblet이 동작하기 위해 필요한 것들이 뭔지 알려줌
riblet이 필요한 것들을 충족시켜주면 riblet을 자식으로 붙일 수 있게 됨
riblet은 자기가 할일을 하다가 listener를 통해서 부모에게 결과를 알려줌
