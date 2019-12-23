---
layout: post
title:  "스프링 입문 section 1 정리안"
date:   2019-12-23 05:57:55 +0900
categories: spring
---
인프런에 있는 [예제로 배우는 스프링 입문(개정판)][inflearn-link] 강의를 보고 정리한 Spring의 기본 개념들


# 3강 (프로젝트 살펴보기)

### pet-clinic 프로젝트를 이용
[PetClinic Github][pet-clinic github]

* 메이븐에 패키지를 실행 : 프로젝트를 빌드해서  패키지 파일을 만듦
* 프로젝트의 타입에 아무것도 지정하지 않으면 기본적으로 jar(자바 아카이브)
* jar파일 실행하는 명령어 

```
java -jar target/*.jar
```



* 항상 자바 파일을 실행하기 전에 메이븐 패키지를 실행해야함

```
./mvnw package
```

- 플러그인이 실행되어야 로컬호스트에서 실행가능 




# 4강 (프로젝트 과제 풀이)

```java
@Query("SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.firstName LIKE %:firstName%")
    @Transactional(readOnly = true)
```

* :firstName 콜론 다음 파라미터는 항상 바뀐다는 것을 알려주는 것이기 때문에 와일드카드를 콜론 앞에 사용해야함 

### AGE만 dao에 추가했을 경우 

#### 에러 

```
user lacks privilege or object not found: OWNER0_.AGE
```

* AGE가 컬럼에 없기 때문에 생기는 오류

#### 해결방법

* `Application.properties` 에 데이터베이스 스키마파일의 정의를 볼 수 있음 

* 스키마 파일에서 데이터베이스를 고쳐주면 됨




--------------------




#### 에러

```
Caused by: org.hsqldb.HsqlException: row column count mismatch
```

* 데이터를 넣을 때 주로 발생하는 에러 (컬럼에 맞게 데이터를 넣지 않아서)

#### 해결방법

* 데이터 INSERT구문을 수정


[inflearn-link]:https://www.inflearn.com/course/spring_revised_edition
[pet-clinic github]: https://github.com/spring-projects/spring-petclinic