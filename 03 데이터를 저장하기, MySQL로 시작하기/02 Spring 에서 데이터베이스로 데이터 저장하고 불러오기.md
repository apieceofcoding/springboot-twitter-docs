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



## 2. Spring Boot 프로젝트에 데이터베이스 설정하기

### 의존성 추가하기

Spring Boot 프로젝트에 데이터베이스 관련 의존성을 추가합니다. `build.gradle.kts` 파일에 다음 의존성을 추가합니다.

```gradle
dependencies {
    // Spring Data JPA
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    
    // H2 데이터베이스 드라이버
    runtimeOnly("com.h2database:h2")
}
```





### 데이터베이스 연결 설정하기

`application.yml` 파일 (또는 application.properties) 에 데이터베이스 연결 정보를 설정합니다:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: ""
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  h2:
    console:
      enabled: true
```



## 3. 엔티티(Entity) 클래스 만들기

엔티티 클래스는 데이터베이스 테이블과 매핑되는 Java 클래스입니다. JPA 어노테이션을 사용하여 엔티티를 정의합니다.

### 게시글(Post) 엔티티 클래스 예시

```java
@Getter
@Setter
@Table(name = "posts")
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String content;

    private LocalDateTime createdAt;

    public void updateContent(String content) {
        this.content = content;
    }
}
```

**JPA 엔티티에서 record를 사용할 수 없습니다.**

- 가변성(Mutability) 필요: JPA는 엔티티의 상태를 관리하기 위해 필드값을 변경할 수 있어야 합니다. 하지만 record는 불변(immutable) 클래스로, 한번 생성되면 필드값을 변경할 수 없습니다.
- 프록시 생성: JPA는 지연 로딩(lazy loading)을 구현하기 위해 엔티티의 프록시 클래스를 생성합니다. record는 final 클래스이므로 프록시를 생성할 수 없습니다. (프록시는 원본 클래스를 상속하여 만들어집니다. 즉, final 클래스는 상속이 불가능하므로 프록시를 만들 수 없습니다.)


따라서 엔티티는 일반 클래스로 작성해야 합니다. 대신 Lombok의 @Getter, @Setter 등을 사용하여 코드를 간결하게 만들 수 있습니다.



생성자 관련 롬복 어노테이션

- `@AllArgsConstructor` : 모든 필드를 인자로 가지는 생성자를 만듭니다. 직접 정적 메소드로 구현해도 좋습니다만, 편의상 위 어노테이션을 사용했습니다.
- `@NoArgsConstructor` : 기본 생성자 (파라미터 없는 생성자) 를 만듭니다. `@AllArgsConstructor` 때문에 다른 생성자가 만들어지면, 자바가 기본으로 만들어주는 기본 생성자는 더 이상 만들어지지 않습니다. 하지만, 우리는 JPA 사용을 위해 기본 생성자가 필요하여 이 어노테이션을 추가했습니다.
- `@Builder` : 원하는 필드로만 구성된 객체를 읽기 쉽고, 실수를 방지할 수 있도록 도와주는 빌더 패턴을 사용할 수 있도록 도와줍니다.






## 4. 리포지토리(Repository) 인터페이스 만들기

리포지토리는 데이터베이스 접근을 담당하는 인터페이스입니다. Spring Data JPA는 리포지토리 인터페이스를 구현하여 데이터베이스 작업을 수행합니다.

### JPA 저장소 구현체 예시

```java
public interface JpaPostRepository extends JpaRepository<Post, Long> {
  
}
```

위처럼만 해도 사용할 수 있습니다.



하지만 우리는 자체적으로 PostRepository 인터페이스를 먼저 만들어 놓았기 때문에 이를 사용할 수 있도록 만들어 봅시다.

```java
public interface JpaPostRepository extends JpaRepository<Post, Long>, PostRepository {
    
    @Override
    default List<Post> findAllPaged(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("id").descending());
        return findAll(pageable).getContent();
    }
}
```

- PostRepository 인터페이스 안의 나머지 메소드는 JpaRepository 의 기본 구현체인 `SimpleJpaRepository` 에 이미 다 구현이 되어 있습니다. 따라서 우리는 `findAllPaged` 만 구현하면 됩니다.
  - save, findAll, findById, deleteById 등



## 5. 컨트롤러에서 PostRepository 직접 사용하기

이전에 만들어놓았던 PostController 에서 PostRepository 인터페이스를 사용하고 있으므로, PostController 는 변경할 부분이 없습니다.

대신 PostRepository 에 구현한 2개의 구현체 중에 어느 것을 선택할지 결정해주어야 하는데요.

이때는 스프링의 **의존성 주입(DI, Dependency Injection)** 을 사용하면 됩니다. 

```java
@Configuration
public class RepositoryConfig {

    @Bean
    public PostRepository postRepository() {
        return new InMemoryPostRepository();
    }
}
```

- 위와 같이 `@Configuration` 설정파일을 만들어놓고, `@Bean` 을 통해 특정 클래스를 빈으로 등록해주면 됩니다.



만약 JpaPostRepository 로 하고 싶다면 아래와 같이 하면 됩니다.

```java
@Configuration
public class RepositoryConfig {

    @Bean
    public PostRepository postRepository(JpaPostRepository jpaPostRepository) {
        return jpaPostRepository;
    }
}
```

- 우리가 만든 JpaPostRepository 인터페이스는 `JpaRepository` 인터페이스를 상속 받고 있고, 이 인터페이스의 기본 구현체가 `SimpleJpaRepository` 입니다. 그리고 스프링 자동설정에 의해 이미 빈으로 등록되어 있습니다.
- 따라서 JPA 가 제공하는 위 구현체를 사용하려면, 파라미터로 JpaPostRepository 를 넣어주고 그대로 반환해주면 됩니다.





## 6. CRUD란 무엇인가요?

CRUD는 데이터베이스에서 수행하는 기본적인 데이터 조작 작업의 약자입니다.

- **C(Create)**: 데이터 생성 - 새로운 정보를 데이터베이스에 추가합니다.
- **R(Read)**: 데이터 조회 - 저장된 정보를 검색하고 읽어옵니다.
- **U(Update)**: 데이터 수정 - 기존 정보를 변경합니다.
- **D(Delete)**: 데이터 삭제 - 저장된 정보를 제거합니다.

Spring Data JPA를 사용하면 이러한 CRUD 작업을 매우 간단하게 구현할 수 있습니다. 위에서 살펴본 `JpaRepository` 인터페이스는 이러한 기본적인 CRUD 작업을 위한 메서드를 모두 제공합니다.

우리도 위 Post 예제 에서 CRUD 를 이미 모두 만들었습니다.



## 7. H2 Database

http://localhost:8080/h2-console

브라우저를 통해 위 주소에 들어가면, H2 database 를 시각적으로 조회하고 제어해볼 수 있습니다.

- JDBC URL 이 `jdbc:h2:mem:testdb` 이 맞는지 꼭 확인하고 Connect 버튼을 눌러주세요.



들어가서 데이터베이스를 조회, 생성, 삭제, 수정해봅시다.

H2 DB 는 간단한 인메모리 저장소이지만, 이렇게 사용자 편의 기능까지 제공하여 개발용으로 많이 사용됩니다.



