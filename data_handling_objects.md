# 데이터를 다루는 객체들

<br /><br />

* 종류
---

<br />

1. DTO (Data Transfer Object)
```
1) 데이터 전송 객체로서, 클라이언트와 서버 간 데이터 교환을 위해 사용.

2) 주로 비즈니스 계층과 프레젠테이션 계층 사이에서 데이터를 전달하는 데 사용.

3) DTO는 주로 데이터 전송을 위한 용도로 사용되며,
   클라이언트 요청이나 서버 응답에 사용될 데이터를 담고 있다.
```

<br />

2. DO (Domain Object)
```
1) 도메인 객체로서, 비즈니스 도메인의 데이터와 비즈니스 로직을 함께 포함하는 객체.
   도메인 객체는 도메인의 규칙을 구현하고, 데이터를 캡슐화한다.

2) 일반적으로 데이터베이스와의 매핑을 담당하며,
   Mybatis 프레임워크와 함께 사용된다.
   일반적으로 엔티티(Entity)와 동일한 개념으로 사용될 수 있다.
```

<br />

3. DAO (Data Access Object)
```
1) 데이터베이스와의 상호 작용을 캡슐화하는 객체.
   주로 데이터베이스 연산을 수행하는 메서드들을 포함(예시 -> CRUD 생성(Create), 조회(Retrieve), 갱신(Update), 삭제(Delete))

2) DAO는 일반적으로 데이터베이스와의 통신을 위한 연결 관리와 트랜잭션 관리 등의 역할을 수행.
   (JPA를 사용하는 경우에는 DAO 대신에 Repository 패턴을 주로 사용한다.)
```

<br />

4. VO (Value Object)
```
1) 값 객체로서, 불변(immutable)하고 값에 따라 동일성이 결정되는 객체.
   주로 비즈니스 로직에 의해 정의된 특정 값의 조합을 나타낸다.
   (예시 -> 좌표를 표현하는 Point 객체가 있을 때, 두 점이 같은 위치인지를 비교할 때 VO를 사용할 수 있다.)

2) VO는 주로 도메인 모델에서 사용되며, 도메인 개념에 집중.
```

<br />

5. Entity (Domain Class)
```
1) 데이터베이스의 테이블과 매핑되는 자바 클래스.
   주로 JPA 어노테이션을 사용하여 매핑을 정의. (예시 -> @Entity, @Table, @Column 등)

2) 엔티티 클래스는 데이터베이스의 레코드를 나타내며, 비즈니스 로직을 포함할 수도 있다.
   JPA에서는 엔티티를 사용하여 데이터베이스와의 상호 작용을 추상화하고,
   ORM 기술을 통해 객체와 데이터베이스 간의 매핑을 처리한다.
```
