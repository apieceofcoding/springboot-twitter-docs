# JWT 기반 로그인, 토큰 기반 인증 시스템 구현

이번에는 세션과 달리 토큰 기반 인증방식인 JWT(JSON Web Token) 로그인 기능을 Spring Boot 애플리케이션에서 구현하는 방법을 알아보겠습니다.



## 1. JWT란 무엇일까?

JWT(JSON Web Token)는 웹에서 사용자를 식별하기 위한 '디지털 신분증'이라고 생각하면 됩니다.

### JWT의 개념 쉽게 이해하기

예시로 설명하면 다음과 같습니다.

**놀이공원 입장권과 비슷합니다**

- 입장권에는 방문객 정보, 입장 시간, 만료 시간이 있죠.
- JWT도 사용자 정보, 발급 시간, 만료 시간 등을 담고 있습니다

**위조가 불가능한 특별한 도장이 찍혀있습니다**

- JWT는 서버만 알고 있는 비밀 키로 '서명'되어 있어 위조가 불가능합니다
- 마치 특수하게 만들어진 홀로그램처럼, 진짜와 가짜를 구분할 수 있죠

### JWT의 장점

세션처럼 서버에 저장할 필요가 없어서 서버에 부담이 적습니다. 또한 서버가 여러대 여도 서버 입장에서는 이것을 저장하고 관리하는 것이 아니므로, 그대로 사용하면 됩니다. 따라서 웹사이트 뿐만 아니라 모바일 앱에서도 사용하기 좋습니다. 웹 브라우저에서 자주 사용되는 쿠키를 사용하지 않아도 되니까요.

### JWT 사용 시 주의할 점

- 유효기간을 정하여 꼭 만료되게 만들어야 합니다. 영원히 지속되는 토큰이면, 중간에 노출될 경우 악용할 확률이 높아질 겁니다.
- JWT는 **누구나 내용을 볼 수 있기 때문에** 비밀번호 같은 민감, 개인정보는 넣으면 안됩니다.
- 클라이언트(웹 브라우저나 앱 등)에서 토큰을 안전하게 보관해야 합니다.



## 2. Spring Security 설정하기

Spring Security를 사용하여 JWT 기반 로그인 기능을 구현합니다.

### 2.1 의존성 추가

`build.gradle` 파일에 Spring Security와 JWT 관련 의존성을 추가합니다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

### 2.2 보안 설정 클래스 생성

Spring Security 설정을 위한 클래스를 생성합니다.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final MemberDetailsService memberDetailsService;
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final AuthenticationEntryPoint authenticationEntryPoint;
    private final AccessDeniedHandler accessDeniedHandler;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .exceptionHandling(exception -> exception
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler)
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 2.3 MemberDetailsService 구현

Spring Security의 `UserDetailsService`를 구현하여 회원 정보를 로드합니다.

```java
@Service
@RequiredArgsConstructor
public class CustomMemberDetailsService implements UserDetailsService {
    
    private final MemberRepository memberRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Member member = memberRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("회원을 찾을 수 없습니다: " + username));
        
        return new org.springframework.security.core.userdetails.User(
            member.getUsername(),
            member.getPassword(),
            Collections.singletonList(new SimpleGrantedAuthority("ROLE_MEMBER"))
        );
    }
}
```

## 3. JWT 인증 구현하기

JWT(JSON Web Token)를 사용하여 인증을 구현합니다.

### 3.1 JWT 유틸리티 클래스

JWT 토큰을 생성하고 검증하기 위한 유틸리티 클래스를 생성합니다.

