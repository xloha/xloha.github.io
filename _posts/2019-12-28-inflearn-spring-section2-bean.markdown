---
layout: post
title:  "스프링 입문 section 2 - Spring Bean"
date:   2019-12-28 18:05:55 +0900
categories: spring
---
Spring bean의 기본 개념

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>

# 7강(스프링 빈)

```java
OwnerController ownerController = new OwnerController();
//BEAN이 아님
//일반 객체

OwnerController bean = applicationContext.getBean(OwnerController.class)
//BEAN임 
//why? application context가 관리하는 객체에서 가져온 것이기 때문
```

* Application context가 알고있는 객체를 Bean이라고 함 
* 오로지 Bean들만 의존성 주입이 됨  


<br/>

----------------------------------

<br/>

* `@Controller` : `@Component` 애노테이션을 사용한 메타 애노테이션

* **lifecycle callback** : IoC Container를 만들고, 그 안에 Bean을 등록할 때 사용하는 여러가지 인터페이스들

  * **lifecycle callback**에 포함된 인터페이스들은 `@Component` 를 찾아서 그 클래스의 인스턴스를 만들어서 Bean으로 등록하는 Annotation Process (애노테이션 처리기)가 등록되어있음.    

<br/>

* `@SpringBootApplication` 애노테이션 안에 `@ComponentScan`라는 애노테이션이 어느 지점부터 컴포넌트를 찾아보라고 알려줌

  *  `@ComponentScan` 에 지정되어 있는 범위 안에서, 다음과 같은 애노테이션이 붙어있는 모든 클래스들을 찾아서 Bean으로 등록을 해줌(`@ComponentScan` 이 Bean으로 등록을 해주지는 않음)

    * `@Controller`
    * `@Configuration`
    * `@Service`
    * `@Repository` : 스프링 데이터 JPA가 제공해주는 기능에 의해서 Bean으로 등록이 됨

      * 특정한 Annotation이 없더라도 특정한 interface를 상속받은 경우에 interface를 상속받고 있는 클래스를 찾아서 그 클래스의 interface의 구현체를 내부적으로 만들어서 bean으로 등록을 해줌

      ```java
      public interface OwnerRepository extends Repository<Owner, Integer> {
        ...
      }
      ```
  
  
<br/>   

### BEAN으로 만드는 방법

1. `@ComponentScan`
2. 직접 등록하는 방법

<br/>

### 1. Sample Controller를 `@ComponentScan`을 사용해서 Bean으로 등록

**[ SampleController.java ]**

```java
package org.springframework.samples.petclinic.sample;

import org.springframework.stereotype.Controller;

@Controller
public class SampleController {
  
}
```

**[ SampleControllerTest.java ]**

```java
package org.springframework.samples.petclinic.sample;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;

//스프링 BOOT 기반으로 테스트를 작성하는 방법
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleControllerTest {
    @Autowired
    ApplicationContext applicationContext;

    @Test
    public void TestDI() {
        SampleController bean = applicationContext.getBean(SampleController.class);
        assertThat(bean).isNotNull();
    }
}
```

* Application Context에 Sample Controller가 Bean으로 등록이 되어있는지 확인하는 테스트 코드

<br/>

### 2. 직접 등록하기

* Bean 설정 파일
    * XML
    * **자바 설정 파일**

<br/>
     
   **[ SampleController.java ]**

   ```java
   package org.springframework.samples.petclinic.sample;
   
   import org.springframework.stereotype.Controller;
   //@Controller
   //에노테이션을 쓰지 않아도 됨
   public class SampleController {
   }
   ```

   **[ SampleConfig.java ]**

   ```java
   package org.springframework.samples.petclinic.sample;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class SampleConfig {
       @Bean
       public SampleController sampleController() {
         //메소드에서 리턴하는 객체자체가 IoC 컨테이너 안에 Bean으로 등록이 됨
           return new SampleController();
       }
   }
   ```



### 빈을 꺼내는 방법

1. `@Autowired` 를 사용하기

```java
@Controller
class OwnerController {

    private static final String VIEWS_OWNER_CREATE_OR_UPDATE_FORM = "owners/createOrUpdateOwnerForm";
  	@Autowired
    private OwnerRepository owners;
  	@Autowired
    private VisitRepository visits;

// 생성자는 없어도 됨
//    public OwnerController(OwnerRepository clinicService, VisitRepository visits) {
//        this.owners = clinicService;
//        this.visits = visits;
//   }
...
}
```

2. ApplicationContext에서 직접 꺼내서 사용하기 ( 권장X )

```java
OwnerController bean = applicationContext.getBean(OwnerController.class);
```



[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition