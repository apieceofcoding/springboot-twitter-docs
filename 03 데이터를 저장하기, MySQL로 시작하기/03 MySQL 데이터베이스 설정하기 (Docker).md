# MySQL 데이터베이스 설정하기



## 1. Docker로 MySQL 설치하기

### Docker 설치 확인

먼저 Docker가 설치되어 있는지 확인합니다:

```bash
docker --version
```

Docker가 설치되어 있지 않다면, 다음 명령어로 설치할 수 있습니다:

```bash
# macOS
brew install --cask docker

# Windows
# https://www.docker.com/products/docker-desktop 에서 다운로드
```



### MySQL 이미지 다운로드

MySQL 이미지를 다운로드합니다:

```bash
docker pull mysql:8.0
```



### MySQL 컨테이너 실행

MySQL 컨테이너를 실행합니다:

```bash
docker run --name mysql-twitter \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=twitterdb \
  -e MYSQL_USER=dev \
  -e MYSQL_PASSWORD=dev123 \
  -p 3306:3306 \
  -d mysql:8.0
```

각 매개변수의 의미:
- `--name mysql-twitter`: 컨테이너 이름
- `-e MYSQL_ROOT_PASSWORD`: root 사용자의 비밀번호
- `-e MYSQL_DATABASE`: 생성할 데이터베이스 이름
- `-e MYSQL_USER`: 생성할 일반 사용자 이름
- `-e MYSQL_PASSWORD`: 일반 사용자의 비밀번호
- `-p 3306:3306`: 호스트의 3306 포트를 컨테이너의 3306 포트에 매핑
- `-d`: 백그라운드에서 실행



### MySQL 컨테이너 상태 확인

```bash
docker ps
```



### MySQL 컨테이너 접속

```bash
docker exec -it mysql-twitter mysql -u root -p
```

비밀번호를 입력하면 MySQL 프롬프트가 나타납니다.



## 2. Spring Boot 프로젝트 설정하기

### MySQL 의존성 추가

`build.gradle` 파일에 MySQL 의존성을 추가합니다:

```gradle
dependencies {
    // MySQL 드라이버
    runtimeOnly 'mysql:mysql-connector-java'
}
```



### 데이터베이스 연결 설정

`application.properties` 파일에 MySQL 연결 정보를 설정합니다:

