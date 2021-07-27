---
layout: post
title:  "(iOS) Init - 초기화"
date: 2021-07-27 13:40:00 +0900
categories: iOS swift
tags:
- init
- 초기화
---

🌈💕해윙~ 블로그 개오랜만~~~~❤️⭐️💜

오늘은 swift의 init에 대해서 알아보자

<br>

### ❓ `init`

### ✅  Designated init : 지정 이니셜라이저

### ✅  Convenience init : 편의(보조) 이니셜라이저

### ✅  Required init : 필수 이니셜라이저

<br>
<br>

# 1. `Designated init` (지정 이니셜라이저)

### [ 사용방법 ]
* `Optional` 타입 : 이니셜라이저는 자동으로 값을 `nil`로 초기화  
* **초기화하는 중에 상수 속성의 값은 수정 가능**하며, 초기화가 끝난 후에는 명확한 값으로 설정됨

<br>

```swift
class Person() {
    var name: String
    var age: Int?
    let sex: String

    init(name: String, age: String, sex: String) {
        self.name = name
        self.age = age
        self.sex = sex
    }
}
```
<br>

# 2. `Convenience init` (편의 이니셜라이저)

### 클래스의 원래 이니셜라이저인 `init`을 도와주는 역할 
👉 더 적은 입력으로 초기화를 편리하게 할 수 있도록 도와줌

<br>

### [ 사용방법 ]
* Convenience init을 사용하려면 ***Designated init 선언 필수***
* Designated init 파라미터 중 일부를 기본값으로 설정하고, Convenience init 안에서 Designated init을 호출하여 초기화
* Convenience init은 같은 클래스에서 다른 이니셜라이저를 호출해야 함

<br>

```swift
class Person() {
    init() {

    }

    convenience init() {

    }
}
```
<br>

# 3. `Required init` (필수 이니셜라이저)

### 모든 자식 클래스에서 반드시 구현해야하는 이니셜라이저
👉 자식클래스에서 구현할 때도 `required` 키워드 사용 필수

* required는 오버라이드를 기본적으로 포함하고 있음

<br>

```swift
class Person() {
    var name: String
    var age: Int?
    let sex: String
    var mood: String

    init() {}

    required init(mood: String) {
        self.mood = mood
    }
}

class eunyeong: Person {
    required init(mood: String) {
        mood = "happy"
    }
}
```

<br>

# 4. `Initializer Delegation for Value Types` (값 타입을 위한 이니셜라이저 딜리게이션)

### **값 타입**에서 다른 이니셜라이저를 참조하도록 함
👉 이니셜라이저 안에서 `self.init` 을 사용함

### [ 사용방법 ]
* **값 타입은 상속을 지원하지 않음**
* 값 타입을 위한 사용자 이니셜라이저를 정의하면 **더 이상 기본 이니셜라이저에 접근할 수 없음**
    * 이러한 제약은 필수적인 설정을 제공하는 복잡한 이니셜라이저 대신, 자동 이니셜라이저를 사용하여 사고를 방지

<br>

```swift
struct Size {
    var width = 0.0
    var height = 0.0
}

struct Point {
    var x = 0.0
    var y = 0.0
}

struct Rect {
    var origin = Point()
    var size = Size()
    init() {}

    // 멤버 이니셜라이저와 같은 기능
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }

    // 기존에 제공되는 이니셜라이저의 이점을 사용
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

<br>

> ### **멤버 이니셜라이저** 
> 구조체 인스턴스 멤버 속성을 초기화하는 축약방법 
>
> 사용자 이니셜라이저를 정의하지 않으면 자동으로 멤버 이니셜라이저를 받음
>
> ```swift
> struct Size {
>     var width = 0.0
>     var height = 0.0
> }
> 
> let size = Size(width: 2.0, height: 2.0)
> ```
>