# <JPA 스터디 4장, 엔티티 매핑>

### JPA 매핑 시 사용하는 Annotatoin

- 객체와 테이블 매핑: `@Entity`, `@Table`
- 기본 키 매핑: `@Id`
- 객체와 컬럼 매핑: `@Column`
- 연관관계 매핑: `@ManyToOne`, `@JoinColumn`

## 1. @Entity

JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 어노테이션을 필수로 붙여야 한다.

### @Entity 주의 사항

- `기본 생성자`는 필수다(파라미터가 없는 public 또는 protected 생성자)
- `final 클래스`, `enum`, `interface`, `inner 클래스`에는 사용할 수 없다.
- 저장할 필드에 `final`을 사용하면 안 된다.

## 2. @Table

`@Table`은 엔티티와 매핑할 테이블을 지정, 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

## 3.  다양한 매핑 사용

```java
package jpabook.start; 
import javax.persistence.*; 
import java.util.Date; 

@Entity 
@Table (name="MEMBER") 
public class Member { 
	
	@Id 
	@Column (name = "ID") 
	private String id; 
	
	@Column (name - "NAME") 
	private String username; 
	
	private Integer age; 

	//== 추가 == 
	@Enumerated (EnumType. STRING) 
	private RoleType roleType; // 1

	@Temporal (TemporalType. TIMESTAMP) 
	private Date createdDate; // 2

	@Temporal (TemporalType. TIMESTAMP)  
	private Date lastModifiedDate; // 2
 
	@Lob 
	private String description; //3
}

//Getter, Setter 
package jpabook.start; 

public enum RoleType { 
	ADMIN, USER
}
```

1. roleType : 자바의 `enum`을 사용하려면 `@Enumerated` 어노테이션으로 매핑해야 한다.
2. createdDate, lastModifiedDate : 자바의 날짜 타입은 `@Temporal`을 사용해서 매핑한다.
3. 회원을 설명하는 필드는 길이 제한이 없다. 따라서 데이터 베이스의 `VARCHAR` 타입 대신에 `CLOB` 타입으로 저장해야 한다. `@Lob`을 사용하면 `CLOB`, `BLOB` 타입을 매핑할 수 있다.
    
    <aside>
    💡 *CLOB?
    
    LOB이란 Large Object의 약자로 대용량 데이터를 저장할 수 있는 데이터 타입. 일반적으로 그래픽, 이미지, 사운드등 비정형 데이터를 저장할때 LOB타입을 사용합니다. 문자형 대용량 데이터는 CLOB나 NCLOB, 그래픽, 이미지, 동영상등의 대이터는 BLOB를 주로 사용.
    
    </aside>
    

## 4. 데이터베이스 스키마 자동 생성

테이블의 유니크 제약조건을 만들어 주는 `@Table`의 `uniqueConstraints` 속성을 사용할 수 있다.

```java
@Entity (name="Member") 
@Table (name="MEMBER", uniqueConstraints = { 
	@UniqueConstraint( //추가 
		name = "NAME AGE UNIQUE", 
		columnNames = {"NAME", "AGE"} )}) 
public class Member { 

	@id 
	@Column (name = "id") 
	private String id; 
	
	@column (name = "name") 
	private String username; 
	
	private Integer age;
	...
}
```

## 5. 기본 키 매핑

데이터베이스마다 기본 키를 생성하는 방식을 해결하기 위한 JPA의 해결법

- 직접 할당 : 기본 키를 애플리케이션에서 직접 할당
- 자동 생성 : 대리키 사용
    - `IDENTITY` : 기본 키 생성을 데이터베이스에 위임한다.
    - `SEQUENCE` : 데이터베이스 시퀀스를 사용해서 기본 키를 할당한다.
    - `TABLE` : 키 생성 테이블을 사용한다.

오라클 데이터베이스는 시퀀스를 제공하지만 `MYSQL`은 시퀀스를 제공하지 않는다. 대신에 `MySQL`은 기본 키 값을 자동으로 채워주는 `AUTO_INCREMENT` 기능을 제공한다. 따라서 `SEQUENCE`나 `IDENTITY` 전략은 사용하는 데이터베이스에 의존한다.

