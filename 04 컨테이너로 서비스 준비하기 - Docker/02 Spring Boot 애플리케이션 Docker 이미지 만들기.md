# Spring Boot 애플리케이션 Docker 이미지 만들기



# 1. Jar 파일 생성하고 spring boot 앱 실행하기

먼저 spring boot 앱을 실행시킬 수 있는 jar 파일을 생성해야 합니다.

```java
./gradlew clean build
```

- clean 은 이전 빌드결과를 지웁니다. (`build/` 폴더)
- build 에 포함되는 주요 task들
  1. **`compileJava`**
     - `src/main/java` 아래의 Java 소스 코드 컴파일
  2. **`processResources`**
     - `src/main/resources`에 있는 리소스 복사 (예: `application.yml`, `static` 파일 등)
  3. **`compileTestJava`**
     - `src/test/java`의 테스트 코드 컴파일
  4. **`processTestResources`**
     - `src/test/resources` 리소스 복사
  5. **`test`**
     - 단위 테스트 실행 (`JUnit`, `Mockito` 등)
     - **테스트 실패 시 빌드 중단됨**
  6. **`jar`**
     - 일반 JAR 파일 생성
     - 실행 가능한 Spring Boot JAR 아님
  7. **`bootJar`** (Spring Boot 플러그인이 있을 때)
     - 실행 가능한 fat JAR 생성
     - 모든 의존성과 리소스를 포함한 단일 파일



build 가 완료되면 `build/libs/` 아래에 `springboottwitter-0.0.1-SNAPSHOT.jar` (bootJar 결과물) 로 우리 애플리케이션을 실행할 수 있습니다.

```bash
java -jar build/libs/springboottwitter-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.4.4)
...
```



우리는 실행가능한 JAR 즉, fat jar 만 필요하니 일반 JAR 는 생성하지 않게 해봅시다.

```build.gradle.kts
tasks.jar {
    enabled = false
}
```

- `jar` 태스크를 비활성화하여 일반 jar 는 만들어지지 않게 합니다.





# 2. Dockerfile 작성하기

Dockerfile은 Docker 이미지를 빌드하기 위한 규칙이 포함된 텍스트 파일입니다. 

Spring Boot 애플리케이션을 위한 Dockerfile을 작성해보겠습니다.



### 기본 Dockerfile 구조

```dockerfile
# JDK 21 기반의 OpenJDK 이미지를 사용
FROM openjdk:21-jdk-slim

# 작업 디렉토리 설정
WORKDIR /app

# 빌드된 JAR 파일을 컨테이너에 복사
COPY build/libs/*.jar app.jar

# 8080 포트 노출
EXPOSE 8080

# 애플리케이션 실행
ENTRYPOINT ["java", "-jar", "app.jar"]
```





# 3. Spring Boot 애플리케이션 준비하기

**Docker 이미지 빌드하기**

```bash
docker build -t springboot-twitter:latest .
```

- `-t springboot-app:latest`: 이미지 이름과 태그 지정



# 4. Docker 컨테이너 실행하기

**기본 실행**

```bash
docker run -d -p 8080:8080 --name springboot-twitter-container springboot-twitter:latest
```



여기서 문제가 발생합니다.

mysql 컨테이너가 띄워진 상황에서 spring boot 컨테이너를 실행하면 mysql 에 연결하지 못하게 되는데요.

이는 spring boot 컨테이너에서 localhost 는 내 PC 가 아닌 컨테이너 자신을 가리키기 때문입니다.



이를 해결할 수 있는 가장 간단한 방법은 docker compose 를 통해 두 컨테이너를 한 네트워크로 연결하여 같이 실행하는 것입니다.





# 5. Docker compose 로 두 컨테이너 함께 실행하기

**docker-compose.yaml**

```yaml
services:
  springboot-twitter:
    container_name: springboot-twitter
    image: springboot-twitter:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-twitter:3306/twitterdb
      - SPRING_DATASOURCE_USERNAME=dev
      - SPRING_DATASOURCE_PASSWORD=dev123
    networks:
      - twitter-network
    depends_on:
      - mysql-twitter

  mysql-twitter:
    container_name: mysql-twitter
    image: mysql:9.3
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: twitterdb
      MYSQL_USER: dev
      MYSQL_PASSWORD: dev123
    ports:
      - "3306:3306"
    networks:
      - twitter-network

networks:
  twitter-network:
    driver: bridge
```

