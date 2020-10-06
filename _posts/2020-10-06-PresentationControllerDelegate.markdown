---
layout: post
title:  "(iOS) PresentationControllerDelegate - 모달 닫기 전에 경고창 띄우기"
date:   2020-10-06 11:10:00 +0900
categories: iOS swift 
tags:
- PresentationControllerDelegate
- alert
- isModalInPresentation
---
<br>

[EUNALARMI](https://github.com/EunYeongKim/EUNALARMI)를 만들면서 메모에 대한 CRUD기능을 만들어보려고 유튜브에 kxcoding강의를 듣고 있던 중에 기억할만한 좋은 내용이 있어서 적어본다.  

메모가 수정됐다면 이를 확인하고, 모달방식으로 띄워진 시트를 닫기 전에 경고창을 띄우는 방식이다.  
<br>

❓ `isModalInPresentation, PresentationControllerDelegate`

✅  `isModalInPresentation = true` 일 경우 delegate 메소드가 실행됨

<br>

# 1. Delegate 설정

`viewWillAppear(), viewWillDisappear()`에서 편집화면이 표시되기 직전에 `presentationController`의 delegate를 설정했다가, 편집화면이 사라지기 직전에 delegate가 해제될 수 있도록 설정해 줌  


```swift
override func viewDidLoad() {
        super.viewDidLoad()
        configureUI()
        memoTextView.delegate = self
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        navigationController?.presentationController?.delegate = self
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        navigationController?.presentationController?.delegate = nil
    }
```

<br>

# 2. `isModalInPresentation`의 값 설정

메모가 편집되었을 경우 `pulldown방식`으로 시트를 닫기 전에 delegate메소드를 호출할 수 있도록 함
* `isModalInPresentation` : 모달 방식으로 동작해야하는지 결정해야하는 flag로 사용됨
    * `true` : 시트가 모달방식으로 동작되고, 시트를 `pull down`으로 내리기 전에  delegate메소드를 호출해줌 
    * *단, iOS13 이전 버전에서는 사용할 수 없음*


```swift
extension NewMemoViewController: UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        if let original = originalMemoContent, let edited = textView.text {
            if #available(iOS 13.0, *) {
                isModalInPresentation = original != edited
            }
        }
    }
}
```
* 텍스트가 편집됐다면, 모달방식으로 동작하도록 함  

<br>

# 3. `UIAdaptivePresentationControllerDelegate` 작성

`isModalInPresentation = true`인 상태에서 `pull down`하면 sheet가 사라지지 않고, `presentationControllerDidAttemptToDismiss` 메소드가 호출됨  

이 메소드에서 alert창을 띄우도록 코드를 작성함  


```swift
extension NewMemoViewController: UIAdaptivePresentationControllerDelegate {
    func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
        let alert = UIAlertController(title: "알림", message: "편집한 내용을 저장할까요?", preferredStyle: .alert)
        let okAction = UIAlertAction(title: "확인", style: .default) { [weak self] (action) in
            self?.didTapSave(action)
        }
        alert.addAction(okAction)
        
        let cancelAction = UIAlertAction(title: "취소", style: .cancel) { [weak self] (action) in
            self?.didTapClose(action)
        }
        alert.addAction(cancelAction)
        
        present(alert, animated: true, completion: nil)
    }
}
```



