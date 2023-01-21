# EntityMapping

### 엔티티 매핑 소개
- 객체와 테이블 매핑 : @Entity, @Table
- 필드와 컬럼 매핑 : @Column
  - @Column(name = "name") : 데이터베이스 컬럼 명이 name일 때, DB 컬럼명과 변수명이 다를 때 사용
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
- 운영 같은 경우에는 데이터가 엄청 많을 떄 alter를 잘못 치면 시스템이 중단 상태가 될 수도 있다.
- 만약 alter를 한다면 스크립트를 직접짜서 DB에서 테스트를 해보고 문제 없다면 검수 받고 적용하는 것을 권장한다.

### hibernate.hbm2ddl.auto
```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```
- create : 기존테이블 삭제 후 다시 생성(DROP + CREATE)
- create-drop : create와 같으나 종료시점에 테이블 DROP
  - testcase 실행킬 때 보통 사용
- update : 변경분만 반영(운영 DB에는 사용하면 안됨)
  - drop table하지 않고 컬럼을 추가하고 싶을 때(alter)
  - 지우는 것에 대해서는 update 되지 않음
- validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
- node : 사용하지 않음
  - 기능 자체를 주석처리 해도된다.
  

### DDL 생성 기능
- 제약조건 추가 : 회원 이름은 필수, 10자 초과 X
    - @Column(nullable = false, length = 10)
- @Column(unique = true, length = 10)
  - 유니크 제약조건을 넣거나 하는 것들은 실행 자체에 영향을 주진 않고, 단순하게 DDL을 생성하는데 영향을 준다.
- 유니크 제약조건 추가
    - @Table(uniqueConstraints = {@UniqueConstraint(name = "Name_AGE_UNIQUE", columnNames = {"NAME", "AGE"}})
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 주지 않는다.

### 필드와 컬럼 매핑
```java
@Entity
public class Member {
    @Id
    private Long id;

    // DB의 name과 username이 매칭
    @Column(name = "name")
    private String username;
    // Integer쓰면 DB에서 가장 적절한 숫자타입이 만들어진다.
    private Integer age;

    //객체에서 Enum타입을 사용하고 싶을 때 DB에는 Enum타입이 없기 때문에 @Enumerated 어노테이션 사용
    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    // Java와 달리 DB는 날짜/시간/날짜시간이 구분되어 있기 때문에 이것을 명시해줘야된다.
    // DATE/TIME/TIMESTAMP
    @Temporal(TemporalType.TIMESTAMP)
    private Date createDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    // varchar를 넘어서는 큰 컨텐츠를 넣고 싶으면 @Lob사용
    @Lob
    private String description;
    public Member() {
    }
}
```
#### 실행결과
```sql
create table Member (
       id bigint not null,
        age integer,
        createDate timestamp,
        description clob,
        lastModifiedDate timestamp,
        roleType varchar(255),
        name varchar(255),
        primary key (id)
    )
```

### 매핑 어노테이션 정리
- @Column : 칼럼 매핑
- @Temporal : 날짜 타입 매핑
- @Enumerated : enum 타입 매핑
- @Lob : BLOB, CLOB 매핑
- @Transient : DB와 매핑하고 싶지 않을때, 메모리 상에서만 사용하고 싶을 때

### @Column 속성
- name
  - 필드와 매핑할 테이블의 컬럼 이름
  - 기본값은 객체의 필드 이름
- insertable / updatable
  - 등록, 변경 가능 여부(false등록 시 insert/update X)
  - 기본값은 True
- nullable(DDL)
  - null 값의 허용 여부 설정. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.
- unique(DDL)
  - @Table의 uniqueConstraints와 같지만 이름 지정은 할 수 없어 잘 쓰이지 않는다.
  - 보통 사용한다면 이름을 정할 수 있는 @Table에서 유니크 제약조건을 건다.
- length(DDL)
  - 문자 길이 제약조건, String 타입에만 사용 가능
  - 기본값은 255
- columnDefinition(DDL)
  - 데이터베이스 컬럼 정보를 직접 줄 수 있다.
  - ex) varchar(100) default 'EMPTY'
- precision / scale(DDL)
  - BigDecimal(BingInteger) 타입에서 사용한다.
  - precision은 소수점을 포함한 전체 자릿수, scale은 소수의 자릿수
  - double, float 타입은 적용 X

### @Enumerated 속성
- value
  - EnumType.ORDINAL(기본값)
    - enum 순서를 데이터베이스에 저장
    - enum 데이터가 추가, 수정, 삭제 될 시 순서가 달라지므로 운영상 이슈가 발생가능하다.
    - 사용권장X
  - EnumType.STRING
    - enum 이름을 데이터베이스에 저장
    - 사용권장O

### @Temporal 속성
날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용  
LocalDate, LocalDateTime을 사용할 때는 생략 가능(최신 하이버네이트 지원)
- value
  - TemporalType.DATE
    - 날짜, 데이터베이스 date 타입과 매핑
    - ex) 2023-10-11
  - TemporalType.TIME
    - 시간, 데이터베이스 time 타입과 매핑
    - ex) 11:11:11
  - TemporalType.TIMESTAMP: 날짜와 시간, 데이터베이스 timestamp 타입과 매핑
  - ex) 2023-10-11 11:11:11


