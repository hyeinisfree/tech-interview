# JVM, JRE, JDK

## Java의 개발 철학
`WORA = Write Once, Run Anywhere`  
`한 번 쓰고 모든 곳에서 실행한다`

## JVM
Java Virtual Machine의 줄임말.  
JVM은 OS 위에서 CPU가 Java를 인식, 실행할 수 있게 하는 가상 컴퓨터이다.

JVM은 2가지 기본 기능이 있다.
1. 자바 프로그램이 어느 기기, 어느 운영체제 상에서도 실행될 수 있게 만들어 준다. => WORA
2. 자바 프로그램의 메모리를 효율적으로 관리 & 최적화해 준다.  

Java는 JVM을 통해 플랫폼에 관계없이 실행할 수 있다.   
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F0kg24%2Fbtq4YOOQH4J%2FEF2ISOpkYA36a1flwtLEmK%2Fimg.png)

Java 소스코드, 즉 원시코드(*.java)는 CPU가 인식을 하지 못하므로 기계어로 컴파일을 해줘야한다.
하지만 Java는 이 JVM 이라는 가상머신을 거쳐서 OS에 도달하기 때문에 OS가 인식할 수 있는 기계어로 바로 컴파일 되는게 아니라 JVM이 인식할 수 있는 Java bytecode(*.class)로 변환된다.
 
Java compiler 가 .java 파일을 .class 라는 Java bytecode로 변환한다.

> 💡 여기서 Java compiler는 JDK를 설치하면 bin 에 존재하는 javac.exe를 말한다. (JDK에 Java compiler가 포함되어 있다). 
> javac 명령어를 통해 .java를 .class로 컴파일 할 수 있다.

변환된 bytecode는 기계어가 아니기 때문에 OS에서 바로 실행되지 않는다.  
이 때, JVM이 OS가 bytecode를 이해할 수 있도록 해석해준다. 따라서 Byte Code는 JVM 위에서 OS 상관없이 실행될 수 있는 것이다.  
OS에 종속적이지 않고, Java 파일 하나만 만들면 어느 디바이스든 JVM 위에서 실행할 수 있다.

### 바이트코드란
가상 컴퓨터(VM)에서 돌아가는 실행 프로그램을 위한 이진 표현법.
 
자바 바이트 코드(Java bytecode)는 JVM이 이해할 수 있는 언어로 변환된 자바 소스코드를 의미한다.
자바 컴파일러에 의해 변환된 코드의 명령어 크기가 1바이트라서 자바 바이트 코드라고 불리고 있다.
 
바이트 코드는 다시 실시간 번역기 또는 JIT 컴파일러에 의해 바이너리 코드로 변환된다.

> 💡 **바이너리 코드란?**  
> 바이너리 코드 또는 이진 코드라고 한다.
> 컴퓨터가 인식할 수 있는 0과 1로 구성된 이진코드

> 💡 **기계어란?**  
> 0과 1로 이루어진 바이너리 코드이다.
> 기계어가 이진코드로 이루어졌을 뿐 모든 이진코드가 기계어인 것은 아니다.
> 기계어는 특정한 언어가 아니라 CPU가 이해하는 명령어 집합이며, CPU 제조사마다 기계어가 다를 수 있다.

즉, CPU가 이해하는 언어는 바이너리 코드, 가상 머신이 이해하는 코드는 바이트 코드이다.


## JVM 구성요소

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtclVx%2Fbtq4Xfml6Dy%2Fnzb5xxlGG1fr5iBGUMv77K%2Fimg.png)

JVM은 크게 아래와 같이 이루어져 있다.  

- 클래스 로더(Class Loader)
- 실행 엔진(Execution Engine)
  - 인터프리터(Interpreter)
  - JIT 컴파일러(Just-in-Time)
  - 가비지 콜렉터(Garbage collector)
- 런타임 데이터 영역 (Runtime Data Area)

### JIT 컴파일러 
JIT 컴파일(just-in-time compliation) 또는 동적 번역(dynamic translation) 이라고 한다.  
JIT 컴파일러는 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일러이다.
 
인터프리터 방식의 단점을 보완하기 위해 도입되었다.  
인터프리터 방식으로 실행하다가 적절한 시점에 바이트 코드 전체를 컴파일하여 기계어로 변경하고, 이후에는 해당 더 이상 인터프리팅 하지 않고 기계어로 직접 실행하는 방식이다.
 
기계어(컴파일된 코드)는 캐시에 보관하기 때문에 한 번 컴파일된 코드는 빠르게 수행하게 된다.  
물론 JIT 컴파일러가 컴파일하는 과정은 바이트 코드를 인터프리팅하는 것보다 훨씬 오래걸리므로 한 번만 실행되는 코드라면 컴파일 하지 않고 인터프리팅하는 것이 유리하다.  
따라서 JIT 컴파일러를 사용하는 JVM들은 내부적으로 해당 메서드가 얼마나 자주 수행되는지 체크하고 일정 정도를 넣을때에만 컴파일을 수행한다.  
 
자바에선 자바 컴파일러가 자바 프로그램 코드를 바이트 코드로 변환한 다음, 실제 바이트 코드를 실행하는 시점에서 자바 가상 머신(JVM, 정확히는 JRE)이 바이트 코드를 JIT 컴파일을 통해 기계어로 변환한다.  
 
### 가비지 컬렉션
Garbage Collection.  
 
JVM은 가비지 컬렉션이라는 프로세스를 통해 메모리를 관리한다. 가비지 컬렉션은 자바 프로그램에서 사용되지 않는 메모리를 지속적으로 찾아내서 제거하는 역할을 한다.


## JRE
Java SE Runtime Environment (자바 런타임 환경)  
 
JVM + 자바 클래스 라이브러리(Java Class Library) 등으로 구성되어 있다.  
컴파일 된 Java 프로그램을 실행하는데 필요한 패키지이다.

> **💡 SDK란?**  
> Software Development Kit (소프트웨어 개발 키트)  
> 하드웨어 플랫폼, 운영체제 또는 프로그래밍 언어 제작사가 제공하는 툴이다. 키트의 요소는 제작사마다 다르다.  
> SDK의 대표적인 예로, JDK 등이 있다.  
> SDK를 활용하여 애플리케이션을 개발할 수 있다.

## JDK
Java SE Development Kit (자바 개발 키트)  
 
Java 를 사용하기 위해 필요한 모든 기능을 갖춘 Java용 SDK (Software Development Kit)이다.  
JDK 는 JRE를 포함하고 있다.
JRE에 있는 모든 것 뿐만 아니라 컴파일러(javac)와 jdb, javadoc 과 같은 도구도 있다.  

즉, JDK는 프로그램을 생성, 실행, 컴파일할 수 있다.

> **요약**   
> JDK는 자바 프로그램을 실행, 컴파일, 개발용 도구.  
> JRE, JVM를 모두 포함하는 포괄적이 키트이다.  
> 
> JRE는 자바 프로그램을 실행할 수 있게 하는 도구이다. JVM을 포함하고 있다.  

### 📗참고
- [[JAVA] JVM이란? 개념 및 구조 (JDK, JRE, JIT, 가비지 콜렉터...)](https://doozi0316.tistory.com/entry/1주차-JVM은-무엇이며-자바-코드는-어떻게-실행하는-것인가?category=935577)