**spring boot 애플리케이션 컨테이너**

- `springboot-twitter` 서비스는 `springboot-twitter:latest` 이미지를 사용하여 실행되거나, 현재 디렉토리에서 `Dockerfile` 이 있다면 이를 사용하여 이미지를 빌드합니다.
- `port` 는 호스트:컨테이너 입니다. 즉 내 PC 8080 을 컨테이너 8080 포트와 매핑합니다.
- `environment`는 Spring Boot 애플리케이션의 환경 변수들을 설정합니다:

- `SPRING_DATASOURCE_URL`은 MySQL 데이터베이스에 연결할 URL을 설정합니다. 이 경우 MySQL 컨테이너는 `mysql-twitter`라는 이름으로 참조됩니다.
- `SPRING_DATASOURCE_USERNAME`과 `SPRING_DATASOURCE_PASSWORD`는 MySQL 사용자명과 비밀번호를 설정합니다.
- `depends_on`은 `springboot-twitter` 서비스가 `mysql-twitter` 서비스가 시작된 후에 실행되도록 보장합니다.
- `networks`는 `springboot-twitter` 서비스가 `twitter-network` 네트워크에 연결되도록 설정하여 MySQL 서비스와 통신할 수 있도록 합니다.



**mysql 컨테이너**

- `myql-twitter` 서비스는 `mysql:9.3` 이미지를 사용하여 MySQL 서버를 실행합니다.
- `environment`에서는 MySQL의 root 비밀번호, 데이터베이스 이름, 사용자명, 비밀번호를 설정합니다:

- `MYSQL_ROOT_PASSWORD`는 MySQL root 계정의 비밀번호입니다.
- `MYSQL_DATABASE`는 생성할 데이터베이스 이름을 설정합니다.
- `MYSQL_USER`와 `MYSQL_PASSWORD`는 MySQL 사용자 계정을 설정합니다.
- `ports`는 호스트의 `3306` 포트를 MySQL 컨테이너의 `3306` 포트에 매핑하여 외부에서 MySQL에 접근할 수 있도록 설정합니다.
- `networks`는 `mysql-twitter` 서비스가 `twitter-network` 네트워크에 연결되도록 설정하여 Spring Boot와 통신할 수 있도록 합니다.



**네트워크 정의**

- `networks`에서는 `twitter-network`라는 사용자 정의 네트워크를 정의합니다.
- `driver: bridge`는 `bridge` 네트워크 드라이버를 사용하여, 컨테이너들이 동일 네트워크 내에서 서로 통신할 수 있도록 설정합니다.



이제 아래 명령어로 도커 컴포즈를 실행해봅시다.

```bash
docker-compose up -d
```

- docker-compose 는 docker desktop 설치시 같이 설치됩니다.
- `-d` 는 백그라운드 실행옵션입니다.







### 여전히 springboot-twitter 가 실행 실패할 경우

depends_on 옵션 덕분에 mysql-twitter, springboot-twitter 컨테이너 순으로 실행되는 것은 맞습니다.

하지만 mysql-twitter 컨테이너가 완전히 실행되고 나서 springboot-twitter 가 실행하는 것을 보장하는 것은 아닙니다.

따라서 이 경우 간단한 조치로 springboot-twitter 가 임시로 몇초간 대기 후 실행하도록하여 해결해봅시다.

이는 임시적인 해결책일 뿐 실제 운영에서는 이보다는 다른 방법을 추천드립니다, 저희는 개발 테스트환경이므로 이렇게 진행해보겠습니다.

**Dockerfile 수정**

```Docker
ENTRYPOINT ["sh", "-c", "sleep 10 && java -jar app.jar"]
```

**이미지 다시 빌드**

```bash
docker build -t springboot-twitter:latest .
```

**도커 컴포즈 실행**

```bash
docker-compose up -d
```



