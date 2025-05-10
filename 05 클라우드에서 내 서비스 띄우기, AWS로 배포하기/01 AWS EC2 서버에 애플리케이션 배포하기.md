# AWS EC2 서버에 애플리케이션 배포하기



## AWS 계정 생성 및 IAM 사용자 설정

AWS 클라우드 서비스를 사용하기 위해서는 먼저 AWS 계정이 필요합니다. AWS 계정을 생성하고 보안을 위해 IAM 사용자를 설정하는 방법을 알아보겠습니다.

#### **AWS 계정 생성하기**

- AWS 웹사이트(https://aws.amazon.com) 접속
- 계정 생성 버튼 클릭
- 이메일, 비밀번호, 계정 이름 입력
- 신용카드 정보 및 개인정보 입력
- 전화번호 인증 완료



루트 사용자는 AWS 계정 생성시 같이 만들어집니다.

- 이 계정은 삭제도 어렵고 최고 권한을 가지고 있으므로 MFA(다중 인증) 꼭 해두시길 권장합니다.

IAM 계정을 만들고 평소에는 루트사용자 대신 IAM 계정으로 접속하는게 보안에 좋습니다. (AWS 공식 제안사항)



#### IAM 사용자 생성하기

- AWS 콘솔에서 IAM 서비스 접속
- 사용자 추가 버튼 클릭
- 사용자 이름 및 액세스 유형 선택
- 권한 설정 (AdminAccessor 권한 부여)
- 사용자 생성 완료
- 액세스 키와 비밀 액세스 키 저장



## EC2 인스턴스 생성 및 설정

EC2는 AWS의 가상 서버입니다. 우리의 애플리케이션을 호스팅할 EC2 인스턴스를 생성하고 설정하는 방법을 알아보겠습니다.

1. EC2 인스턴스 생성하기
   - AWS 콘솔에서 EC2 서비스 접속
   - 인스턴스 시작 버튼 클릭
   - Amazon Linux 2023 AMI 선택
   - t2.micro 인스턴스 타입 선택 (프리티어)
   - 키 페어 생성 및 다운로드
   - 네트워크 설정 구성
   - 스토리지 설정 (기본값 8GB)
   - 인스턴스 시작

2. 보안 그룹 설정
   - 인바운드 규칙 추가
   - SSH (포트 22) 접속 허용
   - HTTP (포트 80) 접속 허용
   - HTTPS (포트 443) 접속 허용
   - 애플리케이션 포트 (8080) 접속 허용



## 서버 환경 설정

EC2 인스턴스에 접속하여 필요한 소프트웨어를 설치하고 환경을 설정합니다.

**SSH 접속**

```bash
ssh -i your-key.pem ec2-user@your-instance-public-dns
```

앞으로 간단하게 접속하려면 아래 설정을 추가하면 좋습니다.

```bash
# ssh 관리하는 폴더로 들어감 
cd ~/.ssh

# pem 파일을 이동시키고
mv ~/Downloads/test-keypair.pem .

# 파일소유자만 읽을 수 있도록 권한을 변경합니다.
chmod 400 test-keypair.pem

# ~/.ssh/config 파일 편집
vi config

# vi 편집기 안에서
Host test
  HostName 13.112.173.59
  User ec2-user
  IdentityFile ~/.ssh/aws/test-keypair.pem
  
# 이제 :wq! 를 타이핑해서 저장하고 vi 에디터를 종료합니다.
```

```bash
ssh test
```





**시스템 업데이트**

```bash
sudo dnf update -y
```

**Docker 설치**

```bash
sudo dnf install -y docker

# Docker 서비스 시작
sudo systemctl start docker

# Docker 서비스 부팅 시 자동 시작 설정
sudo systemctl enable docker
```



> ####  `dnf`란?
>
> `dnf`는 **Dandified YUM**의 줄임말로,
>  Red Hat 계열 리눅스(RHEL, CentOS, Fedora, Amazon Linux 등)에서 사용하는 **패키지 관리자**입니다.



편의를 돕는 부가설정 

```bash
# 현재 사용자(예: ec2-user)를 docker 그룹에 추가 (sudo 없이 사용 가능하게)
sudo usermod -aG docker $USER

# 반영을 위해 로그아웃 후 다시 로그인하거나 아래 명령 실행
newgrp docker

# 설치 확인위해 버전 조회
docker version
```







## 애플리케이션 배포

Docker를 사용하여 애플리케이션을 배포하고 실행합니다.



**내 PC**

linux 용 도커이미지 만들기 (EC2 서버는 리눅스)

```bash
docker buildx build --platform linux/amd64 -t apiece/springboot-twitter-linux:latest .
```

Docker hub 에 푸시하기 (먼저 docker login 필요)

```bash
docker push apiece/springboot-twitter-linux:latest
```



**EC2**

```bash
ssh test
```

```bash
vi docker-compose.yml

services:
  springboot-twitter:
    container_name: springboot-twitter
    image: apiece/springboot-twitter-linux:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-twitter:3306/twitterdb
      - SPRING_DATASOURCE_USERNAME=dev
      - SPRING_DATASOURCE_PASSWORD=dev123
    depends_on:
      - mysql-twitter
    command: sh -c "echo ' Waiting for MySQL...' && sleep 10 && exec java -jar app.jar"
    networks:
      - twitter-network

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



스프링부트 및 MySQL 실행

```bash
docker-compose up -d
```

- `-d`:  (detach 모드) 백그라운드에서 실행합니다.



## 서비스 모니터링

배포된 애플리케이션의 상태를 모니터링하고 관리하는 방법을 알아봅니다.

#### 컨테이너 상태 확인

```bash
docker ps -a
```



#### 로그 확인

```bash
docker logs -f 123a
```

- `-f` 는 실시간 로그 출력입니다.
- 123a 대신 container id 나 container 이름을 적어주시면 됩니다.



#### API 확인

이제 외부에서 인터넷을 통해 저희가 만든 api 를 사용해봅시다.

```bash
# 게시글 생성 예시
curl --location 'http://ec2-13-211-173-59.ap-southeast-2.compute.amazonaws.com:8080/api/posts' \
--header 'Content-Type: application/json' \
--data '{
    "content": "게시글 입니다."
}'

