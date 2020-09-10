---
layout: post
title:  "(iOS) Delegate Pattern"
date:   2020-09-03 15:12:00 +0900
categories: iOS swift delegatePattern
---
<br>

❓ Delegate Pattern이란?

1. 데이터를 공급하는 역할(대부분 datasource라는 접미어를 가지고 있음)
2. 이벤트를 처리하는 역할

<br>

# Delegate Pattern

* 하나의 객체는 다른 객체를 자신의 대리자로 지정

* 그리고 자신이 제공하는 일부 기능을 대리자가 대신 수행하도록 위임함

  Ex ) 테이블 뷰와 같은 기능... -> 목록 표시하고 상세보기 누르면 상세화면과 같은 다른 화면으로 이동

<br>

## 테이블 뷰는 항목을 선택했을 때 어떤 작업을 실행해야하는지 정확하게 알지 못하고, 공통적인 부분만 뽑아서 일반화 시키는 것도 불가능 -> 즉 이 부분을 대리자가 대신 처리하도록 위임함

<br>
<br>

# Delegate Pattern 개요

* Delegate pattern에는 두개의 객체가 존재함

* **Table View**(기능을 위임하는 테이블 뷰), **Delegate Object**(테이블 뷰의 일부기능을 대신 수행하는 delegate object)

  >  dataSource 접미어나, delegate접미어가 붙어있는 속성에 위임할 객체를 저장하는 방식으로 delegate를 지정함 

* 테이블 뷰는 특정 항목이 선택되었을 때  delegate로 지정된 객체에게 처리를 요청함(Sending Request)

  ​	delegate에 구현되어있는 특별한 메소드를 호출하는 방식으로 요청함(delegate 객체는 table view가 호출 할 메소드를 구현해야 함)

  > 특정 시점에 호출되는 메소드는 **프로토콜**로 선언되어 있음
  >
  > delegate객체는 반드시 프로토콜에 선언되어 있는대로 메소드를 구현해야 함 

* delegate 패턴으로 구현한 메소드는 우리가 직접 호출하는 메소드는 아니고, 객체에서 이벤트가 발생하면 delegate로 지정된 객체에서 이벤트를 처리하는 메소드가 구현되어 있는지 확인 함 

  * 만약 **구현이 되어있다면, 그 메소드를 호출해서 이벤트 처리를 위임함** (즉 table view가 delegate객체에 구현되어 있는 메소드를 호출함)

<br>

# Delegate Pattern을 구현하기

1. 개발자 문서를 통해서 delegate가 필요한지 확인, 구현해야하는 메소드가 선언되어 있는 프로토콜 확인 

2. 인터페이스 빌더에서 두 객체를 연결하거나, 코드를 통해 두 객체를 연결함

   table view처럼 화면에 표시되어 있는 컨트롤들은 대부분 자신이 포함되어 있는 ViewController를 Delegate로 지정함

3. Delegate로 지정한 클래스에서 직접 프로토콜을 구현하거나, Extension으로 프로토콜 구현을 추가함

<br>

# iOS Delegate Pattern에서의 Delegate객체의 역할

1. 데이터를 공급하는 역할(대부분 datasource라는 접미어를 가지고 있음)
2. 이벤트를 처리하는 역할

<br>

# Table View로 Delegate Pattern 알아보기

* 테이블 뷰가 있는 곳의 View Controller를 dataSource로 지정하면 **View Controller**가 Table View에게 **데이터를 공급하는** Delegate객체로 지정됨(datascoure로 지정됨)

* 테이블 뷰가 있는 곳의 View Controller를 delegate로 지정하면 **View Controller**가 Table View에서 발생하는 **이벤트를 대신 처리하는** Delegate객체로 지정됨

```markdown
* 테이블 뷰는 데이터가 필요할 때 마다 View Controller클래스에 구현되어 있는 메소드를 호출 함
* 이벤트를 발생할 때 마다  View Controller클래스에 구현되어 있는 메소드를 호출 함
* table view는 cell이 선택되었다는 것을 delegate에게 알려주고, delegate는 테이블뷰를 대신해서 이벤트를 처리함
```
> 클래스 선언에 직접 프로토콜 구현을 추가해도 되지만, **extension** 추가하면 코드가 좀 더 깔끔해짐


<br>

# ✅ Delegate Pattern의 장점

* 개발 유연성 증가 
  * 원하는 데이터를 원하는 방식으로 출력 가능 
  * 이벤트 처리 또한 아무런 제약없이 자유롭게 구현가능

<br>

