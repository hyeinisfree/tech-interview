# JPA 설정
### JPA yml 설정
- database-plaform : 어떤 데이터베이스에 접속할 것인지에 대한 명시적 설정이 가능하다.
- open-in-view : true이면 뷰 렌더링 시점에 영속성 컨텍스트가 존재하지 않기 때문에 준영속 상태의 객체의 프록시를 초기화할 수 없다면 영속성 컨텍스트를 오픈된 채로 뷰 렌더링 시점까지 유지하여 뷰 렌더링 시점에서도 영속성 컨텍스트를 다룰 수 있게 다. 즉, 작업 단위를 요청 시작 시점부터 뷰 렌더링 완료 시점까지로 확장한다.
- generate-ddl : true이면 최초 데이터베이스 스키마를 자동 생성한다.

### 하이버네이트 전용 속성
- hibernate.show_sql : 하이버네이트가 실행한 SQL을 출력한다.
- hibernate.format_sql : 하이버네이트가 실행한 SQL을 출력할 때 보기 쉽게 정렬해준다.
- hibernate.use_sql_comments : 쿼리를 출력할 때 주석도 함께 출력한다.
- hibernate.id.new_generator_mappings : JPA 표준에 맞춘 새로운 키 생성 전략을 사용한다.
- hibernate.ddl-auto
  - create : 초기화 과정에서 테이블을 생성한다. 만약 기존에 있었다면 삭제 후 생성한다.
  - create-drop : 초기화 과정에서 테이블을 생성하고, 종료 시점에 만들어진 테이블을 삭제한다. Unit Test 용으로 많이 사용된다.
  - update : 컬럼 추가/삭제와 같이 테이블 변경 사항에 대해서 조치하게 한다. 테이블 생성/삭제는 관여하지 않는다.
  - validate : Entity와 Table Mapping 오류만 점검하고, 오류 발견 시에 Exception을 발생시킨다.
  - none : none
- hibernate.physical_naming_strategy : Java Code내에 Entity와 Attribute를 데이터베이스의 테이블과 컬럼으로 Mapping 할 때 Naming을 어떤 전략으로 다루는 지를 설정한다. 값을 따로 지정하지 않으면 전체 소문자 + Under Score로 단어가 이어지는 Snake-case가 적용되며 이때 사용되는 구현체는 org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy 이다. org.hibernate.boot.model.naming.PhysicalNamingStrategyStadardImpl은 Snake-case로 변환 없이 Naming하게 된다.

### JPA 어플리케이션 코드
```java
import javax.persistence.*;
import java.util.List;

public class JpaMain {
  public static void main(String[] args) {
    //[엔티티 매니저 팩토리] - 생성
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("project name");
    
    //[엔티티 매니저] - 생성
    EntityManager em = emf.createEntityManager();

    //[트랜잭션] - 획득
    EntityTransaction tx = em.getTransaction();

    try {
        tx.begin();     //[트랜잭션] - 시작
        logic(em);      //비즈니스 로직 실행
        tx.commit();    //[트랜잭션] - 커밋
    } catch (Exception e) {
        tx.rollback();  //[트랜잭션] - 롤백
    } finally {
        em.close();     //[엔티티 매니저] - 종료
    }
    emf.close();        //[엔티티 매니저 팩토리] - 종료
  }

  private static void logi(EntityManager em) {...}
}
```

코드는 크게 3가지로 나뉘어 있다.
- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직

### **엔티티 매니저 설정**
- **엔티티 매니저 팩토리 생성**
  spring boot라면 application.yml의 설정 정보를 통해 엔티티 매니저 팩토리를 생성한다. 설정 정보를 통해 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체 따라서 데이터베이스 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용이 아주 크다. 따라서 **엔티티 매니저 팩토리는 애플리케이션 전체에서 한 번만 생성하고 공유해서 사용해야 한다.**

