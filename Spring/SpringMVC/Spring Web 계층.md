# Spring Web 계층
### Web Layer
- 흔히 사용하는 컨트롤러(@Controller)와 JSP/Freemarker 등 뷰 템플릿 영역이다.
- 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(@ControllerAdvice) 등 외부 요청과 응답에 대한 전반적인 영역을 이야기한다.

### Service Layer
- @Service에 사용되는 서비스 영역이다.
- 일반적으로 Controller와 Dao의 중간 영역에서 사용된다.
- @Transactional이 사용되어야 하는 영역이기도 하다.

### Repository Layer
- Database와 같이 데이터 저장소에 접근하는 영역이다.
- Dao(Data Access Object) 영역으로 이해하면 쉽다.

### Dtos
- Dto(Data Transfer Object)는 계층 간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역을 얘기한다.
- 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을 이야기한다.

### Domain Model
- 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 한다.
- 이를 테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다.
- @Entity가 사용된 영역 역시 도메인 모델이라고 이해하면 된다.
- 다만, 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아니다.
- VO처럼 값 객체들도 이 영역에 해당하기 때문이다.

### ✅ 추가
- **많은 사람들이 오해하고 있는 것이, Service에서 비즈니스 로직을 처리해야 한다는 것이다.**
- 하지만, 전혀 그렇지 않다. Service는 트랜잭션, 도메인 간 순서 보장의 역할만 한다.
- Web(Controller), Service, Repository, Dto, Domain, 이 5가지 레이어에서 비즈니스 처리를 담당해야 할 곳은 어디일까?
    - 바로 Domain이다.
- 기존에 서비스로 처리하던 방식을 트랜잭션 스크립트라고 한다.
- 모든 로직이 서비스 클래스 내부에서 처리된다. 그러다 보니 서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 하게 된다.
<br></br>
- **절대로 Entity 클래스를 Request/Response 클래스로 사용해서는 안된다.**
- Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스이다. Entity 클래스를 기준으로 테이블이 생성되고, 스키마가 변경된다.
- 수많은 서비스 클래스나 비즈니스 로직들이 Entity 클래스를 기준으로 동작한다. Entity 클래스가 변경되면 여러 클래스에 영향을 끼치지만 Request와 Response용 Dto는 View를 위한 클래스라 정말 자주 변경이 필요하다.
- View Layer와 DB Layer의 역할 분리를 철저하게 하는 게 좋다. 실제로 Controller에서 결괏값으로 여러 테이블을 조인해서 줘야 할 경우가 빈번하므로 Entity 클래스만으로 표현하기 어려운 경우가 많다.
- 꼭 Entity 클래스와 Controller에서 쓸 Dto는 분리해서 사용해야 한다.

### 📗 참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