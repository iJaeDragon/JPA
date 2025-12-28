# 엔티티 매핑 (Entity Mapping)

---

## 1. 엔티티 매핑 개요

엔티티 매핑은 **객체와 관계형 데이터베이스 테이블을 어떻게 연결할 것인가**에 대한 규칙이다.
JPA를 사용하는 순간, 테이블이 아닌 **객체 설계가 중심**이 된다.

### 엔티티 매핑의 4가지 축

* 객체 ↔ 테이블 매핑
* 필드 ↔ 컬럼 매핑
* 기본 키 매핑
* 연관관계 매핑

---

## 2. 객체와 테이블 매핑

### @Entity

```java
@Entity
public class Member {

    @Id
    private Long id;

    private String username;
}
```

* JPA가 관리하는 객체임을 선언
* 반드시 **기본 생성자(public/protected)** 필요
* 제약사항

  * final 클래스 사용 불가
  * enum, interface, inner class 사용 불가
  * final 필드 사용 불가

#### @Entity 속성

| 속성   | 설명                | 기본값  |
| ---- | ----------------- | ---- |
| name | JPQL에서 사용할 엔티티 이름 | 클래스명 |

> 특별한 이유가 없다면 기본값 사용 권장

---

### @Table

```java
@Entity
@Table(name = "MEMBER")
public class Member {

    @Id
    private Long id;

    private String username;
}
```

| 속성                | 설명           |
| ----------------- | ------------ |
| name              | 매핑할 테이블 이름   |
| uniqueConstraints | DDL 유니크 제약조건 |

---

## 3. 데이터베이스 스키마 자동 생성

### 개념

* 애플리케이션 실행 시점에 **DDL 자동 생성**
* 객체 중심 개발 가능
* **운영 환경에서는 직접 사용 금지**

### hibernate.hbm2ddl.auto 옵션

| 옵션          | 설명             | 사용 권장 환경 |
| ----------- | -------------- | -------- |
| create      | 기존 테이블 삭제 후 생성 | 개발 초기    |
| create-drop | 종료 시 테이블 삭제    | 테스트      |
| update      | 변경분만 반영        | ❌ 운영 금지  |
| validate    | 매핑 검증만 수행      | 운영       |
| none        | 미사용            | 운영       |

> 운영 서버: **validate 또는 none만 사용**

### persistence.xml 설정 예제
```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
```

### 실행시 생성되는 DDL (H2기준)
```sql
create table MEMBER (
    id bigint not null,
    username varchar(255),
    primary key (id)
);
```

---

### DDL 생성 보조 기능

```java
@Column(nullable = false, length = 10)
```

* DDL 생성 시에만 영향
* **JPA 실행 로직에는 영향 없음**

---

## 4. 필드와 컬럼 매핑

### 주요 매핑 어노테이션

| 어노테이션       | 설명          |
| ----------- | ----------- |
| @Column     | 컬럼 매핑       |
| @Enumerated | enum 매핑     |
| @Temporal   | 날짜 타입 매핑    |
| @Lob        | BLOB / CLOB |
| @Transient  | 컬럼 매핑 제외    |

---

### @Column 주요 속성

| 속성               | 설명          |
| ---------------- | ----------- |
| nullable         | NOT NULL 제약 |
| length           | 문자열 길이      |
| unique           | 단일 컬럼 유니크   |
| columnDefinition | 컬럼 정의 직접 지정 |

---

### 예제

```
@Entity
public class Member {

    @Id
    private Long id;

    @Column(nullable = false, length = 20)
    private String username;

    @Column(unique = true)
    private String email;
}
```

### @Enumerated

```java
public enum RoleType {
    USER, ADMIN
}
```

```java
// EnumType.ORDINAL 사용 시 0, 1(인덱스 순서)로 저장되어 순서 변경 시 치명적 오류 발생
@Enumerated(EnumType.STRING)
private RoleType roleType;
```

### DB 저장 값 예시
```text
USER
ADMIN
```

* **ORDINAL 절대 사용 금지**
* enum 순서 변경 시 데이터 오류 발생

---

### @Temporal

* java.util.Date 사용 시 필수
* LocalDate, LocalDateTime 사용 시 생략 가능

---

### @Lob

* String → CLOB
* byte[] → BLOB

---

## 5. 기본 키 매핑

### 기본 키 매핑 어노테이션

```java
@Id
@GeneratedValue
```

### 기본 키 전략

| 전략       | 설명                | DB         |
| -------- | ----------------- | ---------- |
| IDENTITY | DB AUTO_INCREMENT | MySQL      |
| SEQUENCE | DB 시퀀스 사용         | Oracle, H2 |
| TABLE    | 키 생성 테이블          | 모든 DB      |
| AUTO     | 방언에 따라 자동 선택      | 기본값        |

---

### IDENTITY 전략 특징

* persist 시점에 즉시 INSERT
* AUTO_INCREMENT 기반
* 쓰기 지연 전략 사용 불가

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

---

### SEQUENCE 전략

```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GEN",
    sequenceName = "MEMBER_SEQ",
    allocationSize = 1
)
public class Member {

    @Id
    @GeneratedValue(
        strategy = GenerationType.SEQUENCE,
        generator = "MEMBER_SEQ_GEN"
    )
    private Long id;
}
```
* 시퀀스 기반
* allocationSize 기본값 = 50
* 성능 최적화를 위한 미리 할당 전략

---

### TABLE 전략

* 모든 DB 사용 가능
* 성능 가장 낮음
* 실무에서는 거의 사용하지 않음

```
@Entity
@TableGenerator(
    name = "MEMBER_SEQ_GEN",
    table = "MY_SEQUENCES",
    pkColumnValue = "MEMBER_SEQ",
    allocationSize = 1
)
public class Member {

    @Id
    @GeneratedValue(
        strategy = GenerationType.TABLE,
        generator = "MEMBER_SEQ_GEN"
    )
    private Long id;
}
```
---

## 6. 권장 식별자 전략

* 기본 키 조건

  * null 아님
  * 유일
  * 변경 불가

### 권장 방식

> **Long 타입 대리키 + 자동 생성 전략**

* 자연키 사용 지양
* 비즈니스 변경에 안전
* 의미 없는 대리키 사용
* 의미 없는 대리키 사용

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;
}
```

---

## 핵심 요약

* 엔티티 매핑은 JPA의 출발점
* 운영 환경에서는 스키마 자동 생성 절대 주의
* Enum은 반드시 STRING
* 기본 키는 Long + 대리키 전략

---
