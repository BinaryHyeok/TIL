# 연관관계 매핑 기본

- 객체의 참조와 테이블의 외래 키를 매핑
- 용어
  - 방향(Direction): 단방향, 양방향
  - 다중성(Multiplicity): 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)이해
  - 연관관계의 주인(Owner): 객체 양방향 연관관계는 관리 주인이 필요

<img width="1066" alt="image" src="https://user-images.githubusercontent.com/94176133/212537538-c42be9fc-1b7f-4f65-8dd0-1d058c688d26.png">

<br>

## 객체를 테이블에 맞추어 모델링  
- 참조 대신에 외래 키를 그대로 사용
  ```java
    //member
    @Entity
    public class Member {
        @Id
        @GeneratedValue
        @Column(name = "MEMBER_ID")
        private Long id;

        @Column(name = "USERNAME")
        private String username;

        @Column(name = "TEAM_ID")
        private long teamId;
    }

    //team
    @Entity
    public class Team {

        @Id
        @GeneratedValue
        @Column(name = "TEAM_ID")
        private Long id;
        private String name;
    }
  ```

- 외래 키 식별자를 직접 다룸
  ```java

  //팀 저장
  Team team = new Team();
  tema.setName("TeamA");
  em.persist(team);

  //회원 저장
  Memeber member =  new Member();
  member.setName("member1");
  member.setTeamId(team.getId());
  em.psersist(member);
  ```

- 식별자로 다시 조회, 객체 지향적인 방법은 아니다.
  ```java
  //조회
  Member findMember = em.find(Member.class, member.getId());

  //연관관계가 없음
  Team findTeam = em.find(Team.class, team.getId());
  ```

-> 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
- 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에는 이런 큰 간격이 있다.

## 객체 지향 모델링(단방향)