### @Lob 속성
데이터베이스 BLOB, CLOB 타입과 매핑
- @Lob에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
  - CLOB
    - String, char[], java.sql.CLOB
  - BLOB
    - byte[], java.sql.BLOB

### @Transient 속성
- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용
- @Transient  
    private Integer temp;


## 기본 키 매핑 어노테이션
- @Id
- @GeneratedValue


### 기본 키 매핑 방법
- 직접 할당 : @Id만 사용
- 자동 생성(@GeneratedValue)
  - IDENTITY : 데이터베이스에 위임, MYSQL
  - SEQUENCE : 데이터베이스 시퀀스 오브젝트에 사용, ORACLE
    - @SequenceGenerator 필요
  - TALBE : 키 생성용 테이블 사용, 모든 DB에서 사용
    - @TableGenerator 필요
  - AUTO : 방언에 따라 자동 지정

### IDENTITY 전략
```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```
- 기본 키 생성을 데이터베이스에 위임
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용
    (예: MySQL의 AUTO_INCREMENT)
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
- AUTO_INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID값을 알 수 있음
- IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회
- IDENTITY 에서는 DB에 값을 넣어야 pk를 알 수 있으므로, JPA에서 알 수 있는 방법이 없다. 따라서 IDENTITY에서는 transaction commit시점이 아닌 persist에서 INSERT QUERY를 날린다. 따라서 버퍼링해서 커밋하는 것이 IDENTITY에서는 불가능하다.

### SEQUENCE 전략
```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        // 매핑할 데이터베이스 시퀀스 이름
        sequenceName = "MEMBER_SEQ",
        initialValue = 1, allocationSize = 1)
public class Member{
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```
- 시퀀스에서는 persist 시 다음값을 DB에서 받아오므로 commit시에 버퍼링 가능하다.

### @SequenceGenerator
- 주의 : allocationSize 기본값 = 50
- name
  - 식별자 생성기 이름
  - 기본값 : 필수
- sequenceName
  - 데이터베이스에 등록되어있는 시퀀스 이름
  - 기본값 : hibernate_sequence
- initialValue
  - DDL 생성 시에만 사용된다. 시퀀스 DDL을 생성할 때 처음 1 시작하는 수를 지정한다.
- allocationSize
  - 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨) 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값은 반드시 1로 설정해야 한다.
  - allocationSize만큼 DB에서 미리 확보해 놓고, 웹서버 메모리에서 쌓아간다.
- catalog, schema
  - 데이터베이스 catalog, schema 이름

### TABLE 전략
- 키 생성 전용 테이블 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
- 장점 : 모든 데이터베이스 적용 가능
- 단점 : 성능

```java
create table MY_SEQUENCES (
sequence_name varchar(255) not null,
next_val bigint,
primary key ( sequence_name )
)
```
```java
@Entity
@TableGenerator(
name = "MEMBER_SEQ_GENERATOR",
table = "MY_SEQUENCES",
pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
public class Member {
@Id
@GeneratedValue(strategy = GenerationType.TABLE,
generator = "MEMBER_SEQ_GENERATOR")
private Long id;
```

## @TableGenerator 속성
- name
  - 식별자 생성기 이름
  - 필수생성
- table
  - 키생성 테이블명
  - 기본값 : hibernate_sequence
- pkColumnName
  - 시퀀스 컬럼명
  - 기본값 : sequence_name
- valueColumnNa
  - 시퀀스 값 컬럼명
  - 기본값 : next_val
- pkColumnValue
  - 키로 사용할 값 이름
  - 기본값 : 엔티티 이름
- initialValue
  - 초기 값, 마지막으로 생성된 값이 기준이다.
  - 기본값 : 0
- allocationSize
  - 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)
  - 기본값 : 50
- catalog, schema
  - 데이터베이스 catalog, schema 이름
- uniqueConstraints(DDL)
  - 유니크 제약 조건을 지정할 수 있다.

### 권장하는 식별자 전략
- 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대채키)를 사용하자
  - ex) pk를 주민등록번호를 사용할 떄 ->  다른 테이블에서 pk를 주민등록번호로 쓰고 있음 -> 주민등록번호를 못쓰게된다면? -> 마이그레이션이 엄청나다.
- 권장 : Long형 + 대체키 + 키 생성전략 사용(비즈니스를 키로 끌고오는 것은 권장하지 않는다.)


### 데이터 중심 설계의 문제점
- 객체 설계를 테이블 설계에 맞춘 방식
- 테이블의 외래키를 객체에 그대로 가져옴
- 객체 그래프 탐색이 불가능
- 참조가 없으므로 UML도 잘못되었다.  

-> 테이블 끼리 아이디만 가지고 있으므로 참조가 끊긴다.  
-> id를 가져오는 것이 아니고 진짜 객체를 가져오는 것이 필요하다.
-> 연관관계 매핑 필요
