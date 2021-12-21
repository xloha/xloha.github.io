---
layout: post
title:  "(iOS) Clean Architecture - 클린아키텍쳐"
date: 2021-12-21 13:30:00 +0900
categories: iOS swift
tags:
- ios
- swift
- clean architecture
---

# 크린토피아

[원 글](https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37) 보고 정리

> ⭐️참고할 글 들
>
> [RxSwift + Clean Architecutre](https://medium.com/tiendeo-tech/ios-rxswift-clean-architecture-d7e9eaa60ba)
>
> [Clean Architecture + MVVM](https://www.programmersought.com/article/82166656779/)
>
> ⭐️참고한 글
>
> [medium 영어 문서](https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37)
>
> [Clean Architecture + MVVM 영화 찾기](https://ios-development.tistory.com/555)

<br>

# 1 ] Dependency Inversion

## 🟡 Dependency Inversion : 소프트웨어 모듈을 분리하는 특정 형태

1. 상위 레벨 모듈은 하위 레벨 모듈에 의존해서는 안됨. 둘 다 **추상화**(인터페이스, 프로토콜)에 의존해야함
2. 추상화는 세부사항에 의존해서는 안됨. 세부사항(구체적인 구현)은 추상화에 의존해야함

<br>

## 🟡 Dependency Injection과 Dependency Inversion의 차이?

* ### Dependency Injection

```swift
class A {
    var b: B
    
    init() {
        b = B()
    }
    
    func start() {
        b.doSomething()
    }
}

class B {
    func doSomething() {
    }
}
```

* 클래스 A는 클래스B에 의존 ➡️ B에 대한 변경은 A에 대한 변경을 필요로 함  ➡️ 이 의존성이 클래스A의 재사용성을 낮춤

<br>

## 1) 주입

```swift
class A {
    var b: B
    
    init(b: B) {
        self.b = b
    }
    
    func start() {
        b.doSomething()
    }
}

class B {
    func doSomething() {
    }
}
```

* 클래스A 내부에 B를 초기화하는 대신에 **A를 init할 때 B를 주입**함

<br>

## 2) 추상화

```swift
protocol BInterface {
    func doSomething()
}

class A {
    var b: BInterface
    
    init(b: BInterface) {
        self.b = b
    }
    
    func start() {
        b.doSomething()
    }
}

class B: BInterface {
    func doSomething() {
    }
}

class C: BInterface {
    func doSomething() {
    }
}
```

* 클래스A는 클래스 A와 B를 느슨하게 연결하기 위해 클래스B 자체가 아닌 **클래스B의 프로토콜의 추상화에 의존**해야함  

    ✅ 이제 B 프로토콜을 준수하는 클래스, 즉 클래스B 또는 클래스C 를 클래스A에 주입할 수 있음!

<br>

* ##  Dependency Inversion

    두 모듈 사이에 추상화를 도입하여, 강력한 결합을 제거하고 코드를 상호교환 가능하게 만듦

<br>

>  참고자료
>
>  [Swift DI - 예시 포함](https://medium.com/flawless-app-stories/practical-dependency-inversion-in-swift-1c1142161a8)

<br>
<br>

# 2 ] Clean Architecture의 Layers (3+1)

클린 아키텍쳐에는 일반적으로 **프레젠테이션, 도메인, 데이터** 3가지 레이어가 존재

여기에 진입점에 **앱 레이어**를 추가하고, **각 레이어에 종속성을 주입**함


* ### APP Layer

  * 종속성 주입이 포함된 앱의 진입점

* ### Domain Layer

  * 비즈니스 로직 포함

* ### Presentation Layer

  * UI를 다룸 (Views & ViewModels)

* ### Data Layer

  * 원격 또는 로컬 데이터 소스에서 데이터 가져옴

<br>
<br>

# 3 ] 각 Layer의 의존성

>  종속성 : 소프트웨어가 다른 소프트웨어에 의존할 때 참조하는데 사용되는 광범위한 소프트웨어 엔지니어링 용어
>
>  A → B : A는 B에 의존함. 즉, A는 B의 모든 것을 볼 수 있음

<img src="/assets/image/dependencyDiagram.png" style="zoom: 100%;"/>

* ### Domain Layer

  * 다른 계층에 의존하지 않음
  * 독립적인 Layer로 다른 레이어에서 사용하는 것과 상관없이 앱의 비즈니스 로직을 담고 있음

* ### Presentation Layer

  * 비즈니스 로직을 얻기 위해 Domain Layer에 의존

* ### Data Layer

  * 비즈니스 로직을 수행할 수 있도록 Domain Layer에 데이터를 제공하는 방법을 알아야하므로 Domain Layer에 의 (Dependency Inversion을 준수하여 수행할 수 있음)

* ### Presentation Layer & Data Layer

  * 모두 Domain Layer에 의존
  * Presentation Layer와 Data Layer는 서로 의존하지 않음
  * 프레젠테이션 계층의 클래스가 도메인 계층의 클래스를 사용하면 프레젠테이션 계층이 도메인 계층에 종속 됨

* ### App Layer

  * 모든 계층에 의존함
  * 앱의 진입점이고, 모든 것을 제어함 ➡️ Dependency Injection 역할을 함 ➡️ 모두에 종속됨


<img src="/assets/image/CleanArchitectureDiagram.png" style="zoom: 100%;"/>

<br>
<br>

## 🟡 데이터 흐름

1. View에서 이벤트 발생 ➡️ Presenter(ViewModel)에서 '무엇'인지 판단
2. Presenter는 UseCase 실행
3. UseCase는 User와 Repository를 결합
4. 각 Repository는 network, DB관련 Store에서 데이터 반환
5. 다시 UI에 업데이트 (Store -> Repository -> UseCase -> ViewModel -> View(UI))

