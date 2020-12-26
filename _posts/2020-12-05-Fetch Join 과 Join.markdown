---
layout: post
title:  "FetchType 쌩까는 JPA 의 만행과 FetchMode"
date:   2020-12-05 16:42:12 +0900
categories: 개발
---

##### n+1 을 공부하면서 다음과 같은 내용을 배웠다.

* 페치타입을 lazy -> eager 로 바꿔도 n+1 이 사라지진 않는다
* n+1 문제의 해결에는 fetch join 을 쓰자

페치 조인은 정확히 무엇이고 일반 조인과는 뭐가 다를까

JPA -> JPQL -> SQL 에서 글로벌 페치 전략을 참고하지 않는다는것과 관련이 있는걸까?

에 대한 궁금증으로 찾아봤다.

# Hibernate 의 Fetch 전략

하이버네이트의 페치 전략에 관한 docs 에서 일부 글을 발췌해온다.

### 

```text
하이버네이트는 어플리케이션에서 연관된 것들을 탐색해야 하는 경우 연관된 객체를 가져오기 위해 페치 전략을 쓴다.
페치 전략은 o/r 매핑 데이터나 특정 hql 또는 criteria 쿼리에 정의할 수 있다.

하이버네이트는 다음과 같은 페치 전략을 정의한다:

Join Fetching : OUTER JOIN 을 사용해서 하이버네이트는 연관된 객체나 컬렉션을 같은 SELECT 절에서 조회해옵니다.

...
하이버네이트는 컬렉션에서는 lazy select 페치를, 일반 객체에서는 lazy proxy 페치를 기본적값으로 사용합니다.
```

### Fetch Join


```sql
FROM Employee emp
JOIN emp.department dep
```

```sql
FROM Employee emp
JOIN FETCH emp.department dep
```

department 를 lazy 하게 설정했을때

##### 첫번째 쿼리에서는 Employee 만 반환된다. department 객체가 쓰고싶다면 추가로 select 쿼리가 한번 더 발생할 것이다

##### 두번째 쿼리에서는 하나의 select 쿼리에 Employee 와 department 객체까지 반환될 것이다.

한번의 select 에서 연관된 객체 전부를 가져오는게 fetch join 이라는것을 알았다.

뭐 하는 일 자체는 eager loading 과 같은거겠지만 jpql -> sql 생성시에는 lazy / eager 종류를 애초에 따지고 있지 않기 때문에 

페치 전략을 eager 로 바꿔도 n+1 이 해결될 수 없다.

그럼 fetch join 을 사용하고 eager 하게 로딩하면 똑같이 n+1 문제가 발생하지 않네.

라고 생각할 수 있지만 이것은 어플리케이션의 쿼리 퍼포먼스 이슈와 관련이 있다.

result set 이 작은 경우는 eager 로 설정해도 상관없지만 

어느 정도 규모가 있는 곳에서는 lazy + fetch join 이 best practice 라고 뽑힌다.


### @FetchMode 와 fetch type 과의 관계

지금까지 eager 니 lazy 니 했던것들은 전부 FetchType 에 해당한다.

> @FetchType : 데이터를 언제 가져올지

> @FetchMode : 데이터베이스에서 어떻게 가져올지

페치모드 중 가장 유명한 두가지가 있다.

##### Join (디폴트) : 1개의 SELECT 에서 Outer Join 을 사용해 연관된 것들 모두 가져온다.
##### Subselect : 2개의 SELECT 를 사용해서 연관된 컬렉션들을 가져온다.

초개가 fetch modem 의 JOIN 을 보고 처음 든 생각은 다음과 같다.

##### 이거 fetch join 이랑 똑같은거 아냐?

하지만 다음과 같은 이유로 FetchMode.JOIN 은 지양한다고 한다.

1. JPQL -> SQL 과정에서 글로벌 페치 전략을 쌩깐다 했었는데 이 FetchMode.JOIN 도 쌩깐다고 한다.

2. 엔티티의 primary id 를 가지고 엔티티를 가져올때만 이 모드가 유용하다

3. JOIN 모드를 명시적으로 쓰면 페치타입은 무시되고 항상 eager 로 작동한다.
-> 페치 타입과의 콜라보를 맛보려면 SUBSELECT 혹은 SELECT 를 써야 한다. 

아직도 2번은 잘 이해가 안되나 더 찾아봐야겠다.






 



