---
layout: post
title:  "(iOS) iOS라이브러리 교체 방법"
date: 2020-12-14 13:40:00 +0900
categories: iOS swift 
tags:
- library
---

iOS 기존 라이브러리 교체하는 방법이다!

<br>

✅ `라이브러리 Path 확인`

# 1. XCode 페이지 트리에서 경로 확인
<img src="/assets/image/라이브러리교체1.png" style="zoom: 80%;"/>

교체하려는 라이브러리 선택 후 `Show in Finder`
<br>

# 2. 파일 경로에서 라이브러리 교체
<img src="/assets/image/라이브러리교체2.png" style="zoom: 60%;"/>

파일 경로에서 교체하려는 라이브러리로 파일 교체
<br>

# 3. 경로 잡기
<img src="/assets/image/라이브러리교체3.png" style="zoom: 80%;"/>

올바른 경로로 잡기 위해서 파인더에서 `교체한 라이브러리를 다시 XCode의 페이지 트리의 알맞은 위치로 끌어옴`
<br>

# 4. 경로 확인
<img src="/assets/image/라이브러리교체4.png" style="zoom: 80%;"/>

파일을 선택하고 `File Inspector`에서 올바른 경로로 잡혔는지 확인하여 다른 컴퓨터, 노트북에서 빌드할 수 있도록 한다.
<br>