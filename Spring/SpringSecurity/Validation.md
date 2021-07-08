# Validation
### Bean Validation
> Java에서는 Bean Validation이라는 이름으로 어노테이션을 데이터 검증을 위한 메타데이터로 사용하는 방법을 제시하고 있다.

### Hibernate Validator
> Bean Validation 명세에 대한 구현체이다.
- Field나 getter에 기본 제공 Annotation을 이용해서 심플하게 validation을 처리할 수 있다.
- Service나 Bean에서 사용하기 위해서는 @Validated와 @Valid를 추가해야 한다.

### 📗 참고
- [Validation 어디까지 해봤니?](https://meetup.toast.com/posts/223)
- [스프링 부트에서 Request 유효성 검사하는 방법, 서버 개발한다면 꼭 해야하는 작업 Spring Validation](https://jeong-pro.tistory.com/203)