```java
@Component
public class JwtUtils {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private int jwtExpirationMs;
    
    /**
     * JWT 토큰 생성
     * @param authentication 인증 정보
     * @return JWT 토큰
     */
    public String generateJwtToken(Authentication authentication) {
        UserDetails userPrincipal = (UserDetails) authentication.getPrincipal();
        
        return Jwts.builder()
            .setSubject(userPrincipal.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date((new Date()).getTime() + jwtExpirationMs))
            .signWith(getSigningKey(), SignatureAlgorithm.HS512)
            .compact();
    }
    
    /**
     * JWT 토큰에서 사용자명 추출
     * @param token JWT 토큰
     * @return 사용자명
     */
    public String getUserNameFromJwtToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
    
    /**
     * JWT 토큰 검증
     * @param authToken JWT 토큰
     * @return 유효성 여부
     */
    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(authToken);
            return true;
        } catch (SecurityException e) {
            log.error("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            log.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            log.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            log.error("JWT claims string is empty: {}", e.getMessage());
        }
        
        return false;
    }
    
    /**
     * 서명 키 생성
     * @return 서명 키
     */
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

### 3.2 JWT 인증 필터

JWT 토큰을 검증하고 인증 정보를 설정하는 필터를 생성합니다.

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtUtils jwtUtils;
    private final MemberDetailsService memberDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String jwt = parseJwt(request);
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                String username = jwtUtils.getUserNameFromJwtToken(jwt);
                
                UserDetails userDetails = memberDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                    userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            log.error("Cannot set user authentication: {}", e.getMessage());
        }
        
        filterChain.doFilter(request, response);
    }
    
    /**
     * 요청에서 JWT 토큰 추출
     * @param request HTTP 요청
     * @return JWT 토큰
     */
    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");
        
        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);
        }
        
        return null;
    }
}
```

### 3.3 인증 예외 처리

인증 및 권한 관련 예외를 처리하는 클래스를 생성합니다.

```java
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException authException) throws IOException {
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        
        Map<String, Object> body = new HashMap<>();
        body.put("status", HttpServletResponse.SC_UNAUTHORIZED);
        body.put("error", "Unauthorized");
        body.put("message", authException.getMessage());
        body.put("path", request.getServletPath());
        
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(response.getOutputStream(), body);
    }
}
```

```java
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException {
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        
        Map<String, Object> body = new HashMap<>();
        body.put("status", HttpServletResponse.SC_FORBIDDEN);
        body.put("error", "Forbidden");
        body.put("message", accessDeniedException.getMessage());
        body.put("path", request.getServletPath());
        
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(response.getOutputStream(), body);
    }
}
```

## 4. 로그인 서비스 구현하기

로그인과 관련된 비즈니스 로직을 처리하는 서비스를 구현합니다.

```java
@Service
@RequiredArgsConstructor
public class AuthService {
    
    private final AuthenticationManager authenticationManager;
    private final MemberRepository memberRepository;
    private final JwtUtils jwtUtils;
    
    /**
     * 로그인
     * @param request 로그인 요청
     * @return 로그인 응답
     * @throws AuthenticationException 인증 실패 시
     */
    public LoginResponse login(LoginRequest request) {
        // 인증 처리
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword()));
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        
        // JWT 토큰 생성
        String jwt = jwtUtils.generateJwtToken(authentication);
        
        // 회원 정보 조회
        Member member = memberRepository.findByUsername(request.getUsername())
            .orElseThrow(() -> new UsernameNotFoundException("회원을 찾을 수 없습니다: " + request.getUsername()));
        
        // 응답 생성
        return LoginResponse.builder()
            .token(jwt)
            .type("Bearer")
            .id(member.getId())
            .username(member.getUsername())
            .name(member.getName())
            .email(member.getEmail())
            .build();
    }
    
    /**
     * 현재 로그인한 회원 정보 조회
     * @return 회원 정보
     */
    public MemberResponse getCurrentMember() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        
        Member member = memberRepository.findByUsername(userDetails.getUsername())
            .orElseThrow(() -> new UsernameNotFoundException("회원을 찾을 수 없습니다: " + userDetails.getUsername()));
        
        return MemberResponse.builder()
            .id(member.getId())
            .username(member.getUsername())
            .name(member.getName())
            .email(member.getEmail())
            .createdAt(member.getCreatedAt())
            .build();
    }
}
```

