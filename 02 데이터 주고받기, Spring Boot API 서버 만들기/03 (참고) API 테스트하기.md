# 3. API 테스트하기

## 1. Postman 사용하기

Postman은 API를 테스트하기 위한 인기 있는 도구입니다. 다음과 같은 요청을 보내 API를 테스트할 수 있습니다:

1. **게시글 목록 조회**:

   - GET http://localhost:8080/api/posts

2. **특정 게시글 조회**:

   - GET http://localhost:8080/api/posts/1

3. **새 게시글 작성**:

   - POST http://localhost:8080/api/posts

   - Body (JSON):

     ```json
     {
       "content": "안녕하세요! 이것은 첫 번째 게시글입니다."

     }
     ```

4. **게시글 수정**:

   - PUT http://localhost:8080/api/posts/1

   - Body (JSON):

     ```json
     {
       "content": "게시글이 수정되었습니다."
     }
     ```

5. **게시글 삭제**:

   - DELETE http://localhost:8080/api/posts/1



## 2. curl 명령어 사용하기

터미널에서 curl 명령어를 사용하여 API를 테스트할 수도 있습니다:

```bash
# 게시글 목록 조회
curl -X GET http://localhost:8080/api/posts

# 특정 게시글 조회
curl -X GET http://localhost:8080/api/posts/1

# 새 게시글 작성
curl -X POST http://localhost:8080/api/posts \
  -H "Content-Type: application/json" \
  -d '{"content":"안녕하세요!"}'

# 게시글 수정
curl -X PUT http://localhost:8080/api/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"content":"게시글이 수정되었습니다."}'

# 게시글 삭제
curl -X DELETE http://localhost:8080/api/posts/1
```