### 5-1 기본 키 직접 할당 전략

@Id 적용 가능 자바 타입

- **자바 기본형**
- **자바 래퍼(wrapper)형**
- **String**
- **java.util.Date**
- **java.sql.Date**
- **java.math.BigDecimal**
- **java.math.BigInteger**

### 5-2 **IDENTITY 전략**

`IDENTITY`는 기본 키 생성을 데이터베이스에 위임하는 전략. `IDENTITY` 전략은 지금 설명한 AUTO INCREMENT를 사용한 예제처럼 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.

```java
private static void logic (EntityManager em) { 
	Board board = new Board(); em.persist (board); 
	System.out.println("board.id = " + board.getId()); 
}
// board.id = 1
```

<aside>
💡 엔티티가 영속 상태가 되려면 식별자가 반드시 필요하다. 그런데 `IDENTITY`식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 `em.persist()`를 호출하는 즉시 `INSERT SQL`이 데이터베이스에 전달된다. 따라서 이 전략은 트랜잭션을 지원하는 `쓰기 지연이` 동작하지 않는다.

</aside>

### 5-3 SEQUENCE 전략

```java
@Entity
@SequenceGenerator(
	name = "BOARD_SEQ_GENERATOR".
	sequenceName = ”BOARD_SEQ”, //매핑할 데이터베이스 시퀀스 이름
	initialvalue = 1, 
	allocationsize = 1)
public class Board {

	@IdQGeneratedValue(
	strategy = GenerationType.SEQUENCE,
	generator = "BOARD_SEQ_GENERATOR")
	private Long id;
	...
}
```

`sequenceName` 속성의 이름으로 `BOARD_SEQ`를 지정 했는데 JPA는 이 시퀀스 생성기를 실제 데이터베이스의 BOARD_SEQ 시퀀스와 매핑한다.

<aside>
💡 SquenceGenerator.`allocationSize`의 기본값이 `50`인 이유는 최적화 때문이다 allocationSize 값이 50이면 시퀀스를 한 번에 50 증가 시킨 다음에 1~50까지는 메모리에서 식별자를 할당한다. 이 최적화 방법은 시퀀스 값을 선점 하므로 여러 `JVM`이 동시에 동작 해도 기본 키 값이 충돌하지 않는 장점이 있다. 반면에 데이터베이스에 직접 접근해서 데이터를 등록할 때 시퀀스 값이 한번에 많이 증가한다는 점을 염두해 두어야 한다. 참고로 앞서 설명한 `hibernate.id.new_generator_mappings` 속성을`true`로 설정해야 지금까지 설명한 최적화 방법이 적용된다.

</aside>

### 5-3 **TABLE 전략**

TABLE 전략은 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다.

```java
@Entity
@TableGenerator(
	name = "BOARD_SEQ_GENERATOR",
	table = ”MY_SEQUENCES",
	pkColumnValue = ”BOARD_SEQ”, 
	allocationsize = 1)
public class Board {
	@Id
	@GeneratedValue(
		strategy = GenerationType.TABLE,
		generator = '' BOARD_SEQ_GENERATOR''
	)
	private Long id;
	...
}
```

<aside>
💡 TABLE 전략과 최적화 TABLE 전략은 값을 조회하면서 `SELECT 쿼리`를 사용하고 다음 값으로 증가시키기 위해 `UPDATE 쿼리`를 사용한다. 이 전략은 `SEQUENCE 전략`과 비교해서 데이터베이스와 한번 더 통신하는 단점이 있다. TABLE 전략을 최적화하려면 `TableGenerator.allocationSize`를 사용하면 된다.

</aside>

### 5-4 **AUTO 전략**

`GenerationType.AUTO`는 선택한 데이터베이스 방언에 따라 `IDENTITY`, `SEQUENCE`, `TABLE` 전략 중 하나를 자동으로 선택한다. AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것이다. `AUTO`를 사용할 때 `SEQUENCE`나 `TABLE` 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 한다.

