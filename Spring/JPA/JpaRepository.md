# JpaRepository
- domain 패키지에 만든 Entity 클래스로 Database를 접근하게 해준다.
- 보통 ibatis나 MyBatis 등에서 Dao라고 불리는 DB Layer 접근자이다.
- JPA에서는 Repository라고 부르며 인터페이스로 생성한다.
- 단순히 인터페이스를 생성 후, JpaRepository<Entity 클래스, PK 타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성된다.
- @Repository를 추가할 필요도 없다. 여기서 주의할 점은 Entity 클래스와 기본 Entity Repository는 함께 위치해야 하는 점이다. 둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없다.
- 나중에 프로젝트 규모가 커져 도메인별로 프로젝트를 분리해야 한다면 이때 Entity 클래스와 기본 Repository는 함께 움직여야 하므로 도메인 패키지에서 함께 관리한다.

### 📗 참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