---
layout: post
title:  "스프링 입문 section 3 - Proxy Pattern"
date:   2020-01-26 17:32:00 +0900
categories: spring proxy-pattern
---
Proxy Pattern을 사용해서 Spring AOP구현해보기

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>

# 10강 (Proxy Pattern)

* 다음 그림을 구현해 보자

    ![image-20200126163948759](/assets/image/image-20200126163948759.png)

<br/>

## CreditCard가 지불 방식을 정하는 Payment



**[ Payment.java ]**

```java
package org.springframework.samples.petclinic.proxy;

public interface Payment {
    void pay(int amount);
}
```

<br/>

**[ Cash.java ]**

```java
package org.springframework.samples.petclinic.proxy;

public class Cash implements Payment {

    @Override
    public void pay(int amount) {
        System.out.println(amount + "현금 결제");
    }
}

```

<br/>

**[ CreditCard.java ]**

```java
package org.springframework.samples.petclinic.proxy;

public class CreditCard implements Payment{
    Payment cash = new Cash();

    @Override
    public void pay(int amount) {
        if(amount > 100) {
            System.out.println(amount + " 신용 카드 ");
        } else {
            cash.pay(amount);
        }
    }
}
```

* `CreditCard는 일종의 Proxy`로 CreditCard를 사용하는데 문제가 있다면 Cash로 fallback됨

<br/>

**[ Store.java ]**

```java
package org.springframework.samples.petclinic.proxy;

public class Store {

    Payment payment;

    public Store(Payment payment) {
        this.payment = payment;
    }

    public void buySomething() {
        payment.pay(100);
    }
}
```

* Store입장에서는 `Payment`라는 인터페이스만 쓰지만 CreditCard를 주면 Cash를 쓸지 CreditCard를 쓸지 알아서 판단하는 것임

<br/>


## 성능 측정이 되는 Payment

**[ CashPerf.java ]**

```java
package org.springframework.samples.petclinic.proxy;

import org.springframework.util.StopWatch;

public class CashPerf implements Payment{
    Payment cash = new Cash();

    @Override
    public void pay(int amount) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        cash.pay(amount);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```

* CashPerf라는 프록시는 자동으로 BEAN이 만들어질 때 만들어짐
* 원래는 Cash또는 CreditCard가 BEAN으로 등록이 되어야하는데 성능측정을 하는 CashPerf가 등록이 되서 CashPerf의 로직이 수행됨

<br/>

**[ Store.java ]**

```java
package org.springframework.samples.petclinic.proxy;

public class Store {

    Payment payment;

    public Store(Payment payment) {
        this.payment = payment;
    }

    public void buySomething(int amount) {
        payment.pay(amount);
    }
}
```

<br/>

**[ StoreTest.java ]**

```java
package org.springframework.samples.petclinic.proxy;

import org.junit.jupiter.api.Test;
import org.springframework.util.StopWatch;

import static org.junit.jupiter.api.Assertions.*;

class StoreTest {
    @Test
    public void TestPay() {
        CashPerf cashPerf = new CashPerf();
        Store store = new Store(cashPerf);
        store.buySomething(100);
    }
}
```

* `Store.java`, `Cash.java` ... **클라이언트 코드는 바뀌지 않은 채로** 성능을 측정하는 코드를 넣은 `CashPerf라는 Proxy Class`가 성능 측정을 해줌 

<br/>

## 원래의 Payment의 경우

```java
class StoreTest {
    @Test
    public void TestPay() {
        Cash cash = new Cash();
      	CreditCard creditCard = new CreditCard();
      
        Store store = new Store(cash);
        store.buySomething(100);
    }
}
```

* 원래의 Store는 `Cash` 또는 `CreditCard`를 넣어서 썼지만 `CashPerf`를 사용하도록 코드를 바꾼 것이기 때문에 **기존코드를 건드리지 않고 성능측정을 할 수 있음**

<br/>



```java
public interface OwnerRepository extends Repository<Owner, Integer> {

    @Query("SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%")
    @Transactional(readOnly = true)
    Collection<Owner> findByLastName(@Param("lastName") String lastName);
  
  ...
}
```

* `@Transactional` annotation이 붙어있으면 `OwnerRepository` 객체 타입의 프록시가 새로 만들어짐
* `@Transactional` : 트랜잭션이 앞뒤로 붙어있는 프록시로 다음 코드를 생략할 수 있도록 함
  * SetAutoCommit = false;
  * sql 실행
  * commit or Rollback

<br/>
<br/>


> * test file로 이동 또는 생성 : command + shift + T
>
> * 실행 : control + shift + R 





[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition
