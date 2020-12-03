---
layout: post
title:  "(iOS) iOS빌드 오류 시 DerivedData"
date: 2020-12-03 15:40:00 +0900
categories: iOS swift 
tags:
- DerivedData
---

Xcode 11.4버전, 12.0버전을 쓰고 있는데 같은 소스를 다른 Xcode에서 빌드하면 빌드 오류가 발생한다.

이를 해결하기 위해서는 `DerivedData`에 있는 폴더를 지워줘야한다.

<br>

❓ `DerivedData`

✅  `/Users/a60080252/Library/Developer/Xcode/DerivedData`