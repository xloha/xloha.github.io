---
layout: post
title:  "스프링 입문 section 2 - Dependency Injection"
date:   2020-01-26 15:30:00 +0900
categories: spring DI
---
Spring DI의 기본 개념

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>

# 8강(의존성 주입)

**[ OwnerControlle.java ]**

```java
//생성자를 사용해서 주입을 받음
//private final OwnerRepository owners;
//private VisitRepository visits;

//public OwnerController(OwnerRepository clinicService, VisitRepository visits) {
//  this.owners = clinicService;
//  this.visits = visits;
//}

@Autowired
private OwnerRepository owners;
```



## Autowired

*  필드에 사용 가능 

    * 필드로 바로 주입받는 방법

        ```java
        @Autowired                   //final불가
        private OwnerRepository owners;
        ```

* Setter에 사용가능

  * spring IoC Container가 `class 인스턴스`를 만들고 나서 Setter를 통해서 IoC컨테이너에 들어있는 Bean중에 `OwnerRepository` 타입을 찾아서 넣어줌

    ```java
    private OwnerRepository owners;
        
    @Autowired
    public void setOwners(OwnerRepository owners) {
        this.owners = owners;
    }
    ```

*  생성자에 사용가능

    ```java
    @Autowired
    public OwnerController(OwnerRepository clinicService, VisitRepository visits) {
        this.owners = clinicService;
        this.visits = visits;
    }
    ```

    * 생성자를 사용해서 주입을 하는 것
    * @Autowired 사용가능
        * spring 4.3부터 어떤 클래스에 생성자가 하나뿐이고 그 생성자로 주입받는 레퍼런스 변수들이 빈으로 등록되어있다면 그 빈을 자동으로 주입해주도록 spring framework가 해줌 (따라서 @Autowired는 생략 가능)


* BEAN으로 등록되지 않은 Class를 DI한다면...

  ```java
  @Autowired
  //BEAN으로 등록되어 있지 않은 SampleRepository를 작성 후 애플리케이션 실행
  private SampleRepository sampleRepository;
  ```

  * OwnerController에 필요한 의존성을 넣어줄 수 없기 때문에 오류가 남
  * 의존성 주입이 만족이 되지 않을 경우 뜨는 에러 : `No qualifying bean of type 'org.springframework.samples.petclinic.owner.sampleRepository' available`

<br/>

## 셋 중에서 어떤 방법을 사용하면 좋을까?

* spring framework reference에서 권장하는 방법

  * **생성자**
  * why? 필수적으로 사용해야하는 reference없이는 **OwnerController인스턴스를 만들지 못하도록 강제**할 수 있기 때문임
    * 이 클래스는 OwnerRepository없이는 제대로 동작할 수 없기 때문에 **클래스 입장에서 OwnerRepository는 반드시 있어야하는 객체** (이를 강제하기 위한 가장 좋은 수단은 생성자를 사용하는 것)
    * `Field injection`, `Setter injection`**은 의존성 주입이 제대로 되지 않아도 OwnerController 인스턴스를 만들 수 있음**

<br/>
  

>  BUT, 순환참조 발생 시 애플리케이션이 뜨지 않는 오류 발생(A가 B참조 , B가 A참조  => 둘 다 생성자 injection을 사용한다면 못만듦)
>
> * Setter injection, Field injection은 일단 인스턴스를 만든 다음에 서로의 인스턴스를 주입해 줄 수 있기 때문에 Circular Dependency(상호 참조)문제 해결 가능
> * 하지만 circular Dependency가 발생하지 않게끔 하는 것이 좋음





[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition