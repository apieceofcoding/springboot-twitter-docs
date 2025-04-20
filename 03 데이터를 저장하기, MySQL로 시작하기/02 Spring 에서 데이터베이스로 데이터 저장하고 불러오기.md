# Spring에서 데이터베이스로 데이터 저장하고 불러오기



## 1. Spring Data JPA 소개

Spring Data JPA는 Spring 애플리케이션에서 데이터베이스를 쉽게 사용할 수 있도록 도와주는 라이브러리입니다. 

JPA(Java Persistence API)를 기반으로 하며, 반복적인 데이터베이스 접근 코드를 줄이고 객체 지향적인 방식으로 데이터를 처리할 수 있게 해줍니다.
Java Persistent API 에서 나오는 API란, 흔히 우리가 말하는 HTTP 요청을 통해 데이터를 주고받는 REST API의 API와 상당히 유사한 의미로 사용됩니다.
REST API는 클라이언트와 서버가 데이터를 주고받기 위한 **통신 인터페이스**이고, Java Persisent API는 자바에서 자바 코드를 통해 **데이터를 다루기 위한 인터페이스**입니다. 

인터페이스란 기능이 약속되어있는 설계도라고 생각하면 좋습니다. 사용자 입장에서는 이 규칙만 이용하면 기능을 편하게 사용할 수 있습니다. 
이 인터페이스에 대한 대부분의 상세 구현은 이미 선조 개발자들이 잘 만들어 놓았으며, 우리는 이를 인터페이스만 알고 사용하면 됩니다



### JPA란?

JPA는 Java 객체와 데이터베이스 테이블 간의 매핑을 정의하는 표준 API입니다. JPA를 사용하면 SQL 쿼리를 직접 작성하지 않고도 Java 객체를 통해 데이터베이스를 조작할 수 있습니다.

### Spring Data JPA의 장점

**간결한 코드**: 반복적인 CRUD 작업을 위한 코드를 줄일 수 있습니다.

**객체 지향적인 접근**: 데이터베이스 테이블을 Java 객체로 다룰 수 있습니다.

**유연한 쿼리 작성**: 메서드 이름만으로 쿼리를 생성할 수 있습니다.

**트랜잭션 관리**: Spring의 트랜잭션 관리 기능을 활용할 수 있습니다.



## 2. Spring Boot 프로젝트에 데이터베이스 설정하기

### 의존성 추가하기

Spring Boot 프로젝트에 데이터베이스 관련 의존성을 추가합니다. `build.gradle` 파일에 다음 의존성을 추가합니다.

```gradle
dependencies {
    // Spring Data JPA
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // 데이터베이스 드라이버 (예: H2, MySQL, PostgreSQL 등)
    runtimeOnly 'com.h2database:h2'  // H2 데이터베이스
    // runtimeOnly 'mysql:mysql-connector-java'  // MySQL
}
```

### 데이터베이스 연결 설정하기

`application.properties` 또는 `application.yml` 파일에 데이터베이스 연결 정보를 설정합니다:

```properties
# H2 데이터베이스 설정 (개발용)
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true

# JPA 설정
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```



## 3. 엔티티(Entity) 클래스 만들기

엔티티 클래스는 데이터베이스 테이블과 매핑되는 Java 클래스입니다. JPA 어노테이션을 사용하여 엔티티를 정의합니다.

### 기본 엔티티 클래스 예시

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column
    private LocalDateTime createdAt;
}
```



### 주요 JPA 어노테이션

- `@Entity`: 이 클래스가 엔티티임을 나타냅니다.
- `@Table`: 매핑할 테이블 정보를 지정합니다.
- `@Id`: 기본키를 지정합니다.
- `@GeneratedValue`: 기본키 생성 전략을 지정합니다.
- `@Column`: 컬럼 정보를 지정합니다.
- `@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`: 관계를 지정합니다.



## 4. 리포지토리(Repository) 인터페이스 만들기

리포지토리는 데이터베이스 접근을 담당하는 인터페이스입니다. Spring Data JPA는 리포지토리 인터페이스를 구현하여 데이터베이스 작업을 수행합니다.

### 기본 리포지토리 인터페이스 예시

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // 기본 CRUD 메서드는 JpaRepository에서 제공됩니다.
    
    // 메서드 이름으로 쿼리 생성
    List<User> findByName(String name);
    Optional<User> findByEmail(String email);
    
    // @Query 어노테이션으로 쿼리 정의
    @Query("SELECT u FROM User u WHERE u.createdAt >= :startDate")
    List<User> findUsersCreatedAfter(@Param("startDate") LocalDateTime startDate);
}
```



