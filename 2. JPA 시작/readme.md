
# JPA 시작 – Hello JPA 실습 정리

> 김영한 「자바 ORM 표준 JPA 프로그래밍」 기반
> JPA 입문 단계에서 반드시 이해해야 할 설정 + 동작 흐름 정리

---

## 1. Hello JPA 실습의 목적

Hello JPA 실습은 **JPA의 전체 동작 흐름을 최소 코드로 경험**하는 것이 목적이다.

* JPA 프로젝트 기본 구조 이해
* JPA 설정 파일의 역할 이해
* 엔티티 매핑 경험
* JPA가 SQL을 어떻게 생성·실행하는지 확인
* JPQL의 필요성 체감

---

## 2. 개발 환경 구성

### 2.1 H2 데이터베이스

#### H2를 사용하는 이유

* 가벼움 (약 1.5MB)
* 설치 및 실행 간단
* 웹 콘솔 제공
* MySQL / Oracle 모드 지원
* 실습용으로 최적

공식 사이트: [https://www.h2database.com/](https://www.h2database.com/)

---

### 2.2 Maven 소개

Maven은 **자바 빌드 및 의존성 관리 도구**이다.

* 라이브러리 자동 다운로드
* 의존성 충돌 관리
* 표준 프로젝트 구조 제공

> 최근에는 Gradle도 많이 사용되지만, JPA 입문 학습에는 Maven이 이해하기 쉽다.

---

## 3. 프로젝트 생성

### 3.1 기본 설정

* Java 8 이상 (강의 기준 8 권장)
* Maven 프로젝트

```text
groupId    : jpa-basic
artifactId : ex1-hello-jpa
version    : 1.0.0
```

---

### 3.2 라이브러리 설정 (pom.xml)

```xml
<dependencies>
    <!-- JPA 구현체: Hibernate -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.3.10.Final</version>
    </dependency>

    <!-- H2 데이터베이스 -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.199</version>
    </dependency>
</dependencies>
```

> JPA는 표준이고, 실제 동작은 Hibernate가 수행한다.

---

## 4. JPA 설정 – persistence.xml

### 4.1 persistence.xml 역할

* JPA의 **설정 파일**
* 위치 고정: `/META-INF/persistence.xml`
* 하나 이상의 `persistence-unit` 정의 가능

---

### 4.2 persistence.xml 주요 구성

```xml
<persistence version="2.2">
  <persistence-unit name="hello">
    <properties>
      <!-- JDBC 필수 설정 -->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>

      <!-- 데이터베이스 방언 -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

      <!-- 로그 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
    </properties>
  </persistence-unit>
</persistence>
```

---

## 5. 데이터베이스 방언(Dialect)

### 5.1 방언이 필요한 이유

* JPA는 DB에 독립적
* 하지만 DB마다 SQL 문법이 다름

예시:

* 문자열 타입: VARCHAR / VARCHAR2
* 페이징: LIMIT / ROWNUM
* 함수: SUBSTRING / SUBSTR

➡️ 방언은 **DB 고유 문법을 JPA가 처리하기 위한 번역기**

---

### 5.2 대표적인 방언

```text
H2       : org.hibernate.dialect.H2Dialect
Oracle   : org.hibernate.dialect.Oracle10gDialect
MySQL    : org.hibernate.dialect.MySQL5InnoDBDialect
```

Hibernate는 40개 이상의 방언을 지원한다.

---

## 6. JPA 구동 방식

### 6.1 핵심 객체

* **EntityManagerFactory**

  * 애플리케이션 전체에서 하나만 생성
  * 비용이 매우 큼

* **EntityManager**

  * 트랜잭션 단위로 생성
  * 쓰레드 간 공유 금지

---

## 7. 엔티티 매핑

### 7.1 엔티티 정의

```java
@Entity
public class Member {

    @Id
    private Long id;

    private String name;
}
```

* `@Entity`: JPA 관리 대상
* `@Id`: 기본 키(PK)

---

### 7.2 테이블 자동 생성 결과

```sql
create table Member (
   id bigint not null,
   name varchar(255),
   primary key (id)
);
```

---

## 8. JPA 실습 – CRUD

### 8.1 반드시 기억할 주의사항

* EntityManagerFactory는 하나만
* EntityManager는 공유 금지
* **모든 데이터 변경은 트랜잭션 안에서 수행**

---

## 9. JPQL 소개

### 9.1 JPQL이 필요한 이유

* JPA는 엔티티 중심 개발
* 단건 조회는 `find()`로 충분
* 조건 검색은 SQL이 필요

➡️ 객체를 대상으로 하는 쿼리 언어 필요

---

### 9.2 JPQL 개념

* SQL을 추상화한 객체 지향 쿼리 언어
* 테이블이 아닌 **엔티티를 대상**으로 쿼리

```jpql
select m from Member m where m.id >= 2
```

* SQL과 문법 유사
* DB에 독립적

---

### 9.3 JPQL vs SQL 차이

| 구분  | JPQL  | SQL    |
| --- | ----- | ------ |
| 대상  | 엔티티   | 테이블    |
| 의존성 | DB 독립 | DB 종속  |
| 목적  | 객체 조회 | 데이터 조회 |

---

## 10. 핵심 정리

* Hello JPA는 **JPA 전체 흐름을 압축한 실습**
* 설정 → 엔티티 → 트랜잭션 → CRUD → JPQL
* JPA는 SQL을 숨기지만, SQL을 몰라도 되는 것은 아님

> JPA는 SQL을 제거하는 기술이 아니라,
> **SQL을 더 잘 쓰게 만들어 주는 기술**이다.

---

## 11. 다음 학습 포인트

* 영속성 컨텍스트
* 엔티티 생명주기
* JPQL 심화
* 연관관계 매핑
