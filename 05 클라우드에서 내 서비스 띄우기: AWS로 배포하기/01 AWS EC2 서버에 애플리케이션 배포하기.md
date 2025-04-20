# AWS EC2 서버에 애플리케이션 배포하기



## AWS 계정 생성 및 IAM 사용자 설정

AWS 클라우드 서비스를 사용하기 위해서는 먼저 AWS 계정이 필요합니다. AWS 계정을 생성하고 보안을 위해 IAM 사용자를 설정하는 방법을 알아보겠습니다.

1. AWS 계정 생성하기
   - AWS 웹사이트(https://aws.amazon.com) 접속
   - 계정 생성 버튼 클릭
   - 이메일, 비밀번호, 계정 이름 입력
   - 신용카드 정보 및 개인정보 입력
   - 전화번호 인증 완료

2. IAM 사용자 생성하기
   - AWS 콘솔에서 IAM 서비스 접속
   - 사용자 추가 버튼 클릭
   - 사용자 이름 및 액세스 유형 선택
   - 권한 설정 (EC2FullAccess 권한 부여)
   - 태그 추가 (선택사항)
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

SSH 접속
```bash
ssh -i your-key.pem ec2-user@your-instance-public-dns
```

시스템 업데이트
```bash
sudo yum update -y
```

Java 설치
```bash
sudo yum install java-17-amazon-corretto -y
```

Docker 설치
```bash
sudo yum install docker -y
sudo service docker start
sudo usermod -a -G docker ec2-user
```



## 애플리케이션 배포

Docker를 사용하여 애플리케이션을 배포하고 실행합니다.

Docker 이미지 가져오기
```bash
docker pull your-docker-image:latest
```

컨테이너 실행
```bash
docker run -d -p 8080:8080 --name your-app your-docker-image:latest
```

컨테이너 상태 확인
```bash
docker ps
```



## 서비스 모니터링

배포된 애플리케이션의 상태를 모니터링하고 관리하는 방법을 알아봅니다.

로그 확인
```bash
docker logs your-app
```

리소스 사용량 모니터링
```bash
docker stats
```

컨테이너 관리
```bash
# 컨테이너 중지
docker stop your-app

# 컨테이너 시작
docker start your-app

# 컨테이너 재시작
docker restart your-app
```



## 문제 해결

배포 과정에서 발생할 수 있는 일반적인 문제들과 해결 방법을 알아봅니다.

1. 연결 문제 해결
   - 보안 그룹 설정 확인
   - 네트워크 ACL 확인
   - 인스턴스 상태 확인

2. 애플리케이션 문제 해결
   - 로그 분석
   - 메모리 사용량 확인
   - 디스크 공간 확인

3. 성능 최적화
   - 인스턴스 타입 조정
   - 자동 스케일링 설정
   - 로드 밸런서 구성
