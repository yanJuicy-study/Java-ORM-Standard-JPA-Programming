## 엔티티 매니저 팩토리와 엔티티 매니저

```java
// 엔티티 매니저 팩토리 생성
EntityManagerFactory emf =
	Persistence.createEntityManagerFactory("persistence-unit name");
    
// 엔티티 매니저 생성
EntityManger em = emf.createEntityManager();
```

엔티티 매니저 팩토리를 생성하는 비용은 상당히 크다.
따라서 한 개만 만들어서 애플리케이션 전체에서 공유하도록 설계한다.
반면, 엔티티 매니저를 생성하는 비용하는 비용은 거의 들지 않는다.
따라서 엔티티 매니저 팩토리 하나에서 다수의 엔티티 매니저를 생성시킨다.

![](https://velog.velcdn.com/images/hjun0917/post/4b0ec885-eba6-4a84-9ee3-02e12ab3e5df/image.png)


> 엔티티 매니저 팩토리는 여러 스레드가 동시에 접근해도 안전하므로 서로 다른 스레드 간에 공유해도 되지만, 엔티티 매니저는 여러 쓰레드가 동시에 접근하면 동시성 문제가 발생하므로 쓰레드 간에 절대 공유하면 안된다.

#### 동시성 문제에 대하여 ([참고 자료](https://www.inflearn.com/questions/92588/%EC%97%94%ED%8B%B0%ED%8B%B0%EB%A7%A4%EB%8B%88%EC%A0%80-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C))

스프링 프레임워크는 엔티티 매니저를 쓰레드에서 안전하게 동작하도록 지원해준다.

따라서 실무에서 엔티티 매니저에 동시성 문제가 발생하는 일은 거의 없다.

반면에 스프링을 사용하지 않고, 엔티티 매니저를 직접 생성하고 관리할 때는 동시성 문제가 발생할 수 있다. 
하나의 엔티티 매니저를 생성해서 공유변수 같은 곳에 넣어버리면 큰일 나는 것이다.

## 영속성 컨텍스트란?
>JPA 를 이해하는 데 가장 중요한 용어 중 하나는  **영속성 컨텍스트(persistence context)**이다.

직역하자면 **_엔티티를 영구 저장하는 환경_** 이라는 뜻이다.

`persist()` 메서드는 정확히 말하자면
_**엔티티 매니저를 사용해서 엔티티를 영속성 컨텍스트에 저장한다.**_

> 영속성 컨텍스트는 논리적인 개념에 가깝고 눈에 보이지도 않는다.
그리고 엔티티 매니저를 생성할 때 하나 만들어진다.
이 엔티티 매니저를 통해 영속성 컨텍스트에 접근하고 관리한다.
<p style="color:red">여러 엔티티 매니저가 같은 영속성 컨텍스트에 접근할 수도 있다. 지금은 위와 같이 생각하자.</p>

![](https://velog.velcdn.com/images/hjun0917/post/2451e8e2-8d30-4209-9bc2-c1d389e77352/image.png)


## 엔티티의 생명주기

4가지 상태가 존재한다.
- 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 상태
- 영속(managed) : 영속성 컨텍스트에 저장된 상태
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed) : 삭제된 상태

![](https://velog.velcdn.com/images/hjun0917/post/511e0896-c48b-476d-bdbc-51ca8f4d2e5d/image.png)
 
```java
// 객체를 생성한 상태(비영속)
Member member = new Member();

// 객체를 저장한 상태(영속)
em.persist(member);

// 객체를 영속성 컨텍스트에서 분리(준영속)
em.detech(member);

// 객체를 삭제한 상태(삭제)
em.remove(member);
```

## 영속성 컨텍스트의 이점
- 1차 캐시
(1차 캐시에 저장)
![](https://velog.velcdn.com/images/hjun0917/post/f207ed94-3716-457a-995a-c0a387202158/image.png)
(1차 캐시에서 조회)
![](https://velog.velcdn.com/images/hjun0917/post/a59f9782-5604-4428-8b98-79514f2b4f8e/image.png)
(데이터베이스에서 조회)
![](https://velog.velcdn.com/images/hjun0917/post/691d9e52-804a-414a-9c7c-82b2d6520444/image.png)

- 동일성(identity) 보장
1차 캐시로 반복 가능한 읽기(repeatable read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공
- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
트랜잭션 커밋을 하는 순간 `INSERT` SQL 을 보낸다.
```java
// 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
ts.begin();

em.persist(memberA);
em.persist(memberB);
// 아직 INSERT SQL 을 보내지 않음

// 커밋하는 순간 INSERT SQL 을 보냄
ts.commit(); // 트랜잭션 커밋
```
- 변경 감지(dirty checking)
엔티티의 변경을 감지해, 커밋 시점에 데이터베이스와 동기화 시킨다.
```java
// 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
ts.begin();

Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memeberA.setUsername("kim");

// em.update(member) 같은 코드가 있어야하지 않을까 하지만 update 메서드는 존재하지도 않을 뿐더러 필요가 없다.

// 커밋하는 순간 dirty checking 을 통해서 해당 변경사항을 데이터 베이스에 동기화 시킨다.
ts.commit(); 
```

- 지연 로딩(lazy loading)

## 플러시
> 영속성 컨텍스트의 변경내용을 데이터베이스에 반영

- 변경 감지
- 수정된 엔티티 쓰기 지연 -> SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송
(등록, 수정, 삭제 쿼리)

#### 참고
- 영속성 컨텍스트를 비우지 않음
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
- 트랜잭션이라는 작업 단위가 중요! -> 커밋 직전에만 동기화 하면 됨

## 영속성 컨텍스트를 플러시하는 방법
- em.flush() - 직접 호출
- 트랜잭션 커밋 - 플러시 자동 호출
- JPQL 쿼리 실행 - 플러시 자동 호출

## 준영속 상태
- 영속 -> 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
- 영속성 컨텍스트가 제공하는 기능을 사용하지 못함

## 준영속 상태로 만드는 방법
- em.detach(entity)
특정 엔티티만 준영속 상태로 전환

- em.clear()
영속성 컨텍스트를 완전히 초기화

- em.close()
영속성 컨텍스트를 종료
