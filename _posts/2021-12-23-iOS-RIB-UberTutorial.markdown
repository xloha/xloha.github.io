---
layout: post
title:  "(iOS) RIBs - Uber tutorial"
date: 2021-12-23 16:40:00 +0900
categories: iOS
tags:
- iOS
- RIB
- architecture
---

# Dynamic Dependencies vs Static Dependencies

### Dependency
부모RIB에서 만들 때 넣어줘야하는 *의존성들을 나열한 프로토콜*

### Component
구체적인 구현체
> Addition
> 자기 자신과 하위 RIB의 종속성을 소유하는 책임을 가짐
> 
> **LoggedInComponent에서 하위 RIB인 OffGame, TicTacToe의 의존성을 채택해서 소유**
> ```swift
> final class LoggedInComponent: Component<LoggedInDependency>, OffGameDependency, TicTacToeDependency
> ```
> * 의존성에 포함되어있는 것들은 주로 DI Tree 아래로 전달될 어떤 상태를 포함하고 있음
> * 비용이나 성능상의 이유로 RIB간에 공유됨


---