### 5-5**기본 키 매핑 정리**

`em.persist()`를 호출한 직후에 발생하는 일을 식별자 할당 전략별로 정리하면 다음과 같다.

- 직접 할당 : em.persist()를 호출하기 전에 애플리케이션에서 직접 식별자 값을 할당해야 한다. 만약 식별자 값이 없으면 예외가 발생한다.
- SEQUENCE : 데이터베이스 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- TABLE : 데이터베이스 시퀀스 생성용 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- IDENTITY : 데이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다. (IDENTITY 전략은 테이블에 데이터를 저장해야 식별자 값을 획득할 수 있다.)

## 6. 필드와 컬럼 매핑

**Hibrernate - DDL Auto 속성**

| 옵션 | 설 |
| --- | --- |
| create | 기존 테이블을 삭제하고 새로 생성한다. DROP + CREATE |
| create-drop | create 속성에 추가로 애플리케이션을 종요할 때 생성한 DDL을 제거한다. DROP + CREATE + DROP |
| update | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다 |
| validate | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. |
| none | 자동 생성 기능을 사용하지 않는다. |

### 6-1 **@Column**

@Column은 객체 필드를 테이블 컬럼에 매핑한다. insertable, updatable 속성은 데이터베이스에 저장되어 있는 정보를 읽기만 하고 실수로 변경하는 것을 발지하고 싶을 때 사용한다.

### 6-2 **@Enumerated**

`EnumType.ORDINAL`은 enum에 정의된 순서대로 ADMIN은 0, USER는 1 값이 데이터베이스에 저장된다.장점 : 데이터베이스에 저장되는 `데이터 크기`가 작다단점 : 이미 저장된 enum의 `순서`를 변경할 수 없다

`EnumType.STRING`은 enum 이름 그대로 ADMIN은 'ADMIN', usersms 'USER'라는 문자로 데이터베이스에 저장된다.장점 : 저장된 enum의 순서가 바뀌거나 enum이 추가되어도 `안전`하다단점 : 데이터베이스에 저장되는 데이터 크기가 ORDINAL에 비해서 크다.

### 6-3 @Temporal

날짜 타입을 매핑할 때 사용한다.

`@Temporal`을 생략하면 자바의 Date와 가장 유사한 timestamp로 정의된다.

```java
@Temporal(TemporalType.DATE)
private Date date; //날짜

@Temporal(TemporalType.TIME)
private Date time; //시간

@Temporal(TemporalType.TIMESTAMP)
private Date timestamp; "날짜와시간

//== 생성된 DDL==//
date date,
time time,
timestamp timestamp,
```

### 6-4 **@Lob**

@Lob에는 지정할 수 있는 속성이 없다. 대신에 매핑하는 필드 타입이 문자면 `CLOB`으로 매핑하고 나머지는 BLOB으로 매핑한다.

### 6-5 **@Transient**

객체에 `임시로` 어떤 값을 보관하고 싶을 때 사용한다.

```java
@Transient
private Integer temp;
```

### 6-6 **@Access**

@Access를 설정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다.

- 필드 접근 : `AccessType.FIELD`로 지정한다. 필드에 직접 접근한다. 필드 접근 권한이 `private`이어도 접근할 수 있다.
- 프로퍼티 접근 : `AccessType.PROPERTY`로 지정한다. 접근자(Getter)를 사용한다.

## 7. 데이터 중심 설계의 문제점

<aside>
💡 데이터 중심 설계의 문제점
엔티티 설계가 이상하다는 생각이 들었다면 객체지향 설계를 의식하는 개발자고, 그렇지 않고 자연스러웠다면 데이터 중심의 개발자일 것이다. 특히 테이블의 외래키를 개체에 그대로 가져온 부분이 문제이다. 왜냐하면 관계형 데이터베이스는 연관된 객체를 찾을 때 외래 키를 사용해서 조인하면 되지만 객체에는 조인이라는 기능이 없다. 객체는 연관된 객체를 찾을 때 참조를 사용해야 한다.

</aside>