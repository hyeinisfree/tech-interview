# JWT
### Spring Security 기본 설정
- **config/SecurityConfig.java**
  ```java
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(WebSecurity web) {
      web
          .ignoring()
          .antMatchers("/h2-console/**", "/favicon.ico");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http
            .authorizeRequests()
            .antMatchers("/api/hello").permitAll()
            .anyRequest().authenticated();
    }
  }
  ```

  - @EnableWebSecurity : 기본적인 Web 보안을 활성화
  - 추가적인 설정을 위해 WebSecurityConfigurer를 implements 하거나 WebSecurityConfigurerAdapter를 extends하는 방법이 있다.
  - WebSecurityConfigurerAdapter의 configure 메소드를 오버라이드하여 사용한다.
    - http.authorizeRequests() : HttpServletRequest를 사용하는 요청들에 대한 접근제한을 설정하겠다.
    - http.antMatchers("/api/hello").permitAll() : /api/hello에 대한 요청은 인증없이 접근을 허용하겠다는 의미이다.
    - http.anyRequest().authenticated() : 나머지 요청들은 모두 인증되어야 한다는 의미이다.
  - web.ignoring().antMatchers("/h2-console/**, "/favicon.ico") : /h2-console/ 하위 모든 요청과 파비콘은 모두 무시하는 것으로 설정

### Data 설정
- **entity/User.java**
  ```java
  @Entity
  @Table(name = "user")
  @Getter
  @Setter
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {

      @JsonIgnore
      @Id
      @Column(name = "user_id")
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long userId;

      @Column(name = "username", length = 50, unique = true)
      private String username;

      @JsonIgnore
      @Column(name = "password", length = 100)
      private String password;

      @Column(name = "nickname", length = 50)
      private String nickname;

      @JsonIgnore
      @Column(name = "activated")
      private boolean activated;

      @ManyToMany
      @JoinTable(
        name = "user_authority",
        joinColumns = {@JoinColumn(name = "user_id", referencedColumnName = "user_id")},
        inverseJoinColumns = {@JoinColumn(name = "authority_name", referencedColumnName = "authority_name")})
      private Set<Authority> authorities;
  }
  ```

- **entity/Authority.java**
  ```java
  @Entity
  @Table(name = "authority")
  @Getter
  @Setter
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  public class Authority {

      @Id
      @Column(name = "authority_name", length = 50)
      private String authorityName;
  }
  ```

### JWT 설정
- application.yml
  ```yaml
  jwt:
    header: Authorization
    secret: bHVuaXQtc3ByaW5nLWJvb3QtcHJvamVjdC1ieS0yeWVzZXVsLWxvbmdlci1sb25nZXItbG9uZ2VyLWxvbmdlci1sdW5pdC1wbGVhc2U=
    token-validity-in-seconds: 86400
  ```
  - 이 튜토리얼에서는 HS512 알고리즘을 사용하기 때문에 Secret Key는 64Byte 이상이 되어야 한다.
  - 토큰의 만료시간은 86400초로 설정한다.

- build.gradle
  ```
  dependencies {
    implementation group: 'io.jsonwebtoken', name: 'jjwt-api', version: '0.11.2'
    implementation group: 'io.jsonwebtoken', name: 'jjwt-impl', version: '0.11.2'
    implementation group: 'io.jsonwebtoken', name: 'jjwt-jackson', version: '0.11.2'
    
  }
  ```
  - JWT 관련 라이브러리들을 추가한다.

### JWT 코드 작성
- **jwt/TokenProvider.java**
  - 토큰의 생성, 토큰의 유효성 검증을 담당한다.
  ```java
  @Component
  public class TokenProvider implements InitializingBean {

      private final Logger logger = LoggerFactory.getLogger(TokenProvider.class);

      private static final String AUTHORITIES_KEY = "auth";

      private final String secret;
      private final long tokenValidityInMilliseconds;

      private Key key;

      public TokenProvider(
        @Value("${jwt.secret}") String secret,
        @Value("${jwt.token-validity-in-seconds}") long tokenValidityInSeconds) {
        this.secret = secret;
        this.tokenValidityInMilliseconds = tokenValidityInSeconds * 1000;
      }

      @Override
      public void afterPropertiesSet() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        this.key = Keys.hmacShaKeyFor(keyBytes);
      }

      public String createToken(Authentication authentication) {
        String authorities = authentication.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.joining(","));

        long now = (new Date()).getTime();
        Date validity = new Date(now + this.tokenValidityInMilliseconds);

        return Jwts.builder()
            .setSubject(authentication.getName())
            .claim(AUTHORITIES_KEY, authorities)
            .signWith(key, SignatureAlgorithm.HS512)
            .setExpiration(validity)
            .compact();
      }

      public Authentication getAuthentication(String token) {
        Claims claims = Jwts
                .parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();

        Collection<? extends GrantedAuthority> authorities =
            Arrays.stream(claims.get(AUTHORITIES_KEY).toString().split(","))
              .map(SimpleGrantedAuthority::new)
              .collect(Collectors.toList());

        User principal = new User(claims.getSubject(), "", authorities);

        return new UsernamePasswordAuthenticationToken(principal, token, authorities);
      }

      public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            return true;
        } catch (io.jsonwebtoken.security.SecurityException | MalformedJwtException e) {
            logger.info("잘못된 JWT 서명입니다.");
        } catch (ExpiredJwtException e) {
            logger.info("만료된 JWT 토큰입니다.");
        } catch (UnsupportedJwtException e) {
            logger.info("지원되지 않는 JWT 토큰입니다.");
        } catch (IllegalArgumentException e) {
            logger.info("JWT 토큰이 잘못되었습니다.");
        }
        return false;
      }
  }
  ```
  - IntializingBean을 implements해서 afterPropertiesSet을 Override한 이유 : 빈이 생성이 되고 주입을 받은 후에 secreat값을 Base64 Decode해서 key 변수에 할당하기 위해서
  - createToken 메소드 : Authentication 객체의 권한정보를 이용해서 토큰을 생성하는 역할을 한다. Authentication 객체를 파라미터로 받아서 권한, 만료시간을 설정하고 토큰을 생성한다.
  - getAuthentication 메소드 : Token에 담겨있는 정보를 이용해 Authentication 객체를 리턴하는 메소드이다. 토큰을 파라미터로 받아서 클레임을 만들고 이를 이용해 유저 객체를 만들어서 최종적으로 Authentication 객체를 리턴한다.
  - validateToken 메소드 : 토큰을 파라미터로 받아서 토큰의 유효성 검증을 수행한다. 토큰을 파싱해보고 발생하는 익셉션들을 캐치, 문제가 있으면 false, 정상이면 true를 리턴한다.

- **jwt/JwtFilter.java**
  - JWT를 위한 커스텀 필터를 만들기 위해 생성한다.

  ```java
  public class JwtFilter extends GenericFilterBean {

      private static final Logger logger = LoggerFactory.getLogger(JwtFilter.class);

      public static final String AUTHORIZATION_HEADER = "Authorization";

      private TokenProvider tokenProvider;

      public JwtFilter(TokenProvider tokenProvider) {
        this.tokenProvider = tokenProvider;
      }

      @Override
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
        throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String jwt = resolveToken(httpServletRequest);
        String requestURI = httpServletRequest.getRequestURI();

        if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
            Authentication authentication = tokenProvider.getAuthentication(jwt);
            SecurityContextHolder.getContext().setAuthentication(authentication);
            logger.debug("Security Context에 '{}' 인증 정보를 저장했습니다, uri: {}", authentication.getName(), requestURI);
        } else {
            logger.debug("유효한 JWT 토큰이 없습니다, uri: {}", requestURI);
        }

        filterChain.doFilter(servletRequest, servletResponse);
      }

      private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
      }
  }
  ```
  - JwtFilter는 TokenProvider를 주입받는다.
  - GenericFilterBean을 extends해서 doFilter 메소드를 Override, 실제 필터링 로직은 doFilter 내부에 작성한다.
  - doFilter 메소드 : 토큰의 인증정보를 현재 실행 중인 SecurityContext에 저장하는 역할을 수행한다. resolveToken을 통해 토큰을 받아와서 유효성 검증을 하고 정상 토큰이면 SecurityContext에 저장한다.
  - resolveToken 메소드 : Request Header에서 토큰 정보를 꺼내온다. 필터링을 하기 위해서는 토큰 정보가 필요하다.

- **jwt/JwtSecurityConfig.java**
  - TokenProvider, JwtFilter를 SecurityConfig에 적용할 때 사용한다.
  ```java
  public class JwtSecurityConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

      private TokenProvider tokenProvider;

      public JwtSecurityConfig(TokenProvider tokenProvider) {
          this.tokenProvider = tokenProvider;
      }

      @Override
      public void configure(HttpSecurity http) {
          JwtFilter customFilter = new JwtFilter(tokenProvider);
          http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
      }
  }
  ```
  - SecurityConfigurerAdapter를 extends하고 TokenProvider를 주입받아서 JwtFilter를 통해 Security로직에 필터를 등록한다.

- **jwt/JwtAuthenticationEntryPoint.java**
  - 유효한 자격증명을 제공하지 않고 접근하려 할 때 401 Unauthorized 에러를 리턴한다.

  ```java
  @Component
  public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

      @Override
      public void commence(HttpServletRequest request,
                          HttpServletResponse response,
                          AuthenticationException authException) throws IOException {
        // 유효한 자격증명을 제공하지 않고 접근하려 할때 401
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
      }
  }
  ```

- **jwt/JwtAccessDeniedHandler.java**
  - 필요한 권한이 존재하지 않는 경우에 403 Forbidden 에러를 리턴한다.
  ```java
  @Component
  public class JwtAccessDeniedHandler implements AccessDeniedHandler {

      @Override
      public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
        //필요한 권한이 없이 접근하려 할때 403
        response.sendError(HttpServletResponse.SC_FORBIDDEN);
      }
  }
  ```

- **config/SecurityConfig.java**
  - jwt 관련 5개의 클래스를 SecurityConfig에 추가한다.
  ```java
  @EnableWebSecurity
  @EnableGlobalMethodSecurity(prePostEnabled = true)
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
      private final TokenProvider tokenProvider;
      private final CorsFilter corsFilter;
      private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
      private final JwtAccessDeniedHandler jwtAccessDeniedHandler;

      public SecurityConfig(
              TokenProvider tokenProvider,
              CorsFilter corsFilter,
              JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint,
              JwtAccessDeniedHandler jwtAccessDeniedHandler
      ) {
          this.tokenProvider = tokenProvider;
          this.corsFilter = corsFilter;
          this.jwtAuthenticationEntryPoint = jwtAuthenticationEntryPoint;
          this.jwtAccessDeniedHandler = jwtAccessDeniedHandler;
      }

      @Bean
      public PasswordEncoder passwordEncoder() {
          return new BCryptPasswordEncoder();
      }

      @Override
      public void configure(WebSecurity web) {
          web.ignoring()
                  .antMatchers(
                          "/h2-console/**"
                          ,"/favicon.ico"
                          ,"/error"
                  );
      }

      @Override
      protected void configure(HttpSecurity httpSecurity) throws Exception {
          httpSecurity
                  // token을 사용하는 방식이기 때문에 csrf를 disable합니다.
                  .csrf().disable()

                  .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)

                  .exceptionHandling()
                  .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                  .accessDeniedHandler(jwtAccessDeniedHandler)

                  // enable h2-console
                  .and()
                  .headers()
                  .frameOptions()
                  .sameOrigin()

                  // 세션을 사용하지 않기 때문에 STATELESS로 설정
                  .and()
                  .sessionManagement()
                  .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

                  .and()
                  .authorizeRequests()
                  .antMatchers("/api/hello").permitAll()
                  .antMatchers("/api/authenticate").permitAll()
                  .antMatchers("/api/signup").permitAll()

                  .anyRequest().authenticated()

                  .and()
                  .apply(new JwtSecurityConfig(tokenProvider));
      }
  }
  ```
  - @EnableGlobalMethodSecurity : @PreAuthorize 어노테이션을 메소드 단위로 추가하기 위해서 적용한다.
  - SecurityConfig는 TokenProvider, JwtAuthenticationEntryPoint, JwtAccessDeniedHandler를 주입받는다.
  - PasswordEncoder는 BCryptPasswordEncoder를 사용한다.
  - configure 메소드
    - 토큰을 사용하기 때문에 csrf설정은 disable하고, Exception을 핸들링할 때 만들어놓은 클래스를 사용하도록 추가한다.
    - 세션을 사용하지 않기 때문에 세션 설정을 STATELESS로 설정한다.
    - 로그인 API, 회원가입 API는 토큰이 없는 상태에서 요청이 들어오기 때문에 모두 permitAll 설정을 한다.
    - JwtFilter를 addFilterBefore로 등록했던 JwtSecurityConfig 클래스도 적용한다.

### 외부 통신 시 사용할 DTO 클래스들을 생성
- **dto/LoginDto.java**
  - 로그인 시 사용한다.
  ```java
  @Getter
  @Setter
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  public class LoginDto {

      @NotNull
      @Size(min = 3, max = 50)
      private String username;

      @NotNull
      @Size(min = 3, max = 100)
      private String password;
  }
  ```
  - Lombok 어노테이션이 추가되어 있고 @Valid 관련 어노테이션을 추가한다.
  - username, password 2개의 필드를 가지고 있다.

- **dto/TokenDto.java**
  - Token 정보를 Response할 때 사용한다.
  ```java
  @Getter
  @Setter
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  public class TokenDto {

      private String token;
  }
  ```

- **dto/UserDto.java**
  - 회원가입 시 사용한다.
  ```java
  @Getter
  @Setter
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  public class UserDto {

      @NotNull
      @Size(min = 3, max = 50)
      private String username;

      @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
      @NotNull
      @Size(min = 3, max = 100)
      private String password;

      @NotNull
      @Size(min = 3, max = 50)
      private String nickname;
  }
  ```

### Repository 생성
- **repository/UserRepository.java**
  ```java
  public interface UserRepository extends JpaRepository<User, Long> {
      @EntityGraph(attributePaths = "authorities")
      Optional<User> findOneWithAuthoritiesByUsername(String username);
  }
  ```
  - JpaRepository를 extends하면 findAll, save 등의 메소드를 기본적으로  사용할 수 있게된다.
  - findOneWithAuthoritiesByUsername 메소드 : username을 기준으로 User 정보를 가져올 때 권한 정보도 같이 가져오게 된다.
  - @EntityGraph : 쿼리가 수행 될때 Lazy 조회가 아니라 Eager 조회로 authorities 정보를 같이 가져오게 된다.

- **repository/AuthorityRepository.java**
  ```java
  public interface AuthorityRepository extends JpaRepository<Authority, String> {
  }
  ```

### CustomUserDetailsService 클래스 생성
- **service/CustomUserDetailsService.java**
  ```java
  @Component("userDetailsService")
  public class CustomUserDetailsService implements UserDetailsService {
      private final UserRepository userRepository;

      public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
      }

      @Override
      @Transactional
      public UserDetails loadUserByUsername(final String username) {
        return userRepository.findOneWithAuthoritiesByUsername(username)
            .map(user -> createUser(username, user))
            .orElseThrow(() -> new UsernameNotFoundException(username + " -> 데이터베이스에서 찾을 수 없습니다."));
      }

      private org.springframework.security.core.userdetails.User createUser(String username, User user) {
        if (!user.isActivated()) {
            throw new RuntimeException(username + " -> 활성화되어 있지 않습니다.");
        }
        List<GrantedAuthority> grantedAuthorities = user.getAuthorities().stream()
                .map(authority -> new SimpleGrantedAuthority(authority.getAuthorityName()))
                .collect(Collectors.toList());
        return new org.springframework.security.core.userdetails.User(user.getUsername(),
                user.getPassword(),
                grantedAuthorities);
      }
  }
  ```
  - UserDetailsService를 implements하고 UserRepository를 주입받는다.
  - loadUseerByUsername 메소드를 오버라이드해서 로그인시에 DB에서 유저정보와 권한정보를 가져오게 된다. 해당 정보를 기반으로 userdetails.User 객체를 생성해서 리턴한다.

### 로그인 API 생성
- **controller/AuthController.java**
  ```java
  @RestController
  @RequestMapping("/api")
  public class AuthController {
      private final TokenProvider tokenProvider;
      private final AuthenticationManagerBuilder authenticationManagerBuilder;

      public AuthController(TokenProvider tokenProvider, AuthenticationManagerBuilder authenticationManagerBuilder) {
          this.tokenProvider = tokenProvider;
          this.authenticationManagerBuilder = authenticationManagerBuilder;
      }

      @PostMapping("/authenticate")
      public ResponseEntity<TokenDto> authorize(@Valid @RequestBody LoginDto loginDto) {

          UsernamePasswordAuthenticationToken authenticationToken =
                  new UsernamePasswordAuthenticationToken(loginDto.getUsername(), loginDto.getPassword());

          Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
          SecurityContextHolder.getContext().setAuthentication(authentication);

          String jwt = tokenProvider.createToken(authentication);

          HttpHeaders httpHeaders = new HttpHeaders();
          httpHeaders.add(JwtFilter.AUTHORIZATION_HEADER, "Bearer " + jwt);

          return new ResponseEntity<>(new TokenDto(jwt), httpHeaders, HttpStatus.OK);
      }
  }
  ```
  - TokenProvider, AuthenticationManagerBuilder를 주입받는다.
  - 로그인 API 경로는 /api/authenticate 이고 Post 요청을 받는다.
  - LoginDto의 username, password를 파라미터로 받고 이를 이용해 UsernamePasswordAuthenticationToken을 생성한다.
  - authenticationToken을 이용해서 Authentication 객체를 생성하려고 authenticate 메소드가 실행 될 때 loadUserByUsername 메소드가 실행된다.
  - Authentication 객체를 생성하고 이를 SecurityContext에 저장하고 Authentication 객체를 createToken 메소드를 통해서 JWT Token을 생성한다.
  - JWT Token을 Response Header에도 넣어주고 TokenDto를 이용해서 Response Body에도 넣어서 리턴한다.

### 간단한 Util 메소드 생성
- **util/SecurityUtil.java**
  ```java
  public class SecurityUtil {

      private static final Logger logger = LoggerFactory.getLogger(SecurityUtil.class);

      private SecurityUtil() {
      }

      public static Optional<String> getCurrentUsername() {
        final Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null) {
            logger.debug("Security Context에 인증 정보가 없습니다.");
            return Optional.empty();
        }

        String username = null;
        if (authentication.getPrincipal() instanceof UserDetails) {
            UserDetails springSecurityUser = (UserDetails) authentication.getPrincipal();
            username = springSecurityUser.getUsername();
        } else if (authentication.getPrincipal() instanceof String) {
            username = (String) authentication.getPrincipal();
        }

        return Optional.ofNullable(username);
      }
  ```
  - getCurrentUsername 메소드 : Security Context의 Authentication 객체를 이용해 username을 리턴해주는 간단한 유틸성 메소드이다.
  - Security Context에 Authentication 객체가 저장되는 시점은 JwtFilter의 doFilter 메소드에서 Request가 들어올 때 SecurityContext에 Authentication 객체를 저장해서 사용하게 된다.

  ### UserService 클래스 생성
  - 회원가입, 유저정보조회 등의 메소드를 위해 생성한다.
  - **service/Userservice.java**

      ```java
      @Service
      public class UserService {
          private final UserRepository userRepository;
          private final PasswordEncoder passwordEncoder;

          public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
              this.userRepository = userRepository;
              this.passwordEncoder = passwordEncoder;
          }

          @Transactional
          public User signup(UserDto userDto) {
              if (userRepository.findOneWithAuthoritiesByUsername(userDto.getUsername()).orElse(null) != null) {
                  throw new RuntimeException("이미 가입되어 있는 유저입니다.");
              }

              //빌더 패턴의 장점
              Authority authority = Authority.builder()
                      .authorityName("ROLE_USER")
                      .build();

              User user = User.builder()
                      .username(userDto.getUsername())
                      .password(passwordEncoder.encode(userDto.getPassword()))
                      .nickname(userDto.getNickname())
                      .authorities(Collections.singleton(authority))
                      .activated(true)
                      .build();

              return userRepository.save(user);
          }

          @Transactional(readOnly = true)
          public Optional<User> getUserWithAuthorities(String username) {
              return userRepository.findOneWithAuthoritiesByUsername(username);
          }

          @Transactional(readOnly = true)
          public Optional<User> getMyUserWithAuthorities() {
              return SecurityUtil.getCurrentUsername().flatMap(userRepository::findOneWithAuthoritiesByUsername);
          }
      }
      ```

      - UserService는 UserRepository, PasswordEncoder를 주입받는다.
      - signup 메소드 : 회원가입 로직을 수행하는 메소드이다. username이 DB에 존재하지 않으면 Authority와 User 정보를 생성해서 UserRepository의 save 메소드를 통해 DB에 정보를 저장한다.
      - 유저, 권한정보를 가져오는 메소드 2개
          - getUserWithAuthorities는 username을 기준으로 정보를 가져온다.
          - getMyUserWithAuthorities는 SecurityContext에 저장된 username의 정보만 가져온다.

### UserController 클래스 생성
- **controller/UserController.java**
  ```java
  @RestController
  @RequestMapping("/api")
  public class UserController {
      private final UserService userService;

      public UserController(UserService userService) {
          this.userService = userService;
      }

      @GetMapping("/hello")
      public ResponseEntity<String> hello() {
          return ResponseEntity.ok("hello");
      }

      @PostMapping("/signup")
      public ResponseEntity<User> signup(
              @Valid @RequestBody UserDto userDto
      ) {
          return ResponseEntity.ok(userService.signup(userDto));
      }

      @GetMapping("/user")
      @PreAuthorize("hasAnyRole('USER','ADMIN')")
      public ResponseEntity<User> getMyUserInfo() {
          return ResponseEntity.ok(userService.getMyUserWithAuthorities().get());
      }

      @GetMapping("/user/{username}")
      @PreAuthorize("hasAnyRole('ADMIN')")
      public ResponseEntity<User> getUserInfo(@PathVariable String username) {
          return ResponseEntity.ok(userService.getUserWithAuthorities(username).get());
      }
  }
  ```
  - signup 메소드는 UserDto를 파라미터로 받아서 UserService의 signup 메소드를 호출한다.
  - getMyUserInfo 메소드는 @PreAuthorize를 통해서 USER, ADMIN 두가지 권한 모두 허용하고, getUserInfo 메소드는 ADMIN 권한만 호출할 수 있도록 설정한다.

### 📗 참고
- [[Server] 토큰 기반 인증 VS 서버 기반 인증](https://mangkyu.tistory.com/55)
- [[Server] JWT(Json Web Token)란?](https://mangkyu.tistory.com/56)
- [[SpringBoot] SpringBoot로 SpringSecurity 기반의 JWT 토큰 구현하기](https://mangkyu.tistory.com/57)
- [[SpringBoot] JWT + 스프링부트](https://velog.io/@ayoung0073/springboot-JWT)
- [Spring Security 와 JWT 겉핥기](https://bcp0109.tistory.com/301)
- [인프런 Spring Boot JWT Tutorial](https://www.inflearn.com/course/스프링부트-jwt/dashboard)