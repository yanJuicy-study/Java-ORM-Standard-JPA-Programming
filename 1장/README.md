# JPA 소개

[SQL을 직접 다룰 때 발생하는 문제점](https://www.notion.so/SQL-a76a1a3f81f94405b320d5549acc9005)

[패러다임의 불일치](https://www.notion.so/f6a27b4bd459416ba02899a2ead1dc8c)

SQL을 직접 다룰 때 발생하는 문제점

1. **자바로 개발하는 “애플리케이션”이 관계형 데이터베이스를 저장소로 사용하는 이유?**

→ 가장 대중적이고 신뢰할 만한 안전한 데이터 저장소이기 때문이다.

1. **SQL**

→ 데이터베이스에 데이터를 관리할 때, 쓰는 언어

→ 애플리케이션(자바 기반) → JDBC API 사용 → SQL → 데이터베이스에 전달

---

- **반복, 반복 그리고 또 반복**
    
    **회원 관리 기능**
    
    - 전제
        - 테이블 이미 완성
        - CRUD 기능 개발
    - 상황
        
        **→ 회원 조회, 등록, 삭제 등의 기능을 만들어야 함.**
        
    
    ```java
    public classMember{
        private String memberId;
        private String name;
    }
    ```
    
    ```java
    public classMemberDAO{
    
        public Member find(String memberId) {...} //메소드 찾기 기능
    }
    ```
    
    **회원 조회 기능**
    
    1. 회원 조회용 SQL 작성
    
    `SELECT MEMBER_ID, NAME FORM MEMBER M WHERE MEMBER_ID =?`
    
    1. JDBC API 사용하여 SQL 실행
    
    *`ResultSet* re = stmt.executeQuery(sql);`
    
    1. 조회 결과 Member 객체로 매핑
    
    ```java
    String memberId = rs.getString("MEMBER_ID");
    String name = rs.getString("NAME");
    
    Member member = new Member();
    member.setMemberId(memberId);
    member.setName(name);
    ```
    
    **회원 등록 기능**
    
    1. 회원 등록 기능 추가
    
    ```java
    public class MemberDAO {
    		public Member find(String memberId) {...}
    		public void save (Member member) {...} //추가
    }
    ```
    
    1. 회원 등록용 SQL을 작성
    
    ```java
    String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES(?. ?)";
    ```
    
    1. 회원 객체의 값을 꺼내서 등록 SQL에 전달
    
    ```java
    pstmt.setString(1, member.getMemberId());
    pstmt.setString(2, member.getMember());
    ```
    
    1. JDBC API를 사용해서 SQL을 실행
    
    ```java
    pstmt.executeUpdate(sql);
    ```
    
    **회원 삭제 기능**
    
    …
    
    database는 객체는 달리 data 중심의 구조를 가지므로 **객체를 database에 직접 저장하거나 조회할 수는 없습니다.** 따라서 바로 위 “JDBC API를 사용해서 SQL을 실행” 부분과 같이 **개발자가 객체지향 application과 database 중간에서 SQL과 JDBC API를 사용해서 변환 작업을 직접 해주어야 합니다.** 
    
    그리고!!!! 여기서 또 하나의 문제가 발생합니다. 
    
    **바로 객체를 database에 CRUD하려면 너무 많은 SQL과 JDBC API를 코드로 작성해야 한다는 점입니다. 심지어 테이블 마다 비슷하게 반복해야 합니다.**
    
    예) 100개의 **테이블**을 필요로 하는 어플리케이션이 있음.
    
    → 각 **테이블**에 수많은 **SQL**을 넣고 그렇게 만든 1개의 테이블이 100개 있어야 하는 것입니다.
    
    <코드 예시>
    
    나중에…
    
    결론 → 100개의 테이블에 각각 **SQL**을 넣는 수작업이 매우 비효율적이다.
    
    <궁금한 단어들>
    
    -**애플리케이션**
    
    -**SQL(핵심)**
    
    -**JDBC API**
    
- **SQL에 의존적인 개발**
    
    ```java
    ****<상황극1>****
    
    ****서보성**** : “아~ 회원 관리 기능, 회원 조회 기능, 회원 등록 기능, 회원 삭제 기능 등등… 이제 업무 끝났다^^ 퇴근 해야지~~”
    
    (갑자기 휴대폰에서 ‘지잉지잉~~’ 울리는 소리가 들린다.)
    
    ****서보성**** : “어? 뭐지? 지금 새벽이라 전화올 곳이 없는데…?”
    
    (휴대폰을 보고 그 어느 때보다 명절에 어울리는 자세로 (좌)절을 하게 되었는데…)
    
    ****서보성**** : “아니…”
    
    ****휴대폰 문자**** : “기능 구현 추가입니다. 회원의 연락처도 함께 추가해주세요^^”
    
    (그날 이후로 보성이는 회사에서 볼 수 없었다…)
    ```
    
    위 상황극에서 말하고 싶은 것은 회원의 연락처도 함께 저장해달라는 요구입니다.
    
    멘탈이 나가겠죠? 
    
    그럼 어떤 식으로 멘탈이 나가는지 한번 알아보겠습니다.
    
    **등록 코드 변경 → 테이블에 TEL 컬럼 추가**
    
    ```java
    public class Member {
    		private String memberId;
    		private String name;
    		private String tel; //칼럼(세로열), 추가하는 부분
    }
    ```
    
    <칼럼 적용 예시>
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c0eb30c8-0962-4ea0-b285-9e5a2d22c7c3/Untitled.png)
    
    1. **INSERT SQL 수정(연락처 저장을 위해)**
    
    ```java
    String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES(?,?,?)"
    ```
    
    1. **회원 객체의 연락처 값 꺼냄 → 등록 SQL에 전달**
    
    ```java
    pstmt.setString(3, member.getTel());
    ```
    
    - 연락처 database에 저장을 위해 **SQL과 JDBC API 수정**
    - 테스트 및 database에 연락처 데이터 저장 확인
    
    **조회 코드 변경 → 조회 SQL에 연락처 컬럼 추가하지 않았음.**
    
    1. 회원 조회용 SQL 수정
    
    ```java
    SELECT MEMBER_ID, NAME, TEL FROM MEMBER WHERE MEMBER_ID = ?
    ```
    
    1. 연락처 조회 결과를 Member 객체에 추가로 매핑 → 작성 후 연락처 값이 출력됨.
    
    ```java
    ...
    String tel = rs.getString("TEL");
    member.setTel(tel); //추가 매핑
    ```
    
    **수정 코드 변경**
    
    MemberDAO.update()를 열어 UPDATE SQL을 확인해보니 TEL 컬럼 추가하지 않아 수정되지 않는 문제 발생 → UPDATE SQL과 MemberDAO.update()의 일부 코드를 변경해서 연락처가 수정되도록 함.
    
    → 만약 회원 객체가 database가 아닌 java collection(list or set 등)에 보관했다면 필드를 추가한다고 해서 이렇게 많은 코드를 수정할 필요 없을 것이다.
    
    ```java
    list.add(member); //등록
    Member member = list.get(xxx); //조회
    member.setTel("xxx"); //수정
    ```
    
    **연관된 객체**
    
    추가 요청사항 → 회원은 어느 한 팀에 필수로 소속되어야 함.(Member 객체에 team 필드 추가되어 있음)
    
    1. 회원 정보 출력 시, 연관된 팀이름도 함께 출력
    
    ```java
    class Member {
    		private String memberId;
    		private String name;
    		private String tel;
    		private Team team; //추가
    }
    
    class Team {
    		...
    		private String teamName;
    		...
    }
    ```
    
    1. 팀 이름 출력 → 코드 실행해보니 member.getTeam()의 값이 항상 null 값 → 가설1 : 회원과 연관된 팀이 없다고 판단 후 database 확인 / 모든 회원 팀에 소속됨.
    
    ```java
    member.getName();
    member.getTeam().getTeamName(); //추가
    ```
    
    1. MemberDAO에 추가된 findWithTeam() → MemberDAO 코드 열어서 확인 find() 메소드(회원 출력)는 회원만 조회한 다음 SQL을 그대로 유지했습니다.
    
    ```java
    public class MemberDAO {
    
    		public Member find(String memberId) {...}
    		public Member findWithTeam(String memberId) {...}
    }
    //코드 실행 -> 
    ```
    
    ```java
    SELECT MEMBER_ID, NAME, TEL FROM MEMBER M // SQL
    ```
    
    1. 새로운 findWithTeam() 메소드는 다음 SQL로 회원과 연관된 팀 함께 조회
    
    ```java
    SELECT M.MEMBER_ID, M.NAME, M.TEL, T.TEAM_ID, T.TEAM_NAME
    FROM MEMBER M
    JOIN TEAM T
    		ON M.TEAM_ID = T.TEAM_ID
    ```
    
    **결론**
    
    → DAO를 열어서 SQL을 확인 후 원인 파악 가능, 회원 조회 코드를 public Member find()에서 public Member findWithTeam()으로 변경 후 문제 해결
    
    **단원에 대한 결론**
    
    → Member 객체와 연관된 Team 객체를 사용할 수 있는지 없는지 판단은 SQL을 보고 알 수 있습니다.
    
    여기서 문제가 발생하는데, **이 방식의 큰 문제는 data 접근 계층을 사용해서 SQL을 사용해도 어쩔 수 없이 DAO를 열어서 어떤 SQL이 실행되는지 확인 해야하는 점입니다.**
    
    **이게 뭔가요?**
    
    - **SQL**
        
        **관계형 데이터베이스에 정보를 저장하고 처리하기 위한 프로그래밍 언어**
        
    - **DAO**
        
        database의 data에 접근하기 위한 객체, database 접근을 하기 위한 logic과 **business logic**을 분리하기 위해 사용
        
    
    위와 같이 Member나 Team처럼 비즈니스 요구사항을 모델링한 객체를 entity라고 하는데, 위와 같이 계속해서 data를 열고 값을 넣고 다시 돌리고 또 요구사항이 들어와서 다시 넣고 돌리는 것이 반복되다보면 나중에는 entity를 신뢰 할 수 없게 된다. 왜냐하면 요구사항을 까먹을 수도 있고 협업을 할 때에는 어떤 사람이 어느 값을 넣었는지 알 수 없을 수도 있기 때문이다. 그래서 일일이 DAO를 열어서 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는지 일일이 확인해야 된다.
    
    이것은 진정한 의미의 계층 분할이 아니다. 물리적으로 SQL과 JDBC API를 데이터 접근 계층에 숨기는데 성공했을지는 몰라도 논리적으로 entity와 아주 강한 의존관계를 가지고 있다.
    
    이런 강한 의존관계 때문에 회원을 조회할 때는 물론, 회원 객체에 필드를 하나 추가할 때도 DAO의 CRUD 코드와 SQL 대부분을 변경해야하는 문제가 발생한다.
    
    **application에서 SQL을 직접 다룰 때 발생하는 문제점**
    
    - 진정한 의미의 계층 분할이 어렵다
    - entity를 신뢰할 수 없다.
    - SQL에 의존적인 개발을 피하기 어렵다.
