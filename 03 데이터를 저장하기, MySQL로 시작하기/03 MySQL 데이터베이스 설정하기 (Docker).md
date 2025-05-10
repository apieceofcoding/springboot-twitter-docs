# MySQL 데이터베이스 설정하기



## 1. Docker로 MySQL 설치하기

### Docker 설치 확인

먼저 Docker가 설치되어 있는지 확인합니다:

```bash
docker --version
```



Docker가 설치되어 있지 않다면, 아래 사이트에서 설치할 수 있습니다.

도커 데스크탑: https://www.docker.com/products/docker-desktop



### MySQL 이미지 다운로드

MySQL 이미지를 다운로드합니다:

```bash
docker pull mysql:9.3
```

- pull 을 하지않아도, `docker run` 을 할 때 해당 도커 이미지가 없다면 자동으로 `docker pull` 을 실행하게 됩니다.



### MySQL 컨테이너 실행

MySQL 컨테이너를 실행합니다:

```bash
docker run --name mysql-twitter \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=twitterdb \
  -e MYSQL_USER=dev \
  -e MYSQL_PASSWORD=dev123 \
  -p 3306:3306 \
  -d \
  mysql:9.3
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
root123
```

비밀번호를 입력하면 MySQL 프롬프트가 나타납니다.



**MySQL 명령어**

```shell
use twitterdb;

show tables;
```





## 2. Spring Boot 프로젝트 설정하기

### MySQL 의존성 추가

`build.gradle` 파일에 MySQL 의존성을 추가합니다:

```gradle
dependencies {
    // MySQL 드라이버
    implementation("com.mysql:mysql-connector-j")
}
```

참고) maven repository : https://mvnrepository.com/artifact/com.mysql/mysql-connector-j/9.3.0



### 데이터베이스 연결 설정

`application.properties` 파일에 MySQL 연결 정보를 설정합니다:

```properties
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/twitterdb
    username: dev
    password: dev123
```

각 설정의 의미:
- `spring.datasource.url`: 데이터베이스 연결 URL
- `spring.datasource.username`: 데이터베이스 사용자 이름
- `spring.datasource.password`: 데이터베이스 비밀번호
- `spring.jpa.properties.hibernate.format_sql`: SQL 쿼리 포맷팅 (저는 한 줄로 깔끔히 보려고 추가하지 않았습니다. 기본설정 false)





## 3. 데이터베이스 관리 도구 설치하기

#### Intellij Database Navigator 플러그인 설치 💡

DB 를 쉽게 연결하고, 조회, 생성, 수정, 삭제 등 직관적으로 쉽게 조작하기 위한 도구를 설치합니다.

- Intellij > Shift 2번 > Plugins 검색 > Database Navigator 설치
  - Hostname: localhost
  - Port: 3306
  - Username: dev
  - Password: dev123



#### MySQL Workbench 설치 (옵션1)

MySQL Workbench는 MySQL 데이터베이스를 시각적으로 관리할 수 있는 도구입니다.

1. [MySQL Workbench 다운로드 페이지](https://dev.mysql.com/downloads/workbench/)에서 운영체제에 맞는 버전을 다운로드합니다.
2. 설치 파일을 실행하여 설치를 완료합니다.
3. MySQL Workbench를 실행하고 새로운 연결을 생성합니다:



#### DBeaver 설치 (옵션1)

DBeaver는 다양한 데이터베이스를 지원하는 범용 데이터베이스 관리 도구입니다.

1. [DBeaver 다운로드 페이지](https://dbeaver.io/download/)에서 운영체제에 맞는 버전을 다운로드합니다.
2. 설치 파일을 실행하여 설치를 완료합니다.
3. DBeaver를 실행하고 새로운 데이터베이스 연결을 생성합니다:



이 외에도 여러 좋은 도구가 있습니다. 

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



