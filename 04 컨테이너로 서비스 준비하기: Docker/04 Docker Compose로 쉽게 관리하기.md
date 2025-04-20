# Docker Compose로 쉽게 관리하기

## 1. Docker Compose 소개

Docker Compose는 여러 Docker 컨테이너를 정의하고 실행하기 위한 도구입니다. YAML 파일을 사용하여 애플리케이션의 서비스를 정의하고, 단일 명령어로 모든 서비스를 시작하거나 중지할 수 있습니다.



### Docker Compose의 장점

**다중 컨테이너 관리**: 여러 컨테이너를 하나의 파일로 정의하고 관리할 수 있습니다.

**환경 일관성**: 개발, 테스트, 운영 환경에서 동일한 설정을 사용할 수 있습니다.

**간편한 명령어**: 복잡한 Docker 명령어 대신 간단한 명령어로 컨테이너를 관리할 수 있습니다.

**서비스 의존성 관리**: 서비스 간의 의존성을 정의하고 순서대로 시작할 수 있습니다.



## 2. Docker Compose 설치하기

### Docker Desktop에 포함된 Docker Compose

Docker Desktop for Windows 또는 macOS를 설치하면 Docker Compose가 함께 설치됩니다.



### Linux에서 설치하기

```bash
# Docker Compose 다운로드
sudo curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 실행 권한 부여
sudo chmod +x /usr/local/bin/docker-compose

# 설치 확인
docker-compose --version
```



## 3. docker-compose.yml 파일 작성하기

### 기본 구조

```yaml
version: '3.8'

services:
  # 서비스 정의
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/twitterdb
      - SPRING_DATASOURCE_USERNAME=dev
      - SPRING_DATASOURCE_PASSWORD=dev123

  db:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=twitterdb
      - MYSQL_USER=dev
      - MYSQL_PASSWORD=dev123
      - MYSQL_ROOT_PASSWORD=rootpassword
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```



### 주요 설정 항목

- **version**: Docker Compose 파일 형식 버전
- **services**: 실행할 서비스(컨테이너) 정의
- **build**: Dockerfile 위치 또는 빌드 컨텍스트
- **image**: 사용할 Docker 이미지
- **ports**: 포트 매핑 (호스트:컨테이너)
- **environment**: 환경 변수 설정
- **depends_on**: 서비스 의존성 정의
- **volumes**: 볼륨 마운트 정의
- **networks**: 네트워크 설정



## 4. Docker Compose 명령어

### 기본 명령어

```bash
# 서비스 시작
docker-compose up

# 백그라운드에서 서비스 시작
docker-compose up -d

# 서비스 중지
docker-compose down

# 서비스 상태 확인
docker-compose ps

# 서비스 로그 확인
docker-compose logs

# 특정 서비스 로그 확인
docker-compose logs web

# 서비스 재시작
docker-compose restart

# 특정 서비스 재시작
docker-compose restart web
```



### 빌드 관련 명령어

```bash
# 이미지 빌드
docker-compose build

# 이미지 빌드 및 서비스 시작
docker-compose up --build

# 특정 서비스만 빌드
docker-compose build web
```



## 5. Spring Boot + MySQL 예제

### 프로젝트 구조

```
springboot-mysql-docker/
├── src/
│   └── main/
│       ...
├── Dockerfile
├── docker-compose.yml
└── build.gradle
```



### Dockerfile

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```



### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/twitterdb
      - SPRING_DATASOURCE_USERNAME=dev
      - SPRING_DATASOURCE_PASSWORD=dev123
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
      - SPRING_JPA_SHOW_SQL=true

  db:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=twitterdb
      - MYSQL_USER=dev
      - MYSQL_PASSWORD=dev123
      - MYSQL_ROOT_PASSWORD=root123
    volumes:
      - mysql-data:/var/lib/mysql
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

volumes:
  mysql-data:
```



### 5실행 방법

```bash
# 애플리케이션 빌드
./gradlew build

# Docker Compose로 서비스 시작
docker-compose up -d

# 서비스 상태 확인
docker-compose ps

# 로그 확인
docker-compose logs -f
```



## 6. 고급 Docker Compose 기능

### 네트워크 설정

```yaml
version: '3.8'

services:
  web:
    # ... 기존 설정 ...
    networks:
      - frontend
      - backend

  db:
    # ... 기존 설정 ...
    networks:
      - backend

networks:
  frontend:
  backend:
```



### 서비스 확장

```bash
# 서비스 인스턴스 수 지정
docker-compose up -d --scale web=3
```



### 환경 변수 파일 사용

`.env` 파일 생성:

```
MYSQL_DATABASE=twitterdb
MYSQL_USER=dev
MYSQL_PASSWORD=dev123
MYSQL_ROOT_PASSWORD=rootpassword
```

`docker-compose.yml`에서 환경 변수 파일 사용:

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    env_file:
      - .env
    # ... 기존 설정 ...
```



### 헬스 체크

```yaml
version: '3.8'

services:
  web:
    # ... 기존 설정 ...
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    # ... 기존 설정 ...
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```



## 7. Docker Compose 프로덕션 환경

### 다중 환경 설정

`docker-compose.yml`과 `docker-compose.prod.yml` 파일을 분리하여 관리:

```bash
# 개발 환경
docker-compose up -d

# 프로덕션 환경
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```



### 프로덕션 환경 설정 예시

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    restart: always
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_JPA_HIBERNATE_DDL_AUTO=none
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    restart: always
    deploy:
      placement:
        constraints:
          - node.role == manager
```

