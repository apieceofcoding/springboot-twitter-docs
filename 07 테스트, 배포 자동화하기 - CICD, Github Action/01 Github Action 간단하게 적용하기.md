# GitHub Actions 간단하게 적용하기: CI/CD 파이프라인 구축

GitHub Actions는 GitHub에서 제공하는 CI/CD(지속적 통합/지속적 배포) 도구로, 코드 변경 사항을 자동으로 테스트하고 배포할 수 있게 해줍니다. 이 문서에서는 Spring Boot 애플리케이션에 GitHub Actions를 적용하는 방법을 알아보겠습니다.



## 1. GitHub Actions란?

GitHub Actions는 GitHub 저장소에서 직접 CI/CD 워크플로우를 구축할 수 있게 해주는 도구입니다. 코드를 GitHub에 푸시할 때마다 자동으로 테스트를 실행하고, 필요에 따라 배포까지 진행할 수 있습니다.



### GitHub Actions의 주요 개념

- **워크플로우(Workflow)**: 자동화된 프로세스를 정의하는 YAML 파일입니다. `.github/workflows` 디렉토리에 저장됩니다.
- **작업(Job)**: 워크플로우 내에서 실행되는 작업 단위입니다. 여러 단계(Step)로 구성됩니다.
- **단계(Step)**: 작업 내에서 실행되는 개별 명령어입니다.
- **액션(Action)**: 재사용 가능한 워크플로우 단위입니다. GitHub 마켓플레이스에서 다양한 액션을 찾을 수 있습니다.
- **러너(Runner)**: 워크플로우를 실행하는 서버입니다. GitHub에서 호스팅하는 러너를 사용하거나 자체 러너를 설정할 수 있습니다.



## 2. GitHub Actions 워크플로우 설정하기



### 워크플로우 파일 생성하기

GitHub 저장소에 `.github/workflows` 디렉토리를 생성하고 그 안에 워크플로우 파일을 만듭니다. 예를 들어, `spring-boot-ci.yml` 파일을 생성합니다.

```yaml
name: Spring Boot CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 21
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        
    - name: Build with Gradle
      run: ./gradlew build
      
    - name: Run tests
      run: ./gradlew test
```

이 워크플로우는 다음과 같은 작업을 수행합니다:

1. `main` 브랜치에 푸시하거나 풀 리퀘스트를 생성할 때 실행됩니다.
2. Ubuntu 최신 버전에서 실행됩니다.
3. 코드를 체크아웃합니다.
4. JDK 21을 설정합니다.
5. Gradle을 사용하여 프로젝트를 빌드합니다.
6. 테스트를 실행합니다.



### 워크플로우 파일 커스터마이징하기

위의 기본 워크플로우를 필요에 따라 커스터마이징할 수 있습니다. 예를 들어, 캐싱을 추가하여 빌드 속도를 향상시킬 수 있습니다:

```yaml
name: Spring Boot CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 21
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        
    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-
        
    - name: Build with Gradle
      run: ./gradlew build
      
    - name: Run tests
      run: ./gradlew test
```

이 워크플로우는 Gradle 패키지를 캐싱하여 빌드 속도를 향상시킵니다.



## 3. GitHub Actions로 배포 자동화하기



### GitHub Actions 시크릿 설정하기

GitHub Actions에서 민감한 정보(예: API 키, 비밀번호)를 사용할 때는 시크릿을 사용해야 합니다. 시크릿은 GitHub 저장소의 설정에서 설정할 수 있습니다.

**GitHub 시크릿 설정 방법**

1. GitHub 저장소로 이동합니다.
2. "Settings" 탭을 클릭합니다.
3. 왼쪽 사이드바에서 "Secrets and variables" > "Actions"를 클릭합니다.
4. "New repository secret" 버튼을 클릭합니다.
5. 시크릿 이름과 값을 입력하고 "Add secret" 버튼을 클릭합니다.



**워크플로우에서 시크릿 사용하기**

워크플로우 파일에서 시크릿을 사용하려면 `${{ secrets.SECRET_NAME }}` 형식을 사용합니다:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_HUB_USERNAME }}
    password: ${{ secrets.DOCKER_HUB_TOKEN }}
```







### GitHub Actions로 Docker 이미지 빌드 및 푸시하기

GitHub Actions를 사용하여 Docker 이미지를 빌드하고 Docker Hub에 푸시할 수 있습니다:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 21
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        
    - name: Build with Gradle
      run: ./gradlew build
      
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
      
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}
        
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_HUB_USERNAME }}/springboot-twitter-linux:latest
```

이 워크플로우는 다음과 같은 작업을 수행합니다:

1. 코드를 체크아웃합니다.
2. JDK 21을 설정합니다.
3. Gradle을 사용하여 프로젝트를 빌드합니다.
4. Docker Buildx를 설정합니다.
5. Docker Hub에 로그인합니다.
6. Docker 이미지를 빌드하고 Docker Hub에 푸시합니다.





## 4. GitHub Actions로 클라우드 서비스에 배포하기 (EC2 + Docker)

Docker Hub에 이미지를 올려두고, AWS EC2에서 이미지를 pull하여 배포 및 실행하는 자동화 방법을 소개합니다.

### 준비 사항
- EC2 인스턴스에 Docker, Docker-compose 가 설치되어 있어야 합니다.
- EC2 인스턴스에 접속할 수 있는 SSH 키가 있어야 합니다.
- EC2 인스턴스의 보안그룹에서 22(SSH), 8080(서비스 포트) 등이 열려 있어야 합니다.
- GitHub 저장소의 시크릿에 아래 정보를 등록해야 합니다:
  - `EC2_HOST`: EC2 퍼블릭 IP 또는 도메인
  - `EC2_USER`: EC2에 접속할 사용자명 (예: ec2-user, ubuntu 등)
  - `EC2_KEY`: EC2에 접속할 PEM 키 (private key, 여러 줄 입력 가능)

### 워크플로우 (docker-compose로 배포 단계 추가)

```yaml
name: Build, Push, and Deploy to EC2 (docker-compose)

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 21
        uses: actions/setup-java@v3
        with:
          java-version: '21'
          distribution: 'temurin'
      - name: Build with Gradle
        run: ./gradlew build
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/springboot-twitter-linux:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2 via SSH (docker-compose)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            # docker-compose.yml 파일 작성 (혹은 git pull 등으로 복사)
            cat > docker-compose.yml <<EOF
            version: '3'
            services:
              springboot-twitter:
                image: ${{ secrets.DOCKER_HUB_USERNAME }}/springboot-twitter-linux:latest
                container_name: springboot-twitter
                ports:
                  - "8080:8080"
                environment:
                  - SPRING_DATASOURCE_URL=jdbc:mysql://${{ secrets.RDS-ENDPOINT }}:3306/twitterdb
                  - SPRING_DATASOURCE_USERNAME=dev
                  - SPRING_DATASOURCE_PASSWORD=dev123
            EOF
            docker-compose pull
            docker-compose down || true
            docker-compose up -d
```

- SSH로 접속 후, docker-compose.yml을 작성(혹은 git pull 등으로 복사)하고, `docker-compose up -d`로 실행합니다.
- 기존 컨테이너는 `docker-compose down`으로 정리 후 재실행합니다.
- 환경변수에 RDS 엔드포인트와 DB 정보를 넣어줍니다.

이렇게 하면 코드 push → Docker Hub 빌드/업로드 → EC2에서 docker-compose로 자동 배포까지 한 번에 자동화할 수 있습니다.



