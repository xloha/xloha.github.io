---
layout: post
title:  "(iOS) RIBs - 앱과 비즈니스로직"
date: 2022-01-03 10:40:00 +0900
categories: ios
tags:
- iOS
- RIB
- architecture
---

### ❓ `Composition(합성, 조립)`

### ✅  앱을 구성하고 있는 코드를 로직의 특성에 따라서 분류해보고, 여러 아키텍쳐들은 이 로직을 어디서 처리하는지 파악하면서 아키텍쳐의 전반적인 이해도 높임

# 앱의 로직의 분류

| 외부 디펜던시 | |
| :--- | :--- |
| 데이터 저장 | 메모리캐시, 바이너리, 데이터베이스, 파일 등 |
| 서비스 | 네트워크, 블루투스, 위치 서비스 등|

| 비즈니스 로직 | |
| :--- | :--- |
| 네비게이션 | 화면의 이동 (present, dismiss, push, pop) |
| 코디네이션 | 각종 layer를 조합해 앱이 사용자를 위해 하는 일 |

| UI | |
| :--- | :--- |
| 뷰 | UIView, UIViewController |
| 프레젠테이션 | 이미지, 색상, 폰트 등 UI모델 변환 |


# RIBs, VIPER

* UI : View
* 외부 디펜던시 : Builder, Component
* 코디네이션 : Interactor
* 네비게이션 : Router

Builder : 외부서비스들을 Router나, Interactor나, View가 사용할 수 있게끔 전달하는 역할

## Builder
리블렛 객체들을 생성하는 역할을 함

* `build` 메소드에서 리블렛에 필요한 객체들을 생성함
* `router`를 return하는데, return된 라우터는 부모 리블렛이 사용함

### Component
리블렛의 Component는 리블렛에 필요한 객체들을 담는 바구니

이 바구니는 자식 리블렛에 필요한 것도 담는 바구니임

자식들의 dependency를 부모 컴포넌트가 채택하도록 함 

## Listener
부모 리블렛에게 이벤트를 전달할 때 사용됨

 