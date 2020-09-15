---
layout: post
title:  "(iOS) User Defaults"
date:   2020-09-15 17:00:00 +0900
categories: iOS swift 
tags:
- User Defaults
---
<br>

# User Defaults

* 설정 데이터와 같은 작은 데이터를 저장할 때 사용하는 로컬 저장소

* Data는 키와 값의 쌍으로 저장함 **(Key : Value)**

* 메모리에 임시적으로 저장되는 값이 아니라 **Sandbox내에 물리적인 파일로 저장됨** (이 파일을 **Defaults Database**라고 부르기도 함)

* 고유한 format을 가지고 있기 때문에 파일을 직접 수정할 수 없고, 반드시 User Defaults 파일을 사용해야 함

  > 개발자 문서에 최대 4GB까지 저장할 수 있다고 나와있지만 KB이상의 데이터를 저장하기에는 적합하지 않음
  >
  > 이미지처럼 상대적으로 큰 데이터를 저장하는 용도로 사용하면 안됨



* 스마트폰 로컬환경에서 실행되는 작업중에서 가장 느린 작업은 디스크에서 파일을 로딩하는 것
  * 값을 읽고 저장할 때 마다 디스크에 접근한다면 성능 저하 발생
* User Defaults는 전체 데이터를 메모리에 **캐쉬**로 저장함
  * 메모리에 저장된 데이터와 파일로 저장된 데이터는 자동으로 동기화되기 때문에 우리는 어디에 있는지 신경 쓸 필요X
  * PC로 인해서 디스크 접근으로 인한 성능이슈는 발생하지 않지만 전체 데이터를 메모리에 저장하기 때문에 저장된 데이터가 크다면 메모리 문제가 크게 발생함
    *  1000개의 데이터가 저장되어있고, 하나의 데이터만 읽어오더라도 나머지 필요없는 999개의 데이터도 메모리에 저장됨
    * 이런 특징 때문에 소량의 작은 데이터를 저장하는 용도로 사용해야함!
* (Bool, String, Data, Array, Dictionary, Date, URL)과 같은 형식들은 별도의 변환없이 바로 저장 가능
  * 나머지 형식은 NSCoding Protocol을 채용하고 NSKidArchiver로 Archiving한다음 데이터 형식으로 저장

<br>

# User Defaults에 저장

* UserDefault에 값을 저장할 때는 같은 이름을 가진 class를 사용함

* 새로운 instance를 생성하고 UserDefault조작에 사용할 수 있지만 보통 스탠다드 속성이 return하는 singletone 인스턴스를 사용함

  * key와 연관된 값이 저장되어 있다면 값을 교체하고, 그렇지 않다면 새로운 값으로 추가함

  * key는 오타가 날 가능성이 높기 때문에 속성이나 전역상수로 선언하고 사용하는 것이 좋음

```swift
let key = "sampleKey"
UserDefaults.standard.set(12.34, forKey: key)
```

* User Default를 사용할때는 올바른 형식으로 값을 읽는 것이 중요함
  * 키 이름을 지을때는 값의 형식을 유추할 수 있는 이름을 사용하면 문제를 줄이는데 도움이 됨

<br>

# User Defaults에서 삭제

```swift
//1. nil로 설정
UserDefaults.standard.set(nil, forKey: key)
// 2. removeObject()
UserDefaults.standard.removeObject(forKey: key)
```

<br>

# 설정 데이터 저장

* User Defaults는 간단한 설정 데이터를 저장할 때 사용함

  * 보통 설정 데이터들은 기본값을 가지고 있는데 기본값은 앱을 설치한 다음 처음시작할 때 한번만 설정해야 함
  * 그렇지 않으면 매번 앱을 실행할 때마다 기본값으로 초기화되어버림

<br>

**[  AppDelegate.swift ]**

```swift
import UIKit

//GlobalScope로 변수 선언
// 가상의 키
let thresholdKey = "thresholdkey"
// 최초 실행시점을 판단하는데 사용
let initialLanchKey = "initialLanchKey"

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

   var window: UIWindow?

   func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
      
    // 앱 설치 후 한번만 실행되어야할 때 활용되는 패턴
    if !UserDefaults.standard.bool(forKey: initialLanchKey) {
        let defaultSettings = [thresholdKey: 123] as [String: Any]
        // 다수의 기본값을 한번에 등록
        UserDefaults.standard.register(defaults: defaultSettings)
        UserDefaults.standard.set(true, forKey: initialLanchKey)
        
        print("Initial Lanch")
    }
      
      return true
   }
}
```

* `bool(forKey: ) `: 키에 해당하는 값이 없을 때 false를 return

<br>

# UserDefaults 값이 변경되었을 경우

**[ UserDefaultsViewController.swift ]**

```swift
var token: NSObjectProtocol?
    
    deinit {
        if let token = token {
            NotificationCenter.default.removeObserver(token)
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
      
        token = NotificationCenter.default.addObserver(forName: UserDefaults.didChangeNotification, object: nil, queue: OperationQueue.main, using: { [weak self] (noti) in
            self?.updateDateLabel()
        })
    }
```

* `UserDefaults.didChangeNotification` : User Default가 업데이트 되었다는 것만 알려줌
  * 만약 특정 키의 저장되어 있는 값이 업데이트 되었는지 확인하고 싶다면 이전 값을 어딘가에 저장해두었다가 Notification이 전달되는 시점에 직접 비교하는 방식으로 구현함