# 컨트롤러로 API 만들기

## 1. 컨트롤러란 무엇인가요?

컨트롤러는 **클라이언트의 요청을 받아서 처리하는 역할을 하는 클래스**입니다. 이전에 배운 API의 개념을 실제 코드로 구현하는 부분이라고 생각하시면 됩니다.

마치 식당의 웨이터처럼, 손님(클라이언트)의 주문(요청)을 받아서 주방(서비스)에 전달하고, 결과를 손님에게 전달하는 역할을 합니다.

Spring Boot에서는 `@Controller` 또는 `@RestController` 어노테이션을 사용하여 컨트롤러를 만들 수 있습니다.



## 2. 기본적인 컨트롤러 만들기

### 간단한 컨트롤러 예시

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "안녕하세요!";
    }
}
```

- `@RestController`: 이 클래스가 REST API 컨트롤러임을 나타냅니다.
- `@GetMapping("/hello")`: GET 요청으로 `/hello`에 접근하면 이 메서드가 실행됩니다.



### 다양한 HTTP 메서드 처리하기

```java
@RestController
@RequestMapping("/api/posts")
@RequiredArgsConstructor
public class PostController {

    private final PostService postService;

    // 게시글 목록 조회
    @GetMapping
    public List<Post> getAllPosts() {
        return postService.findAll();
    }

    // 특정 게시글 조회
    @GetMapping("/{id}")
    public Post getPost(@PathVariable Long id) {
        return postService.findById(id);
    }

    // 새 게시글 작성
    @PostMapping
    public Post createPost(@RequestBody Post post) {
        return postService.save(post);
    }

    // 게시글 수정
    @PutMapping("/{id}")
    public Post updatePost(@PathVariable Long id, @RequestBody Post post) {
        return postService.update(id, post);
    }

    // 게시글 삭제
    @DeleteMapping("/{id}")
    public void deletePost(@PathVariable Long id) {
        postService.delete(id);
    }
}
```



## 3. 요청 데이터 받기

### URL 파라미터 받기 (query string)

```java
@GetMapping("/search")
public List<Post> searchPosts(
    @RequestParam(required = false) String keyword,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size
) {
    return postService.search(keyword, page, size);
}
```

- `@RequestParam`: URL의 쿼리 파라미터를 받습니다.
- `required = false`: 파라미터가 필수가 아님을 나타냅니다.
- `defaultValue`: 파라미터가 없을 때의 기본값을 지정합니다.



### 경로 변수 받기 (path variable)

```java
@GetMapping("/users/{userId}/posts/{postId}")
public Post getPost(
    @PathVariable Long userId,
    @PathVariable Long postId
) {
    return postService.findByUserIdAndPostId(userId, postId);
}
```

- `@PathVariable`: URL 경로의 변수 부분을 받습니다.
- `{userId}`와 `{postId}`는 실제 값으로 대체됩니다.



### 요청 본문 받기 (request body)

```java
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}
```

- `@RequestBody`: HTTP 요청의 본문을 객체로 변환합니다.
- JSON 형식의 데이터를 자동으로 Java 객체로 변환합니다.



## 4. 응답 데이터 보내기

### 기본 응답

```java
@GetMapping("/hello")
public String hello() {
    return "안녕하세요!";
}
```

### JSON 응답

```java
@GetMapping("/user")
public User getUser() {
    return new User("홍길동", "hong@example.com");
}
```



### HTTP 상태 코드 지정

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User savedUser = userService.save(user);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(savedUser);
}
```



## 5. 예외 처리하기

### 기본 예외 처리

```java
@ExceptionHandler(PostNotFoundException.class)
public ResponseEntity<String> handlePostNotFound(PostNotFoundException e) {
    return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(e.getMessage());
}
```

### 전역 예외 처리

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleAllExceptions(Exception e) {
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body("서버 오류가 발생했습니다.");
    }
}
```

