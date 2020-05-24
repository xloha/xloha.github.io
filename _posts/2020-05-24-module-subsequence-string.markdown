---
layout: post
title:  "(모듈) 연속된 가장 긴 문자열"
date:   2020-05-24 18:10:00 +0900
categories: java algorithm 
---

<br>

# 연속된 가장 긴 문자열

## 방법 1) split 이용

```java
		String str = "0000111111111011010101010110011111000";

		int max = 0;
		String[] strA = str.split("0");
		for(int i = 0; i < strA.length; i++) {
			if(strA[i].contains("1")) {
				max = Math.max(max, strA[i].length());
			}
		}
		System.out.println(max);
```

<br>

## 방법 2) for문 

```java
		String str = "0000111111111011010101010110011111000";
		
		int max = 0;
		for(int i = 0; i < str.length(); i++) {
			if(str.charAt(i) == '0') continue;
			char cur = str.charAt(i);
			int total = 1;
			for(int j = i+1; j < str.length(); j++) {
				if(cur != str.charAt(j)) {
					i = j;
					break;
				}
				total++;
			}
			max = Math.max(max, total);
		}
		System.out.println(max);
```

