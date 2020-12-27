---
layout: post
title:  "Covering Index"
date:   2020-12-25 23:42:12 +0900
categories: 개발
---
# 쿼리 최적화에 필요한 인덱스
초개가 회사에서 처음 일을 시작할 때 가장 어려워했던것, 가장 많이 지적받았던 부분 그리고 지금도 어려워하고 있는 부분이 바로 인덱스를 만들고 쿼리를 짜는것이다.

~~(JPA 를 통한 추상화의 기쁨은 잠깐이었다)~~

인덱스는 만들면 해당 인덱스가 적용된 컬럼 정보와 주소등을 가진 인덱스 파일이 생긴다.

엄청나게 많은 데이터베이스의 로우를 내가 원하는 조건에 맞춰 다 훑어볼 수는 없는것 아닌가 (Full Scan ㅇㅅㅇ...)

그래서 인덱스 파일에서 내가 원하는 범위를 쉽게 찾을 수 있게 도와준다. (Range Scan)

요것이 어떻게 저장되어있냐?

B-Tree 라는 트리를 사용해서 저장된다. B-Tree 는 맨 끝에 리프노드를 가지고 있고 각 리프노트의 depth 는 동일하다.

항상 정렬되어있기 때문에 오름차순 또는 내림차순으로 찾으면 된다.

대충 어떤 특징이 있는가?

* insert, update, delete 문에서 성능이 떨어지고 select 절에서 효력을 발휘한다.

당근 새로운 레코드가 생기면 인덱스 파일도 수정해야되니까...

검색이 많은곳에서 진정한 힘을 발휘한다.

* 분포도가 고른 컬럼에 인덱스를 써야 좋다.

남자/여자 라던가 예/아니오 등 해당 컬럼에 쓸 수 있는 값의 종류가 별로 많지 않은 곳은 좋지 않다.

* 다중컬럼 인덱스를 잘 쓰면 좋다

단일 인덱스를 여러개 만들바에는 다중컬럼 인덱스로 포함시키는게 낫다.


# Covering Index

쿼리의 조건을 충족하는데 필요한 모든 데이터를 오직 인덱스에서 추출할 수 있는 경우 커버링 인덱스라고 부른다

사용자가 `#나에게주는선물#힐링` 따위를 쓸 수 있도록 해시태그 테이블을 만들어보자.
  
```mysql
create table Hashtag
(
	id bigint auto_increment primary key,
	name varchar(150) null,
	reviewStatus varchar(20) charset utf8 not null,
	dateCreated datetime not null
)
```
#### 컬럼
id : 자동으로 1씩 올라가는 primary key

name : 해시태그 이름

reviewStatus : 해시태그를 써도 되는지에 대한 리뷰 여부 (PENDING, ACCEPTED, REJECTED 중 하나가 들어간다)

dateCreated : 해시태그가 만들어진 날짜

---

### 요구사항1 특정 문자열의 해시태그가 있는지 검색

초개는 `힐링` 이 이미 해시태그로 저장됐으면 그냥 넘기고 없다면 새로운 row 로 등록하고싶다. 

```mysql
SELECT * FROM Hashtag WHERE name = '힐링'
```

explain 으로 실행계획을 살펴보자

![explain_no_index](/public/img/explain_no_index.png)

`key` 에 인덱스가 쓰이긴 했지만 `Extra` 에 Using Index 등이 나오지 않는다.

해당 로우를 찾기 위해 인덱스를 사용한건 맞지만, 결국 정보를 가져오기 위해서는 테이블 블록 접근을 했다는 뜻이다. 

쿼리를 바꿔보자

```mysql
SELECT name FROM Hashtag WHERE name = '힐링'
```
해당 쿼리에 대한 explain 은 다음과 같다.

![explain_index](/public/img/explain_index.png)
 
짠 Using Index 라는 정보가 추가됐다.

이렇게 쓰였을때 커버링 인덱스라고 한다 ㅇㅅㅇ






 



