# Spring Data JPA
- JPA는 인터페이스로서 자바 표준명세서이다.
- 인터페이스인 JPA를 사용하기 위해서는 구현체가 필요하다.
- 대표적으로 Hibernate, Eclipse Link 등이 있다.
- 하지만 Spring에서 JPA를 사용할 때는 이 구현체들을 직접 다루진 않는다.
- 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 Spring Data JPA라는 모듈을 이용하여 JPA 기술을 다룬다.
  - JPA ← Hibernate ← Spring Data JPA
- Hibernate를 쓰는 것과 Spring Data JPA를 쓰는 것 사이에는 큰 차이가 없다. 그럼에도 스프링 진영에서는 Spring Data JPA를 개발했고, 이를 권장하고 있다.
- 이렇게 한 단계 더 감싸놓은 Spring Data JPA가 등자한 이유는 크게 두 가지가 있다.
  - 구현체 교체의 용이성
    - Hibernate 외에 다른 구현체로 쉽게 교체하기 위함
  - 저장소 교체의 용이성
    - 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함

### 📗 참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