## 5. 요청 및 응답 DTO 생성하기

클라이언트와 서버 간의 데이터 전송을 위한 DTO를 생성합니다.

```java
public record LoginRequest(
    @NotBlank(message = "아이디는 필수입니다.")
    String username,
    
    @NotBlank(message = "비밀번호는 필수입니다.")
    String password
) {}
```

```java
public record LoginResponse(
    String token,
    String type,
    Long id,
    String username,
    String name,
    String email
) {
    public static LoginResponse of(String token, Member member) {
        return new LoginResponse(
            token,
            "Bearer",
            member.getId(),
            member.getUsername(),
            member.getName(),
            member.getEmail()
        );
    }
}
```

## 6. 컨트롤러 구현하기

클라이언트의 요청을 처리하고 응답을 반환하는 컨트롤러를 구현합니다.

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {
    
    private final AuthService authService;
    
    /**
     * 로그인
     * @param request 로그인 요청
     * @return 로그인 응답
     */
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request) {
        LoginResponse response = authService.login(request);
        return ResponseEntity.ok(response);
    }
    
    /**
     * 현재 로그인한 회원 정보 조회
     * @return 회원 정보
     */
    @GetMapping("/me")
    public ResponseEntity<MemberResponse> getCurrentMember() {
        MemberResponse response = authService.getCurrentMember();
        return ResponseEntity.ok(response);
    }
    
    /**
     * 로그아웃
     * @return 성공 메시지
     */
    @PostMapping("/logout")
    public ResponseEntity<Map<String, String>> logout() {
        SecurityContextHolder.clearContext();
        
        Map<String, String> response = new HashMap<>();
        response.put("message", "로그아웃되었습니다.");
        
        return ResponseEntity.ok(response);
    }
}
```

## 7. 로그인 시도 제한 기능 구현하기

무차별 대입 공격(Brute Force Attack)을 방지하기 위해 로그인 시도 횟수를 제한하는 기능을 구현합니다.

### 7.1 로그인 시도 기록 엔티티

로그인 시도 기록을 저장할 엔티티를 생성합니다.

```java
@Entity
@Table(name = "login_attempts")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
public class LoginAttempt {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String username;
    
    @Column(name = "attempt_time", nullable = false)
    private LocalDateTime attemptTime;
    
    @Column(name = "ip_address")
    private String ipAddress;
    
    @Column(name = "user_agent")
    private String userAgent;
    
    @Column(name = "success", nullable = false)
    private boolean success;
    
    public static LoginAttempt createAttempt(String username, String ipAddress, String userAgent, boolean success) {
        return LoginAttempt.builder()
            .username(username)
            .attemptTime(LocalDateTime.now())
            .ipAddress(ipAddress)
            .userAgent(userAgent)
            .success(success)
            .build();
    }
}
```

### 7.2 로그인 시도 기록 저장소

로그인 시도 기록을 저장하고 조회하기 위한 저장소를 생성합니다.

```java
public interface LoginAttemptRepository extends JpaRepository<LoginAttempt, Long> {
    
    List<LoginAttempt> findByUsernameAndAttemptTimeAfter(String username, LocalDateTime time);
    
    List<LoginAttempt> findByIpAddressAndAttemptTimeAfter(String ipAddress, LocalDateTime time);
    
    @Query("SELECT COUNT(l) FROM LoginAttempt l WHERE l.username = :username AND l.attemptTime > :time AND l.success = false")
    int countFailedAttemptsByUsername(@Param("username") String username, @Param("time") LocalDateTime time);
    
    @Query("SELECT COUNT(l) FROM LoginAttempt l WHERE l.ipAddress = :ipAddress AND l.attemptTime > :time AND l.success = false")
    int countFailedAttemptsByIpAddress(@Param("ipAddress") String ipAddress, @Param("time") LocalDateTime time);
}
```

### 7.3 로그인 시도 제한 서비스

로그인 시도 제한과 관련된 비즈니스 로직을 처리하는 서비스를 구현합니다.

```java
@Service
@RequiredArgsConstructor
public class LoginAttemptService {
    
