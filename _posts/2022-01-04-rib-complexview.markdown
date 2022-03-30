---
layout: post
title:  "(iOS) RIBs - 복잡한 뷰"
date: 2022-01-04 10:40:00 +0900
categories: ios
tags:
- iOS
- RIB
- architecture
---

## AppDelegate
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    ...

    let result = AppRootBuilder(dependency: AppComponent()).build()
    self.launchRouter = result.launchRouter
    self.urlHandler = result.urlHandler
    launchRouter?.launch(from: window)

    ...
}
```
* AppRootBuilder를 만들어서 `Build`를 하고 `launch`함
* `launchRouter`는 앱의 맨 처음 리블렛에만 사용함
    * 맨 처음에는 부모 리블렛이 없기 때문에 attach를 할 것이 없기 때문

<br>    

## AppRootInteractor

```swift
override func didBecomeActive() {
    super.didBecomeActive()

    router?.attachTabs()
}
```
* `didBecomeActive`가 MVC에서의 `viewDidLoad`와 비슷한 역할
* 리블렛이 처음 RIBs 트리에 붙여질 때 부모에게 attach가 되서 활성화될 때 맨 처음 불림
    * 앱이 로딩이 되고 앱 루트를 붙이자마자, router에 attachTab이 붙어서 3개의 tab을 만들어줌
    * 이 때 또 각 탭의 리블렛들이 불림 
        * FinanceHome의 Interactor에서 SuperPayDashboard 리블렛을 붙여주면 됨


## FinanceHomeRouter

```swift
protocol FinanceHomeInteractable: Interactable, SuperPayDashboardListener {
    var router: FinanceHomeRouting? { get set }
    var listener: FinanceHomeListener? { get set }
}

protocol FinanceHomeViewControllable: ViewControllable {
	func addDashboard(_ view: ViewControllable)
}

final class FinanceHomeRouter: ViewableRouter<FinanceHomeInteractable, FinanceHomeViewControllable>, FinanceHomeRouting {
  
	private let superPayDashboardBuildable: SuperPayDashboardBuildable
    private var superPayRouting: Routing?
    ...
	
    func attachSuperPayDashboard() {
		if superPayRouting != nil {
			return
		}

		let router = superPayDashboardBuildable.build(withListener: interactor)

		let dashboard = router.viewControllable
		viewController.addDashboard(dashboard)

		self.superPayRouting = router
		attachChild(router)
	}
}
```

* 자식 리블렛을 붙이려면 먼저 Builder에 build메소드를 호출해서 라우터를 받아줘야함
* `Build`를 해줄 땐 `listener`를 파라미터로 넘겨줘야하는데 자식리블렛의 listener는 `비즈니스 로직을 담당하는 interactor`
    * HomeInteractor가 SuperPayDashboardListener를 채택해야함
* superPayViewController는 present를 할 것이 아니라 subview를 해줘야 함
* 자식 리블렛을 붙일 때 (실수로 or 버그로) 똑같은 자식을 두번이상 붙이지 않도록 방어로직을 추가해 줌
    * 자식 라우터를 붙인 다음 property로 들고 있는 방식 사용


## FinanceHomeViewController

```swift
func addDashboard(_ view: ViewControllable) {
    let vc = view.uiviewController

    addChild(vc)
    stackView.addArrangedSubview(vc.view)
    vc.didMove(toParent: self)
}
```
* viewController의 `didMove(toParent:)`를 호출해야지 viewController의 lifecycle을 유지할 수 있음


## 변동이 있는 데이터, 여러 곳에서 쓰일 수 있는 데이터는 스트림으로 관리하는 것이 좋음

## SuperPayDashboardInteractor

```swift
final class SuperPayDashboardInteractor: PresentableInteractor<SuperPayDashboardPresentable>, SuperPayDashboardInteractable, SuperPayDashboardPresentableListener {

    weak var router: SuperPayDashboardRouting?
    weak var listener: SuperPayDashboardListener?

	private let dependency: SuperPayDashboardInteractorDependency

    init(
		presenter: SuperPayDashboardPresentable,
		dependency: SuperPayDashboardInteractorDependency
	) {
		self.dependency = dependency
        super.init(presenter: presenter)
        presenter.listener = self
    }

    ... 

}
```

* 생성자가 변경될 때마다 고쳐야하는 것들이 많아짐
    * 생성자, 빌더 
* Interactor가 필요한 것을 protocol로 선언해두는 것이 좋음
    * 이렇게하면 나중에 변경사항이 생겨도 고쳐야할 것이 줄어들음
* SuperPayDashboard는 화면에 극히 일부를 담당하는(UI를 그리는 역할이 더 큰,,)  build에서 balance를 생성해서 불러오기보다는, 부모로부터 받는 것이 더 좋아보임

## SuperPayDashboardInteractor

```swift
protocol SuperPayDashboardInteractorDependency {
	var balance: ReadOnlyCurrentValuePublisher<Double> { get }
}

final class SuperPayDashboardInteractor: PresentableInteractor<SuperPayDashboardPresentable>, SuperPayDashboardInteractable, SuperPayDashboardPresentableListener {

    ...

	private let dependency: SuperPayDashboardInteractorDependency

    init(
		presenter: SuperPayDashboardPresentable,
		dependency: SuperPayDashboardInteractorDependency
	) {
		self.dependency = dependency
        super.init(presenter: presenter)
        presenter.listener = self
    }

    ...

}
```
* 부모로부터 받고싶은 Dependency는 `SuperPayDashboardDependency`에 선언함


* 리블렛끼리 소통하고 정보를 주고받을 때는 리블렛의 두뇌인 `Interactor`끼리 정보를 주고 받음

## 부모 -(데이터)-> 자식
* **Stream을 이용**
    * 리블렛은 여러 자식 리블렛을 가질 수 있기 때문`(1:N)` -> 이런 관계에서는 Stream이 좋음
    * Stream은 **다수의 Subscriber한테 동시에 값을 전달하는게 간편하기 때문**
     
## 자식 -(데이터) -> 부모
* 하나의 부모 리블렛밖에 가질 수 없기 때문에 delegate pattern을 이용함