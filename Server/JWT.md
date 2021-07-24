# JWT란?
> RFC 7519 웹 표준
> JSON 객체를 사용해서 토큰 자체에 정보들을 저장하고 있는 Web Token

### JWT 구성
- JWT는 Header, Payload, Signature 3개의 부분으로 구성되어 있다.
  - Header는 Signature를 해싱하기 위한 알고리즘 정보들이 담겨있다.
  - Payload는 서버와 클라이언트가 주고받는, 시스템에서 실제로 사용될 정보에 대한 내용들을 담고 있다.
  - Signature는 토큰의 유효성 검증을 위한 문자열. 이 문자열을 통해 서버에서는 이 토큰이 유효한 토큰인지를 검증할 수 있다.
  
### JWT 장점
  - 중앙의 인증서버, 데이터 스토어에 대한 의존성 없음, 시스템 수평 확장 유리
  - Base64 URL Safe Encoding > URL, Cookie, Header 모두 사용 가능

### JWT 단점
  - Payload의 정보가 많아지면 네트워크 사용량 증가, 데이터 설계 고려 필요
  - 토큰이 클라이언트에 저장, 서버에서 클라이언트의 토큰을 조작할 수 없음