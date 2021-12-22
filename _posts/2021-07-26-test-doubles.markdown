---
layout: post
title:  "Test Doubles"
date: 2021-07-27 13:40:00 +0900
categories: iOS test
tags:
- iOS
- test
---

### ❓ `Test Doubles`

### ✅  실제 객체를 대신해서 테스팅에 사용하는 모든 방법을 일컬어 호칭
실제 객체를 사용하기 어렵고 모호한 때 대신해 줄 수 있는 객체를 만들어 테스트를 수행하도록 도움

    틀린부분이 있을 수 있댬,,,,,,예제는 제가 맘대로 적었기 때문,,

> ⭐️참고한 글 들
> 
> [Test Double](https://brunch.co.kr/@tilltue/55)
> 

<br>
<br>

# `Dummy`

### 객체는 전달되지만 사용되지 않는 객체
Ex) 함수 파라미터에 전달되는 빈 객체

<br>

# `Fake`

### 동작하는 구현을 가지고 있지만 실제 프로덕션에는 적합하지 않은(동일하지 않은) 객체

단순화된 버전의 동작을 제공하기도 함

Ex) 인메모리 데이터베이스


<br>

# `Stub`

### 테스트에서 호출 된 요청에 대해 ***미리 준비해 둔 결과를 제공***
테스트를 위해 프로그래밍 된 내용 이외에는 응답하지 않음

> 모든기능 대신 일부기능(테스트하고자 하는 기능)에 집중하여 임의로 구현

```swift
class Request {
    let query: String
}

class Response {
    let movies: [String]
}

class MovieListRepositoryStub {
    private let movies = Response(movies: ["라이온킹", "백설공주", "인어공주", "알라딘"])

    func fetchMovieList(request: Request) -> Response {
        return self.movies
    }
}
```
<br>

# `Spy`

### `Stub`의 역할을 가지면서 호출된 내용에 대해서 약간의 ***정보를 기록***
테스트에서 확인하기 위한 정보를 기록

Ex) 메일링 서비스에서 발송된 메일의 갯수를 기록

```swift
class Request {
    let query: String
}

class Response {
    let movies: [String]
}

class MovieListRepositoryStub {
    private let movies = Response(movies: ["라이온킹", "백설공주", "인어공주", "알라딘"])
    var isCalled = false

    func fetchMovieList(request: Request) -> Response {
        self.isCalled = true
        return self.movies
    }
}

/// MovieListRepositoryStub 테스트
func test_MovieListRepository {
    let stub = MovieListRepositoryStub()

    stub.fetchMovieList(request: Request())

    XCTAssertEqual(stub.isCalled, true)
}

```

<br>

# `Mock`

### 호출에 대한 기대를 명세하고, 해당 내용에 따라 동작하도록 프로그래밍 된 객체
테스트에서 호출 시 동작이 잘 되었는지 확인하는데 사용

> ### `Spy`
> 확인해야할 매개변수가 많아지면 test를 읽기 어려워지고, `isCalled`와 같은 변수가 많이 생성될 것임
> 
> ➡️ 
> 
> ### `Mock`
> 값을 내부적으로 확인하기 때문에 test를 clean하게 작성할 수 있도록 도와줌

```swift
class Request {
    let query: String
}

class Response {
    let movies: [String]
}

class MovieListRepositoryMock {
    private let movies = Response(movies: ["라이온킹", "백설공주", "인어공주", "알라딘"])
    private var isCalled = false

    func fetchMovieList(request: Request) -> Response {
        return self.movies
    }

    func verify(isCalled: Bool) {
        XCTAssertEqual(self.isCalled, isCalled)
    }
}

/// MovieListRepositoryMock 테스트
func test_MovieListRepository {
    let sut = MovieListRepositoryMock()

    sut.fetchMovieList(request: Request())
    sut.verify(isCalled: true)
}

```
<br>

`Mock`을 사용하면 구현부와 결합되는 부분이 많아짐 (메소드 명과, 인자값이 어떻게 전달되어야할지 지정)
구현부와 결합되어있을 때 리팩토링시 방해요소가 됨

Ex) 함수명 인자명 변경 시, 인자 순서 변경 시 ➡️ 테스트 수정 필요

<br>

> ### `sut` (`system under test`)
>
> 테스트의 대상 (테스트 하고자 하는 클래스, 메소드 등을 선언할 때 사용)

<br>