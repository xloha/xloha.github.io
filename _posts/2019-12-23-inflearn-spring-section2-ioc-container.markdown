---
layout: post
title:  "스프링 입문 section 2 - IoC Container란?"
date:   2019-12-23 18:51:55 +0900
categories: spring
---
Spring IoC Container의 기본 개념

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함
  
##   
  
##     
  
# 6강(스프링 IoC Container)

* IoC 컨테이너 : **Application Context**  또는  BeanFactory 둘 중에 사용할 수 있음 

  * **Application Context**가  **BeanFactory를 상속**받기 때문에 같은 일을 한다고 볼 수 있지만 더 다양한 일들을 할 수 있다!

  

* **IoC 컨테이너** (스프링 제공) : 빈을 만들고, 빈들 사이의 의존성을 엮어주고 만들어져 있는 빈들을 제공해주는 일을 함 

  * `OwnerController, OwnerRepository, PetController, PetRepository` 등은 전부 IoC 컨테이너 안에 빈으로 등록이 되어있음.

  * **하지만 모든 클래스가 Bean으로 등록이 되어있지는 않음!**

    * `Owner.java, NamedEntity.java, BaseEntity.java, Pet.java...` 클래스들은 빈으로 등록되어 있지 않음

    * intelli j에서 콩 표시가 보인다면 Bean!

    ![image-20191223163551108](/assets/image/image-20191223163551108.png)

    * annotation 또는, 특정한 interface를 상속한다거나 직접 Bean으로 등록하는 방법 등이 있음  

    **[ annotation 사용 ]**

    ```java
    @Controller
    class OwnerController {
        ...
    }
    ```

    **[ interface 상속 ]**

    ```java
    public interface OwnerRepository extends Repository<Owner, Integer> {
        ...   
    }
    ```

    **[ 직접 Bean으로 등록하는 방법 ]**

    ```java
        @Bean
        public JCacheManagerCustomizer petclinicCacheConfigurationCustomizer() {
            return cm -> {
                cm.createCache("vets", cacheConfiguration());
            };
        }
    ```

    * 객체를 만들어서 return하면 만들어진  `JCacheManagerCustomizer` 타입의 객체가 return이 됨
    * `JCacheManagerCustomizer` 타입의 객체가 스프링 IoC 컨테이너 안에 Bean으로 등록이 됨 

         
##  
    
##  
    
##      
* 만들어서 등록이 된 Bean들은 스프링의 IoC컨테이너가 서로 의존성 주입을 해 줌 

  ```java
  class OwnerController {
  	private OwnerRepository repository;
  	
    public OwnerController(OwnerRepository repository) {
      this.repository = repository;
    }
  }
  ```

  * IoC 컨테이너, 즉 Application Context가 `OwnerRepository` 타입의 빈을 찾아서 넣어주는 것임

* **의존성 주입은 Bean끼리만 가능함** => 스프링 IoC 컨테이너 안에 들어있는 객체들끼리만  의존성 주입을 해 줌,  IoC컨테이너가 관리하는 Bean들을 가져오는 방법을 제공함

  * 스프링 IoC 컨테이너 밖에 있는 객체에 의존성 주입을 해주지는 않음
    
##     

```java
@AutoWired
ApplicationContext applicationContext;

@Test
 public void getBean() {
        applicationContext.getBeanDefinitionNames();
    }
```

* `ApplicationContext` 안에 모든 Bean들이 들어있음 (이 Bean들에 접근할 수 있는 interface를 ApplicationContext가 상속을 받고 있기 때문에 들어 있는 Bean들을 조회할 수 있음)
* `applicationContext.getBeanDefinitionNames();` : 들어있는 모든 Bean들의 이름을 가져올 수 있음
  
##  
  
```java
@Test
    public void getBean() {
        OwnerController bean = applicationContext.getBean(OwnerController.class);
        assertThat(bean).isNotNull(); 
    }
```

* `OwnerController`를 타입으로 꺼내오면 실제 객체가 나온다는 것을 확인할 수 있음 
  
##  
  
##  
  
**[ OwnerController.java ]**

```java
@Controller
class OwnerController {
private final ApplicationContext applicationContext;

    public OwnerController(OwnerRepository clinicService, VisitRepository visits, ApplicationContext applicationContext) {
        this.owners = clinicService;
        this.visits = visits;
        this.applicationContext = applicationContext;
    }

    @GetMapping("/bean")
    @ResponseBody
    public String bean() {
        return "Bean: " + applicationContext.getBean(OwnerRepository.class);
    }
}
```

* 하지만 `ApplicationContext`를 이렇게 사용하는 일은 별로 없음 

* `ApplicationContext`가 가지고 있는 Bean들은 알아서 주입을 해주기 때문에 굳이 `getBean()`으로 가져와서 주입을 해 줄 필요는 없음 
  
##  
  
  
## 

```java
 @GetMapping("/bean")
    @ResponseBody
    public String bean() {
        return "Bean: " + owners;
    }
```

* 이렇게만 해줘도 Bean을 가져올 수 있음



```java
@GetMapping("/bean")
    @ResponseBody
    public String bean() {
        return "Bean: " + applicationContext.getBean(OwnerRepository.class)
            + "\n Owner: " + this.owners;
    }
```

![image-20191223172454777](/assets/image/image-20191223172454777.png)

* 같은 해쉬 값을 보여줌 => 즉, 둘이 **같은 인스턴스**임
* **싱글톤 scope의 객체** : 어떤 인스턴스 하나를 application 전반에서 계속해서 재사용 (매번 새로 만드는 것 **X**)
  *  멀티쓰레드 상황에서 싱글톤 scope을 구현하는 것은 번거로운 일인데,  IoC 컨테이너를 사용하면 소스코드에 특별한 코드를 작성하지 않아도 **IoC컨테이너에 등록되어 있는 Bean을 사용하면 편하게 singleton scope를 사용할 수 있음** (이 역시 IoC 컨테이너를 사용하는 하나의 이유이기도 함)

 

[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition