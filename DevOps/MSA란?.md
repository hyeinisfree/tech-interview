# MSA란?
- 모노리틱(Monolithic) 아키텍처 스타일 : 전통적인 방식의 개발 방법으로, 전체 애플리케이션이 하나로 되어있어서 보통 동일한 개발 툴을 사용해 개발되며, 배포 및 테스트도 하나의 애플리케이션만 수행하면 되기 때문에 개발 및 환경설정이 간단합니다. 또한 각 컴포넌트들이 함수로 호출 되기 때문에 성능에 제약이 덜하고, 운영 관리가 용이합니다. 이런 장점 때문에 작은 볼륨의 시스템을 개발할 때는 매우 유용하지만 시스템이 커지기 시작하고 여러 컴포넌트들이 더해지면 문제가 발생하기 시작합니다.
- MSA(Micro Service Architecture) : 단일 프로그램을 각 컴포넌트 별로 나누어 작은 서비스의 조합으로 구축하는 방법이다. 각 컴포넌트는 서비스 형태로 구현되고 API를 이용하여 타 서비스와 통신하게 됩니다. 각 서비스는 독립된 서버로 타 컴포넌트와 의존성이 없기 때문에 독립된 배포를 하게 됩니다. 또한 각 컴포넌트가 독립된 서비스로 개발되어있기 때문에 부분적인 확장이 가능합니다. 온라인 쇼핑몰에서 주문 서비스에 트래픽이 증가한다면 해당 서버만 확장을 해주면 됩니다.

### 📗 참고
  - [https://www.redhat.com/ko/topics/microservices](https://www.redhat.com/ko/topics/microservices)
  - [http://clipsoft.co.kr/wp/blog/마이크로서비스-아키텍처msa-개념/](http://clipsoft.co.kr/wp/blog/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98msa-%EA%B0%9C%EB%85%90/)
  - [https://webdevtechblog.com/microservice-architecture란-ca9825087050](https://webdevtechblog.com/microservice-architecture%EB%9E%80-ca9825087050)
  - [https://wooaoe.tistory.com/57](https://wooaoe.tistory.com/57)