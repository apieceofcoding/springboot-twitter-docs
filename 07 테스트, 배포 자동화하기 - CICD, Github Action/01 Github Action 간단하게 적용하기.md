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
        tags: ${{ secrets.DOCKER_HUB_USERNAME }}/springboot-twitter:latest
```

이 워크플로우는 다음과 같은 작업을 수행합니다:

1. 코드를 체크아웃합니다.
2. JDK 21을 설정합니다.
3. Gradle을 사용하여 프로젝트를 빌드합니다.
4. Docker Buildx를 설정합니다.
5. Docker Hub에 로그인합니다.
6. Docker 이미지를 빌드하고 Docker Hub에 푸시합니다.



### GitHub Actions로 클라우드 서비스에 배포하기

GitHub Actions를 사용하여 클라우드 서비스(예: AWS, Azure, Google Cloud)에 애플리케이션을 배포할 수 있습니다. 다음은 AWS Elastic Beanstalk에 배포하는 예시입니다:

```yaml
name: Deploy to AWS Elastic Beanstalk

on:
  push:
    branches: [ main ]

jobs:
  deploy:
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
      
    - name: Generate deployment package
      run: zip -r deploy.zip . -x "*.git*"
      
    - name: Deploy to EB
      uses: einaregilsson/beanstalk-deploy@v21
      with:
        aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        application_name: springboot-twitter
        environment_name: production
        region: us-east-1
        deployment_package: deploy.zip
```

이 워크플로우는 다음과 같은 작업을 수행합니다:

1. 코드를 체크아웃합니다.
2. JDK 21을 설정합니다.
3. Gradle을 사용하여 프로젝트를 빌드합니다.
4. 배포 패키지를 생성합니다.
5. AWS Elastic Beanstalk에 배포합니다.



## 4. GitHub Actions 시크릿 설정하기

GitHub Actions에서 민감한 정보(예: API 키, 비밀번호)를 사용할 때는 시크릿을 사용해야 합니다. 시크릿은 GitHub 저장소의 설정에서 설정할 수 있습니다.

### GitHub 시크릿 설정 방법

1. GitHub 저장소로 이동합니다.
2. "Settings" 탭을 클릭합니다.
3. 왼쪽 사이드바에서 "Secrets and variables" > "Actions"를 클릭합니다.
4. "New repository secret" 버튼을 클릭합니다.
5. 시크릿 이름과 값을 입력하고 "Add secret" 버튼을 클릭합니다.



### 워크플로우에서 시크릿 사용하기

워크플로우 파일에서 시크릿을 사용하려면 `${{ secrets.SECRET_NAME }}` 형식을 사용합니다:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_HUB_USERNAME }}
    password: ${{ secrets.DOCKER_HUB_TOKEN }}
```



## 5. GitHub Actions 워크플로우 모니터링하기

GitHub Actions 워크플로우의 실행 상태는 GitHub 저장소의 "Actions" 탭에서 확인할 수 있습니다. 각 워크플로우 실행의 상세 정보를 확인하고, 로그를 확인할 수 있습니다.

### 워크플로우 실행 상태 확인하기

1. GitHub 저장소로 이동합니다.
2. "Actions" 탭을 클릭합니다.
3. 워크플로우 실행 목록에서 원하는 실행을 클릭합니다.
4. 작업 및 단계의 실행 상태를 확인합니다.

### 워크플로우 실행 로그 확인하기

1. 워크플로우 실행 상세 페이지에서 작업을 클릭합니다.
2. 단계를 클릭하여 로그를 확인합니다.





## 6. GitHub Actions 워크플로우 최적화하기

GitHub Actions 워크플로우를 최적화하여 실행 시간을 단축하고 리소스를 절약할 수 있습니다.

### 워크플로우 최적화 팁

1. **캐싱 사용**: Gradle 등의 패키지 매니저 캐싱을 사용하여 빌드 시간을 단축합니다.
2. **병렬 작업 실행**: 독립적인 작업은 병렬로 실행하여 전체 실행 시간을 단축합니다.
3. **조건부 실행**: 필요할 때만 작업을 실행하도록 조건을 설정합니다.
4. **최소한의 단계 사용**: 불필요한 단계를 제거하여 워크플로우를 간소화합니다.



### 최적화된 워크플로우 예시

```yaml
name: Optimized Spring Boot CI

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
        
    - name: Build and test with Gradle
      run: ./gradlew build
      
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: echo "Deploying to production..."
      # 실제 배포 명령어 추가
```

이 워크플로우는 다음과 같은 최적화를 적용합니다:

1. Gradle 패키지를 캐싱하여 빌드 시간을 단축합니다.
2. 빌드와 테스트를 하나의 단계로 통합하여 단계 수를 줄입니다.
3. 배포 작업은 `main` 브랜치에 푸시할 때만 실행되도록 조건을 설정합니다.
4. 배포 작업은 빌드 작업이 성공한 후에만 실행되도록 `needs` 키워드를 사용합니다.



GitHub Actions를 사용하면 Spring Boot 애플리케이션의 CI/CD 파이프라인을 쉽게 구축할 수 있습니다. 이렇게 하면 코드 변경 사항을 자동으로 테스트하고 배포하여 개발 생산성을 향상시키고 품질을 유지할 수 있습니다. 개발자가 배포 명령어나 스크립트를 일일이 실행시키지 않더라도, 자동으로 배포하는 시스템을 만들 수 있습니다.
