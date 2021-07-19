# Spring Rest Docs
- Spring REST Docs의 목표는 RESTful 서비스에 대한 정확하고 읽기 쉬운 문서를 생성하도록 돕는 것입니다.
- 고품질 문서를 작성하는 것은 어렵습니다. 그 어려움을 완화하는 한 가지 방법은 작업에 잘 맞는 도구를 사용하는 것입니다. 이를 위해 Spring REST Docs는 기본적으로 [Asciidoctor](https://asciidoctor.org/) 를 사용한다 . Asciidoctor는 일반 텍스트를 처리하고 필요에 맞게 스타일이 지정되고 배치된 HTML을 생성합니다. 원하는 경우 Markdown을 사용하도록 Spring REST Docs를 구성할 수도 있습니다.
- Spring REST Docs는 Spring MVC의 [테스트 프레임워크](https://docs.spring.io/spring-framework/docs/5.0.x/spring-framework-reference/testing.html#spring-mvc-test-framework) , Spring WebFlux `[WebTestClient](https://docs.spring.io/spring-framework/docs/5.0.x/spring-framework-reference/testing.html#webtestclient)`또는 [REST Assured 3로](http://rest-assured.io/) 작성된 테스트에서 생성된 스니펫을 사용합니다 . 이 테스트 기반 접근 방식은 서비스 문서의 정확성을 보장하는 데 도움이 됩니다. 스니펫이 올바르지 않으면 이를 생성하는 테스트가 실패합니다.
- RESTful 서비스를 문서화하는 것은 주로 해당 리소스를 설명하는 것입니다. 각 리소스 설명의 두 가지 핵심 부분은 소비하는 HTTP 요청의 세부 정보와 생성하는 HTTP 응답입니다. Spring REST Docs를 사용하면 이러한 리소스와 HTTP 요청 및 응답으로 작업하여 서비스 구현의 내부 세부 사항으로부터 문서를 보호할 수 있습니다. 이 분리는 구현이 아닌 서비스의 API를 문서화하는 데 도움이 됩니다. 또한 문서를 다시 작업하지 않고도 구현을 발전시킬 수 있습니다.

### 📗 참고

- 공식 문서 : [https://docs.spring.io/spring-restdocs/docs/2.0.5.RELEASE/reference/html5/#getting-started](https://docs.spring.io/spring-restdocs/docs/2.0.5.RELEASE/reference/html5/#getting-started)
- 공식 가이드 : [https://spring.io/guides/gs/testing-restdocs/](https://spring.io/guides/gs/testing-restdocs/)
- [https://gaemi606.tistory.com/entry/Spring-Boot-REST-Docs-적용하기](https://gaemi606.tistory.com/entry/Spring-Boot-REST-Docs-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)
- [https://velog.io/@max9106/Spring-Spring-rest-docs를-이용한-문서화](https://velog.io/@max9106/Spring-Spring-rest-docs%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%AC%B8%EC%84%9C%ED%99%94)