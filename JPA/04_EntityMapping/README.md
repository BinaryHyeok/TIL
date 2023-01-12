# EntityMapping

### 엔티티 매핑 소개
- 객체와 테이블 매핑 : @Entity, @Table
- 필드와 컬럼 매핑 : @Column
- 기본 키 매핑 : @Id
- 연관관계 매핑 : @ManyToOne, @JoinColumn

### Entity
- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
- 주의
    - 기본 생성자는 필수(파라미터가 없는 public 또는 protected 생성자)
    - final 클래스, enum, interface, inner 클래스 사용 X
    - 저장할 필드에 final 사용 X

### DDL
데이터 정의어(Data Defination Language, DDL)  
데이터베이스의 테이블의 생성, 변경, 삭제를 담당하는 명령어  
대표적으로 CREATE, ALTER, DROP, RENAME, TRUNCATE가 있다.

### 데이터베이스 스키마 자동 생성
- DDL을 애플리케이션 실행시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

### 데이터베이스 스키마 자동 생성 - 주의
- 운영 장비에는 절대 create, create-drop, update 사용하면 안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none

### hibernate.hbm2ddl.auto
- create : 기존테이블 삭제 후 다시 생성(DROP + CREATE)
- create-drop : create와 같으나 종료시점에 테이블 DROP
- update : 변경분만 반영(운영 DB에는 사용하면 안됨)
- validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
- node : 사용하지 않음 

### DDL 생성 기능
- 제약조건 추가 : 회원 이름은 필수, 10자 초과 X
    - @Column(nullable = false, length = 10)
- 유니크 제약조건 추가
    - @Table(uniqueConstraints = {@UniqueConstraint(name = "Name_AGE_UNIQUE", columnNames = {"NAME", "AGE"}})
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 주지 않는다.