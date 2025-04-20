# Docker란 무엇인가요?



## 1. Docker 소개

Docker는 애플리케이션을 개발, 배포, 실행하기 위한 오픈 플랫폼입니다. Docker를 사용하면 애플리케이션을 인프라에서 분리하여 소프트웨어를 빠르게 제공할 수 있습니다.



### 1.1 Docker의 핵심 개념

**컨테이너(Container)**: 애플리케이션과 그 실행에 필요한 모든 파일과 설정을 포함하는 독립적인 실행 환경입니다. 가상머신보다 가볍고 빠르게 시작할 수 있습니다.

**이미지(Image)**: 컨테이너를 생성하기 위한 읽기 전용 템플릿입니다. 애플리케이션 코드, 런타임, 라이브러리, 도구 등이 포함됩니다.

**Dockerfile**: 이미지를 빌드하기 위한 지침이 포함된 텍스트 파일입니다.

**Docker Hub**: Docker 이미지를 공유하고 다운로드할 수 있는 공개 레지스트리입니다.

### 1.2 Docker의 장점

**일관성**: "내 컴퓨터에서는 잘 되는데..."라는 문제를 해결합니다. 개발, 테스트, 운영 환경이 동일하게 유지됩니다.

**이식성**: 한 환경에서 다른 환경으로 쉽게 이동할 수 있습니다.

**격리**: 각 애플리케이션이 독립적인 환경에서 실행되어 충돌을 방지합니다.

**효율성**: 가상머신보다 적은 리소스를 사용하고 빠르게 시작/종료할 수 있습니다.

**확장성**: 마이크로서비스 아키텍처에 적합하며, 필요에 따라 쉽게 확장할 수 있습니다.



## 2. Docker vs 가상머신

### 가상머신(VM)의 구조

가상머신은 호스트 OS 위에 하이퍼바이저를 설치하고, 그 위에 게스트 OS를 실행합니다. 각 가상머신은 자체 OS를 가지고 있어 리소스를 많이 사용합니다.

### Docker의 구조

Docker는 호스트 OS의 커널을 공유하고, 애플리케이션과 필요한 라이브러리만 컨테이너에 포함시킵니다. OS를 포함하지 않기 때문에 가볍고 빠릅니다.

### 비교

| 특성 | 가상머신 | Docker |
|------|---------|--------|
| 크기 | 수 GB | 수 MB |
| 시작 시간 | 분 단위 | 초 단위 |
| 리소스 사용 | 많음 | 적음 |
| 격리 수준 | 높음 | 중간 |
| 이식성 | 중간 | 높음 |



## 3. Docker 설치하기

### Windows에서 설치하기

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop) 다운로드
2. 설치 프로그램 실행 및 지시에 따라 설치
3. WSL 2(Windows Subsystem for Linux)가 필요할 수 있음



### macOS에서 설치하기

1. [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop) 다운로드
2. 설치 프로그램 실행 및 지시에 따라 설치



## 4. Docker 기본 명령어

### Docker 버전 확인

```bash
docker --version
```

### Docker 실행 상태 확인

```bash
docker info
```

### 이미지 관련 명령어

```bash
# 이미지 목록 확인
docker images

# 이미지 다운로드
docker pull [이미지명]:[태그]

# 이미지 삭제
docker rmi [이미지명]:[태그]
```

### 컨테이너 관련 명령어

```bash
# 컨테이너 실행
docker run [옵션] [이미지명]:[태그]

# 실행 중인 컨테이너 목록 확인
docker ps

# 모든 컨테이너 목록 확인 (중지된 컨테이너 포함)
docker ps -a

# 컨테이너 중지
docker stop [컨테이너ID 또는 이름]

# 컨테이너 삭제
docker rm [컨테이너ID 또는 이름]

# 컨테이너 로그 확인
docker logs [컨테이너ID 또는 이름]
```



## 5. 간단한 Docker 컨테이너 실행해보기

### Hello World 실행

```bash
docker run hello-world
```

이 명령어는 처음 실행 시 hello-world 이미지를 다운로드하고, 간단한 메시지를 출력한 후 종료됩니다.



### Nginx 웹 서버 실행

```bash
# Nginx 이미지 다운로드 및 실행
docker run -d -p 80:80 --name my-nginx nginx

# 브라우저에서 http://localhost 접속하여 확인
```

이 명령어는:
- `-d`: 백그라운드에서 실행
- `-p 80:80`: 호스트의 80번 포트를 컨테이너의 80번 포트에 연결
- `--name my-nginx`: 컨테이너 이름 지정
- `nginx`: 사용할 이미지

