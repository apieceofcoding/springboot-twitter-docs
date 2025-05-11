# 03 Docker MySQL을 RDS로 이전하기



### 1. 애플리케이션과 데이터베이스 관리

EC2 에서 애플리케이션과 데이터베이스를 동시에 관리할 때

EC2 에 장애가 발생하면 둘 다 문제가 발생할 수 있습니다. 

또한 한정된 EC2 메모리, CPU, Disk 자원을 같이 쓰는 것도 문제가 됩니다.

특히 데이터베이스에 있는 무슨일이 있어도 데이터는 복구가 될 수 있도록 더 특별히 관리를 해주어야 합니다.

이를 위해 EC2 에 있는 애플리케이션과 데이터베이스를 분리해보겠습니다.



AWS(Amazon Web Services) 는 **AWS RDS**(Amazon Relational Database Service)라는 **관리형 관계형 데이터베이스 서비스를 제공**합니다. 쉽게 말하면, AWS가 **데이터베이스 설치, 유지보수, 백업, 복구, 보안 패치, 스케일링** 같은 귀찮은 작업들을 대신 해주는 서비스라고 보면 됩니다.





기존 docker-compose로 관리하던 MySQL 컨테이너를 AWS RDS(MySQL)로 이전하고, 

Spring Boot 서비스가 이를 사용하도록 설정해보도록 하겠습니다.



## 2. AWS RDS(MySQL) 인스턴스 생성

1. AWS 콘솔에서 RDS 서비스로 이동합니다.
2. "데이터베이스 생성" 클릭
3. 엔진 유형: **MySQL** 선택
4. DB 인스턴스 식별자, 마스터 사용자 이름/비밀번호 입력
5. **퍼블릭 액세스: 허용하지 않음(비공개, Private)** 으로 설정
6. EC2 인스턴스와 같은 VPC, 가급적 같은 서브넷(또는 라우팅 가능한 서브넷)에 RDS를 생성
7. 보안그룹은 EC2 인스턴스가 속한 보안그룹에서만 3306 포트 인바운드를 허용
8. 나머지 옵션은 기본값 또는 필요에 따라 설정 후 생성

> **TIP:** 퍼블릭 액세스를 허용하지 않으면 RDS에 공인 IP가 할당되지 않아 외부에서는 직접 접근할 수 없습니다. EC2 인스턴스와 같은 VPC/서브넷에 있으면 EC2에서만 접근 가능합니다.





## 3. RDS 엔드포인트 및 접속 정보 확인

- RDS 인스턴스 생성 후, "엔드포인트"와 "포트" 정보를 확인합니다.
- 엔드포인트는 프라이빗 DNS 주소입니다(공인 IP 없음).
- 예시:
  - 엔드포인트: `mydb.xxxxxxxx.ap-northeast-2.rds.amazonaws.com`
  - 포트: `3306`
  - 사용자명/비밀번호: 생성 시 입력한 값

> **참고:** EC2 인스턴스에서만 이 엔드포인트로 접속이 가능합니다. 로컬 PC에서는 직접 접속할 수 없습니다.





## 4. EC2에서 RDS로 접속 테스트

**mysql client 설치 (mysql 연결도구)**

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf update -y
sudo dnf install -y mysql-community-client

mysql --version
mysql  Ver 8.0.42 for Linux on x86_64 (MySQL Community Server - GPL)
```



**RDS 접속**

EC2 인스턴스에 접속(ssh) 후, 아래와 같이 mysql 클라이언트로 RDS에 접속이 되는지 확인합니다.

```bash
$ mysql -h mysql-twitter.c10a6gu2qvpz.ap-southeast-2.rds.amazonaws.com -u admin -p
Enter password:
```

- RDS 생성할 때 적었던 패스워드를 입력합니다.
- 만약 접속이 안 된다면, VPC/서브넷/보안그룹 설정을 다시 확인해야 합니다.



**데이터베이스 생성**

```bash
CREATE DATABASE twitterdb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

개발용 계정 생성

```bash
CREATE USER 'dev'@'%' IDENTIFIED BY 'dev123';
GRANT ALL PRIVILEGES ON twitterdb.* TO 'dev'@'%';
FLUSH PRIVILEGES;
```





## 5. Docker compose 설정 변경

**EC2 접속**

```bash
vi docker-compose.yml
```

```yaml
services:
  springboot-twitter:
    container_name: springboot-twitter
    image: apiece/springboot-twitter-linux:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-twitter.c10a6gu2qvpz.ap-southeast-2.rds.amazonaws.com:3306/twitterdb
      - SPRING_DATASOURCE_USERNAME=dev
      - SPRING_DATASOURCE_PASSWORD=dev123
```

- url 에 작성할 RDS 엔드포인트는 프라이빗 DNS 주소입니다.



**docker-compose 실행**

```bash
docker-compose up -d
```

```bash
# docker 컨테이너 상태 확인
docker ps

# docker 컨테이너 로그 확인
docker logs -f springboot-twitter
```



이제 API 를 테스트 해보면서 잘 작동이되는지 확인해보세요.

우리는 처음으로 클라우드에 내 애플리케이션을 배포하여 테스트까지 완료했습니다. 앞으로는 트위터 애플리케이션에서 아직 개발되지 않은 다양한 요구사항을 구현하고 배포해보도록 하겠습니다.
