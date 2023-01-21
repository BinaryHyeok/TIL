# 고급 매핑

## 상속관계 매핑
- 관계형 데이터베이스는 상속 관계X
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑 : 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑
<img width="1282" alt="image" src="https://user-images.githubusercontent.com/94176133/213847692-d764636c-554c-4883-ab9b-5c75efa3d8e7.png">

- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
  - 각각 테이블로 변환 -> 조인 전략
  - 통합 테이블로 변환 -> 단일 테이블 전략
    - 하나의 테이블에 모든 컬럼을 넣는다.
  - 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략
    - 공통되는 컬럼을 각각 가지도록 설정

<br>

## 주요 어노테이션
- @Inheritance(strategy=InheritanceType.XXX)
  - JOINED : 조인 전략
  - SINGLE_TABLE : 단일 테이블 전략
  - TABLE_PER_CLASS : 구현 클래스마다 테이블 전략
- @DiscriminatorColumn(name="DTYPE")
  - 넣어두면 엔티티명이 DTYPE이라는 컬럼에 들어간다.
- @DiscriminatorValue("XXX")
  - DTYPE에 들어가는 이름을 변경

<br>
  
## 조인 전략
객체화도 잘되고, 정규화도 되고, 설계입장에서도 깔끔하게 설계된다.
<img width="1414" alt="image" src="https://user-images.githubusercontent.com/94176133/213848016-bfe6153b-79cb-4620-84a5-479bd759be73.png">

  장점
  - 테이블 정규화
  - 외래 키 참조 무결성 제약조건 활용가능
    - 하나의 식별자로 참조가 가능하여 설계가 깔끔해진다.
  - 저장공간 효율화
  
단점
  - 조회시 조인을 많이 사용, 성능 저하
  - 조회 쿼리가 복잡함
  - 데이터 저장시 INSERT SQL 2번 호출

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Item {
 ```
 <img width="190" alt="image" src="https://user-images.githubusercontent.com/94176133/213859137-acfb8cf6-e521-486c-a0da-8bfd0be76248.png">


<br>

## 싱글 테이블 전략
- JPA에서 제공하는 기본값
- 성능이 가장 잘 나오는 방식
- @DiscriminatorColumn이 없어도 DTYPE 자동 생성
  
<img width="475" alt="image" src="https://user-images.githubusercontent.com/94176133/213859309-01da74f9-56f3-42f2-a5bc-26b680da5bb3.png">

장점
- 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.

단점
- 자식 엔티티가 매핑한 컬럼은 모두 null 허용
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 상황에 따라서는 조회 성능이 오히려 느려질 수 있다.

<br>

## 구현 클래스마다 테이블 전략
이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않는 전략!
<img width="1448" alt="image" src="https://user-images.githubusercontent.com/94176133/213859529-0bf5c1eb-b129-4437-93d9-9bf18414402a.png">

장점
- 서브 타입을 명확하게 구분해서 처리할 때 효과적
- not null 제약조건 사용 가능

단점
- 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요)
- 자식 테이블을 통합해서 쿼리하기 어려움

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
```
-> Item을 추상클래스로 변경

<br>

## @MappedSuperclass
공통 매핑 정보가 필요할 때 사용(id, name)
<img width="1171" alt="image" src="https://user-images.githubusercontent.com/94176133/213860311-70f1e6fb-a907-4e5f-9be2-5fcb6d4e7a2d.png">

DB의 모든 테이블에 공통적으로 컬럼을 넣어야 할 때 엔티티 마다 직접 넣지않고 속성만 상속받아 사용할 수 있도록 해주는 것

- 상속관계 매핑 X
- 엔티티X, 테이블과 매핑X
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가(em.find(BaseEntity) 불가)
- 직접 생성해서 사용할 일이 없으므로 **추상 클래스 권장**
- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
- 참고: @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능

```java
// 베이스엔티티
@MappedSuperclass
public abstract class BaseEntity {

	private String createdBy;
	private LocalDateTime createdDate;
	private String lastModifyBy;
	private LocalDateTime lastModifiedDate;
}

// 상속엔티티
@Entity
public class Member extends BaseEntity {
}

@Entity
public class Team extends BaseEntity {
}
```
