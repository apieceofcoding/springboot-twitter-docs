# 컨트롤러로 API 만들기

## 1. 컨트롤러란 무엇인가요?

컨트롤러는 **클라이언트의 요청을 받아서 처리하는 역할을 하는 클래스**입니다. 이전에 배운 API의 개념을 실제 코드로 구현하는 부분이라고 생각하시면 됩니다.

마치 식당의 웨이터처럼, 손님(클라이언트)의 주문(요청)을 받아서 주방(서비스)에 전달하고, 결과를 손님에게 전달하는 역할을 합니다.

Spring Boot에서는 `@Controller` 또는 `@RestController` 어노테이션을 사용하여 컨트롤러를 만들 수 있습니다.





## 2. 기본적인 컨트롤러 만들기

### 간단한 컨트롤러 예시

```java
@RestController
public class PostController {

    @GetMapping("/posts")
    public String getPosts() {
        return "안녕하세요!";
    }
}
```

- `@RestController`: 이 클래스가 REST API 컨트롤러임을 나타냅니다.
- `@GetMapping("/posts")`: GET 요청으로 `/posts`에 접근하면 이 메서드가 실행됩니다.





## 3. 응답 데이터 보내기

### JSON 응답

```json
{
  "id": 1,
  "content": "안녕하세요!",
  "createdAt": "2024-05-01T12:34:56.789"
}
```
웹 브라우저로 전달될 데이터는 위와 같은 JSON 형식입니다.

반면, Spring Boot에서는 이 데이터를 '객체'로 다루면 편합니다.
```java
public record Post(
    Long id;
    String content;
    LocalDateTime createdAt;
) { 
    
}
```


```java
@GetMapping("/posts")
public Post getPosts() {
    return new Post(1L, "안녕하세요!", LoacalDateTime.now());
}
```
Spring Boot에서는 객체를 JSON으로 자동 변환해주는 기능을 제공하기 때문에,
우리는 별도로 변환 코드를 만들 필요없이 객체를 그대로 반환하면 됩니다.
참고로 Jackson 라이브러리가 이 작업을 처리하며, Spring Boot 에 기본적으로 포함되어 있습니다.



### 다양한 HTTP 메서드 처리하기

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {

    // 게시글을 저장할 Map (임시 데이터 저장소)
    private Map<Long, Post> posts = new HashMap<>();
    private AtomicLong idGenerator = new AtomicLong(1);

    // 게시글 목록 조회
    @GetMapping
    public List<Post> getAllPosts() {
        return new ArrayList<>(posts.values());
    }

    // 특정 게시글 조회
    @GetMapping("/{id}")
    public Post getPost(@PathVariable Long id) {
        return posts.get(id);
    }

    // 새 게시글 작성
    @PostMapping
    public Post createPost(@RequestBody Post post) {
        long id = idGenerator.getAndIncrement();
        post.setId(id);
        posts.put(id, post);
        return post;
    }

    // 게시글 수정
    @PutMapping("/{id}")
    public Post updatePost(@PathVariable Long id, @RequestBody Post post) {
        post.setId(id);
        posts.put(id, post);
        return post;
    }

    // 게시글 삭제
    @DeleteMapping("/{id}")
    public void deletePost(@PathVariable Long id) {
        posts.remove(id);
    }
}
```
이 코드는 실제 데이터베이스나 서비스 없이, 컨트롤러 내부의 Map을 임시 저장소로 사용하여 게시글을 관리하는 예시입니다.
- `Map<Long, Post> posts` : 게시글을 메모리에 저장하는 임시 저장소입니다.
- `AtomicLong idGenerator` : 게시글의 고유 id를 자동으로 증가시켜 생성합니다.
- 각 HTTP 메서드별로 게시글을 생성, 조회, 수정, 삭제할 수 있습니다.
- 실제 서비스에서는 데이터베이스와 서비스 레이어를 사용하지만, 여기서는 개념 이해를 위해 간단하게 구현했습니다.



### HTTP 상태 코드 지정



#### ResponseEntity 사용

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User savedUser = userService.save(user);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(savedUser);
}
```
위 예시는 ResponseEntity를 사용해 상태 코드를 지정하는 방법입니다.



#### 어노테이션 기반 상태 코드 지정

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}
```
- `@ResponseStatus(HttpStatus.CREATED)` 어노테이션을 사용하면, 메서드가 반환될 때 지정한 상태 코드(여기서는 201 Created)로 응답합니다.
- ResponseEntity를 사용하지 않고도 간단하게 상태 코드를 지정할 수 있습니다.


















---



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
@PostMapping("/posts")
public Post createPost(@RequestBody Post post) {
    return postService.save(post);
}
```

- `@RequestBody`: HTTP 요청의 본문을 객체로 변환합니다.
- JSON 형식의 데이터를 자동으로 Java 객체로 변환합니다.
