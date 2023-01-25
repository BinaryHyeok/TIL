# 프록시와 연관관계 관리

## 프록시
- em.find() vs em.getReference()
- em.find() : 데이터베이스를 통해서 실제 엔티티 객체 조회
- em.getReference() : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회
  - DB에 쿼리가 나가지않아도 객체가 조회된다.

### 프록시 특징
- 실제 클래스를 상속 받아서 만들어짐
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)
- 프록시 객체는 실제 객체의 참조(target)를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출
  <img width="1085" alt="image" src="https://user-images.githubusercontent.com/94176133/213915993-9c87c902-43b8-4419-835b-63c229dcee8b.png">
- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님. 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
- 프록시 객체는 원본 엔티티를 상속받음, **따라서 타입 체크시 주의해야함(== 비교 실패, 대신 instance of 사용)**
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생 
  (하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림)

### 프록시 확인
- 프록시 인스턴스의 초기화 여부 확인  
  PersistenceUnitUtil.isLoaded(Object entity)

- 프록시 클래스 확인 방법  
  entity.getClass().getName() 출력

- 프록시 강체 초기화  
org.hibernate.Hibernate.initialize(entity);

- JPA 표준은 강제 초기화 없음  
강제 호출: member.getName()

<br>

## 즉시 로딩과 지연 로딩
- Member와 Team을 같이 사용하는 경우가 적다면?  
지연로딩을 사용하여 프록시로 가져온다.
```java
@Entity
public class Member {

private String name; 
@ManyToOne(fetch = FetchType.LAZY) //** @JoinColumn(name = "TEAM_ID")
private Team team;
..
}
```
실제 team을 사용하는 시점에 초기화된다.

- Member와 Team을 항상 같이 사용한다면?  
즉시 로딩 EAGER을 사용해서 함께 조회한다.
```java
@Entity
public class Member {

private String name; 
@ManyToOne(fetch = FetchType.EAGER) //** @JoinColumn(name = "TEAM_ID")
private Team team;
..
}
```
JPA 구현체는 가능하면 조인을 사용해서 SQL 한번에 함께 조회

<br>

### 프록시와 즉시로딩 주의
- 가급적 지연 로딩만 사용(특히 실무에서)
- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생
- **즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.**
  - SQL로 번역되어 멤버를 가져왔을 때 EAGER로 되어있다면 TEAM 내용도 들어있어야 되기때문에 추가 쿼리가 생긴다.
  - 처음 쿼리 1 + 추가 쿼리 N
- **@ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정**
- @OneToMany, @ManyToMany는 기본이 지연 로딩
- 일단 모든 연관관계를 지연로딩으로 설정한다.  
  - 필요한 것들은 fetchJoin으로 가져온다.

<br>

## 영속성 전이: CASCADE
- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
- 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장  


기존 코드
```java
Child child1 = new Child();
Child child2 = new Child()

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
em.persist(child1);
em.persist(child2);
```

Cascade 적용코드
```java
// cascade적용
@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)

Child child1 = new Child();
Child child2 = new Child()

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
```

em.persist(parent)만 해도 child도 등록된다.

<br>

### 영속성 전이: CASCADE - 주의
- 영속성 전이는 연관관계 매핑하는 것과 아무 관련이 없음
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐
- 부모엔티티에서 자식엔티티만 관리하면 사용 가능 O
- 자식 엔티티가 다른 엔티티에서 관리가 된다면 사용 X
- 사용가능한 조건
  - 부모와 자식 엔티티의 라이프사이클이 거의 유사할 때
  - 소유자가 하나일 때

### CASCADE의 종류
- ALL: 모두 적용
- PERSIST: 영속
- REMOVE: 삭제
- MERGE: 병합
- REFRESH: REFRESH
- DETACH: DETACH

<br>

### 고아 객체
- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- orphanRemoval = true
  ```java
    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST, orphanRemoval = true)
  ```
- Parent parent1 = em.find(Parent.class, id);  
  parent1.getChildren().remove(0);  
  //자식 엔티티를 컬렉션에서 제거
- DELETE FROM CHILD WHERE ID=?