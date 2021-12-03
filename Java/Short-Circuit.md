# Short Circuit
쇼트 서킷이란, 논리연산자 AND, OR을 나타내기 위해 부호 &&, ||을 사용하는 것을 의미한다.
  
&&, ||와 &, |는 같은 결과를 가지지만 다른 과정을 거친다.
  
&, |: 연산자의 앞 조건식의 결과에 관계없이 뒤 조건식을 실행한다. 조건식을 둘 다 실행한다.  
&&, ||: 연산자의 앞 조건식의 결과에 따라 뒤 조건식의 실행 여부를 결정한다. 이러한 논리연산자를 쇼트서킷이라 한다.
- && 연산의 앞 조건식이 true일 때만 뒤 조건식을 실행한다.
- || 연산의 앞 조건식이 false일 때만 뒤 조건식을 실행한다.

## &&
### 소스 코드
```java
public class ShortCircuit {

    public static void shortCircuit() {

        int x = 0;
        int y = 0;

        if((0 > 1) && (x++ > y)) {
            System.out.println("hello1");
        }
        System.out.printf("&&: x = %d, y = %d\n", x, y);

        if((0 > 1) & (x++ > y)) {
            System.out.println("hello2");
        }
        System.out.printf("&: x = %d, y = %d\n", x, y);
    }

    public static void main(String[] args) {

        shortCircuit();
    }
}
```
### 출력 결과
```bash
&&: x = 0, y = 0
&: x = 1, y = 0
```

## ||
### 소스 코드
```java
public class ShortCircuit {

    public static void shortCircuit() {

        int x = 0;
        int y = 0;

        if((0 < 1) || (x++ > y)) {
            System.out.println("hello1");
        }
        System.out.printf("||: x = %d, y = %d\n", x, y);

        if((0 < 1) | (x++ > y)) {
            System.out.println("hello2");
        }
        System.out.printf("|: x = %d, y = %d\n", x, y);
    }

    public static void main(String[] args) {

        shortCircuit();
    }
}
```
### 출력 결과
```bash
hello1
||: x = 0, y = 0
hello2
|: x = 1, y = 0
```

## 정리
- 쇼트 서킷을 사용함으로써 불필요한 코드 실행을 막아 작업의 연산 속도를 높일 수 있고,
- 앞 조건식의 결과에 따라 뒤 조건식의 실행 여부를 결정할 경우에도 사용할 수 있다.

### 📗참고
- [[Java] 자바 쇼트 서킷 (short-circuit)](https://junior-datalist.tistory.com/214)