    private final LoginAttemptRepository loginAttemptRepository;
    
    private static final int MAX_ATTEMPT = 5;
    private static final int BLOCK_DURATION_MINUTES = 30;
    
    /**
     * 로그인 시도 기록
     * @param username 사용자명
     * @param ipAddress IP 주소
     * @param userAgent 사용자 에이전트
     * @param success 성공 여부
     */
    public void recordLoginAttempt(String username, String ipAddress, String userAgent, boolean success) {
        LoginAttempt attempt = LoginAttempt.createAttempt(username, ipAddress, userAgent, success);
        loginAttemptRepository.save(attempt);
    }
    
    /**
     * 로그인 시도 제한 확인
     * @param username 사용자명
     * @param ipAddress IP 주소
     * @return 로그인 시도 제한 여부
     */
    public boolean isLoginBlocked(String username, String ipAddress) {
        LocalDateTime blockTime = LocalDateTime.now().minusMinutes(BLOCK_DURATION_MINUTES);
        
        // 사용자명 기준 로그인 시도 제한 확인
        int failedAttemptsByUsername = loginAttemptRepository.countFailedAttemptsByUsername(username, blockTime);
        if (failedAttemptsByUsername >= MAX_ATTEMPT) {
            return true;
        }
        
        // IP 주소 기준 로그인 시도 제한 확인
        int failedAttemptsByIpAddress = loginAttemptRepository.countFailedAttemptsByIpAddress(ipAddress, blockTime);
        if (failedAttemptsByIpAddress >= MAX_ATTEMPT) {
            return true;
        }
        
        return false;
    }
    
    /**
     * 로그인 시도 제한 해제
     * @param username 사용자명
     */
    public void unblockLogin(String username) {
        // 로그인 시도 제한 해제 로직 구현
        // 예: 로그인 시도 기록 삭제 또는 성공으로 표시
    }
}
```

### 7.4 로그인 시도 제한 적용

로그인 시도 제한 기능을 로그인 서비스에 적용합니다.

```java
@Service
@RequiredArgsConstructor
public class AuthService {
    
    // 기존 코드...
    private final LoginAttemptService loginAttemptService;
    
    /**
     * 로그인
     * @param request 로그인 요청
     * @param ipAddress IP 주소
     * @param userAgent 사용자 에이전트
     * @return 로그인 응답
     * @throws AuthenticationException 인증 실패 시
     */
    public LoginResponse login(LoginRequest request, String ipAddress, String userAgent) {
        // 로그인 시도 제한 확인
        if (loginAttemptService.isLoginBlocked(request.getUsername(), ipAddress)) {
            throw new AuthenticationException("로그인 시도 횟수가 초과되었습니다. " + BLOCK_DURATION_MINUTES + "분 후에 다시 시도해주세요.") {};
        }
        
        try {
            // 인증 처리
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword()));
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
            
            // JWT 토큰 생성
            String jwt = jwtUtils.generateJwtToken(authentication);
            
            // 회원 정보 조회
            Member member = memberRepository.findByUsername(request.getUsername())
                .orElseThrow(() -> new UsernameNotFoundException("회원을 찾을 수 없습니다: " + request.getUsername()));
            
            // 로그인 시도 기록 (성공)
            loginAttemptService.recordLoginAttempt(request.getUsername(), ipAddress, userAgent, true);
            
            // 응답 생성
            return LoginResponse.builder()
                .token(jwt)
                .type("Bearer")
                .id(member.getId())
                .username(member.getUsername())
                .name(member.getName())
                .email(member.getEmail())
                .build();
        } catch (AuthenticationException e) {
            // 로그인 시도 기록 (실패)
            loginAttemptService.recordLoginAttempt(request.getUsername(), ipAddress, userAgent, false);
            throw e;
        }
    }
}
```

### 7.5 컨트롤러 수정

로그인 시도 제한 기능을 적용하기 위해 컨트롤러를 수정합니다.

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {
    
    // 기존 코드...
    
    /**
     * 로그인
     * @param request 로그인 요청
     * @param request HTTP 요청
     * @return 로그인 응답
     */
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request, HttpServletRequest httpRequest) {
        String ipAddress = getClientIpAddress(httpRequest);
        String userAgent = httpRequest.getHeader("User-Agent");
        
        LoginResponse response = authService.login(request, ipAddress, userAgent);
        return ResponseEntity.ok(response);
    }
    
    /**
     * 클라이언트 IP 주소 가져오기
     * @param request HTTP 요청
     * @return 클라이언트 IP 주소
     */
    private String getClientIpAddress(HttpServletRequest request) {
        String xForwardedForHeader = request.getHeader("X-Forwarded-For");
        if (xForwardedForHeader != null) {
            return xForwardedForHeader.split(",")[0];
        }
        return request.getRemoteAddr();
    }
}
```

## 8. 설정 파일 수정하기

JWT 관련 설정을 `application.yml` 파일에 추가합니다.

```yaml
jwt:
  secret: 9a4f2c8d3b7a1e6f45c8a0b3f267d8b1d4e6f3c8a9d2b5f8e3a9c6b5d2e8f1a
  expiration: 86400000  # 24시간 (밀리초)
```

## 9. API 사용 예시

JWT 기반 로그인 API를 사용하는 방법은 다음과 같습니다:

### 9.1 로그인 요청

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testmember",
    "password": "password123"
  }'
```

### 9.2 로그인 응답

```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZXN0bWVtYmVyIiwiaWF0IjoxNjIxNjk2MDAwLCJleHAiOjE2MjE2OTk2MDB9Q.abcdefghijklmnopqrstuvwxyz",
  "type": "Bearer",
  "id": 1,
  "username": "testmember",
  "name": "Test Member",
  "email": "test@example.com"
}
```

### 9.3 인증된 요청

```bash
curl -X GET http://localhost:8080/api/members/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZXN0bWVtYmVyIiwiaWF0IjoxNjIxNjk2MDAwLCJleHAiOjE2MjE2OTk2MDB9Q.abcdefghijklmnopqrstuvwxyz"
```

## 10. 보안 고려사항

JWT 기반 로그인 기능을 구현할 때 다음과 같은 보안 고려사항을 염두에 두어야 합니다:

### 10.1 JWT 토큰 보안

- JWT 시크릿 키는 충분히 길고 무작위적인 값을 사용해야 합니다.
- JWT 토큰의 만료 시간을 적절히 설정해야 합니다.
- JWT 토큰을 안전하게 저장하고 전송해야 합니다.

### 10.2 비밀번호 보안

- 비밀번호는 반드시 암호화하여 저장해야 합니다.
- 비밀번호 정책을 설정하여 강력한 비밀번호를 사용하도록 유도해야 합니다.
- 비밀번호 재설정 기능을 제공해야 합니다.

### 10.3 CSRF 방지

- JWT를 사용하는 경우 CSRF 토큰이 필요하지 않습니다.
- CORS 설정을 적절히 구성하여 보안을 강화해야 합니다.

### 10.4 XSS 방지

- XSS(Cross-Site Scripting) 공격을 방지하기 위해 사용자 입력을 검증해야 합니다.
- 응답 헤더에 Content-Security-Policy를 설정하여 XSS 공격을 방지할 수 있습니다.

## 11. 결론

JWT 기반 로그인은 사용자가 서비스에 접근하기 위한 인증 과정입니다. Spring Boot 애플리케이션에서 Spring Security와 JWT를 사용하여 안전하고 효율적인 JWT 기반 로그인 기능을 구현할 수 있습니다.

이 문서에서 설명한 내용을 바탕으로 JWT 기반 로그인 기능을 구현해보세요. 필요에 따라 기능을 확장하여 더욱 풍부한 사용자 경험을 제공할 수 있습니다. 