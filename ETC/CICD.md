# CI/CD란?
- CI : 개발자를 위한 자동화 프로세스인 지속적인 통합(Continuous Integration)을 의미합니다.개발자를 위한 자동화 프로세스인 지속적인 통합(Continuous Integration)을 의미합니다. CI를 성공적으로 구현할 경우 애플리케이션에 대한 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 공유 리포지토리에 통합되므로 여러 명의 개발자가 동시에 애플리케이션 개발과 관련된 코드 작업을 할 경우 서로 충돌할 수 있는 문제를 해결할 수 있습니다.
- CD : 지속적인 서비스 제공(Continuous Delivery) 및/또는 지속적인 배포(Continuous Deployment)를 의미하며 이 두 용어는 상호 교환적으로 사용됩니다. 두 가지 의미 모두 파이프라인의 추가 단계에 대한 자동화를 뜻하지만 때로는 얼마나 많은 자동화가 이루어지고 있는지를 설명하기 위해 별도로 사용되기도 합니다.
<br></br>
- 코드 버전 관리를 하는 VCS 시스템(Git, SVN 등)에 PUSH가 되면 자동으로 테스트와 빌드가 수행되어 안정적인 배포 파일을 만드는 과정을 CI(Continuous Integration - 지속적 통합)라고 하며, 이 빌드 결과를 자동으로 운영 서버에 무중단 배포가지 진행되는 과정을 CD(Continuous Deployment - 지속적인 배포)라고 한다.
<br></br>
**✅ 주의할 점**
- 단순히 CI 도구를 도입했다고 해서 CI를 하고 있는 것은 아니다. 마틴 파울러의 블로그(http://bit.ly/2Yv0vFp)를 참고해 보면 CI에 대해 다음과 같은 4가지 규칙을 이야기한다.
  1. 모든 소스 코드가 살아 있고(현재 실행되고) 누구든 현재의 소스에 접근할 수 있는 단일 지점을 유지할 것
  2. 빌드 프로세스를 자동화해서 누구든 소스로부터 시스템을 빌드하는 단일 명령어를 사용할 수 있게 할 것
  3. 테스팅을 자동화해서 단일 명령어로 언제든지 시스템에 대한 건전한 테스트 수트를 실행할 수 있게 할 것
  4. 누구든 현재 실행 파일을 얻으면 지금까지 가장 완전한 실행 파일을 얻었다는 확신을 하게 할 것
  - 여기서 특히나 중요한 것은 테스팅 자동화이다. 지속적으로 통합하기 위해서는 무엇보다 이 프로젝트가 완전한 상태임을 보장하기 위해 테스트 코드가 구현되어 있어야만 한다.
<br></br>
## CI 도구
### Jenkins
- 무료이고 Reference 및 사용자층과 정보가 많음
- Windows, macOS 또는 openSUSE, Red Hat, Ubuntu와 같은 다양한 리눅스 OS에 사용가능
- 설치 및 사용이 간단하다. 하지만 빌드 서버를 구축해야 한다.
- 발전하는 플러그인 생태계 (플러그인은 젠킨스내에서 여러 유용한 기능을 제공하고 생산성과 안정성을 높이게끔 해주는 역할을 함)

### Gihub Action
- Github에서 제공하는 CI/CD  도구
- 사용료는 public 저장소는 무료이며, private저장소는 해당 계정에 부여된 무료 사용량 이후에 과금이 부과됩니다. Github 무료 계정의 전체 비공개 저장소를 기준으로 한달에 500MB 스토리지와 실행 시간 2,000분(minute)까지 제공된다.
- 기존의 Circle CI / Travis CI / Jenkins CI와 같은 서비스 또는 설치형 CI처럼 Github에서도 Actions이라는 CI툴을 선보였으며 별다른 복잡한 절차 없이 Github를 통해 사용할 수 있다는 장점이 있다.
- 워크 플로우 복제 용이
- GitHub와 통합

### Bamboo
- Atlassian에서 개발한 CI 도구로 JIRA 의 대쉬보드나 confluence의 Page에 bamboo build chart 를 붙일수도 있고 JIRA 의 특정 이슈와 관련된 build 내역을 조회하는 등 atlassian 제품을 기존에 사용하고 있다면 각 제품군을 통합해서 더욱 유기적으로 사용할 수 있음.
- 유료 서비스
- Windows, Mac OS X, Linux와 같은 플랫폼에서 사용가능

### Travis CI
- 간결하고 직관적인 웹 인터페이스를 제공하는 인터넷 기반의 CI 서비스
- 무료 버전과 기업용 버전(유료)이 있음
  - 오픈 소스 프로젝트 일 경우(public repo) 무료로 사용 가능
  - private repo일 경우 한달에 69$
- Github와 연동하여 Commit/Push를 기반으로 CI가 자동 동작하며, Push 외에도 Pull Request에도 반응하도록 설계되어 있음

### Circle CI
- Linux, macOS, Android, and Windows 운영체제에서 사용 가능
- 일정 한도 내에서 무료로 사용 가능
  - 한주에 2500 크레딧 부여하고 한번에 하나의 job 수행
  - 무료 제공되는 크레딧으로 한달에 1000분 정도의 빌드시간 사용 가능
  - 유료로 사용할 경우 한달에 15달러
- Git에 Push를 하면 자동으로 테스트를 진행하여 검사 결과를 알려 주어 문제점에 대해 바로 알림을 보내줌. 이를 통해 협업시 발생할 수 있는 파일충돌이나 에러를 잡을 수 있음. 그리고 문제가 없다면 서버에 배포까지 자동으로 이루어짐

### CodeBuild
- AWS에서 제공하는 CI 도구
- 빌드 시간만큼 요금이 부과되는 구조라 초기에 사용하기에는 부담
- 월 100분까지는 무료로 제공
<br></br>
## CD 도구
### CodeDeploy
- 대채제가 없다.
- CodeDeploy는 저장 기능이 없다. 따라서 CI 도구가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요하다. 보통 AWS S3를 이용한다.
- Github Commit id를 통해 깃허브 코드를 가져와 배포할 수 도 있다.
- 즉, CodeDeploy는 빌드도 하고 배포도 할 수 있다. CodeDeploy에서는 깃허브 코드를 가져오는 기능을 지원하기 때문이다. 하지만 이렇게 할 때 빌드 없이 배포만 필요할 때 대응하기 어렵다. 빌드와 배포가 분리되어 있으면 예전에 빌드되어 만들어진 Jar를 재사용하면 되지만, CodeDeploy가 모든 것을 하게 될 땐 항상 빌드를 하게 되니 확장성이 많이 떨어진다. 그래서 웬만하면 빌드와 배포는 분리해야 한다.
<br></br>
**✅ CodePipeLine**
- AWS의 CodeBuild, CodeDeploy를 연결하여 CI/CD PipeLine을 구축한다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile24.uf.tistory.com%2Fimage%2F9977A8335A65DB3101D50A](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile24.uf.tistory.com%2Fimage%2F9977A8335A65DB3101D50A)
<br></br>
## 무중단 배포
- 서비스를 정지하지 않고, 배포할 수 있는 방법
- 무중단 배포 방식
  - AWS에서 블루 그린(Blue-Green) 무중단 배포
  - 도커를 이용한 웹서비스 무중단 배포