- JPA와 문제 해결
    
    **앞에서 언급한 문제점들**
    
    - 100개의 테이블에 각각 **SQL**을 넣는 수작업이 매우 비효율적이다.
    - 진정한 의미의 계층 분할이 어렵다
    - entity를 신뢰할 수 없다.
    - SQL에 의존적인 개발을 피하기 어렵다.
    
    **앞선 문제점 JPA로 해결하기**
    
    핵심 → JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 되는데, 그러면 JPA가 개발자 대신 적절한 SQL을 생성해서 database에 전달한다.
    
    **JPA가 제공하는 CRUD API**
    
    1. 저장 기능 → persist()
    - persist() method는 객체를 database에 저장한다.
    - persist() 호출 시, JPA가 객체와 매핑정보(어떤 객체를 어떤 테이블에 관리할지 정의한 정보)를 보고, 적절한 INSERT SQL을 생성 → database에 전달
    
    1. 조회 기능 → find()
    - find() method는 객체를 database에 조회한다.
    - find() 호출 시, JPA가 객체와 매핑정보(어떤 객체를 어떤 테이블에 관리할지 정의한 정보)를 보고, 적절한 SELECT SQL을 생성 → database에 전달
    
    1. 수정 기능 → method x
    - 별도 method 제공 x
    - 객체를 조회하여 값을 변경하면 트랜잭션을 커밋할 때 database에 적절한 UPDATE SQL이 전달 된다.
        - transaction
            
            데이터베이스의 상태를 변화시키기 해서 수행하는 작업의 단위
            
            데이터베이스의 상태를 변화시킨다는 것은 무얼 의미하는 것일까?
            
            간단하게 말해서 아래의 질의어(SQL)를 이용하여 데이터베이스를 접근 하는 것을 의미
            
            (SELECT, INSERT, DELETE, UPDATE)
            
        - commit
            
            아직 저장되지 않은 data를 database(db)에 저장하고 transaction을 종료시키는 명령
            
        
    1. 연관된 객체 조회
    - JPA는 연관된 객체 사용 시점에서 적절한 SELECT SQL을 실행한다. → JPA를 사용하면, 연관된 객체 조회 가능
