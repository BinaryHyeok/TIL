# 값 타임
## JPA의 데이터 타입 분류
- 엔티티 타입
  - @Entity로 정의하는 객체
  - 데이터가 변해도 식별자로 지속해서 추적 가능
  - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능
- 값 타입
  - int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
  - 식별자가 없고 값만 있으므로 변경시 추적 불가
  - 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

### 값 타입 분류
- 기본값 타입
  - 자바 기본 타입(int, double)
  - 래퍼 클래스(Integer, Long)
  - String
- 임베디드 타입(embedded type, 복합 값 타입)
- 컬렉션 값 타임(collection value type)

### 기본값 타입
- 예) String name, int age
- 생명주기를 엔티티에 의존
  - 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- 값 타입은 공유 X
  - 회원 이름 변경 시 다른 회원의 이름도 함께 변경되면 안됨
- int, double 같은 기본 타입(primitive type)은 절대 공유 X
- 기본 타입은 항상 값을 복사함
- Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경X

### 임베디드 타입(복합 값 타입)
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입(embedded type)이라 함
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함
- int, String과 같은 값 타입

#### 예제
- 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.  

<img width="1337" alt="image" src="https://user-images.githubusercontent.com/94176133/214596812-1d58acf3-73c3-4768-bdea-5bc3ba9c460d.png">

<img width="1296" alt="image" src="https://user-images.githubusercontent.com/94176133/214596881-7fa3e311-f96b-42a6-bbb8-53fdc310cfeb.png">

<img width="1230" alt="image" src="https://user-images.githubusercontent.com/94176133/214596994-19a4a055-5330-4921-ad1e-ce774e6a0f15.png">


### 임베디드 타입 사용법
- @Embeddable: 값 타입을 정의하는 곳에 표시
- @Embedded: 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

### 임베디드 타입의 장점
- 재사용
- 높은 응집도
- Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함

### 임베디드 타입과 테이블 매핑
<img width="979" alt="image" src="https://user-images.githubusercontent.com/94176133/214598917-a16a9607-a21a-438e-8bef-470d82af2f55.png">

- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.
- 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

<br>

### 임베디드 타입과 연관관계
<img width="1287" alt="image" src="https://user-images.githubusercontent.com/94176133/214602762-6dccc62f-ce90-4ffb-8e36-680eb4886808.png">

임베디드 타입이 엔티티를 연관관계로 가질 수도 있다 -> 외래키값만 들고 있으면 되기 떄문

### @AttributeOverride: 속성 재정의
- 한 엔티티에서 같은 값 타입을 사용하면?
- 컬럼 명이 중복됨
- @AttributeOverrides, @AttributeOverride를 사용해서 컬러 명 속성 재정의

### 임베디드 타입과 null
- 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null

<br>

## 값 타입과 불변 객체
