---
layout: post
title:  "[weak self] [unowned self] 캡쳐 리스트"
date: 2021-08-17 11:40:00 +0900
categories: iOS Swift
tags:
- iOS
- Swift
- 캡쳐 리스트
---

### ❓ `캡쳐 리스트`

### ✅  강한 순환 참조 발생을 막기 위해 사용
> weak unowned 모두 레퍼런스 카운팅을 하지 않기 때문에 클로저의 강한 순환 참조를 막기 위해서 사용되는 캡쳐 리스트

> 참고한 글
> 
> [weak unowned](https://velog.io/@msi753/unowned-self)

<br>
<br>


# `캡쳐리스트`
* 클로저 안에서 한 개 이상의 참조 타입을 어떤 참조(`strong`, `weak`, `unowned`)로 캡쳐할지 정의하는 리스트
* 두 개(사용할 참조타입과 클로저)의 인스턴스가 강한 순환 참조가 생기는 것을 방지하기 위해 사용함

<br>

# `weak `
* optional 타입으로 nil이 될 수 있으며, 참조하고 있는 인스턴스가 메모리에서 해제될 시 ARC가 nil로 참조값을 대체
    > 메모리에서 채제된 후에 nil이 되기 때문에 weak 참조변수는 반드시 optional타입
* 참조하고 있는 객체의 생명주기가 짧은 경우 사용

<br>

## 강한 참조에서의 `[weak self]`

```swift
class TestClass{
    var aBlock: (()->())? = nil
    let aConstant = 5
    
    init(){
        print("init")
        aBlock = { [weak self] in   // weak self capture
            print(self.aConstant)
        }
    }
    deinit(){
        print("deinit")
    }
}

var testClass: TestClass? = TestClass()
testClass = nil
```

<br>

## `Closure`를 `locally`하게 사용

```swift
class TestClass {
    let aConstant = 5
    
    init() {
        print("init")
        let aBlock = {
            print(self.aConstant)
        }
    }
    deinit {
        print("deinit")
    }   
}
var testClass: TestClass? = TestClass()
testClass = nil
```


<br>

# `unowned`
* weak와 달리 참조하고 있는 인스턴스가 메모리에서 해제될 시, nil이 되지 않음
* 참조하고 있는 객체가 메모리에서 해제된 후 접근할 시 crash가 날 수 있음 ➡️ 해제된 메모리 영역을 접근하지 않는 확실한 경우에만 사용해야 함
* 따라서 참조하고 있는 객체의 생명주기가 현재 scope보다 더 길거나 같은 경우 사용

<br>

# `self?`(체이닝)  vs  `guard let self = self else { return }`(옵셔널 바인딩)
## * `self?` (체이닝)
* 매번 self를 nil체크 하므로 원치 않은 결과를 만들 수 있음

## * `guard let self = self else { return }` (옵셔널 바인딩)
* 해제되기 전에 self를 캡쳐해버리기 때문에 클로저가 끝날때까지 self의 해제를 지연시킴

<br>

따라서 weak self를 사용할 때에도 이런 사항들을 고려해야 함