---
layout: post
title:  "스프링 입문 section 3 - @AOP 실습"
date:   2020-01-26 18:06:00 +0900
categories: spring @AOP
---
Proxy Pattern을 사용해서 성능측정 AOP구현해보기

[예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리함

<br/>

# 11강 (Spring @AOP 실습)

## 성능을 측정하는 사용자 정의 annotation만들기

### @LogExecutionTime

**[ OwnerController.java ]**

```java
@PostMapping("/owners/new")
@LogExecutionTime
public String processCreationForm(@Valid Owner owner, BindingResult result) {
  ....
}

@GetMapping("/owners/find")
@LogExecutionTime
public String initFindForm(Map<String, Object> model) {
  ...
}
```



**[ LogExecutionTime.java ]**

```java
package org.springframework.samples.petclinic.owner;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)

public @interface LogExecutionTime {
}
```

* `@Target(ElementType.METHOD)` : Annotation을 어디에 쓸 수 있는지 정의	
  * method에 적용
* `@Retention(RetentionPolicy.RUNTIME)` : 어노테이션 정보를 언제까지 유지할 것인가
  * runtime까지 유지
* **Annotation은 아무런 일을 하지 않음 주석과도 같음**! => 이 Annotation을 읽어서 처리하는 무언가가 있어야함
  * Annotation을 읽어서 실제 성능을 측정하는 클래스 필요

**[ LogAspect.java ]**

```java
@Component
@Aspect
public class LogAspect {

    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTIme(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
				
      	//어노테이션이 붙어있는 메소드 실행
        Object proceed = joinPoint.proceed();

        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());
				
      	//실행 후 결과가 있다면 return
        return proceed;
    }
}

```

* `@Component`: BEAN으로 등록하기 위해서
* `@Aspect` : Aspect인 것을 알리기 위해
* `@Around("@annotation(LogExecutionTime)")` : 이 Annotation을 사용한 메소드 안에서 JoinPoint를 파라메타로 받을 수 있음 
  * JoinPoint : `@LogExecutionTime` 이 붙어있는 해당 메소드
  * `@LogExecutionTime` 주변에 이러한 코드를 적용하겠다고 알려줌

<br/>
<br/>

### Spring이 제공해주는 Annotation기반의 AOP를 사용한 @LogExecutionTime

* 내부는 proxy pattern 기반으로 동작함
* `OwnerController.java`는 `Cash.java` 클래스와 같은  Aspect를 적용할 타겟임
* `LogAspect.java`의 경우에는 `CashPerf.java`와 같은 역할



[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition