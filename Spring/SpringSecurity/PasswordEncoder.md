## PasswordEncoder란?
> Spring Security에서 지원하는 비밀번호 단방향 암호화 인터페이스

### PasswordEncoder
- Spring Security에서는 비밀번호를 안전하게 저장할 수 있도록 비밀번호의 단방향 암호화를 지원하는 PasswordEncoder 인터페이스와 구현체들을 제공합니다. 
- 이 인터페이스는 아래와 같이 심플하게 구성되어 있습니다.
```java
public interface PasswordEncoder {

　　// 비밀번호를 단방향 암호화
　　String encode(CharSequence rawPassword);

　　// 암호화되지 않은 비밀번호(raw-)와 암호화된 비밀번호(encoded-)가 일치하는지 비교
　　boolean matches(CharSequence rawPassword, String encodedPassword);

　　// 암호화된 비밀번호를 다시 암호화하고자 할 경우 true를 return하게 설정
　　default boolean upgradeEncoding(String encodedPassword) { return false; };
}
```
- Spring Security 5.3.3에서 공식 지원하는 PasswordEncoder 구현 클래스들은 아래와 같습니다.

  - **BcryptPasswordEncoder** : `BCrypt` 해시 함수를 사용해 비밀번호를 암호화
  - **Argon2PasswordEncoder** : `Argon2` 해시 함수를 사용해 비밀번호를 암호화
  - **Pbkdf2PasswordEncoder** : `PBKDF2` 해시 함수를 사용해 비밀번호를 암호화
  - **SCryptPasswordEncoder** : `SCrypt` 해시 함수를 사용해 비밀번호를 암호화

  - 위 4개의 PasswordEncoder는 Password를 encode할 때, 매번 임의의 salt를 생성해서 encode 하게 되어 있습니다.
  - 예를 들어 BCryptPasswordEncoder Class의 코드를 보면 아래와 같이 되어있습니다.
```java
/*
* BCryptPasswordEncoder.encode() : 암호화
*/
public String encode(CharSequence rawPassword) {

　　if (rawPassword == null) {
　　　　throw new IllegalArgumentException("rawPassword cannot be null");
　　}

　　String salt;

   if (random != null) {
   　　salt = BCrypt.gensalt(version.getVersion(), strength, random);
   } else {
   　　salt = BCrypt.gensalt(version.getVersion(), strength);
   }
   return BCrypt.hashpw(rawPassword.toString(), salt);
}

/**
* BCrypt.gensalt() : Salt 생성
*/
public static String gensalt(String prefix, int log_rounds, SecureRandom random) throws IllegalArgumentException {

　StringBuilder rs = new StringBuilder();
　byte rnd[] = new byte[BCRYPT_SALT_LEN]; // 16byte(128bit) 크기의 Salt 생성

  if (!prefix.startsWith("$2") || (prefix.charAt(2) != 'a' && prefix.charAt(2) != 'y' && prefix.charAt(2) != 'b')) {
      throw new IllegalArgumentException ("Invalid prefix");
  }

  if (log_rounds < 4 || log_rounds > 31) {
      throw new IllegalArgumentException ("Invalid log_rounds");
  }

　random.nextBytes(rnd);

　rs.append("$2");
　rs.append(prefix.charAt(2));
　rs.append("$");
　if (log_rounds < 10)
      rs.append("0");

  rs.append(log_rounds);
  rs.append("$");
  encode_base64(rnd, rnd.length, rs);

  return rs.toString();
}
```

### BCryptPasswordEncoder
- `BCrypt 해시 함수`를 사용해 비밀번호를 해시하는 PasswordEncoder입니다. 
- Bruteforce attack이나 Rainbow table attack과 같은 Password Cracking에 대한 저항력을 높이기 위해 의도적으로 느리게 설정되어 있습니다.
> 전문 장비를 이용하면 한 계정에 대한 비밀번호 입력을 1초에 수억번 이상으로 시도할 수 있습니다. 따라서 이런 유형의 공격을 어렵게 만들기 위해 1개의 암호를 확인하는데 약 1초 정도의 시간이 걸리도록 하는 것을 권장합니다. 각 시스템별로 성능 차이가 있기 때문에 PasswordEncoder가 암호를 해독하는데 걸리는 시간은 달라질 수 있습니다. 따라서 시스템에 맞게 테스트하면서 속도를 조정해줘야 합니다.

- BCryptPasswordEncoder의 속도는 강도(strength)를 조정해서 조절할 수 있습니다. 
- 강도는 4 ~ 31까지 설정할 수 있으며, BcryptPasswordEncoder는 default 강도로 아래와 같이 10을 사용합니다.
```java
public BCryptPasswordEncoder(BCryptVersion version, int strength, SecureRandom random) {

  if (strength != -1 && (strength < BCrypt.MIN_LOG_ROUNDS || strength > BCrypt.MAX_LOG_ROUNDS)) {
    throw new IllegalArgumentException("Bad strength");
  }

  this.version = version;
  this.strength = strength == -1 ? 10 : strength; // 지정하지 않으면 강도를 10으로 설정
  this.random = random;
}
```

아래는 BCryptPasswordEncoder의 강도를 16으로 설정한 예제입니다.

```java
// Create an encoder with strength 16
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(16);
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### Argon2PasswordEncoder

`Argon2 해시 함수` 를 사용해 비밀번호를 해시하는 PasswordEncoder입니다. Argon2는 Password Hasing Competition의 우승자(?)로 Password Cracking을 방지하기 위해 다른 PasswordEncoder와 마찬가지로 의도적으로 느리게 실행되도록 설정되어 있습니다. 마찬가지로 1개의 비밀번호를 확인하는데 약 1초 정도가 걸리도록 속도를 조정해줘야 합니다.

```java
// Create an encoder with all the defaults
Argon2PasswordEncoder encoder = new Argon2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### Pbkdf2PasswordEncoder
- `PBKDF2 해시 함수` 를 사용해 비밀번호를 해시하는 PasswordEncoder입니다. 
- FIPS 인증(Federal Information Processing Standards, 미 연방 시스템 내에서 중요한 데이터를 보호하기 위한 필수 표준) 이 필요한 경우 이 PasswordEncoder를 선택하는 것이 좋습니다.

```java
// Create an encoder with all the defaults
Pbkdf2PasswordEncoder encoder = new Pbkdf2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### SCryptPasswordEncoder
- `SCrypt 해시 함수`를 사용해 비밀번호를 해시하는 PasswordEncoder입니다.
```java
SCryptPasswordEncoder encoder = new SCryptPasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result);
```

### 기타 PasswordEncoder
- PasswordEncoder 이전 버전과의 호환성을 위해 존재하는 다른 구현 클래스들이 많으나, 이는 안전하지 않은 것으로 간주되기 때문에 더이상 사용되지 않습니다. 
- 그러나 기존 레거시 시스템을 마이그레이션하기 어렵기 때문에 제거할 계획이 없습니다. (Spring Security 曰)

## PasswordEncoder 사용
### 1. Spring Security 의존성 주입
- build.gradle

    ```java
    dependencies {
    		implementation 'org.springframework.boot:spring-boot-starter-security'
    }
    ```

### 2. Config 설정
- `PasswordEncoder`는 스프링 시큐리티의 인터페이스 객체이다.
- `PasswordEncoder`가 하는 역할은 이름에서 알수있듯 비밀번호를 암호화하는 역할이다. 구현체들이 하는 역할은 바로 이 암호화를 어떻게 할지, 암호화 알고리즘에 해당한다.
- 그래서 `PasswordEncoder`의 구현체를 대입해주고 이를 스프링 빈으로 등록하는 과정이 필요하다.
- 이와 함께 스프링 시큐리티 의존성을 주입하고, 바로 톰캣 서버로 실행하면 브라우저에서 로그인 프롬프트가 출력된다. 이런 기본적인 설정들을 disable하는 Config 객체를 생성해야 한다.
- Config 객체는 `WebSecurityConfigurerAdapter`를 상속받아서 `configure()`를 구현한다.

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.config.annotation.web.builders.HttpSecurity;
    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
    import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
    import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
    import org.springframework.security.crypto.password.PasswordEncoder;

    @Configuration
    @EnableWebSecurity
    public class JavaConfig extends WebSecurityConfigurerAdapter {

        @Bean
        public PasswordEncoder getPasswordEncoder() {
            return new BCryptPasswordEncoder();
        }

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                    .cors().disable()      // cors 비활성화
                    .csrf().disable()      // csrf 비활성화
                    .formLogin().disable() //기본 로그인 페이지 없애기
                    .headers().frameOptions().disable();
        }
    }
    ```
    - @Configuration : 설정파일이라는 것을 알려주는 어노테이션
    - @Bean : 빈으로 등록하는 어노테이션 (return 타입이 주입됨)
