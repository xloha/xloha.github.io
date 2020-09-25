---
layout: post
title:  "(iOS) Notification - 작업이 끝났음을 알리기"
date:   2020-09-25 17:20:00 +0900
categories: iOS swift 
tags:
- Notification
- NotificationCenter
- viewWillAppear
---
<br>

[EUNALARMI](https://github.com/EunYeongKim/EUNALARMI)를 만들면서 메모에 대한 CRUD기능을 만들어보려고 유튜브에 kxcoding강의를 듣고 있던 중에 기억할만한 좋은 내용이 있어서 적어본다.  

새로운 메모를 추가하는 창에서 Save를 눌러 이전화면으로 돌아갈 때 테이블 뷰의 목록을 업데이트 하는 방법에 관해서이다.

<br>

❓ Notification, NotificationCenter

✅  Notification으로 어떠한 작업이 완료됨을 알리고, Observer로 Notification으로를 관찰하여 작업이 완료됨을 인식하고 일을 처리할 수 있음

<br>

# viewWillAppear

* 아래와 같이 **FullScreen방식**으로 화면을 띄우고, Save를 눌렀을 경우 viewWillAppear에서 목록을 업데이트 해주면 viewWillAppear가 실행되면서 새로운 메모가 보여지게 된다.

  ![fullscreen](/assets/image/스크린샷-2020-09-25-오전-10.18.27.png)

  ```swift
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
  
    memoCollectionView.reloadData()
    memoTableView.reloadData()
  }
  ```



* 하지만 화면이 뜨는 방식을 **모달방식**으로 설정하면 viewWillAppear에 작성한 코드가 실행되지 않는다. (즉 화면이 fullScreen이였을때만 viewWillAppear가 불린다.)

  ![modal](/assets/image/스크린샷-2020-09-25-오전-10.25.52.png)



➡ 따라서 이런 모달방식에서 모달화면이 닫혔을 때 실행해야하는 일이 있을 경우에 **Notification**으로 처리할 수 있다.

<br>

# Notification

**`Notification은 라디오 방송`** 같은 것이고, **`NotificationCenter 는 라디오방송국 센터`** 라고 이해하면 된다고 한다.

<br>

1. Notification을 보낼 viewController에서 Notification을 작성한다

   * extension으로 작성함

   ```swift
   extension NewMemoViewController {
       static let newMemoDidInsert = Notification.Name(rawValue: "newMemoDidInsert")
   }
   ```

2. Notification을 보낸다

   * 메모를 저장하는 액션에서 노티를 보냄

   ```swift
   NotificationCenter.default.post(name: NewMemoViewController.newMemoDidInsert, object: nil)
   ```

3. Notification을 관찰하는 Observer를 추가한다

   * Observer를 추가하는 코드는 한번만 실행하면 되기 때문에 viewDidLoad에 작성
   * **`OperationQueue.main`** 을 전달하여 옵저버가 실행하는 코드를 **메인쓰레드에서 실행**하도록 함
   * 4번째 파라미터로 전달 된 클로저가 3번째 파라미터로 전달된 쓰레드에서 실행됨

   ```swift
    override func viewDidLoad() {
           super.viewDidLoad()
           NotificationCenter.default.addObserver(forName: NewMemoViewController.newMemoDidInsert, object: nil, queue: OperationQueue.main) { [weak self] (noti) in
               self?.memoCollectionView.reloadData()
               self?.memoTableView.reloadData()
           }
       }
   ```

   * Notification구현 시 Observer를 해지하는 것이 가장 중요한 과정임 (앱은 정상적으로 실행되도, 내부에서는 메모리가 낭비되고있음)

4. Observer해제

   ```swift
   var token: NSObjectProtocol?
   
   deinit {
     if let token = token {
       NotificationCenter.default.removeObserver(token)
     }
   }
   
   override func viewDidLoad() {
     super.viewDidLoad()
     token = NotificationCenter.default.addObserver(forName: NewMemoViewController.newMemoDidInsert, object: nil, queue: OperationQueue.main) { [weak self] (noti) in                                                                           
        self?.memoCollectionView.reloadData()                                                                           
        self?.memoTableView.reloadData()                                                                            
        }
   }
   ```

   * addObserver메소드는 Observer를 해제할 사용하는 객체(**토큰**)를 return해줌
   * viewDidLoad에서 추가한 Observer는 View가 **화면에서 사라지기 전**에 해제하거나 **소멸자**에서 해제해줌
     * 여기서는 소멸자에서 Observer를 해제함
  

<br>

# Extension으로 더욱 깔끔하게

내가 작성한 코드의 경우에는 Extension파일을 따로 만들어서 다음과 같이 깔끔하고 가독성 좋게 쓸 수 있도록 하였다.

```swift
extension Notification.Name {
    enum Memo {
        static let Insert = Notification.Name(rawValue: "Insert")
    }
}
```
<br>

```swift
NotificationCenter.default.post(name: Notification.Name.Memo.Insert, object: nil)
```

