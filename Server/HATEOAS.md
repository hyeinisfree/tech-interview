# HATEOAS란?
- HATEOAS는 Hypermedia As The Engine Of Application State의 약자로 하이퍼미디어를 REST API 의 상태 정보를 관리하기 위한 매커니즘으로 활용하는 것을 말합니다.
- HATEOAS를 쓰는 이유는 다음과 같은 기존 REST API의 단점을 보완하기 위해서입니다.
  1. REST API는 앤드포인트 URL이 정해지고 나면 이를 변경하기 어렵다는 단점이 있습니다. 만일 API의 URL을 변경하게 되면 모든 클라이언트의 URL까지 수정해야하기 때문에 번거로워지므로 기존 다른 API를 지속적으로 추가하게 됩니다. 따라서 URL 관리가 어렵게 됩니다.
  2. 전달받은 정적 자원의 상태에 따른 요소를 서버 단에서 구현하기 어렵기 때문에 클라이언트 단에서 이 부분에 대한 로직을 처리해야 합니다.
  - 위 단점들을 links 요소를 통해 href 값의 형태로 보내주기 때문에 자원 상태에 대한 처리를 링크에 있는 URL을 통해 처리할 수 있게됩니다.
- Hateoas란 REST Api를 사용하는 클라이언트가 전적으로 서버와 동적인 상호작용이 가능하도록 하는 것을 의미합니다. 이러한 방법은 클라이언트가 서버로부터 어떠한 요청을 할 때, 요청에 필요한 URI를 응답에 포함시켜 반환하는 것으로 가능하게 할 수 있습니다.
- 클라이언트가 요청한 URI를 통해 응답 값을 보낼 경우. 응답 값과 관련된 내용을 알려준다는 것이다. 예를 들자면, 클라이언트가 GET 메소드로 URI 주소 '/member/1'를 호출한다면 사용자 ID가 1인 '사용자 정보'를 갖고온다고 가정해보자. 이때 동일한 URI로 DELETE 메소드를 호출 할 경우 사용자 삭제가 가능하고, PUT 메소드를 호출할 경우 업데이트라고 한다면 일일히 클라이언트 쪽에 알려줘야 한다는 번거로움이 발생한다. 이것을 줄이고자 호출한 URI로부터 연관된 REST API 주소 정보들을 함께 보내주는 역할을 하는 것이 HATEOAS 이다. 그럼으로써 클라이언트는 서버와 상호 작용하는 방법에 대한 사전 지식이 거의 또는 전혀 필요없이 사용 할 수 있게 되는 것이다.
- HATEOAS는 궁극적으로 등산로의 안내 표지판 같다는 생각이 든다. 각각의 지점마다 가야할 방향과 거리를 알려주는 표지판처럼 각각의 API마다 관련 URI 주소와 내용을 전달해줌으로써 클라이언트가 보다 손쉽게 개발 할 수 있도록 도와주는 역할을 하는 것이다.

### Hateoas 사용 예시
이제 대략적으로 Hateoas, REST API가 무엇인지 알았으니, 해당 개념에 대한 예제를 보고 더욱 구체적으로 이해해 보자.
<br></br>
계좌번호가 "12345"인 계좌의 정보를 조희 하는 경우에 해당 계좌의 상태 ( 잔여 금액 등등..)에 따라 접근 가능한 추가 API들이 LINKS라는 이름으로 제공된다.
<br></br>
![https://blog.kakaocdn.net/dn/bkfAkR/btqYZ5QYPBH/kxfgzK2fwDMa1tMzm8SDCK/img.png](https://blog.kakaocdn.net/dn/bkfAkR/btqYZ5QYPBH/kxfgzK2fwDMa1tMzm8SDCK/img.png)
기존의 전형적인 REST API 응답
<br></br>
![https://blog.kakaocdn.net/dn/rzVz6/btqY5MDjtur/YgmaTlZjevjwZN9nkpM2m1/img.png](https://blog.kakaocdn.net/dn/rzVz6/btqY5MDjtur/YgmaTlZjevjwZN9nkpM2m1/img.png)
Hateoas가 적용된 응답 ( URI 정보를 응답에 추가)
<br></br>
위와 같이 Hateoas를 사용하게 되면 누릴 수 있는 장점은 아래와 같다
- Client 사이드에서는 "rel"의 이름으로 요청 URI를 사용하기 때문에 URI 수정이 발생하더라도 Client 사이드는 수정이 이루어지지 않는다.