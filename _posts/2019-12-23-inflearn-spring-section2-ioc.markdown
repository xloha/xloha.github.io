---
layout: post
title:  "스프링 입문 section 2 - IoC란?"
date:   2019-12-23 18:18:55 +0900
categories: spring IoC
---
Spring IoC의 기본 개념

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>


# 5강(스프링 IoC)

## Inversion of Control : 제어권의 역전 
의존성을 관리하는 일은 스프링 프레임워크(IoC Container)가 해주는 것

### 기존 방법
* 자기가 사용할 의존성을 자기가 만들어서 관리함
* `OwnerRepository`를 코드로 직접 생성하며 의존성을 주입하고 있음

```java
class OwnerController {
	private OwnerRepository repository = new OwnerRepository();
}
```



### IoC 사용

```java
class OwnerController {
	private OwnerRepository repository;
	
  public OwnerController(OwnerRepository repository) {
    this.repository = repository;
  }
	
}
```

* OwnerRepository를 사용을 하지만 **자신이 만들지는 않음** 
  * 누군가가 OwnerController밖에서 누군가가 줄 수 있게끔 
  * 생성자를 통해 받아옴 
* 의존성을 만드는 일은 더 이상 OwnerController 가 하는 것이 아님 (의존성을 관리하는 일은 누군가 밖에서 해주는 것임) => **제어권이 역전되었다고 볼 수 있음** 
* **의존성을 주입해주는 일 : Dependency injection** (dependency injection 또한 IoC라고 볼 수 있음)
  * why? : 의존성을 관리하는 일 자체, 그 제어권을 inversion했기 때문에 (의존성을 관리하는 일이 외부의 누군가로 바꼈기 때문)

<br/>

```java
@Controller
class OwnerController {
    private final OwnerRepository owners;

  public OwnerController(OwnerRepository clinicService) {
          this.owners = clinicService;
      }


   @PostMapping("/owners/new")
      public String processCreationForm(@Valid Owner owner, BindingResult result) {
          if (result.hasErrors()) {
              return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
          } else {
              this.owners.save(owner);
              return "redirect:/owners/" + owner.getId();
          }
      }
}
```

* `this.owners.save(owner);` : this.owners에 아무것도 설정이 되어 있지 않으면 NullPointerException 오류가 일어남

* 하지만 NullPointerException이 발생할 수 없음!

  * why? **생성자가 하나뿐이기 때문에 OwnerController 인스턴스를 만들려면 반드시 OwnerRepository를 만들어서 넣어줄 수 밖에 없음** 
  * 넣어주지 않는 이상 OwnerController를 만들 수 없음 => 이렇게 만들어진 OwnerController는 무조건 OwnerRepository가 있는 상태임

* OwnerController를 보면 OwnerRepository없이는 OwnerController를 못 만들게 되어있음

  * 따라서 이 안에서 사용하는 모든 코드들은 전부 다 안전하게 사용가능

<br/>


* OwnerRepository는 누가 넣어줄까?

  **[ test > OwnerControllerTests.java ]**

  ```java
  @MockBean
  private OwnerRepository owners;
  ```

  * **@MockBean** : Mock객체로 만들어줘서 bean으로 만들어주는 annotation을 사용 
    * `OwnerRepository` 타입의 인스턴스를 스프링이  OwnerControllerTests.java를 만들 때 자동으로 만들어 줌
    * 만들어진 **owners 객체가 자동으로 bean으로 등록**이 됨 


  * **Bean** : 스프링이 관리하는 객체
    * 스프링이 관리하는 Bean으로 등록이 되면 OwnerController를 만들 때 `OwnerRepository` 타입의 빈을 가져와서 주입을 해줌 
    * who? 스프링이 해줌. 스프링에 있는 IoC Controller가 이 일을 해 줌
  * **Spring** : Bean들의 의존성을 관리하고, 객체를 만들어서 Bean으로 등록을 해줌. 그렇게 만들어진 객체들을  스프링 컨테이너 안에 있기 때문에 Bean이라고 부르는 것이고, 그 Bean들의 의존성을 관리해줌.
    * 의존성 관리: 필요한 의존성을 서로 주입을 해준다는 것임(특정한 생성자나, annotation을 보고)




[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition