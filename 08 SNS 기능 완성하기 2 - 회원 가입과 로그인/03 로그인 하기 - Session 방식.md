# 세션 기반 로그인, 사용자 인증 시스템 구현

로그인은 사용자가 서비스에 접근하기 위한 인증 과정입니다. 이 문서에서는 Spring Boot 애플리케이션에서 세션 기반 로그인 기능을 구현하는 방법을 알아보겠습니다.



## 1. 세션이란?
세션(Session)은 웹사이트에서 사용자를 기억하기 위한 방법입니다. 쉽게 설명하면 다음과 같습니다.

### 세션의 개념
- 세션은 마치 "웹사이트가 발급해주는 임시 회원증"과 같습니다
- 로그인에 성공하면 서버가 사용자에게 고유한 세션 ID를 발급합니다
- 브라우저는 이 세션 ID를 쿠키에 저장해두고, 이후 요청할 때마다 함께 전송합니다

### 세션의 동작 방식
1. 사용자가 로그인에 성공하면 서버는 세션을 생성합니다
2. 서버는 세션 ID를 브라우저에게 전달합니다
3. 브라우저는 받은 세션 ID를 쿠키에 저장합니다
4. 이후 모든 요청에서 이 세션 ID를 함께 보내 자신을 증명합니다

### 
세션은 로그인 상태를 서버가 관리합니다. 이게 중요한 포인트인데, 로그인 상태를 관리할 수 있다는 것은 로그인 정보를 쉽게 유지하고 해제할 수 있다는 것입니다. 예를 들어서 이미 로그인하고 있는 유저를 강제로 해제 시킬수도 있습니다.

하지만, 이 부분을 다른 관점에서 바라보면 로그인 상태를 서버가 '관리해야' 한다는 것입니다. 그럼 이 정보가 많아져서 용량이 커지면 문제가 될 수 도 있고, 하나의 서버가 아니라 여러 대의 서버를 구성하면 또 다른 관점에서 관리포인트가 생길 수 있습니다.



## 2. Spring Security 설정하기

Spring Security를 사용하여 세션 기반 로그인 기능을 구현합니다.

### 의존성 추가

`build.gradle` 파일에 Spring Security 의존성을 추가합니다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
}
```



### 보안 설정 클래스 생성

Spring Security 설정을 위한 클래스를 생성합니다.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final UserDetailsService userDetailsService;
    private final AuthenticationEntryPoint authenticationEntryPoint;
    private final AccessDeniedHandler accessDeniedHandler;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(false)
                .expiredUrl("/login?expired")
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/api/auth/login")
                .usernameParameter("username")
                .passwordParameter("password")
                .defaultSuccessUrl("/")
                .failureUrl("/login?error")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/api/auth/logout")
                .logoutSuccessUrl("/login?logout")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
                .permitAll()
            )
            .exceptionHandling(exception -> exception
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler)
            );
        
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
}
```



### UserDetailsService 구현

Spring Security의 `UserDetailsService`를 구현하여 사용자 정보를 로드합니다.

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다: " + username));
        
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"))
        );
    }
}
```



## 3. 인증 예외 처리

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
    private final UserRepository userRepository;
    
    /**
     * 로그인
     * @param request 로그인 요청
     * @return 로그인 응답
     * @throws AuthenticationException 인증 실패 시
     */
    public LoginResponse login(LoginRequest request) {
        // 인증 처리
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.username(), request.password()));
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        
        // 사용자 정보 조회
        User user = userRepository.findByUsername(request.username())
            .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다: " + request.username()));
        
        // 응답 생성
        return LoginResponse.from(user);
    }
    
    /**
     * 현재 로그인한 사용자 정보 조회
     * @return 사용자 정보
     */
    public UserResponse getCurrentUser() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        
        User user = userRepository.findByUsername(userDetails.getUsername())
            .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다: " + userDetails.getUsername()));
        
        return UserResponse.builder()
            .id(user.getId())
            .username(user.getUsername())
            .name(user.getName())
            .email(user.getEmail())
            .createdAt(user.getCreatedAt())
            .build();
    }
}
```



## 5. 요청 및 응답 DTO 생성하기

클라이언트와 서버 간의 데이터 전송을 위한 DTO를 생성합니다.

```java
public record LoginRequest(
    @NotBlank(message = "사용자명은 필수입니다.")
    String username,
    
    @NotBlank(message = "비밀번호는 필수입니다.")
    String password
) {}
```

```java
public record LoginResponse(
    Long id,
    String username,
    String name,
    String email
) {
    public static LoginResponse from(User user) {
        return new LoginResponse(
            user.getId(),
            user.getUsername(),
            user.getName(),
            user.getEmail()
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
     * 현재 로그인한 사용자 정보 조회
     * @return 사용자 정보
     */
    @GetMapping("/me")
    public ResponseEntity<UserResponse> getCurrentUser() {
        UserResponse response = authService.getCurrentUser();
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



## 7. API 사용해서 테스트해보기

세션 기반 로그인 API를 사용하는 방법은 다음과 같습니다.

### 로그인 요청

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```



### 로그인 응답

```json
{
  "id": 1,
  "username": "testuser",
  "name": "Test User",
  "email": "test@example.com"
}
```



### 인증된 요청

```bash
curl -X GET http://localhost:8080/api/users/me \
  --cookie "JSESSIONID=abcdef123456"
```