- Config 클래스에서 WebSecurityConfigurerAdapter 클래스를 상속받아 configure를 오버라이딩 하고, 파라미터인 **[HttpSecurity](https://docs.spring.io/spring-security/site/docs/4.2.13.RELEASE/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)** 클래스를 이용하여 설정한다.
- 패스워드 암호화 방식으로 BCryptPasswordEncoder를 적용했다. BcryptPasswordEncoder는 BCrypt라는 해시 함수를 이용하여 패스워드를 암호화하는 구현체이다.

### 3. 테스트
- JUnit5를 사용하여 테스트
    ```java
    import org.junit.jupiter.api.DisplayName;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.security.crypto.password.PasswordEncoder;

    import static org.junit.jupiter.api.Assertions.*;

    @SpringBootTest
    public class PasswordEncoderTest {
      
       @Autowired
       private UserService userService;

       @Autowired
       private PasswordEncoder passwordEncoder;
      
       @Test
       @DisplayName("패스워드 암호화 테스트")
       void passwordEncode() {
          // given
          String rawPassword = "12345678";

          // when
          String encodedPassword = passwordEncoder.encode(rawPassword);

          // then
          assertAll(
                () -> assertNotEquals(rawPassword, encodedPassword),
                () -> assertTrue(passwordEncoder.matches(rawPassword, encodedPassword))
          );
       } 
    }
    ```
    - 암호화는 `BCryptPasswordEncoder`로 구현된 `encode()`를 이용한다. 파라미터에 평문 패스워드를 주입하면, 암호화된 패스워드를 반환해준다.
    - 그리고 평문 패스워드와 암호화된 패스워드가 서로 같다라는 사실이 증명되어야 로그인을 구현할 수 있을 것이다. 이 때 이 두 문자열을 비교해주는 메소드가 `matches()`이다.
    - `matches()`는 내부에서 평문 패스워드와 암호화된 패스워드가 서로 대칭되는지에 대한 알고리즘을 구현하고 있기 때문에 가능하다.
    - 실제 테스트는 `// then` 아래의 코드인데 두가지를 테스트하기 위해서 `assertAll`을 사용했다.
        - 평문 패스워드와 암호화 패스워드가 서로 다른게 맞는지 -> `assertNotEquals()`
        - `BCryptPasswordEncoder`의 `matches()`를 이용해서 평문 패스워드와 암호화 패스워드를 비교했을때, 같은 패스워드라는 결과를 반환받는지 -> `assertTrue()`

- **String encode(String raw)** : 패스워드 암호화
- **boolean matches(String raw, String encoded)** : 평문 패스워드와 암호화 패스워드가 같은 패스워드인지 비교

### 📗 참고
- [[Spring Security] PasswordEncoder란?](https://velog.io/@corgi/Spring-Security-PasswordEncoder란-4kkyw8gi)
- [[Spring Boot] 스프링 부트에서 비밀번호 암호화하기](https://devlog-wjdrbs96.tistory.com/212)
- [스프링부트에 패스워드 암호화 적용하기 (a.k.a PasswordEncoder)](https://youngjinmo.github.io/2021/05/passwordencoder/)