```properties
# MySQL 데이터베이스 설정
spring.datasource.url=jdbc:mysql://localhost:3306/twitterdb?useSSL=false&serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.username=dev
spring.datasource.password=dev123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA 설정
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

각 설정의 의미:
- `spring.datasource.url`: 데이터베이스 연결 URL
  - `useSSL=false`: SSL 연결 비활성화
  - `serverTimezone=UTC`: 서버 시간대 설정
  - `characterEncoding=UTF-8`: 문자 인코딩 설정
- `spring.datasource.username`: 데이터베이스 사용자 이름
- `spring.datasource.password`: 데이터베이스 비밀번호
- `spring.datasource.driver-class-name`: MySQL 드라이버 클래스
- `spring.jpa.hibernate.ddl-auto`: 테이블 자동 생성 설정
  - `update`: 테이블이 없으면 생성하고, 엔티티가 변경되면 테이블 구조도 변경
- `spring.jpa.show-sql`: SQL 쿼리 로그 출력
- `spring.jpa.properties.hibernate.format_sql`: SQL 쿼리 포맷팅
- `spring.jpa.properties.hibernate.dialect`: Hibernate SQL 방언 설정



## 3. 데이터베이스 관리 도구 설치하기

### MySQL Workbench 설치

MySQL Workbench는 MySQL 데이터베이스를 시각적으로 관리할 수 있는 도구입니다.

1. [MySQL Workbench 다운로드 페이지](https://dev.mysql.com/downloads/workbench/)에서 운영체제에 맞는 버전을 다운로드합니다.
2. 설치 파일을 실행하여 설치를 완료합니다.
3. MySQL Workbench를 실행하고 새로운 연결을 생성합니다:
   - Hostname: localhost
   - Port: 3306
   - Username: dev
   - Password: dev123



### DBeaver 설치 (대안)

DBeaver는 다양한 데이터베이스를 지원하는 범용 데이터베이스 관리 도구입니다.

1. [DBeaver 다운로드 페이지](https://dbeaver.io/download/)에서 운영체제에 맞는 버전을 다운로드합니다.
2. 설치 파일을 실행하여 설치를 완료합니다.
3. DBeaver를 실행하고 새로운 데이터베이스 연결을 생성합니다:
   - Database: MySQL
   - Host: localhost
   - Port: 3306
   - Database: twitterdb
   - Username: dev
   - Password: dev123



그 외에 여러 좋은 도구가 있습니다. 

Sequel Pro 는 Mac 에서 사용하기가 편리하고,

DataGrip 은 IntelliJ 와 UI 가 비슷해서 사용이 더욱 편합니다. (하지만 유료) 



아래는 데이터베이스 관리도구들을 비교한 표인데 사용에 참고하시면 좋을 것 같습니다.

| 항목          | **MySQL Workbench**                 | **DBeaver**                          | **Sequel Pro**                | **DataGrip**                         |
| ------------- | ----------------------------------- | ------------------------------------ | ----------------------------- | ------------------------------------ |
| **제작사**    | Oracle                              | DBeaver Corp. (오픈소스)             | 오픈소스                      | JetBrains                            |
| **지원 OS**   | Windows, macOS, Linux               | Windows, macOS, Linux                | macOS 전용                    | Windows, macOS, Linux                |
| **지원 DB**   | MySQL, MariaDB                      | 대부분의 DBMS 지원                   | MySQL, MariaDB                | 대부분의 DBMS 지원                   |
| **라이선스**  | 무료                                | 무료 (Community) / 유료 (Enterprise) | 무료                          | 유료 (연간 구독)                     |
| **한글 지원** | O                                   | O                                    | X (깨짐 현상 있음)            | O                                    |
| **UI/UX**     | 보통 (전통적 UI)                    | 다양하고 유연함                      | 매우 직관적 (단순)            | 현대적이고 세련됨                    |
| **장점**      | MySQL 전용 최적화, Oracle 공식 지원 | 다양한 DB 지원, 플러그인 풍부        | 간단하고 빠름, mac에서 가볍게 | 강력한 코드 완성, 리팩토링, Git 연동 |
| **단점**      | MySQL 외 DB 불편                    | 일부 기능은 유료                     | mac 전용, 오래된 프로젝트     | 유료, 무거움                         |





## 4. 데이터베이스 성능 최적화 (맛보기)

### 인덱스 생성

자주 조회하는 컬럼에 인덱스를 생성하여 조회 성능을 향상시킬 수 있습니다:

```sql
-- 단일 컬럼 인덱스
CREATE INDEX idx_posts_title ON posts(title);
```



### 쿼리 최적화

1. **EXPLAIN 사용하기**:
   ```sql
   EXPLAIN SELECT * FROM posts WHERE title LIKE '%검색어%';
   ```

2. **불필요한 조인 피하기**:
   ```sql
   -- 나쁜 예
   SELECT p.*, u.* FROM posts p JOIN users u ON p.user_id = u.id;
   
   -- 좋은 예
   SELECT p.* FROM posts p WHERE p.user_id = 1;
   ```

3. **페이징 처리 사용하기**:
   ```sql
   SELECT * FROM posts ORDER BY created_at DESC LIMIT 10 OFFSET 0;
   ```



### 커넥션 풀 설정

`application.properties` 파일에 커넥션 풀 설정을 추가합니다.

```properties
# HikariCP 커넥션 풀 설정
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.max-lifetime=1200000
```



## 5. 보안 설정

### 사용자 권한 관리

```sql
-- 사용자 생성
CREATE USER 'dev'@'localhost' IDENTIFIED BY 'twitterpassword';

-- 데이터베이스 권한 부여
GRANT SELECT, INSERT, UPDATE, DELETE ON twitterdb.* TO 'dev'@'localhost';

-- 권한 적용
FLUSH PRIVILEGES;
```