# 패러다임의 불일치

- **상속**
    
    **상속**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4355fad3-a6e2-420c-8cb8-1390e4fa8a79/Untitled.png)
    
    위 그림처럼 객체는 상속이라는 기능을 가지고 있지만, **테이블은 상속이라는 기능이 없다.**
    
    다만, database modeling에서 supertype, subtype 관계를 사용하여 객체 상속과 유사한 형태로 테이블을 설계할 수 있다. 
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0062125-c2f5-4fa5-bc38-e8d1b0c8d64c/Untitled.png)
    
    위 그림과 같이 DTYPE 컬럼을 사용해서 어떤 자식 테이블과 관계 있는지 정의했다.
    
    DTYPE : 타입을 구분하는 컬럼(상속 표현 방법)
    
    예) DTYPE : MOVIE, DTYPE : BOOK 등
    
    **객체 모델 코드 예시** → **모델?**
    
    ```java
    abstract class Item {
    		Long id;
    		String name;
    		int price;
    }
    
    class Album extends Item {
    		String artist;
    }
    
    class Movie extends Item {
    		String director;
    		String actor;
    }
    
    class Book extends Item {
    		String author;
    		String isbn;
    }
    ```
    
    Album 객체 저장하려면 이 객체를 분리한 후 SQL을 만들어야 함.
    
    ```java
    INSERT INTO ITEM...
    INSERT INTO ALBUM...
    ```
    
    Movie 객체도 마찬가지
    
    ```java
    INSERT INTO ITEM...
    INSERT INTO MOVIE...
    ```
    
    JDBC API 사용해서 코드 완성하려면 부모 객체에서 부모 데이터만 꺼내서 item용 INSERT SQL 작성하고 자식 객체에서 자식 data만 꺼내서 ALBUM용 INSERT SQL을 작성해야 하는데, 작성해야 할 코드량이 만만치 않다. 그리고 자식 타입에 따라서 DTYPE도 저장해야 한다.
    
    조회하는 것도 쉬운 일은 아니다. 예를 들어 Album을 조회한다면 ITEM과 ALBUM table을 조인해서 조회한 다음 그 결과로 Album객체를 생성해야 한다.
    
    이런 과정이 모두 패러다임의 불일치를 해결하려고 소모하는 비용이다. 만약 해당 객체들을 database가 아닌 java collection에 보관한다면 다음 같이 부모 자식이나 타입에 대한 고민없이 해당 collection을 그냥 사용하면 된다.
    
    ```java
    list.add(album);
    list.add(movie);
    
    Album album = list.get(albumId);
    ```
    
    **JPA와 상속**
    
    **JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.**
    
    개발자는 마치 java collection에 객체를 저장하듯 JPA에 객체를 저장하면 된다.
    
    - Album 객체 저장 → persist()
    
    ```java
    jpa.persist(album);
    ```
    
    - JPA는 다음 SQL을 실행해서 객체 ITEM, ALBUM 두 table에 나누어 저장
    
    ```java
    INSERT INTO ITEM...
    INSERT INTO ALBUM...
    ```
    
    - Album 객체 조회 → find()
    
    ```java
    String albumId = "id100";
    Album album = jpa.find(Album.class, albumId);
    ```
    
    - JPA는 ITEM과 ALBUM 두 table을 join하여 필요한 data를 조회하고 그 결과를 반환한다.
    
    ```java
    SELECT I.*, A.*
    		FROM ITEM I
    		JOIN ALBUM A ON I>ITEM_ID = A.ITEM_ID
    ```
    
