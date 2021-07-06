## JPA 어노테이션

### @Entity
- 테이블과 링크될 클래스임을 나타낸다.
- 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭한다.
- ex) SalesManager.java → sales_manager table

### @Id
- 해당 테이블의 PK 필드를 나타낸다.

### @GeneratedValue
- PK의 생성 규칙을 나타낸다.
- 스프링 부트 2.0 에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 된다.

### @Column
- 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다.
- 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.

### ✅ 추가
- **웬만하면 Entity의 PK는 Long 타입의 Auto_increment를 추천한다.(MySQL 기준으로 이렇게 하면 bigint 타입이 된다).**
- 주민등록번호와 같이 비즈니스상 유니크 키나, 여러 키를 조합한 복합키로 PK를 잡을 경우 난감한 상황이 종종 발생한다.
  1. FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황이 발생한다.
  2. 인덱스에 좋은 영향을 끼치지 못한다.
  3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생한다.
  - 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는 것을 추천한다.
<br></br>
- **Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.**
- 자바빈 규약을 생각하면서 getter/setter를 무작정 생성하는 경우가 있다. 이렁헤 되면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경 시 정말 복잡해진다.
- 대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 한다.
- 잘못된 사용 예
  ```java
  public class Order {
    public void setStatus(boolean status) {
      this.status = status
    }
  }

  public void 주문서비스의_취소이벤트() {
    order.setStatus(false)
  }
  ```
- 올바른 사용 예
  ```java
  public class Order {
    public void cancelOrder() {
      this.status = status
    }
  }

  public void 주문서비스의_취소이벤트() {
    order.cancelOrder();
  }
  ```
- 자 그러면 여기서 한 가지 의문이 남습니다. Setter가 없는 이 상황에서 어떻게 값을 채워 DB에 삽입해야 할까요?
  - 기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입하는 것이며, 값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경한느 것을 전제로 한다.
  - 생성자 대신에 @Builder를 통해 제공되는 빌더 클래스를 사용할 수 있다. 생성자나 빌더나 생성 시점에 값을 채워주는 역할은 똑같다. 다만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확히 지정할 수 가 없다.
  - 예를 들어 다음과 같은 생성자가 있다면 개발자가 new Example(b, a)처럼 a와 b의 위치를 변경해도 코드를 실행하기 전까지는 문제를 찾을 수가 없다.
    ```java
    public Example(String a, String b) {
      this.a = a;
      this.b = b;
    }
    ```
  - 하지만 빌더를 사용하게 되면 다음과 같이 어느 필드에 어떤 값을 채워야할지 명확하게 인지할 수 있다.
    ```java
    Example.builder()
        .a(a)
        .b(b)
        .build();
    ```

### 📗 참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