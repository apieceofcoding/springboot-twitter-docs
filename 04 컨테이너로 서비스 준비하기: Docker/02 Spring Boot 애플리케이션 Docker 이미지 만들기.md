# Spring Boot 애플리케이션 Docker 이미지 만들기



## 1. Dockerfile 작성하기

Dockerfile은 Docker 이미지를 빌드하기 위한 지침이 포함된 텍스트 파일입니다. Spring Boot 애플리케이션을 위한 Dockerfile을 작성해보겠습니다.



### 기본 Dockerfile 구조

```dockerfile
# 베이스 이미지 선택
FROM openjdk:17-jdk-slim

# 작업 디렉토리 설정
WORKDIR /app

# JAR 파일 복사
COPY build/libs/*.jar app.jar

# 포트 노출
EXPOSE 8080

# 실행 명령
ENTRYPOINT ["java", "-jar", "app.jar"]
```



### 멀티 스테이지 빌드 (최적화된 방식)

```dockerfile
# 빌드 스테이지
FROM gradle:7.6.1-jdk17 AS build
WORKDIR /app
COPY . .
RUN gradle build --no-daemon

# 실행 스테이지
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

멀티 스테이지 빌드는 최종 이미지 크기를 줄이고 보안을 향상시키는 데 도움이 됩니다.



## 2. Spring Boot 애플리케이션 준비하기

### 애플리케이션 빌드

먼저 Spring Boot 애플리케이션을 빌드합니다:

```bash
# Gradle을 사용하는 경우
./gradlew build

# Maven을 사용하는 경우
mvn clean package
```



### 애플리케이션 설정 확인

`application.properties` 또는 `application.yml` 파일에서 다음 설정을 확인합니다:

```properties
# 서버 포트 설정
server.port=8080

# 데이터베이스 연결 설정 (필요한 경우)
spring.datasource.url=jdbc:mysql://localhost:3306/twitterdb
spring.datasource.username=dev
spring.datasource.password=dev123
```



## 3. Docker 이미지 빌드하기

### 이미지 빌드 명령어

```bash
# 기본 빌드
docker build -t springboot-twitter:latest .

# 멀티 스테이지 빌드
docker build -t springboot-app:twitter -f Dockerfile.multistage .
```



### 빌드 옵션 설명

- `-t springboot-app:latest`: 이미지 이름과 태그 지정
- `-f Dockerfile.multistage`: 사용할 Dockerfile 지정 (기본값은 'Dockerfile')



### 빌드 캐시 활용

Docker는 빌드 과정을 캐시하여 반복적인 빌드 시간을 단축합니다. Dockerfile의 각 명령어는 레이어로 저장되며, 변경된 레이어만 다시 빌드됩니다.



## 4. Docker 컨테이너 실행하기

### 기본 실행

```bash
docker run -d -p 8080:8080 --name springboot-twitter-container springboot-twitter:latest
```



### 환경 변수 설정

```bash
docker run -d -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/twitterdb \
  -e SPRING_DATASOURCE_USERNAME=dev \
  -e SPRING_DATASOURCE_PASSWORD=dev123 \
  --name springboot-twitter-container springboot-twitter:latest
```



### 볼륨 마운트 (로그 파일 등)

```bash
docker run -d -p 8080:8080 \
  -v /path/to/logs:/app/logs \
  --name springboot-twitter-container springboot-twitter:latest
```



## 5. Spring Boot 애플리케이션 최적화

### JVM 옵션 설정

```dockerfile
ENTRYPOINT ["java", "-Xmx512m", "-Xms256m", "-jar", "app.jar"]
```



### 레이어 최적화

Dockerfile의 명령어 순서를 최적화하여 캐시 활용도를 높입니다:

```dockerfile
# 의존성 파일 먼저 복사 (변경이 적은 파일)
COPY build.gradle settings.gradle ./
COPY gradle ./gradle

# 소스 코드 복사 (자주 변경되는 파일)
COPY src ./src

# 빌드 실행
RUN gradle build --no-daemon
```



### .dockerignore 파일 사용

불필요한 파일이 이미지에 포함되지 않도록 .dockerignore 파일을 생성합니다:

```
.git
.gradle
build
!build/libs/*.jar
```



## 6. 문제 해결 및 디버깅

### 컨테이너 로그 확인

```bash
docker logs springboot-container
```

### 컨테이너 내부 접속

```bash
docker exec -it springboot-container /bin/bash
```

### 컨테이너 상태 확인

```bash
docker inspect springboot-container
```

