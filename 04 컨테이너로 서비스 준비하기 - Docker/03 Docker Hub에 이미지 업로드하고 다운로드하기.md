# Docker Hub에 이미지 업로드하고 다운로드하기



## 1. Docker Hub 소개

Docker Hub는 Docker 이미지를 공유하고 다운로드할 수 있는 공개 레지스트리입니다. GitHub와 유사하게 Docker Hub를 통해 이미지를 공유하고 협업할 수 있습니다.



### Docker Hub의 주요 기능

**이미지 저장소**: Docker 이미지를 저장하고 관리할 수 있습니다.

**자동 빌드**: GitHub 등의 저장소와 연동하여 코드 변경 시 자동으로 이미지를 빌드할 수 있습니다.

**팀 협업**: 팀원들과 이미지를 공유하고 협업할 수 있습니다.

**웹훅**: 이미지 푸시 등의 이벤트에 대한 알림을 설정할 수 있습니다.



## 2. Docker Hub 계정 생성 및 로그인

### 계정 생성

1. [Docker Hub 웹사이트](https://hub.docker.com/)에 접속합니다.
2. "Sign Up" 버튼을 클릭하여 계정을 생성합니다.
3. 필요한 정보를 입력하고 계정을 활성화합니다.



### Docker CLI에서 로그인

```bash
docker login
```

로그인 프롬프트가 나타나면 사용자 이름과 비밀번호를 입력합니다.



## 3. Docker 이미지 태깅

Docker Hub에 이미지를 업로드하기 전에 적절한 태그를 지정해야 합니다.

### 태그 형식

```
[사용자명]/[이미지명]:[태그]
apiece/springboot-twitter:latest
```



### 태그 지정 방법

```bash
# 기존 이미지에 새 태그 지정
docker tag springboot-twitter:latest apiece/springboot-twitter:latest

# 빌드 시 태그 지정
docker build -t apiece/springboot-twitter:latest .
```



## 4. Docker Hub에 이미지 업로드하기

### 이미지 푸시

```bash
docker push apiece/springboot-twitter:latest
```



### 푸시 과정 설명

1. Docker CLI는 Docker Hub API를 통해 인증을 확인합니다.
2. 이미지 레이어가 Docker Hub로 전송됩니다.
3. 업로드가 완료되면 Docker Hub 웹사이트에서 이미지를 확인할 수 있습니다.



### 푸시 시 주의사항

- 이미지 이름은 Docker Hub 계정 이름으로 시작해야 합니다.
- 대용량 이미지는 업로드에 시간이 걸릴 수 있습니다.



## 5. Docker Hub에서 이미지 다운로드하기

### 이미지 pull (가져오기)

```bash
docker pull apiece/springboot-twitter:latest
```



### Pull 과정 설명

1. Docker CLI는 Docker Hub API를 통해 이미지 메타데이터를 요청합니다.
2. 필요한 레이어만 다운로드합니다 (이미 일부 레이어가 로컬에 있는 경우).
3. 다운로드가 완료되면 로컬에서 이미지를 사용할 수 있습니다.



### Pull 시 주의사항

- 태그를 지정하지 않으면 `latest` 태그가 기본값으로 사용됩니다.
- 특정 버전의 이미지를 사용하려면 명시적으로 태그를 지정해야 합니다.





# 6. private(비공개) 저장소 사용하기

docker hub 는 현재 1개의 private repository 를 무료로 사용 가능합니다.

### private 저장소 생성

1. Docker Hub 웹사이트에서 "Create Repository" 버튼을 클릭합니다.
2. 저장소 이름을 입력하고 "Private" 옵션을 선택합니다.
3. "Create" 버튼을 클릭하여 저장소를 생성합니다.



### private 저장소 사용

```bash
# 비공개 저장소에 이미지 푸시
docker push apiece/private-twitter:latest

# 비공개 저장소에서 이미지 풀
docker pull apiece/private-twitter:latest
```



