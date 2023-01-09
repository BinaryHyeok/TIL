# 영속성 관리

## Entity Manager Factory & Entity Manager
![image](https://user-images.githubusercontent.com/94176133/211245153-3abb5676-7ff4-4924-96d4-77b9f974eb97.png)

## 영속성 컨텍스트
- JPA를 이해하는데 가장 중요한 용어
- "엔티티를 영구 저장하는 환경"이라는 뜻
- EntityManager.persist(entity)
    - persist메서드는 DB에 저장하는게 아니고 영속성 컨텍스트에 저장한다.
- 영속성 컨텍스트는 논리적인 개념
- 눈에 보이지 않는다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근

![image](https://user-images.githubusercontent.com/94176133/211245540-58c0fc7f-ab96-477e-acdd-8fa035553b6b.png)

## Entity LifeCycle
- 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속(managed) : 영속성 컨텍스트에 관리되는 상태
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed) : 삭제된 상태

![image](https://user-images.githubusercontent.com/94176133/211245831-167c2ae7-27f4-444d-aa50-c091bf872480.png)
    

### 비영속
객체를 생성하고 EntityManager에 연결을 하지 않았고 JPA와 관련이 없는 상태  

### 영속
EntityManager 안에는 영속 컨텍스트가 존재한다.  
EntityManager에 persist로 객체를 넣게되면 영속성 컨텍스트에 객체가 들어가며 **영속 상태**가 된다.  
이때 DB에 저장되지 않는다. -> 트랜잭션 commit시  DB저장

### 준영속, 삭제
엔티티를 영속성 컨텍스트에서 분리, 준영속 상태 -> detach  
객체를 삭제한 상태 -> remove  

### 영속성 컨텍스트의 이점
- 1차 캐시
    - 영속성 컨텍스트에는 1차 캐시가 존재한다.
    - JPA로 조회 시 1차 캐시부터 조회하며, 있을 경우 반환한다.
    - 1차 캐시에 없다면 DB에서 조회하여 찾은 녀석을 1차 캐시에 저장한 뒤 반환한다.
    - 엔티티 매니저는 데이터 트랜잭션 단위로 생성되고 종료된다. 따라서 서비스에서 한명의 고객에 대한 요청만 처리하고 사라지므로 큰 도움이 되지 않는다. 만약 비지니스 로직이 복잡하다면 성능 향상은 있다.

<br>

- 동일성(identity) 보장
    - 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

<br>

- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
    - 영속성 컨텍스트 안에는 1차 캐시 말고도 쓰기 지연 SQL 저장소도 존재한다.
    - persist시 Entity는 1차 캐시에 쓰여지고, JPA는 Entity를 분석하고 INSERT Query를 생성하고 쓰기 지연 SQL 저장소에 쌓아둔다.
    - 트랜잭션을 커밋하면 쓰기 지연 SQL 저장소에 있던 Query문이 flush되면서 DB로 날라간다.
    - **버퍼링 기능을 사용할 수 있게 된다.**
<br>

- 변경 감지(Dirty Checking)
    - 1차 캐시안에는 pk, Entity, 스냅샷이 존재
    - 스냅샷이란 1차캐시로 값을 읽어온 최초 시점의 상태를 복제해놓은 것
    - JPA가 transaction commit 되는시점에 내부적으로 flush가 호출되면서 JPA가 Entity와 스냅샷을 비교한다.
    - 이 때 Entity가 변경되었으면 UPDATE Query를 쓰기 지연 SQL 저장소에 만들어두고, UPDATE Query를 DB에 반영하고 Commit한다.

<br>

- 지연 로딩(Lazy Loading)