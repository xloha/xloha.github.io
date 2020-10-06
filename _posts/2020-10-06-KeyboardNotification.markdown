---
layout: post
title:  "(iOS) Keyboard Notification - textview에서 키보드에 글씨가 잘릴 때"
date:   2020-10-06 11:10:00 +0900
categories: iOS swift 
tags:
- NotificationCenter
- UIResponder.keyboardWillShowNotification
- UIResponder.keyboardWillHideNotification
---
<br>

[EUNALARMI](https://github.com/EunYeongKim/EUNALARMI)를 만들면서 메모에 대한 CRUD기능을 만들어보려고 유튜브에 kxcoding강의를 듣고 있던 중에 기억할만한 좋은 내용이 있어서 적어본다.  

<br>

엄청 긴 텍스트를 작성하게 된다면 키보드에 글씨가 가려져서 수정하거나 텍스트를 확인할 수 없게된다. 이를 위해서는 textview에 여백을 줘야한다.  

keyboard가 보일 때, keyboard가 내려갈 때를 알려주는 Notificaiton을 활용하여 이를 고칠 수 있다.

<br>

❓ `UIResponder.keyboardWillShowNotification, UIResponder.keyboardWillHideNotification`

✅  키보드가 나타날 때, 사라질 때를 알려주는 notification을 이용하여 여백을 주고, 여백을 없앨 수 있다.

<br>

# 문제 상황
<img src="/assets/image/BeforekeyboardNotification.png" alt="여백 주기 전 이미지" style="zoom: 40%;"/> 

<br>

# 1. `Observer` 변수 선언 및 등록
옵저버를 해제할 때 필요한 값을 저장하기 위한 변수
```swift
var willShowToken: NSObjectProtocol?
var willHideToken: NSObjectProtocol?
```
<br>

## 1-1. 키보드가 나타날 때

키보드가 나타날 때 `TextView`와 `TextView의 Scroll`에도 키보드만큼의 여백을 줘야 함

```swift
willShowToken = NotificationCenter.default.addObserver(forName: UIResponder.keyboardWillShowNotification, object: nil, queue: OperationQueue.main, using: { [weak self] (noti) in
    guard let `self` = self else { return }
    if let frame = noti.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
        // 키보드의 높이
        let height = frame.cgRectValue.height
        
        var inset = self.memoTextView.contentInset
        inset.bottom = height
        self.memoTextView.contentInset = inset
        
        inset = self.memoTextView.scrollIndicatorInsets
        inset.bottom = height
        self.memoTextView.scrollIndicatorInsets = inset
    }
})
```
<br>

## 1-2. 키보드가 사라질 때


키보드가 사라질 때 설정했던 여백을 없앰

```swift
willHideToken = NotificationCenter.default.addObserver(forName: UIResponder.keyboardWillHideNotification, object: nil, queue: OperationQueue.main, using: { [weak self] (noti) in
    guard let `self` = self else { return }
    
    var inset = self.memoTextView.contentInset
    inset.bottom = 0
    self.memoTextView.contentInset = inset
    
    inset = self.memoTextView.scrollIndicatorInsets
    inset.bottom = 0
    self.memoTextView.scrollIndicatorInsets = inset
})
```

<br>

# 2. `Observer` 해제
```swift
deinit {
    if let token = willShowToken {
        NotificationCenter.default.removeObserver(token)
    }
    
    if let token = willHideToken {
        NotificationCenter.default.removeObserver(token)
    }
}
``` 

<br>

# 결과
<img src="/assets/image/AfterkeyboardNotification.png" alt="여백 준 후 이미지" style="zoom: 40%;"/> 