### JpaRepository 인터페이스

JpaRepository는 다음과 같은 기본 CRUD 메서드를 제공합니다:

- `save(entity)`: 엔티티를 저장하거나 업데이트합니다.
- `findById(id)`: ID로 엔티티를 조회합니다.
- `findAll()`: 모든 엔티티를 조회합니다.
- `delete(entity)`: 엔티티를 삭제합니다.
- `count()`: 엔티티의 개수를 반환합니다.



## 5. 서비스(Service) 클래스 만들기

서비스 클래스는 비즈니스 로직을 처리하고 리포지토리를 통해 데이터베이스 작업을 수행합니다.

### 기본 서비스 클래스 예시

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    
    public User createUser(User user) {
        user.setCreatedAt(LocalDateTime.now());
        return userRepository.save(user);
    }
    
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("사용자를 찾을 수 없습니다."));
    }
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id);
        user.setName(userDetails.getName());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }
    
    public void deleteUser(Long id) {
        User user = getUserById(id);
        userRepository.delete(user);
    }
}
```



## 6. 컨트롤러(Controller)에서 서비스 사용하기

컨트롤러는 클라이언트의 요청을 받아 서비스를 호출하고 결과를 반환합니다.

### 기본 컨트롤러 클래스 예시

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User createdUser = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        User updatedUser = userService.updateUser(id, user);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```



## 7. CRUD란 무엇인가요?

CRUD는 데이터베이스에서 수행하는 기본적인 데이터 조작 작업의 약자입니다.

- **C(Create)**: 데이터 생성 - 새로운 정보를 데이터베이스에 추가합니다.
- **R(Read)**: 데이터 조회 - 저장된 정보를 검색하고 읽어옵니다.
- **U(Update)**: 데이터 수정 - 기존 정보를 변경합니다.
- **D(Delete)**: 데이터 삭제 - 저장된 정보를 제거합니다.

Spring Data JPA를 사용하면 이러한 CRUD 작업을 매우 간단하게 구현할 수 있습니다. 위에서 살펴본 `JpaRepository` 인터페이스는 이러한 기본적인 CRUD 작업을 위한 메서드를 모두 제공합니다.

우리도 위 User 예제 에서 CRUD 를 이미 모두 만들었습니다.



## 8. API 테스트하기

개발한 API를 테스트하는 방법은 여러 가지가 있습니다. 여기서는 `curl` 명령어를 사용한 테스트 방법을 소개합니다. 

Postman 에 등록하셔도 무방합니다. 또는 IntelliJ 다 Swagger 다른 방법을 이용할 수도 있습니다.



### 사용자 생성 (Create)

```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "홍길동",
    "email": "hong@example.com"
  }'
```



### 사용자 조회 (Read)

특정 사용자 조회:
```bash
curl -X GET http://localhost:8080/api/users/1
```

모든 사용자 조회:
```bash
curl -X GET http://localhost:8080/api/users
```



### 사용자 정보 수정 (Update)

```bash
curl -X PUT http://localhost:8080/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "한조각",
    "email": "han.updated@example.com"
  }'
```



### 사용자 삭제 (Delete)

```bash
curl -X DELETE http://localhost:8080/api/users/1
```



### 응답 확인하기

위 명령어들은 서버로부터 응답을 받게 됩니다. 실제 실행해보고 해당 요청이, 어울리는 응답이 왔는지 확인해보세요.

- 생성 성공: HTTP 201 Created
- 조회 성공: HTTP 200 OK
- 수정 성공: HTTP 200 OK
- 삭제 성공: HTTP 204 No Content

오류가 발생한 경우에는 다음과 같은 상태 코드를 받을 수 있습니다. 오류는 강제로 만들어서 테스트 해볼 수 있습니다. 언제 아래와 같은 예외가 나오는지 파악해두면, 실제 운영 상황에서 빠르게 대처할 수 있는 노하우를 얻어낼 수 있습니다.

- 잘못된 요청: HTTP 400 Bad Request
- 인증 실패: HTTP 401 Unauthorized
- 권한 없음: HTTP 403 Forbidden
- 리소스 없음: HTTP 404 Not Found
- 서버 오류: HTTP 500 Internal Server Error