### Nginx
- 엔진엑스는 웹 서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스 소프트웨어이다.
- 리버스 프록시란 엔진엑스가 외부의 요청을 받아 백앤드 서버로 요청을 전달하는 행위를 이야기한다. 리버스 프록시 서버(엔진엑스)는 요청을 전달하고, 실제 요청에 대한 처리는 뒷단의 웹 애플리케이션 서버들이 처리한다.
- 엔진엑스를 이용한 무중단 배포를 하는 이유는 가장 저렴하고 쉽기 때문
- 기존에 쓰던 EC2에 그대로 적용하면 되므로 배포를 위해 AWS EC2 인스턴스가 하나 더 필요하지 않다.
- 구조는 간단하다. 하나의 EC2 혹은 리눅스 서버에 엔진엑스 1대와 스프링 부트 Jar 2대를 사용하는 것이다.
  - 엔진엑스는 80(http), 443(https) 포트를 할당한다.
  - 스프링 부트1은 8081 포트로 실행한다.
  - 스프링 부트2는 8082 포트로 실행한다.

![https://media.vlpt.us/images/swchoi0329/post/dafeacef-046d-4b82-8c35-f06dc4b4ec15/nginx2.PNG](https://media.vlpt.us/images/swchoi0329/post/dafeacef-046d-4b82-8c35-f06dc4b4ec15/nginx2.PNG)
<br></br>
### 📗 참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱
- [CI/CD(지속적 통합/지속적 제공): 개념, 방법, 장점, 구현 과정](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)
- [CI/CD 개념 및 각종 플랫폼 정리](https://velog.io/@skyni/CICD-개념-및-각종-플랫폼-정리)
- [[CI/CD] Jenkins vs GitLabCI vs Travis](https://owin2828.github.io/devlog/2020/01/09/cicd-1.html)
- [[AWS-CodeDeploy] AWS의 CD 도구, CodeDeploy에 대해서 알아보자.](https://wonit.tistory.com/352)
- [AWS CodePipeLine, CodeBuild, CodeDeploy를 통해 EC2에 배포하기, AWS CI/CD 구축하기 - 1](https://senticoding.tistory.com/91)
- [[CI/CD] Jenkins 과 GitHub Actions의 개념, 차이점](https://choseongho93.tistory.com/295)