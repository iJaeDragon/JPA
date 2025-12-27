# JPA와 모던 자바 데이터 저장 기술

> 김영한 강의 기반 개념 정리 (학습/정리용)

---

## 1. SQL 중심적인 개발의 문제점

### 1.1 반복적인 CRUD 코드

* 엔티티 하나당 항상 다음 SQL 필요

  * `INSERT`, `SELECT`, `UPDATE`, `DELETE`
* 필드 추가/변경 시 모든 SQL 수정 필요
* 개발자는 객체가 아닌 **SQL을 관리**하게 됨

➡️ **생산성 저하, 유지보수 비용 증가**

---

### 1.2 SQL 의존적인 개발 구조

* 객체 설계가 아닌 테이블 구조가 기준
* 객체 변경 = SQL 변경
* 객체지향의 장점(캡슐화, 추상화)이 무력화됨

---

## 2. 객체와 관계형 데이터베이스의 패러다임 불일치

객체지향 언어와 관계형 데이터베이스는 근본 철학이 다르다.

### 2.1 객체지향의 특징

* 추상화
* 캡슐화
* 상속
* 다형성
* 참조(reference)를 통한 연관관계

### 2.2 관계형 DB의 특징

* 테이블 기반 구조
* PK / FK
* JOIN
* 값(value) 중심

➡️ 이 차이로 인해 다양한 문제가 발생

---

### 2.3 상속(Inheritance) 문제

#### 객체

```java
class Album extends Item {}
```

#### 관계형 DB

* 상속 개념 없음
* 슈퍼타입/서브타입 테이블로 분해
* 저장/조회 시 JOIN 필요

#### 문제점

* SQL 복잡
* 객체 조립 로직 필요
* 실무에서 상속 모델링 회피

➡️ 객체지향 포기

---

### 2.4 연관관계(Association) 문제

#### 객체

```java
member.getTeam();
```

#### 테이블

```sql
JOIN TEAM ON MEMBER.TEAM_ID = TEAM.TEAM_ID
```

* 객체: 참조 사용
* 테이블: 외래 키 사용

#### 결과

* 테이블 중심 모델링 → 객체가 망가짐
* 객체 중심 모델링 → SQL 과도하게 복잡

---

### 2.5 객체 그래프 탐색 문제

* 객체는 연관된 객체를 자유롭게 탐색해야 함
* 실제로는 **처음 실행한 SQL 범위까지만 접근 가능**

```java
member.getTeam(); // OK
member.getOrders(); // null
```

➡️ 엔티티를 신뢰할 수 없음

---

### 2.6 동일성(identity) 문제

```java
Member m1 = dao.getMember(id);
Member m2 = dao.getMember(id);
m1 == m2; // false
```

* DB 조회 시 항상 새로운 객체 생성
* 자바 컬렉션과 동작 방식 불일치

---

## 3. ORM(Object-Relational Mapping)

### 3.1 ORM이란?

* 객체와 관계형 DB를 매핑하는 기술
* 객체는 객체답게, DB는 DB답게 설계
* 중간에서 ORM 프레임워크가 매핑 처리

---

## 4. JPA(Java Persistence API)

### 4.1 JPA 개요

* 자바 ORM 기술의 **표준 명세**
* 인터페이스 모음
* 구현체:

  * Hibernate (사실상 표준)
  * EclipseLink
  * DataNucleus

➡️ JPA = 표준, Hibernate = 구현체

---

### 4.2 JPA의 역할

* 객체 ↔ 테이블 매핑
* SQL 자동 생성
* 객체 상태 관리
* 연관관계 처리

개발자는 **객체 중심으로 개발**

---

## 5. JPA를 사용하는 이유

### 5.1 생산성

```java
jpa.persist(member);
jpa.find(Member.class, id);
```

* CRUD SQL 작성 불필요

---

### 5.2 유지보수

* 필드 추가/변경 시 엔티티만 수정
* SQL은 JPA가 자동 처리

---

### 5.3 패러다임 불일치 해결

* 상속
* 연관관계
* 객체 그래프 탐색
* 동일성 보장

---

### 5.4 성능

* 1차 캐시
* 쓰기 지연
* 지연 로딩

➡️ 잘 사용하면 JDBC보다 안정적

---

## 6. JPA의 핵심 메커니즘

### 6.1 1차 캐시와 동일성 보장

* 같은 트랜잭션 내 동일 엔티티 보장

```java
Member m1 = em.find(Member.class, id);
Member m2 = em.find(Member.class, id);
m1 == m2; // true
```

효과:

* SQL 최소화
* Repeatable Read 효과

---

### 6.2 트랜잭션 쓰기 지연 (Write-Behind)

