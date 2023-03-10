# 다양한 연관관계 매핑

## 연관관계 매핑시 고려사항 3가지
- 다중성
- 단방향, 양방향
- 연관관계 주인

### 다중성
- 다대일 : @ManyToOne
- 일대다 : @OneToMany
- 일대일 : @OneToOne
- 다대다 : @ManyToMany
  - 실무에서 쓰면 안된다.

### 단방향, 양방향
- 테이블
  - 외래 키 하나로 양쪽 조인 가능
  - 사실 방향이라는 개념이 없음

- 객체
  - 참조용 필드가 있는 쪽으로만 참조가능
  - 한쪽만 참조하면 단방향
  - 양쪽이 서로 참조하면 양방향


### 연관관계의 주인
- 테이블은 외래 키 하나로 두 테이블이 연관관계를 맺음
- 객체 양방향 관계는 A -> B, B -> A 처럼 참조가 2곳
- 객체 양방향 관계는 참조가 2군데 있음. 둘 중 테이블의 외래 키를 관리할 곳을 지정해야 한다.
- 연관관계의 주인 : 외래 키를 관리하는 참조
- 주인의 반대편 : 외래 키에 영향을 주지 않음, 단순 조회만 가능

<br>

## 다대일[N : 1]
- ### 다대일 단방향
    <img width="1032" alt="image" src="https://user-images.githubusercontent.com/94176133/213225387-60fc0196-cdda-4ed7-b298-e3477b754622.png">

    - 가장 많이 사용하는 연관관계
    - 다대일의 반대는 일대다

<br>

- ### 다대일 양방향
    <img width="1036" alt="image" src="https://user-images.githubusercontent.com/94176133/213226251-b2888401-c9e8-4fb6-8dcd-1704a8d6bfad.png">

    - 외래 키가 있는 쪽이 연관관계의 주인
    - 양쪽을 서로 참조하도록 개발

<br>

## 일대다[1 : N]
- ### 일대다 단방향
    <img width="1159" alt="image" src="https://user-images.githubusercontent.com/94176133/213227342-56bf4f14-0823-4b21-aa5b-504a75698e1b.png">

    - 일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인
    - 테이블 일대다 관계는 항상 다(N)쪽에 외래 키가 있음
    - 객체과 테이블의 차이 떄문에 반대편 테이블의 외래 키를 관리하는 특이한 구조
    - @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 조인 테이블 방식을 사용함(중간에 테이블을 하나 추가)
    - 일대다 단방향 매핑의 단점
      - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
      - 연관관계 관리를 위해 추가로 UPDATE SQL 실행
    - 일대다 단방향 매핑보다는 **다대일 양방향 매핑을 사용**하자


<br>

- ### 일대다 양방향
    <img width="1161" alt="image" src="https://user-images.githubusercontent.com/94176133/213234990-5e394b1d-c673-458d-92eb-b2adba9ffc5d.png">

    - 이런 매핑은 공식적으로는 존재하지 않는다.
    - @JoinColumn(insertable=false, updatable=false)
    - 읽기 전용 필드를 사용해서 양방향처럼 사용하는 방법
    - **다대일 양방향을 사용**하자

<br>

## 일대일[1:1]
- 일대일 관계는 그 반대도 일대일
- 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
  - 주 테이블에 외래 키
  - 대상 테이블에 외래 키
- 외래 키에 데이터베이스 유니크(UNI) 제약조건 추가

- 일대일 : 주 테이블에 외래 키 단방향
  <img width="1125" alt="image" src="https://user-images.githubusercontent.com/94176133/213237609-9da42ce9-98f0-4d2e-a19c-bce932dc6acd.png">
  
  - 다대일(@ManyToOne) 단방향 매핑과 유사

<br>

- 일대일 : 주 테이블에 외래 키 양방향
    <img width="1114" alt="image" src="https://user-images.githubusercontent.com/94176133/213238904-7aeb09ad-fcba-45ec-852b-d1a422efd697.png">

    - 다대일 양방향 매핑 처럼 외래 키가 있는 곳이 연관관계의 주인
    - 반대편은 mappedBy 적용

<br>

- 일대일 : 대상 테이블에 외래 키 단방향
    <img width="1014" alt="image" src="https://user-images.githubusercontent.com/94176133/213240709-63182dda-edd5-4c75-b0f1-af059e24aa0c.png">

    - 단방향 관계는 JPA 지원 X
    - 양방향 관계는 지원

<br>

- 일대일 : 대상 테이블에 외래 키 양방향
    <img width="1033" alt="image" src="https://user-images.githubusercontent.com/94176133/213241113-4d2730c6-6a37-459c-86f5-8cfc48ad1863.png">

    - 일대일 주 테이블에 외래 키 양방향과 매핑 방법은 없음

<br>

## 일대일 정리
- 주 테이블에 외래 키
  - 주 객체가 대상 객체의 참조를 가지는 것 처럼, 주 테이블에 외래 키를 두고 대상 테이블을 찾음
  - 객체지향 개발자 선호
  - JPA 매핑 편리
  - 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
  - 단점 : 값이 없으면 외래 키에 null 허용
- 대상 테이블에 외래 키
    - 대상 테이블에 외래 키가 존재
    - 전통적인 데이터베이스 개발자 선호
  - 장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
  - 단점 : 프록시 기능의 한계로 **지연 로딩으로 설정해도 항시 즉시 로딩됨**

<br>

## 다대다
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
- 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야함
- 객체는 컬렉션을 이용해서 객체 2개로 다대다 관계 가능

### 다대다 매핑의 한계
- 편리해 보이지만 사용X
- 연결 테이블이 단순히 연결만 하고 끝나지 않음
- 주문시간, 수량 같은 데이터가 들어올 수 있음


### 대다대 한계 극복
- 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격)
- @ManyToMany -> @OneToMany, @ManyToOne


### N:M 관계는 1:N, N:1로
- 테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1
- 실전에서는 중간 테이블이 단순하지 않다.
- @ManyToMany는 제약 : 필드 추가 X, 엔티티  테이블 불일치
- 실전에서는 @ManyToMany 사용X

<br>

### @JoinColumn
- 외래 키를 매핑할 때 사용
- name
  - 매핑할 외래 키 이름
  - 기본값 : 필드명_참조하는 테이블의 기본키 컬럼명
- referencedColumnName
  - 외래 키가 참조하는 대상 테이블의 컬럼명
  - 기본값 : 참조하는 테이블의 기본 키 컬럼명
- foreignKey(DDL)
  - 외래 키 제약조건을 직접 지정할 수 있다. 이 속성은 테이블을 생성할 때만 사용한다.
- unique, nullable insertable, updatable, columnDefinition, table
  - @Column의 속성과 같다.

<br>

### @ManyToOne
- 다대일 관계 매핑
- optional
  - false로 설정하면 연관된 엔티티가 항상 있어야 한다.
  - 기본값 : TRUE
- fetch
  - 글로벌 페치 전략을 설정한다.
  - 기본값 : @ManyToOne=FetchType.EAGER, @OneToMany=FetchType.LAZY
- cascade
  - 영속성 전이 기능을 사용한다.
- targetEntity
  - 연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다.

### @OneToMany
- 다대일 관계 매핑
- mappedBy
  - 연관관계 주인 필드를 선택한다.
- fetch
  - 글로벌 페치 전략을 설정한다.
  - 기본값 : @ManyToOne=FetchType.EAGER, @OneToMany=FetchType.LAZY
- cascade
  - 영속성 전이 기능을 사용한다.
- targetEntity
  - 연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다.