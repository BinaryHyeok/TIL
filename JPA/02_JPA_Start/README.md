# JPA 사용 전 셋팅

### Maven에서 라이브러리 추가
```java
     <!-- JPA 하이버네이트 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.3.10.Final</version>
        </dependency>
```
hibernate-entitymanger가 필요한 라이브러리를 가져온다.
hibernate가 jpa 인터페이스를 가지고 있음

### H2 데이터베이스 추가
```java
    <!-- H2 데이터베이스 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.199</version>
        </dependency>
```
본 학습에서는 H2 데이터베이스를 이용해서 실습할 예정이므로 라이브러리 추가하였다.

### JPA 설정하기 - persistence.xml
- /META-INF/persistence.xml 위치
- persistence-unit name으로 이름 지정
- javax.persistence로 시작: JPA 표준 속성
- hibernate로 시작: 하이버네이트 전용 속성

#### persistence.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

<!-- persistence-unit은 보통 데이터베이스 당 하나를 만든다. -->
<persistence-unit name="hello">
<properties>
<!-- 필수 속성 -->
<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
<property name="javax.persistence.jdbc.user" value="sa"/>
<property name="javax.persistence.jdbc.password" value=""/>
<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

<!-- 옵션 -->
<property name="hibernate.show_sql" value="true"/>
<property name="hibernate.format_sql" value="true"/>
<property name="hibernate.use_sql_comments" value="true"/>
<!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
</properties>
</persistence-unit>
</persistence>
```

### 데이터베이스 방언
- JPA는 특정 데이터베이스에 종속 X
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
- 방언 : SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
- hibernate.dialect를 통해서 어떤 DB를 쓰는지 파악하고 JPA가 사용하도록 통일 시켜준다.
![image](https://user-images.githubusercontent.com/94176133/211211396-e383b49d-e04a-4637-a81b-6757e6180a2d.png)

