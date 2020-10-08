---
layout: post
title:  "(iOS) UserNotification - 정해진 요일마다 푸시알람 보내기"
date:   2020-10-08 14:10:00 +0900
categories: iOS swift 
tags:
- UserNotification
- UNCalendarNotificationTrigger
---
<br>

[EUNALARMI](https://github.com/EunYeongKim/EUNALARMI)를 만들면서 정해진 요일마다 푸시 알람을 보낼 수 있는 기능을 구현하였고, [여기서](https://velog.io/@wimes/SwiftUI%EC%97%90%EC%84%9C-LocalNotification-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-4sk60gjlbf) 도움을 많이 받았고, 코드를 참고하였다. 

 `UserNotification, UNCalendarNotificationTrigger`를 이용하였다.

* `Notification` 등록
    * 새로운 알람 등록
    * 수정한 알람 등록
* `Notification` 삭제
    * 알람 삭제
    * 알람 수정하기 전 삭제

<br>

❓ `UserNotification, UNUserNotificationCenter, UNCalendarNotificationTrigger`

✅  반복적으로 정해진 요일마다 앱이 foreground, background에 있을 때 푸시알람을 보낼 수 있다!

<br>

# 1. `Notification`을 도와줄 `struct` 작성

```swift
struct AlarmNotification {
    var id: String
    var title: String
    var body: String
    var hour: Int
    var minute: Int
    var weekday: Int?
}
```
* Notification을 구별할 수 있는 id
* 푸시알림에 띄울 title, body
* 푸시알림을 띄울 시간과 요일

<br>

# 2. `Notification`을 도와줄 class 작성

```swift
import UserNotifications

class AlarmNotificationManager {
    var alarmNotifications = [AlarmNotification]()
    
    func requestPermission() -> Void {
        //알림을 보낼 수 있도록 권한을 사용자에게 요청합니다.
        ...
    }
    
    func scheduleAlarm() -> Void {
        //권한이 있을 경우 Notification을 등록할 수 있도록 합니다.
        ...
    }
    
    func addAlarmNotification(alarm: Alarm) -> Void {
        //alarmNotifications에 Notification을 추가합니다.
        ...
    }
    
    func removeAlarmNotification(alarm: Alarm) -> Void {
        //NotificationCenter에 등록되어 있는 notification을 삭제합니다.
        ...
    }
    
    func scheduleAlarmNotifications() -> Void {
        //alarmNotifications의 notification들을 NotificationCenter에 등록합니다.
        ...
    }
}

```

<br>

# 3. `Permission` 얻기

```swift
func requestPermission() -> Void {
    UNUserNotificationCenter
        .current()
        .requestAuthorization(options: [.alert, .alert]) { granted, error in
            if granted == true && error == nil {
                // permission
            }
        }
}
```

<br>

# 4. 권한이 있을 경우에만 `Notification` 등록

```swift
func scheduleAlarm() -> Void {
    UNUserNotificationCenter.current().getNotificationSettings { settings in
        switch settings.authorizationStatus {
        case .notDetermined:
            self.requestPermission()
        case .authorized, .provisional:
            self.scheduleAlarmNotifications()
        default:
            break
        }
    }
}
```

<br>

# 5. 등록할 `Notification` 배열에 저장

```swift
func addAlarmNotification(alarm: Alarm) -> Void {
    let time = alarm.alarmTime.split(separator: ":")
    guard let hour = Int(time[0]), let minute = Int(time[1]) else { return }
    
    if alarm.alarmCycle.count == 0 {
        alarmNotifications.append(AlarmNotification(id: alarm.alarmNotiId, title: alarm.alarmLabel, body: alarm.alarmTimeLabel, hour: hour, minute: minute, weekday: nil))
    } else {
        for day in alarm.alarmCycle {
            alarmNotifications.append(AlarmNotification(id: alarm.alarmNotiId + String(day.rawValue), title: alarm.alarmLabel, body: alarm.alarmTimeLabel, hour: hour, minute: minute, weekday: day.rawValue + 1))
        }
    }
}
```
> 알람에 대한 Notification을 생성하거나 삭제할 때, 고유한 id로 notification을 구별하여 알림을 보내거나, 삭제할 수 있습니다.  
따라서 알람은 고유한 id값을 가지고 있어야하는데, 고유한 id값을 만들기 위해서 알람을 UserDefaults에 저장할 때
`alarmNotiId = Date() //저장한 현재 시각` 을 변수에 저장합니다.  

> 각 알람은 특정 요일마다 반복할 수도 있기 때문에 요일마다 반복할 경우
`NotificationId(저장한 시각) + 요일의 index` 조합하여 고유한 Notification ID를 만들게 됩니다.

<br>

# 6. `NotificationCenter`에 `notification` 등록
```swift
func scheduleAlarmNotifications() -> Void {
    for notification in alarmNotifications {
        let content = UNMutableNotificationContent()
        content.title = notification.title
        content.body = notification.body
        content.sound = UNNotificationSound.default
        
        var date = DateComponents()
        date.hour = notification.hour
        date.minute = notification.minute
        if let day = notification.weekday {
            date.weekday = day
        }
        
        let trigger = UNCalendarNotificationTrigger(dateMatching: date, repeats: true)
        let request = UNNotificationRequest(identifier: notification.id, content: content, trigger: trigger)
        
        UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
    }
}
```

<br>

# 7. `NotificationCenter`의 `notification` 삭제
```swift
func removeAlarmNotification(alarm: Alarm) -> Void {
    let notiIdList = alarm.alarmCycle.map({ (weekday: WeekDay) -> (String) in
        return alarm.alarmNotiId + String(weekday.rawValue)
    })
    
    UNUserNotificationCenter.current().removePendingNotificationRequests(withIdentifiers: notiIdList)
}
```

> Notification을 삭제할 때는 Notification의 id로 삭제할 수 있는데 이때, 삭제하려는 alarm의 alarmNotiId와 요일의 index를 조합한 Notification ID가 담겨있는 배열을 만들어서 Notification을 삭제할 수 있게됩니다.

<br>

# 8. `foreground` 에서 `Notification`을 처리하도록 `delegate`설정 

**[ Appdelegate.swift ]**
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    UNUserNotificationCenter.current().delegate = self
    return true
}

...

extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.alert, .sound])
    }
}
```

<br>

# 9. `Notification` 사용방법
```swift
func setNotification(alarm: Alarm) {
    let manager = AlarmNotificationManager()
    manager.addAlarmNotification(alarm: alarm)
    manager.scheduleAlarm()
}

func deleteNotification(alarm: Alarm) {
    let manager = AlarmNotificationManager()
    manager.removeAlarmNotification(alarm: alarm)
}

===========================================================

//Notification 등록
setNotification(alarm: alarm)

//Notification 삭제
deleteNotification(alarm: alarm)
```

<br>

# 10. 결과
순서대로  `foreground, background, hold` 

<img src="/assets/image/userNotificationForeground.png" alt="foreground" style="zoom: 25%;"/> <img src="/assets/image/userNotificationBackground.png" alt="background" style="zoom: 25%;"/> <img src="/assets/image/userNotificationHold.png" alt="foreground" style="zoom: 25%;"/> 

### 아직 정해진 요일마다 반복되는지 확인은 못해봤다,,,, 연휴가 끝난 다음에 확인 할 수 있겠지,,,?  