![image](https://user-images.githubusercontent.com/94176133/212538806-e93803ef-efa3-4f83-ac36-7964c200913e.png)

- 객체의 참조와 테이블의 외래 키를 매핑
```java
Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private long teamId;

    //Member가 many, Team이 One
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

- 연관관계 저장
```java
//팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);
//회원 저장
Member member = new Member();
member.setUsername("member1");
member.setTeam(team); // 단방향 연관관계 설정, 참조 저장
em.persist(member);
```

- 참조로 연관관계 조회 - 객체 그래프 탐색
```java
//조회
Member findMember = em.find(Member.class, member.getId());

//참조를 사용해서 연관관계 조회
Team findTeam = findMember.getTeam();
```

- 연관관계 수정
```java
//새로운 팀B
Team teamB = new Team();
teamB.setName("TeamB");
em.persist(teamB);

//회원1에 새로운 팀B 설정
member.setTeam(teamB);
```

<br>

# 양방향 연관관계와 연관관계의 주인
위의 단방향 예제의 경우 멤버에서 팀을 얻어 올 순 있지만, 팀에서 멤버를 얻어올 수는 없다.  
하지만 팀에 레퍼런스만 넣어둔다면 팀에서 멤버를 얻어올 수 있게 된다.  
이것을 **양방향 연관관계** 라고 한다.

## 양방향 매핑
<img width="1322" alt="image" src="https://user-images.githubusercontent.com/94176133/212539753-460d6ad3-650c-49ca-b0d2-d5fc18e94e5a.png">

위의 사진에서 테이블 연관관계의 경우에는 단방향의 테이블 연관관계와 변한 것이 없다.  
테이블 연관관계에서 멤버의 경우 팀 아이디로 팀을 얻어올 수 있고, 팀의 경우 멤버에있는 팀 아이디로 멤버를 얻어올 수 있다.  
사실상 테이블 연관관계는 방향이라는 개념자체가 없고, 외래키 하나로 처리가 가능하다.

- Member 엔티티는 단방향과 동일
```java
 @Id
 @GeneratedValue
 @Column(name = "MEMBER_ID")
 private Long id;
 @Column(name = "USERNAME")
 private String username;
 @ManyToOne
 @JoinColumn(name = "TEAM_ID")
 private Team team;
```

- Team 엔티티는 컬렉션 추가
```java
 @Id
 @GeneratedValue
 @Column(name = "TEAM_ID")
 private Long id;

 private String name;

 @OneToMany(mappedBy = "team")
 private List<Member> members = new ArrayList<>();
```

- 반대 방향으로 객체 그래프 탐색
```java
Member findMember = em.find(Member.class, member.getId());
List<Member> members = findMember.getTeam().getMembers();
```

## 연관관계 주인과 mappedBy
- mappedBy = JPA의 멘탈붕괴 난이도
- mappedBy는 처음에는 이해하기 어렵다.
- 객체와 테이블간에 연관관계 맺는 차이를 이해해야 한다.

### 객체와 테이블이 관계를 맺는 차이
- 객체 연관관계 = 2개
  - 회원 -> 팀 연관관계 1개(단방향)
  - 팀 -> 회원 연관관계 1개(단방향)
- 테이블 연관관계 = 1개
- 회원 <-> 팀의 연관관계 1개(양방향)

### 객체의 양방향 관계
- 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개다.
- 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.
- A -> B (a.getB())
- B -> A (b.getA())

### 테이블의 양방향 연관관계
- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리
- MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐(양쪽 조인 가능)

```sql
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

### 둘 중 하나로 외래 키 관리를 해야한다.
<img width="1363" alt="image" src="https://user-images.githubusercontent.com/94176133/212548882-9f38934a-ffc2-4ef3-9c42-611b0b654bbb.png">

- 만약 어떤 멤버가 팀을 변경한다고 했을 떄
  - Member 객체에서 team을 변경해야 하는가?
  - Team 객체에서 members를 변경해야 하는가?
  - DB 입장에서는 TEAM_ID(FK)만 업데이트 되면 된다.  

-> 둘 중 하나를 정해야 된다.

### 연관관계의 주인(Owner)
양뱡항 매핑 규칙
- 객체의 두 관계중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리(등록, 수정)
- 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용 X
- 주인이 아니면 mappedBy 속성으로 주인 지정

### 누구를 주인으로 해야되는가?
- **외래 키가 있는 곳을 주인으로 정해라**
- DB입장에서 외래키가 있는 곳이 N이고 없는 곳이 1이다. 즉 N : 1
- @ManyToOne(다)쪽이 연관관계 주인이 된다.
- 여기서는 Member.team이 연관관계의 주인

<img width="1122" alt="image" src="https://user-images.githubusercontent.com/94176133/212549311-b990878e-bcef-4d47-bd41-288df09ee158.png">

<br>

### 양방향 매핑시 가장 많이 하는 실수
- 연관관계의 주인에 값을 입력하지 않음
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

///역방향(주인이 아닌 방향)만 연관관계 설정
//member에 팀정보가 업데이트 되지 않음
team.getMembers().add(member);
em.persist(member);
```

- 연관관계의 주인에 값 입력
  - 순수한 객체 관계를 고려하면 항상 양쪽다 값을 입력해야 한다.
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

team.getMembers().add(member);
//연관관계의 주인에 값 설정
member.setTeam(team);
em.persist(member);
```

### 양방향 연관관계 주의
- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
- 연관관계 편의 메소드를 생성하자
    ```java
    public void setTeam(Team team) {
        this.team = team;
        //setTeam할 때 아예 같이 넣는다. 이때 this는 member
        team.getMembers().add(this);
    }
    ```
- 양방향 매핑시에 무한 루프를 조심하자
  - ex) toString(), lombok, JSON 생성 라이브러리
  - lombok 사용시 toString()은 빼고 쓰자
  - controller에서 Entity는 DTO로 변환해서 반환하자

<br>

# 양방향 매핑 정리
- 단방향 매핑만으로도 이미 연관관계 매핑은 완료, 설계는 단방향으로 끝내자
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
- 객체 입장에서 양방향 매핑은 지향하지 않는다.
- JPQL에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨 (테이블에 영향을 주지 않는다.)
- 양방향 매핑은 테이블 변경없이 코드 추가로 완성가능 하므로, 설계 이후 구현단계에서 생각해도 늦지않다.
- 연관관계 주인은 외래키의 위치를 기준으로 정해야 한다.
- 양방향 연관관계 설정 시 한쪽 값만 설정해도 상관없으나 되도로 양쪽에 넣도록 하자
- 연관관계 편의 메소드를 활용하자

<br>
