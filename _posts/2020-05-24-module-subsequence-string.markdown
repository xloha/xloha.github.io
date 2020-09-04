---
layout: post
title:  "(모듈) 연속된 가장 긴 문자열"
date:   2020-05-24 18:10:00 +0900
categories: algorithm java
---

<br>

# 연속된 가장 긴 문자열
주어진 문자열에서 가장 길게 연속된 문자열을 찾는 방법으로 알고리즘 문제에서 생각보다 쓸 일이 많다.  

**for문**을 이용하는 방법은 어떠한 문자열이 있더라도 연속된 가장 긴 문자열을 찾을 수 있지만, **split**을 이용하는 방법은 문자열 내에서 뽑아내고 싶은 결과값이 명확할 때만 사용할 수 있다.

## 방법 1) split 이용

```java
String str = "0000111111111011010101010110011111000";

int max = 0;
String[] strA = str.split("0+");
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