- **엔티티 매니저 생성**
  엔티티 매니저 팩토리에서 엔티티 매니저를 생성한다. 엔티티 매니저를 사용하여 엔티티를 데이터베이스에 CRUD할 수 있다. 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안 된다.

- **종료**
  사용이 끝난 엔티티 매니저와 엔티티 매니저 팩토리는 반드시 종료해야 한다.

### **트랜잭션 관리**
JPA를 사용하여 데이터를 변경하려면 항상 트랜잭션 안에서 이뤄져야 한다. 엔티티 메니저에서 트랜잭션 API를 받아와서 사용한다.
```java
EntityTransaction tx = em.getTransaction(); //트랜잭션 API

  try {
    tx.begin();     //트랜잭션 시작
    logic(em);      //비즈니스 로직 실행
    tx.commit();    //트랜잭션 커밋
  } catch (Exception e) {
    tx.rollback();  //예외 발생 시 트랜잭션 롤백
  }
```

### 비즈니스 로직
```java
private static void logic(EntityManager em) {
  String id = "id";
  Member member = new Member();
  member.setId(id);
  member.setUsername("재두");
  member.setAge(31);

  //등록
  em.persist(member);

  //수정
  member.setAge(20);

  //한 건 조회
  Member findMember = em.find(Member.class, id);
  System.out.println("name = " + member.getUserName() + " age = " + memger.getAge());

  //목록 조회
  String sql = "select m from Member m";
  List<Member> memberList = em.createQuery(sql, Member.class).getResultList();
  System.out.println("member size = " + memberList.size());

  //삭제
  em.remove(member);
}
```

- **등록**
  회원 엔티티 생성 후 `em.persist(member);` 를 실행하여 엔티티를 저장한다. JPA는 회원 엔티티 매핑 정보(어노테이션)을 분석해 (1차 캐시에 없다면) 아래 SQL을 생성하여 데이터베이스에 전달한다.

  ```java
  INSERT INTO MEMBER (ID, NAME, AGE) VALUES ('id', '재두', '31');
  ```

- **수정**
  JPA는 어떤 엔티티가 변경되었는지 추적하는 기능이 있어, 기존에 데이터베이스에 존재하는 값을 변경하는 것이라면 아래와 같은 수정 SQL을 생성하여 데이터베이스에 전달한다.

  ```java
  UPDATE MEMBER SET AGE=20 WHERE ID = 'id';
  ```

- **삭제**
  엔티티를 삭제하려면 엔티티 매니저의 remove() 메소드를 실행하면 된다. 그러면 JPA는 삭제 SQL을 생성하여 데이터베이스에 전달한다.

  ```java
  DELETE FROM MEMGER WHERE ID = 'id';
  ```

- **한 건 조회**
  find() 메소드를 실행하면 JPA에서는 조회할 엔티티 타입과 @Id 어노테이션으로 데이터베이스 테이블의 기본 키와 매핑한 식별자 값을 찾아 (1차 캐시에 해당 값이 없다면) 하나의 엔티티를 조회하는 SQL을 생성하여 데이터베이스에 전달한다. 그리고 조회한 결과 값으로 엔티티를 생성하여 반환한다.

- **목록 조회**
  JPA를 사용하면 개발자는 엔티티 객체 중심으로 개발하고 데이터베이스에 대한 처리는 JPA에 맡겨야 한다. 위 기능들은 모두 SQL을 사용하지 않았으나, 목록 조회와 같은 검색 쿼리는 모든 엔티티 객체를 대상으로 검색해야 하는데 SQL은 테이블을 대상으로 쿼리를 한다. 이 때 JPQL을 사용해야 한다.

- **JPQL(Java Persistence Query Language)**
  SQL을 추상화한 객체지향 쿼리 언어로써 엔티티 객체를 대상으로 쿼리한다. 즉, 클래스와 필드를 대상으로 쿼리한다.

### 📗 참고
- 자바 ORM 표준 JPA 프로그래밍, 김영한