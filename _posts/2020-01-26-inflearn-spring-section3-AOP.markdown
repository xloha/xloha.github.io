---
layout: post
title:  "스프링 입문 section 3 - Spring AOP"
date:   2020-01-26 16:31:00 +0900
categories: spring
---
Spring AOP의 기본 개념

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>

# 9강 (Spring AOP)

* **AOP (Aspect Oriented Programing)** : 관점 지향적인 프로그래밍

```java
class A {
  method a () {
    AAA
    하이루 하이루
    BBB
  }
  
  method b () {
    AAA
    방가 방가
    BBB
  }
  
  method c () {
    AAA
    안녕 안녕
    BBB
  }
}
```

* 똑같은 코드인데 여러 곳에 흩어져 있다면 AAA를 AAAA로 바꾼다고 했을 때 모든 코드를 바꿔야하는 문제 발생
* 공통적인 코드를 따로 모으면 안될까?!

```java
class A {
  method a () {
    하이루 하이루
  }
  
  method b () {
    방가 방가
  }
  
  method c () {
    안녕 안녕
  }
}

class AAABBB{
  method aaabbb(JointPoint point) {
    AAA
    point.execute();
    BBB
  }
}
```

* AAA랑 BBB를 별도의 클래스의 별도의 메소드로 빼내서 사용하자!

<br/>

### PetClinic 프로젝트에서의 AOP

```java
// [ OwnerRepository.java ]
@Query("SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%")
@Transactional(readOnly = true)
    Collection<Owner> findByLastName(@Param("lastName") String lastName);


// [ PetRepository.java ]
@Query("SELECT ptype FROM PetType ptype ORDER BY ptype.name")
@Transactional(readOnly = true)
    List<PetType> findPetTypes();
```

* `@Transactional` : Spring AOP기반으로 만들어져 있는 annotation
*  AOP : 코드가 없는데도 코드가 있는 것처럼 동작하도록 하는 것 

<br/>

### 다양한 AOP 구현 방법

1. 컴파일 (AspectJ)
   * 원래 자바 컴파일 : A.java --------> A.class
   * AOP구현 방법 : A.java ----(AOP)----> A.class
     * 컴파일 중간에 특별한 로직을 끼워넣음
     * 원래 작성한 자바 파일에는 특별한 코드가 없었지만 컴파일을 한 코드에는 그 코드가 들어있게 끔

2. 바이트 코드 조작
   *  A.java --------> A.class
   * AOP구현 방법 :  A.java --------> A.class ----(AOP)----> 메모리
     * 런타임에서 A.class를 사용할 때 ClassLoader가 이 A.class를 읽어와서 메모리에 올릴 때 조작 
     * 원래 Class파일에 아무것도 없는데 클래스를 로딩하는 그 시점 메모리 상에 있는 그 메소드에는 특별한 로직이 들어가 있음
3. **프록시 패턴** (스프링AOP가 사용하는 방법)
   * 디자인 패턴 중 하나( [프록시 패턴][proxy-pattern] )를 사용해서 AOP와 같은 효과를 내는 방법
   * 기존의 코드를 건드리지 않고 새로운 기능을 넣는 것 (클라이언트 코드에 영향을 적게 주면서 그 객체를 다른 객체로 바꾸는 방법)



> 자바 컴파일 과정
>
> ![image-20200126161943326](/assets/image/image-20200126161943326.png)
>
> * 자바 코드를 컴파일러(javac)가 기계어인 자바 바이트 코드로 변환되고, 변환된 코드를 인터프리터가 한줄씩 실행시키면서 애플리케이션을 실행하게 됨
> * 자바 바이트 코드는 클래스 로더에 의해 JVM내로 로드 되고, 메모리상에 배치됨



[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition

[proxy-pattern]:https://refactoring.guru/design-patterns/proxy