* INSERT/UPDATE/DELETE SQL을 모았다가 커밋 시 실행

효과:

* JDBC Batch 처리
* DB 락 최소화
* 성능 향상

---

### 6.3 지연 로딩(Lazy Loading)과 즉시 로딩(Eager Loading)

### 6.3.1 지연 로딩(Lazy Loading) 개념

* 연관된 엔티티를 **즉시 조회하지 않고**, 실제로 사용할 때 조회하는 방식
* 엔티티 조회 시점에는 **프록시 객체**를 주입

```java
Member member = em.find(Member.class, id);
Team team = member.getTeam(); // 이 시점에 SELECT 발생
```

#### 동작 원리

* JPA는 연관 엔티티 대신 **프록시 객체**를 반환
* 프록시 객체는 실제 엔티티를 상속한 가짜 객체
* 프록시 내부에 실제 엔티티 초기화 여부를 관리

```text
Member → Team(Proxy) → 실제 Team
```

* 프록시 객체의 메서드가 호출되는 순간 DB 조회

---

### 6.3.2 즉시 로딩(Eager Loading) 개념

* 엔티티 조회 시점에 연관된 엔티티를 **즉시 함께 조회**
* 기본적으로 JOIN SQL 사용

```java
Member member = em.find(Member.class, id);
```

```sql
SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

#### 특징

* 한 번의 SQL로 연관 엔티티까지 로딩
* 코드상에서는 단순해 보임

---

### 6.3.3 지연 로딩 vs 즉시 로딩 비교

| 구분    | 지연 로딩(LAZY) | 즉시 로딩(EAGER) |
| ----- | ----------- | ------------ |
| 조회 시점 | 실제 사용 시     | 엔티티 조회 시     |
| SQL   | 필요할 때 실행    | 즉시 JOIN 실행   |
| 성능 예측 | 쉬움          | 어려움          |
| 실무 사용 | 권장          | 지양           |
| 기본 전략 | O           | X            |

---

### 6.3.4 즉시 로딩의 문제점 (실무)

#### 1️⃣ 예측 불가능한 SQL

* 연관관계가 많을수록 JOIN 폭발
* 단순 조회 코드가 복잡한 SQL을 발생시킴

#### 2️⃣ N+1 문제 유발

```java
List<Member> members = em.createQuery(
  "select m from Member m", Member.class
).getResultList();
```

* EAGER 설정 시

  * Member 조회 1번
  * Team 조회 N번

➡️ 성능 급격히 저하

#### 3️⃣ 필요 없는 데이터 조회

* 실제로 사용하지 않는 연관 엔티티까지 조회
* 네트워크, 메모리 낭비

---

### 6.3.5 지연 로딩 사용 시 주의사항

#### 1️⃣ 트랜잭션 범위

* 프록시 초기화는 **영속성 컨텍스트가 살아 있어야 가능**

```java
// 트랜잭션 종료 후
member.getTeam(); // LazyInitializationException
```

➡️ 서비스 계층에서 트랜잭션 관리 필수

---

#### 2️⃣ N+1 문제

* 지연 로딩도 잘못 사용하면 N+1 발생
* 해결 방법:

  * fetch join
  * EntityGraph
  * batch size 설정

---

### 6.3.6 실무 권장 전략

1. **모든 연관관계 기본값은 LAZY**
2. 즉시 로딩은 사용하지 않음
3. 필요한 경우에만 조회 전략을 쿼리로 제어

```java
select m from Member m join fetch m.team
```

➡️ 로딩 전략은 **매핑이 아니라 조회에서 결정**

---

### 6.3.7 핵심 정리

* 즉시 로딩은 편해 보이지만 실무에서 위험
* 지연 로딩은 통제 가능하고 예측 가능
* JPA 성능 문제의 대부분은 로딩 전략 이해 부족에서 발생

---

## 7. JDBC vs JPA 비교

| 항목     | JDBC      | JPA      |
| ------ | --------- | -------- |
| CRUD   | SQL 직접 작성 | 자동 처리    |
| 상속     | 복잡        | 자연스럽게 지원 |
| 연관관계   | JOIN 수동   | 객체 참조    |
| 객체 그래프 | 제한적       | 자유       |
| 동일성    | 미보장       | 보장       |
| 유지보수   | 어려움       | 용이       |

---

## 8. 핵심 정리

> ORM은 객체와 RDB,
> 두 기둥 위에 서 있는 기술이다.

* 객체지향을 유지하면서
* 관계형 DB를 사용하는
* 현실적인 표준 기술 = **JPA**

---

## 9. 학습 포인트

* JPA는 만능이 아님
* 객체 설계가 먼저, DB는 그 다음
* 지연 로딩 이해 필수
* 트랜잭션 단위 사고 필요