# 게시글 조회 예시
curl --location 'http://ec2-13-211-173-59.ap-southeast-2.compute.amazonaws.com:8080/api/posts/1'
```

- Postman 을 사용하셔도 됩니다.





#### **리소스 사용량 모니터링**

```bash
docker stats
```

| 속성 이름             | 뜻                                                           |
| --------------------- | ------------------------------------------------------------ |
| **CONTAINER ID**      | 컨테이너 고유 ID (짧게 표시됨)                               |
| **NAME**              | 컨테이너 이름 (`docker-compose.yml`에서 설정한 `container_name`) |
| **CPU %**             | 해당 컨테이너가 사용 중인 **CPU 사용량 비율**                |
| **MEM USAGE / LIMIT** | 현재 사용 중인 **메모리 용량 / 전체 사용 가능한 메모리 제한** |
| **MEM %**             | 메모리 사용률 (%) = 현재 사용량 ÷ 제한 × 100                 |
| **NET I/O**           | 네트워크 입출력 (송신/수신) 총량                             |
| **BLOCK I/O**         | 디스크 입출력 (읽기/쓰기) 총량                               |
| **PIDS**              | 컨테이너 안에서 실행 중인 프로세스 개수 (Process IDs)        |





# 과금 방지에 대한 팁

## AWS 프리티어 (free tier)

- 계정 생성시 프리티어 바로 적용
- 프리티어는 만료되면 바로 과금됩니다.
- 프리티어가 모두 무료는 아닙니다.
- 실습 끝나면 필요없는 리소스는 꺼두는 습관을 가지면 좋습니다.





프리티어는 무료 평가판, 12개월 무료, 언제나 무료 상품을 제공합니다.

### 12개월 무료

- EC2 t2.micro(1 vCPU, 1GB MEM) 또는 t3.micro(2 vCPU, 1GB MEM)
- S3 5GB
- RDS db.t2.micro, db.t3.micro, db.t4g.micro
- 12개월 지나면 과금
- 인스턴스 숫자 * 사용시간 >= 월 750시간 이면 과금

### 언제나 무료

- Lambda 월 1,000,000건
- DynamoDB 25GB
- CloudFront 월 1TB 전송, HTTP/HTTPS 1천만 건

### 무료 평가판

- Lightsail 등



### 무료티어 트래킹 / 알람

- AWS Budget
- AWS User notifications
- 과금을 방지하기 위해 이메일로 알람을 받는 것이 좋습니다.
- Billing and Cost Management 에서 현재 과금 현황을 확인할 수 있습니다.



### 그 외 팁

- EC2 중지시켜두어도 EBS 요금이 청구됩니다.
- EBS 스냅샷, 백업에도 요금이 청구됩니다.
- 다른 리전에 리소스가 있는지도 확인해야 합니다. (E2 글로벌 보기)
- 2024년 2월부터 public IP 주소 과금
  - EC2 인스턴스에 연결된 공인 IPv4 주소는 무료 (월 750시간)
  - Elastic IP 주소나 다른 서비스의 공인 IPv4 주소는 프리 티어 혜택에 포함되지 않습니다.
