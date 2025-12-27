# SQL 중심적인 개발의 문제점

> JPA가 왜 등장했는지를 이해하기 위한 핵심 개념 정리

---

## 1. SQL 중심적인 개발이란?

SQL 중심적인 개발이란, 애플리케이션의 핵심 로직보다 **데이터베이스와 SQL이 설계의 중심**이 되는 개발 방식을 말한다.

* 객체 모델보다 테이블 구조가 우선
* 애플리케이션 코드는 SQL 실행을 위한 도구 역할
* 개발자의 주요 작업이 비즈니스 로직이 아닌 SQL 작성

---

## 2. 반복적이고 지루한 CRUD 문제

### 2.1 CRUD SQL의 무한 반복

엔티티 하나당 항상 다음 SQL이 필요하다.

* INSERT
* SELECT
* UPDATE
* DELETE

```sql
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES (?, ?);
SELECT MEMBER_ID, NAME FROM MEMBER WHERE MEMBER_ID = ?;
UPDATE MEMBER SET NAME = ? WHERE MEMBER_ID = ?;
DELETE FROM MEMBER WHERE MEMBER_ID = ?;
```

➡️ 비즈니스 로직과 무관한 코드가 대부분을 차지

---

### 2.2 필드 변경 시 폭발적인 수정 범위

```java
class Member {
  private String memberId;
  private String name;
  private String tel; // 필드 추가
}
```

* INSERT SQL 수정
* SELECT SQL 수정
* UPDATE SQL 수정
* 관련된 모든 매핑 코드 수정

➡️ **변경에 매우 취약한 구조**

---

## 3. SQL 의존적인 개발 구조

### 3.1 객체보다 SQL이 중심

* 객체는 단순한 DTO 역할
* 객체 설계의 의미가 사라짐
* 캡슐화, 추상화 무력화

```java
// 객체가 아니라 테이블 구조를 그대로 반영
class Member {
  String memberId;
  Long teamId; // FK를 그대로 노출
  String username;
}
```

➡️ 객체지향이 아닌 **테이블 지향 코드**

---

### 3.2 계층 분리가 어려움

```java
Member member = memberDAO.getMember(id);
member.getTeam(); // 동작 여부를 확신할 수 없음
```

* 어떤 SQL을 실행했는지 알아야 객체 사용 가능
* 서비스 계층이 DAO 구현에 종속됨

➡️ 진정한 의미의 계층 분리 실패

---

## 4. 객체와 관계형 DB의 패러다임 불일치 심화

SQL 중심 개발은 객체와 RDB의 차이를 더 크게 만든다.

---

### 4.1 상속 문제

* 객체: 상속은 자연스러운 개념
* DB: 상속 개념 없음

결과:

* 테이블 분해
* JOIN 증가
* 조회/저장 로직 복잡

➡️ 실무에서 상속 모델 회피

---

### 4.2 연관관계 문제

* 객체: 참조 기반
* 테이블: 외래 키 기반

```java
member.getTeam();
```

```sql
JOIN TEAM ON MEMBER.TEAM_ID = TEAM.TEAM_ID;
```

➡️ 객체답게 설계할수록 SQL이 복잡해짐

---

### 4.3 객체 그래프 탐색 제한

```java
member.getTeam(); // OK
member.getOrders(); // null
```

* 처음 실행한 SQL 범위까지만 접근 가능
* 엔티티를 신뢰할 수 없음

➡️ 방어 코드 증가

---

### 4.4 동일성(identity) 문제

```java
Member m1 = dao.getMember(id);
Member m2 = dao.getMember(id);
m1 == m2; // false
```

* 같은 DB 로우 → 다른 자바 객체
* 자바 컬렉션과 동작 방식 불일치

---

## 5. 유지보수 관점의 문제점

### 5.1 변경 비용 증가

* 요구사항 변경 → SQL 수정
* SQL 수정 → 매핑 코드 수정
* 테스트 범위 확대

➡️ 작은 변경이 큰 비용으로 이어짐

---

### 5.2 휴먼 에러 증가

* 컬럼 누락
* 매핑 실수
* SQL 문법 오류

➡️ 런타임 오류 증가

---

## 6. 핵심 정리

* SQL 중심 개발은 단기적으로 단순해 보임
* 규모가 커질수록 유지보수 불가능
* 객체지향의 장점을 살릴 수 없음

> **SQL 중심적인 개발의 한계를 극복하기 위한 대안이 ORM이며,
> 자바 진영의 표준 ORM이 JPA이다.**

---

## 7. 다음 단계

이 문제를 해결하기 위해 등장한 개념:

* ORM(Object-Relational Mapping)
* JPA(Java Persistence API)

➡️ 객체 중심 개발로의 전환
