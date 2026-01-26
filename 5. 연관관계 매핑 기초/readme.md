# JPA 연관관계 매핑 완벽 가이드

## 📚 목차
- [학습 목표](#학습-목표)
- [연관관계가 필요한 이유](#연관관계가-필요한-이유)
- [단방향 연관관계](#단방향-연관관계)
- [양방향 연관관계와 연관관계의 주인](#양방향-연관관계와-연관관계의-주인)
- [정리 및 베스트 프랙티스](#정리-및-베스트-프랙티스)

---

## 학습 목표

이 가이드를 통해 다음 개념들을 완벽하게 이해할 수 있습니다:

- ✅ 객체와 테이블 연관관계의 차이
- ✅ 객체의 참조와 테이블의 외래 키 매핑
- ✅ 방향(Direction): 단방향, 양방향
- ✅ 다중성(Multiplicity): 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
- ✅ 연관관계의 주인(Owner) 개념

---

## 연관관계가 필요한 이유

> "객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다."  
> — 조영호, 《객체지향의 사실과 오해》

### 예제 시나리오

- 회원과 팀이 있다
- 회원은 하나의 팀에만 소속될 수 있다
- 회원과 팀은 **다대일(N:1) 관계**다

### ❌ 잘못된 방법: 테이블에 맞춘 모델링

<img width="640" height="426" alt="image" src="https://github.com/user-attachments/assets/18518cc8-3794-490d-99e0-0f3411cc5040" />

객체를 테이블 구조에 맞춰 설계하면 객체지향의 장점을 잃게 됩니다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    @Column(name = "TEAM_ID")
    private Long teamId;  // ❌ 외래 키를 그대로 사용
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
}
```

#### 문제점 1: 외래 키를 직접 다뤄야 함

```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeamId(team.getId());  // ❌ ID를 직접 설정
em.persist(member);
```

#### 문제점 2: 객체 그래프 탐색 불가능

```java
// 조회
Member findMember = em.find(Member.class, member.getId());

// 연관관계가 없어서 팀을 다시 조회해야 함
Team findTeam = em.find(Team.class, findMember.getTeamId());  // ❌ 번거로움
```

### 핵심 개념

**테이블과 객체의 차이점을 이해하자:**

| 구분 | 연관관계 탐색 방법 |
|------|-------------------|
| **테이블** | 외래 키로 JOIN |
| **객체** | 참조(reference)로 연관 객체 탐색 |

---

## 단방향 연관관계

### ✅ 올바른 방법: 객체 지향 모델링

<img width="643" height="440" alt="image" src="https://github.com/user-attachments/assets/4f60c30d-66fc-4df3-bda9-09b2ea894fcb" />


객체의 참조와 테이블의 외래 키를 매핑합니다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
    
    @ManyToOne  // 다대일 관계
    @JoinColumn(name = "TEAM_ID")  // 외래 키 매핑
    private Team team;  // ✅ 참조로 연관관계 표현
}
```

<img width="637" height="434" alt="image" src="https://github.com/user-attachments/assets/8facb054-c78c-4392-9f51-984e80d22d77" />


### 연관관계 저장

```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team);  // ✅ 참조를 직접 저장
em.persist(member);
```

### 객체 그래프 탐색

```java
// 조회
Member findMember = em.find(Member.class, member.getId());

// 참조를 사용해서 연관관계 조회 (객체 그래프 탐색)
Team findTeam = findMember.getTeam();  // ✅ 간단하고 직관적!
```

### 연관관계 수정

```java
// 새로운 팀B
Team teamB = new Team();
teamB.setName("TeamB");
em.persist(teamB);

// 회원1에 새로운 팀B 설정
member.setTeam(teamB);  // ✅ 참조만 변경하면 됨
```

---

## 양방향 연관관계와 연관관계의 주인

### 양방향 매핑 구조

양방향 매핑을 하면 Team에서도 Member 목록을 조회할 수 있습니다.

#### Member 엔티티 (단방향과 동일)

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

#### Team 엔티티 (컬렉션 추가)

```java
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")  // ⭐ mappedBy로 주인 지정
    private List<Member> members = new ArrayList<>();
}
```

### 역방향 조회

```java
// 조회
Team findTeam = em.find(Team.class, team.getId());

// 역방향으로 객체 그래프 탐색
int memberSize = findTeam.getMembers().size();  // ✅ Team에서 Member 조회
```

---

## 🎯 연관관계의 주인 (mappedBy)

### mappedBy가 필요한 이유

**객체와 테이블의 연관관계 개수 차이:**

| 구분 | 연관관계 개수 |
|------|--------------|
| **객체** | 2개 (Member → Team, Team → Member) |
| **테이블** | 1개 (MEMBER.TEAM_ID 외래 키 하나로 양방향 관리) |

### 객체의 양방향 관계

객체의 양방향 관계는 사실 **서로 다른 단방향 관계 2개**입니다.

<img width="639" height="423" alt="image" src="https://github.com/user-attachments/assets/4b9d34c3-7884-4690-97c1-949539a90662" />

```java
class A {
    B b;  // A → B
}

class B {
    A a;  // B → A
}
```

### 테이블의 양방향 관계

테이블은 **외래 키 하나로 양방향 조인 가능**합니다.

```sql
-- Member → Team 조회
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID;

-- Team → Member 조회
SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID;
```

<img width="637" height="416" alt="image" src="https://github.com/user-attachments/assets/6f8cde3e-e23a-4116-8d49-f21e89a603c2" />

### ⚠️ 문제: 외래 키를 누가 관리할 것인가?

Member의 team과 Team의 members 중 **어느 것으로 외래 키를 관리**해야 할까?

**→ 연관관계의 주인(Owner)을 정해야 합니다!**

<img width="630" height="260" alt="image" src="https://github.com/user-attachments/assets/eada6239-e553-45e3-95f2-0f786b2fbe04" />

---

## 연관관계의 주인 규칙

### 양방향 매핑 규칙

1. **객체의 두 관계 중 하나를 연관관계의 주인으로 지정**
2. **주인만이 외래 키를 관리** (등록, 수정)
3. **주인이 아닌 쪽은 읽기만 가능**
4. 주인은 `mappedBy` 속성 사용 ❌
5. 주인이 아니면 `mappedBy` 속성으로 주인 지정

### 🔑 주인 선택 기준

> **외래 키가 있는 곳을 주인으로 정해라!**

- **Member.team**이 연관관계의 주인 (외래 키 보유)
- Team.members는 읽기 전용 (mappedBy 사용)

<img width="560" height="269" alt="image" src="https://github.com/user-attachments/assets/15423e52-1380-45dc-bf45-0aa56269aca0" />

---

## ⚠️ 가장 흔한 실수

### ❌ 실수 1: 주인이 아닌 쪽에만 값 설정

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 역방향(주인이 아닌 방향)만 연관관계 설정
team.getMembers().add(member);  // ❌ 외래 키에 반영 안됨!

em.persist(member);
```

**결과:**

| ID | USERNAME | TEAM_ID |
|----|----------|---------|
| 1  | member1  | **null** ❌ |

### ✅ 올바른 방법: 주인에 값 설정

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 연관관계의 주인에 값 설정
member.setTeam(team);  // ✅ 필수!

// 순수 객체 관계를 위해 양쪽 다 설정 (권장)
team.getMembers().add(member);

em.persist(member);
```

**결과:**

| ID | USERNAME | TEAM_ID |
|----|----------|---------|
| 1  | member1  | 2 ✅ |

---

## 🎓 양방향 연관관계 주의사항

### 1. 순수 객체 상태를 고려해 항상 양쪽에 값 설정

```java
member.setTeam(team);           // 주인에 설정
team.getMembers().add(member);  // 반대편에도 설정
```

### 2. 연관관계 편의 메소드 생성

```java
@Entity
public class Member {
    // ...
    
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);  // 양방향 설정을 한 번에!
    }
}
```

### 3. 무한 루프 조심

#### ❌ 주의해야 할 케이스:

- `toString()` 메소드
- Lombok의 `@ToString`
- JSON 생성 라이브러리 (Jackson 등)

```java
// ❌ 무한 루프 발생 가능
@Entity
@ToString  // Member → Team → Member → Team → ...
public class Member {
    @ManyToOne
    private Team team;
}
```

**해결책:**
- `toString()` 제외: `@ToString(exclude = "team")`
- 또는 Controller에서 DTO로 변환 후 반환

---

## 정리 및 베스트 프랙티스

### 핵심 정리

1. **단방향 매핑만으로도 연관관계 매핑은 완료**
2. 양방향 매핑은 반대 방향 조회(객체 그래프 탐색) 기능 추가
3. JPQL에서 역방향 탐색이 필요할 때 양방향 추가
4. **단방향 매핑을 먼저 하고, 양방향은 필요할 때 추가** (테이블에 영향 없음)

### 연관관계의 주인 선택 기준

> ⚠️ **비즈니스 로직을 기준으로 선택하면 안 됨!**

✅ **외래 키의 위치를 기준으로 정해야 함**

- 다대일(N:1) 관계에서 **N 쪽이 주인**
- Member(N) : Team(1) → **Member.team이 주인**

### 권장 개발 순서

```
1. 단방향 매핑으로 설계 완료
   ↓
2. 필요시 양방향 추가 (조회 편의성)
   ↓
3. 연관관계 편의 메소드 작성
   ↓
4. 무한 루프 방지 처리
```
