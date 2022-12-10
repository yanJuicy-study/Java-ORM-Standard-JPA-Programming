# 🎯JPA 시작

### 클래스와 테이블 매핑

![](https://velog.velcdn.com/images/hjun0917/post/1b158b40-6117-40cb-9994-ccdbcc6b1492/image.png)

#### @Entity
해당 클래스를 테이블과 매핑한다고 JPA에게 알려준다.

#### `@Table`
엔티티 클래스에 맵핑할 테이블 정보를 알려준다.
(name = "" ) 속성을 통해 테이블에 저장될 테이블 이름을 명시해 줄 수 있다.
이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑한다.
(정확히는 엔티티 이름을 사용한다.)

#### `@Id`
엔티티 클레스의 필드를 테이블의 기본 키(Primary key)에 매핑한다.
해당 어노테이션이 사용된 필드를 식별자 필드라 한다.

#### `@Column`
필드를 컬럼에 매핑한다.
(name = "" ) 속성을 통해 테이블에 저장될 컬럼 이름을 명시해 줄 수 있다.

### JPA 구동 방식

![](https://velog.velcdn.com/images/hjun0917/post/75c02e35-26bc-41af-b2d3-414f6ad9e83d/image.png)


1. 엔티티 매니저 팩토리를 생성
2. 엔티티 매니저를 생성한
3. 사용을 다한 매니저와 팩토리 종료

#### 엔티티 매니저 팩토리
앤티티 매니저 팩토리는 애플리케이션 전체에서 딱 한번만 생성하고 공유하고 사용해야 한다. 

왜??
JPA를 동작시키기 위한 기반 객체를 만들고,
JPA 구현체에 따라서 데이터베이스 커넥션 풀도 생성한다.
즉, 큰 비용이 발생한다.

```java
EntityManagerFactory emf = new EntityManagerFactory("persistence-unit"); 
// persistence-unit 은 메이븐의 xml 폴더에 지정을 하는데 그래들은 어디서 찾는걸까?
```

#### 엔티티 매니저 생성
JPA의 기능 대부분은 엔티티 매니저가 제공한다.
엔티티 매니저를 사용하서 엔티티를 데이터베이스에 등록/수정/삭제/조회할 수 있다.

엔티티 매니저는 내부에 데이터소스(데이터베이스 커넥션)를 유지하며 데이터베이스와 통신하기 때문에, 개발자는 이를 가상의 데이터베이스로 여길 수 있다.

참고로 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안된다.

```java
EntityManager em = emf.creatEntityManager();

```

#### 종료
```java
em.close(); // 엔티티 매니저 종료

emf.close(); // 엔티티 매니저 팩토리 종료
```

### 트랜잭션 관리
JPA를 사용하면 **항상 트랜잭션 안에서 데이터를 변경**해야한다.

```java
EntityTrasaction tx = em.getTransaction(); // 트랜잭션 API
try {
	tx.begin();
    // 비즈니스 로직 실행
    tx.commit();
} catch (Exception e) { 
	tx.rollback();
} finally {
	em.close();
    emf.close();
}

```

### 비즈니스 로직

```java
public static void logic(EntityManager em) {
	String id = "1";
    Member member = new Member();
    member.setId(id);
    member.setUsername("user");
    member.setAge(2);
    
    // 등록
    em.persist(member);
    
    // 수정
    member.setAge(20); // em.update(); X -> 존재하지 않는 메서드
    // em.persist(member) 생략 가능
    
    // 한 건 조회
    Member findMember = em.find(Member.class, id);
    
    // 목록 조회
    List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
    
    // 삭제
    em.remove(member);
}
```

비즈니스 로직을 보면 모든 작업이 엔티티 매니저(em)을 통해 수행되는 것을 볼 수 있다.
즉, 엔티티 매니저를 객체를 저장하는 가상의 데이터베이스로 취급하는 것이다.

JPA는 Member 엔티티의 매핑 정보(어노테이션)를 분석해서 올바른 SQL을 만들어 데이터베이스에 전달한다.

### JPQL
하나 이상의 회원 목록을 조회하는 코드를 보면
SQL과 닮은 쿼리를 사용하는 것을 볼 수 있다.

```java
    // 목록 조회
    List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
```

JPA를 사용하면 개발자는 엔티티 객체를 중심으로 개발하고
데이터베이스에 대한 처리는 JPA에게 맡겨야 한다.
위에서 살펴본 등록, 수정, 삭제, 한건 조회의 예시를 보면 SQL을 전혀 사용하지 않았다.

여기서 문제는 검색 쿼리인데,
JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다.
그런데 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와 엔티티 객체로 만든다음 검색해야하는데, 이는 사실상 불가능하다.

여기서 JPA는 JPQL을 사용한다.
- JPQL : 엔티티 객체를 대상으로 쿼리한다. (클래스와 필드를 대상으로 쿼리한다.)
SQL 문법과 유사하여 `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`등을 사용할 수 있다.
- SQL : 데이터베이스 테이블을 대상으로 쿼리한다.

목록 조회 예제에서 `select m from Member` 가 바로 JPQL 이다.
Member는 엔티티를 말한다. (테이블이 아니다.)
한마디로 JPQL은 데이터베이ㅅ 테이블에 대해 전혀 알지 못한다.

JPQL을 사용하려면 먼저 `em.createQuery("JPQL", 반환 타입)` 메소드를 실행해 쿼리 객체를 생성한 후
쿼리 객체의 `getResuletList()` 메소드를 호출하면 된다.
JPA는 JPQL을 분석해 다음과 같은 적절한 SQL을 만들어 데이터베이스에 요청한다.
```sql
SELECT M.ID, M.USERNAME, M.AGE FROM MEMBER M
```

> JPQL 은 대소문자를 명확하게 구분하고,
SQL 은 관례상 대소문자를 구분하지 않고 사용하는 경우가 많다.
