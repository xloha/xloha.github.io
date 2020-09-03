---
layout: post
title:  "(iOS) outlet & Action"
date:   2020-09-03 14:55:00 +0900
categories: iOS swift
---

<br>
백엔드 개발자가 될 줄 알았고,,,,only 백엔드 러버였는데,,,  

이런 내가 아요 개발을,,,,,,?😭  

주르륵,,,,,인생,,,내 맘대로 되는 것이 없다,,,,  

<br>
<br>

# Outlet and Action

### UI와 코드를 연결하고 싶을 경우 

* outlet : 코드를 통해서 속성에 접근하고 싶을 경우
* action : 컨트롤에서 발생하는 이벤트를 처리하고 싶을 경우



### Outlet

* Outlet은 클래스의 속성으로 추가되고, 반드시 scene과 연결 된 클래스 내부에 추가해야 됨

```swift
@IBOutlet weak var label: UILabel!
```

`@IBOutlet` : 컴파일러에게 outlet으로 연결된 속성이라는 것을 알려줌





### Action

* Action은 Outlet과 달리 메소드와 연결이 됨

```swift
@IBAction func updateLabel(_ sender: Any) {
}
```

`@IBAction` : 컨트롤과 연결되어있다는 Hint로 사용 됨



> 이벤트의 방향
>
> outlet : 코드에서 컨트롤로
>
> action : 컨트롤에서 코드로 



### UI를 컨트롤러로 잘못연결했을 때는 반드시 UI에서 삭제해줘야함!! 그렇지 않으면 `NSUnknownKeyException` 발생



```swift
@IBOutlet weak var label: UILabel!
```

* label이 optional로 설정이 되어있어서 처음에는 nil일 수 있음 
* UI와 연결이 잘 되어있지 않다면 crash가 발생함