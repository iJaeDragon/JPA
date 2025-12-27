# 영속성 관리와 JPA 내부 구조

## 1. JPA에서 가장 중요한 두 가지

JPA를 제대로 이해하기 위해 반드시 짚고 가야 할 핵심은 다음 두 가지다.

1. **객체와 관계형 데이터베이스 매핑 (ORM)**
2. **영속성 컨텍스트(Persistence Context)**

ORM이 "어떻게 매핑할 것인가"에 대한 이야기라면,
영속성 컨텍스트는 **JPA가 객체를 어떻게 관리하고 DB와 언제 동기화하는가**에 대한 핵심 개념이다.

---

## 2. 엔티티 매니저 팩토리와 엔티티 매니저

* **EntityManagerFactory**

  * 애플리케이션 전체에서 하나만 생성
  * DB 커넥션 풀 등 무거운 자원 관리

* **EntityManager**

  * 실제로 엔티티를 저장, 조회, 삭제하는 객체
  * 쓰레드 간 공유 ❌
  * 사용 후 반드시 종료

```java
EntityManager em = emf.createEntityManager();
```

---

## 3. 영속성 컨텍스트란?

* "엔티티를 영구 저장하는 환경"
* **논리적인 개념** (눈에 보이지 않음)
* 엔티티 매니저를 통해서만 접근 가능

```java
em.persist(entity);
```

> 영속성 컨텍스트는 엔티티 매니저 내부에 존재하는 개념적인 저장소다.

---

## 4. 엔티티 매니저와 영속성 컨텍스트 관계

### J2SE 환경

* 엔티티 매니저 : 영속성 컨텍스트 = **1 : 1**

### J2EE / Spring 환경

* 여러 엔티티 매니저가 하나의 영속성 컨텍스트 공유 가능
* **N : 1 구조**

---

## 5. 엔티티의 생명주기

### 1️⃣ 비영속 (new / transient)

* 영속성 컨텍스트와 전혀 관계없는 상태

```java
Member member = new Member();
member.setId("member1");
```

### 2️⃣ 영속 (managed)

* 영속성 컨텍스트에 의해 관리되는 상태

```java
em.persist(member);
```

### 3️⃣ 준영속 (detached)

* 영속성 컨텍스트에서 분리된 상태

```java
em.detach(member);
```

### 4️⃣ 삭제 (removed)

* 삭제 예약 상태

```java
em.remove(member);
```

---

## 6. 영속성 컨텍스트의 이점

* 1차 캐시
* 동일성(identity) 보장
* 트랜잭션을 지원하는 쓰기 지연
* 변경 감지 (Dirty Checking)
* 지연 로딩 (Lazy Loading)

---

## 7. 1차 캐시

* 영속성 컨텍스트 내부의 캐시
* **엔티티 매니저 단위**로 관리

```java
Member findMember = em.find(Member.class, "member1");
```

조회 순서:

1. 1차 캐시 확인
2. 없으면 DB 조회
3. 결과를 1차 캐시에 저장

---

## 8. 동일성(identity) 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
System.out.println(a == b); // true
```

* 같은 영속성 컨텍스트 안에서는 항상 같은 객체 반환
* 애플리케이션 차원에서 **REPEATABLE READ** 수준 보장

---

## 9. 트랜잭션을 지원하는 쓰기 지연

```java
transaction.begin();

em.persist(memberA);
em.persist(memberB);

transaction.commit();
```

* persist 시점에 SQL 실행 ❌
* 커밋 시점에 SQL 실행 ⭕
* 쓰기 지연 SQL 저장소에 쿼리 보관

---

## 10. 변경 감지 (Dirty Checking)

```java
Member member = em.find(Member.class, "memberA");
member.setUsername("hi");

transaction.commit();
```

* update 메서드 호출 ❌
* 스냅샷과 비교하여 변경 사항 자동 반영

---

## 11. 엔티티 삭제

```java
Member member = em.find(Member.class, "memberA");
em.remove(member);
```

* 즉시 삭제되지 않음
* 커밋 시점에 DELETE SQL 실행

---

## 12. 플러시(Flush)

영속성 컨텍스트의 변경 내용을 **데이터베이스에 동기화**하는 과정

### 플러시 시 수행 작업

1. 변경 감지
2. 쓰기 지연 SQL 저장소 등록
3. SQL을 DB에 전송

---

## 13. 플러시 발생 시점

* `em.flush()` 직접 호출
* 트랜잭션 커밋
* **JPQL 실행 시 자동 호출**

```java
em.createQuery("select m from Member m", Member.class)
  .getResultList();
```

---

## 14. 플러시 모드

* `FlushModeType.AUTO` (기본값)

  * 커밋, 쿼리 실행 시 플러시

* `FlushModeType.COMMIT`

  * 커밋 시에만 플러시

```java
em.setFlushMode(FlushModeType.COMMIT);
```

---

## 15. 플러시의 핵심 정리

* 영속성 컨텍스트를 비우지 않음
* DB와 동기화만 수행
* 트랜잭션 단위가 가장 중요

---

## 16. 준영속 상태

* 영속성 컨텍스트의 관리 대상 ❌
* 1차 캐시, 변경 감지, 지연 로딩 모두 사용 불가

### 준영속으로 만드는 방법

```java
em.detach(entity); // 특정 엔티티만
em.clear();        // 전체 초기화
em.close();        // 종료
```

---

## 17. 핵심 요약

* JPA의 본질은 **영속성 컨텍스트**
* SQL 실행 시점은 커밋 기준
* 객체 상태 변경만으로 DB 반영 가능
* 영속 상태 관리가 성능과 설계를 좌우함