- 연관관계
    
    객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 연관된 객체를 조회한다. 반면 테이블은 외래키를 사용해서 다른 테이블과 연관관계를 가지고 조인을 사용해서연관된 테이블을 조회한다.
    
    참조를 사용하는 객체와 외래 키를 사용하는 관계형 데이터베이스 사이의 패러당임 불일치는 객체 지향 모델링을 거의 포기하게 만들 정도로 극복하기 어렵다. 
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e75f49f-4182-4d25-9d5c-81b46e9bc59a/Untitled.png)
    
    위 그림처럼 Member 객체는 Member.team필드에 Team 객체의 참조를 보관해서 Team객체와 관계를 맺는다. 따라서 이 참조 필드에 접근하면 Member와 연관된 Team을 조회할 수 있다.
    
    ```java
    class Member {
    		Team team;
    		...
    		Team getTeam() {
    				return team;
    		}
    }
    
    class Team {
    		...
    }
    
    member.getTeam(); //Member -> team 접근
    ```
    
    MEMBER 테이블은 MEMBER.TEAM_ID 외래키 컬럼을 사용해서 TEAM 테이블과 관계를 맺는다. 이 외래키를 사용해서 MEMBER 테이블과 TEAM 테이블을 조인하면 MEMBER 테이블과 연관된 TEAM 테이블을 조회할 수 있다.
    
    ```java
    SELECT M.*, T.*
    		FROM MEMBER M
    		JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
    ```
    
    조금 어려운 문제도 있는데, 객체는 참조가 있는 방향으로만 조회할 수 있다. 방금 예에서 member.getTeam()은 가능하지만 반대 방향인 team.getMember()는 참조가 없으므로 불가능하다. 반면에 테이블은 외래키 하나로 MEMBER JOIN TEAM도 가능하지만 TEAM JOIN MEMBER도 가능하다.
    
    **객체를 테이블에 맞추어 모델링**
    
    객체와 테이블의 차이를 알아보기 위해 아래와 같이 객체를 단순히 테이블에 맞추어 모델링해보자.
    
    ```java
    class Member {
    		String id; //MEMBER_ID 컬럼 사용
    		Long teamId; //TEAM_ID FK 컬럼 사용
    		String username; //USERNAME 컬럼 사용
    }
    
    class Team {
    		Long id; // TEAM_ID PK 사용
    		String name; // NAME 컬럼 사용
    }
    ```
    
    MEMBER 테이블의 컬럼을 그대로 가져와서 Member 클래스를 만들었다. 이렇게 객체를 테이블에 맞추어 모델링하면 객체를 테이블에 저장하거나 조회할 때는 편리하다. 그런데 여기서 TEAM_ID 외래 키의 값을 그대로 보관하는 teamId 필드에는 문제가 있다. 관계형 데이터베이스는 조인이라는 기능이 있으므로 외래 키의 값을 그대로 보관해도 된다. 하지만 객체는 연관된 객체의 참조를 보관해야 다음처럼 참조를 통해 연관된 객체를 찾을 수 있다.
    
    ```java
    Team team = member.getTeam();
    ```
    
    특정 회원이 소속된 팀을 조회하는 가장 객체지향적인 방법은 이처럼 참조를 사용하는 것이다.
    
    Member.teamId 필드처럼 TEAM_ID 외래 키까지 관계형 데이터베이스가 사용하는 방식에 맞추면 Member 객체와 연관된 Team 객체를 참조를 통해서 조회할 수 없다. 이런 방식을 따르면 좋은 객체 모델링은 기대하기 어렵고 결국 객체지향의 특징을 잃어버리게 된다.
    
    **객체지향 모델링**
    
    객체는 참조를 통해서 관계를 맺는다. 
    
    ```java
    class Member {
    		String id; //MEMBER_ID 컬럼 사용
    		Team team; //참조로 연관관계를 맺는다.
    		String username;  //USERNAME 컬럼 사용
    
    		Team getTeam() {
    				return team;
    		}
    }
    
    class Team {
    		Long id; //TEAM_ID PK 사용
    		String name; //NAME 컬럼 사용
    }
    ```
    
    Member.team필드를 보면 외래키의 값을 그대로 보관하는 것이 아니라 연관된 Team의 참조를 보관한다.
    
    
