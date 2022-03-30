---
layout: post
title:  "(iOS) Swift 함수형 프로그래밍"
date: 2021-12-29 16:40:00 +0900
categories: iOS
tags:
- iOS
- Swift
- 함수형 프로그래밍
---
### ❓ `Swift 함수형 프로그래밍`

### ✅  OOP(객체지향), POP(프로토콜지향), FP(함수형) 을 지원하는 멀티패러다임 언어

# Functional Programming

### 함수를 중심으로 사이드 이펙트가 없도록 프로그래밍 함

> ⭐️참고한 글 들
> 
> [Swift Functional Programming](https://hyunndyblog.tistory.com/163)

|Programing Techinque (프로그래밍 기법)| Functional Programming(함수형 프로그래밍) |
| :--- | :--- |
| Immutable Data |  Side-Effect Free Programming (using programming techinque) |
| High Order Function(고차함수) |  |
| Currying |  |
| Map, Filter, Reduce |  |

* 단순히 이 프로그래밍 기법들을 사용해서 프로그래밍 하는것이 아님

- 데이터를 조작하지 않고, 한번 만들어진 데이터는 변경하지 않고, 변경된 데이터가 필요할 떄는 새로운 데이터를 만들어낸다.
- 데이터를 조작하지 않으면 같은 작업을 수행해도 원본 데이터에 영향을 끼치지 않으므로 사이드 이펙트가 발생하지 않음

<br>

# 순수함수

### 순수함수 : 특정 input에 대해서 항상 동일한 output을 반환하는 함수
> output을 만드는데 input만 사용한다는 의미로, 함수 외부의 값을 사용하지 않아 사이드이펙트가 없음

* output은 input에 의해서만 결정됨
* 함수의 수행과정에서 외부에 있는 값을 사용하지 않음
* 외부의값을 변경하지도 않음
* 외부에 영향을 주지도, 받지도 않으므로 side-effect가 발생하지 않음

> ```swift
> lef offset: CGFloat = 10.0
> function getOffset(curHeight: CGFloat) {
>     return offset + curHeight
> }
> ```
> * 결과를 만드는데 외부값인 offset 사용(let, immutable)
>     * immutable data만 사용하는 함수도 특정 input에 대해서 항상 동일한 output을 내기때문에 순수함수

<br>

# 1급 객체

함수형 프로그래밍에서한 함수를 1급 객체로 취급함

### 1급객체 : 프로그래밍 언어에서 함수의 파라미터로 전달되거나 리턴값으로 사용될 수 있는 객체

### 고차함수 : 함수를 파라미터로 받거나 함수를 리턴하는 함수

함수형 프로그래밍에서는 함수를 1급객체로 사용하므로 당연히 `함수의 합성`도 가능함

<br>

# 함수의 합성

### 함수의 반환값이 다른 함수의 입력값으로 사용되는 것
* 당연히 함수의 반환값과, 이것을 입력으로 받아들이는 타입이 같아야함

> 함수의 합성 예시
> 
> ```swift
> func f1(_ num: Int) -> Int {
>     return num + 3
> }
> 
> func f2(_ i: Int) -> String {
>     return "\(i) 빼기 3은 f1의 파라미터"
> }
> 
> /// 변수에 함수를 할당할 수 있는 고차함수적인 부분
> let result: String = f2(f1(10))
> ```
> 복잡한 함수의 합성
> 
> ```swift
> func ff(_ pf1: @escaping (Int) -> Int, _ pf2: @escaping (Int) -> String) -> (Int) -> (String) {
>   return { (s1: Int) in 
>       return pf2(pf1(s1))
>   } 
> }
> 
> let f3 = ff(f1, f2)
> let result = f3(100)
> ```
> * f3에 ff의 반환값인 (Int) -> String 할당

<br>

# 커링(Currying)
### 여러개의 파라미터를 받는 함수를 하나의 파라미터를 받는 여러개의 함수로 쪼개는 것
* 함수의 합성을 원활하게 하기 위함

> 함수의 output이 다른 함수의 input으로 연결되면서 합성(Composition)이 되는데, 함수들이 서로 chain을 이루면서 연속적으로 연결이 되려면 output과 input타입의 갯수가 같아야함
> 함수의 output은 하나밖에 없으니, input도 모두 하나씩만 갖도록 한다면, 합성이 더 쉬워질 것

커링 전
```swift
func multiply(_ s1: Int, s2: Int) -> Int {
    return s1 * s2
}
```

커링 후
```swift
/// 파라미터 분리
func f1(_ s1: Int) -> Int
func f2(_ s2: Int) -> Int

func multiply(_ s1: Int) -> (Int) -> Int {
    return { s2: in
        return s1 * s2
    }
}

let result = multiply(10)(20)
```

커링 전
```swift
func filterSum(_ arr: [Int], _ n: Int) -> Int {
    return arr.filter({ $0 % n == 0}).reduce(0, +)
}
```

커링 후
```swift
func filterSum2(_ n: Int) -> [Int] -> Int {
    return { arr in
        return arr.filter{ $0 % n == 0}.reduce(0, +)
    }
}

func solution(_ nums: [Int], _ r: Int) -> Int {
    let filterResult = filterSum2(r)

    return result = filterResult(nums)
}
```