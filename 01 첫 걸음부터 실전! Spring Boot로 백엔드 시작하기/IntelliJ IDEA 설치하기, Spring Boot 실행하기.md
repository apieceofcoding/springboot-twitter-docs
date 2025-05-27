

# IntelliJ IDEA 설치하기, Spring Boot 실행하기

Spring Boot 개발을 위해서는 통합 개발 환경(IDE)이 필요합니다. 이 문서에서는 JetBrains사의 IntelliJ IDEA를 설치하는 방법을 안내합니다.



## 1. IntelliJ IDEA 버전 선택

IntelliJ IDEA는 두 가지 버전이 있습니다:

- **Community Edition**: 무료 버전
- **Ultimate Edition**: 유료 버전 (30일 평가판 제공)

Spring Boot 개발을 위해서는 Ultimate Edition을 권장합니다. Ultimate Edition은 다음과 같은 장점이 있습니다:

- Spring Boot 전용 도구 지원
- 데이터베이스 도구 내장
- 웹 개발 도구 지원
- 프레임워크 지원 강화



## 2. 시스템 요구사항

- **운영체제**: Windows, macOS, Linux
- **메모리**: 최소 8GB RAM (16GB 이상 권장)
- **저장공간**: 최소 2.5GB



## 3. 설치 과정

### Windows/macOS 설치

1. [JetBrains 공식 웹사이트](https://www.jetbrains.com/idea/download/)에 접속
2. 운영체제에 맞는 버전 다운로드
3. 다운로드된 설치 파일 실행
4. 설치 마법사의 안내에 따라 진행
   - 원하는 설치 경로 선택
   - 시작 메뉴 폴더 선택
   - 추가 작업 선택 (파일 연결 등)



## 4. 프로젝트 생성

### 무료로 내 첫 스프링부트 프로젝트 만들어보기

1. 개발 환경 설정하기 (IntelliJ CE)
2. Spring Initializr 사용하기
3. Spring Boot 실행하기



### Spring Initializer 사용하기

스프링 프로젝트 환경을 만들기 위해 Spring Initializr 에 들어갑니다.
- 스프링 이니셜라이저: [https://start.spring.io/](https://start.spring.io/)



### 주요 기능

- 프로젝트 타입 선택: Maven 또는 **Gradle**
- 언어 선택: **Java**, Kotlin, Groovy
- Spring Boot 버전 선택 3.5.0
- 프로젝트 메타데이터 입력: Group, Artifact, Name, Description 등: com.apiece.springboot-twiiter
- 의존성(Dependencies) 추가: **Spring Web**, JPA, Lombok 등 다양한 스프링 관련 라이브러리를 선택 가능
- 패키징 선택: **jar** / war
- 자바 버전 선택: **21**




### 트위터 애플리케이션 설정

#### 기본 설정

| 항목 | 추천 설정 | 설명 |
|------|------------|------|
| **Project** | Gradle - Kotlin | 가독성이 좋고 설정이 간편함 (Maven도 가능) |
| **Language** | Java | 가장 널리 쓰이고 자료도 많음 |
| **Spring Boot** | 최신 안정 버전 | 최신 기능과 보안 업데이트 반영(M, SNAPSHOT X) |
| **Group** | `com.apiece` | 패키지 네이밍 규칙 (자유롭게 설정 가능) |
| **Artifact** | `twitter` | 프로젝트 이름 (예: sns, myapp 등) |
| **Name** | `twitter` | 앱 이름 |
| **Description** | `SNS service` | 간단한 설명 |
| **Package name** | `com.apiece.twitter` | 루트 패키지명 |
| **Packaging** | Jar | 대부분의 경우 jar로 충분함 |
| **Java Version** | 21 | 장기 지원 버전(LTS) |





### 필수 의존성 (Dependencies)

| 의존성 | 설명 |
|--------|------|
| **Spring Web** | REST API 개발을 위한 기본 웹 기능 |
| **Lombok** | 코드 간결화를 위한 어노테이션 지원 (Getter/Setter 등) |






> ### 💡 자바 버전은 어떻게 선택하면 좋을까요?
>
> Java 버전 선택 시 장기적으로 지원하는 안정적인 버전인지 확인해볼 필요가 있습니다. 도중에 자바 언어 자체에서 오류가 났을 경우 해당 부분의 버그 수정에 대한 부분을 지원받을 수 있기 때문입니다.
> 1. LTS(Long Term Support) 버전
>    - Java 8 (2014년 3월 ~ 2030년 12월)
>    - Java 11 (2018년 9월 ~ 2032년 1월)
>    - Java 17 (2021년 9월 ~ 2029년 9월)
>    - Java 21 (2023년 9월 ~ 2031년 9월)
>    - Java 25 (2025년 9월 ~ 2033년 9월)
>    - ...
> 2. 현재 프로젝트에서는 Java 21을 선택
>    - 최신 LTS 버전으로 장기 지원 보장
>    - 향상된 성능과 새로운 기능들 포함
>    - 2023년 9월 출시된 안정적인 버전
>
> Java 지원 로드맵 참고: https://www.oracle.com/kr/java/technologies/java-se-support-roadmap.html





### Spring Boot 애플리케이션 실행해보기

#### 1. IntelliJ IDEA 실행 및 프로젝트 열기

1. IntelliJ IDEA를 실행합니다.
2. 시작 화면에서 `Open` 버튼을 클릭하거나, 이미 다른 프로젝트가 열려 있다면 `File > Open` 메뉴를 클릭합니다.
3. Spring Initializr에서 받은 zip 파일을 압축 해제한 폴더를 선택하고 `OK`를 클릭합니다.
4. IntelliJ가 프로젝트를 로딩하고 Gradle 또는 Maven 기반의 프로젝트라면 자동으로 빌드와 의존성 다운로드가 시작됩니다. (이때, 하단에 "Build", "Indexing", "Downloading Dependencies" 같은 메시지가 나올 수 있습니다.)



#### 2. 애플리케이션 실행

1. `TwitterApplication.java` 파일을 더블클릭해서 엽니다.

2. 클래스 안에 다음과 같은 코드가 있을 거예요:

   ```java
   @SpringBootApplication
   public class TwitterApplication {
       public static void main(String[] args) {
           SpringApplication.run(DemoApplication.class, args);
       }
   }
   ```

3. 이 클래스 안 어디든 상관없이 마우스 오른쪽 버튼을 클릭한 다음 **"Run 'TwitterApplication.main()'"** 을 클릭합니다.

   - 상단 툴바 또는 main 코드 옆에 있는 녹색 ▶️ 실행 버튼을 클릭해도 됩니다.



### 예쁘게 실행 결과 보는 팁

- **IntelliJ CE에서 VM 옵션으로 ANSI 컬러 활성화하기**

#### 설정 방법

1. IntelliJ 우측 상단 실행 드롭다운 옆에 있는
    `Edit Configurations…` 클릭
2. 좌측에서 실행할 항목 선택 (예: `TwitterApplication`)
3. 오른쪽에서 VM options 항목에 아래 옵션을 추가:

```bash
-Dspring.output.ansi.enabled=ALWAYS
```
4. Apply > OK 누르고 실행하면, 콘솔 로그에 ANSI 컬러가 적용됩니다.







## 5. 유용한 단축키

| 기능 | Windows/Linux | macOS |
|------|--------------|-------|
| 빠른 검색 | Shift 2번 | Shift 2번 |
| 실행 | Shift + F10 | ⌃ R |
| 종료 | Ctrl + F2 | ⌘ F2 |
| 디버그 | Shift + F9 | ⌃ D |
| 파일 검색 | Ctrl + Shift + N | ⌘ ⇧ O |
| 전체 검색 (문자열 검색) | Ctrl + Shift + F | ⌘ ⇧ F |





## 6. 문제 해결

### 자주 발생하는 문제

1. **JDK를 찾을 수 없음**
   - File → Project Structure에서 JDK 경로 설정

2. **Gradle 빌드 실패**
   - Settings → Build, Execution, Deployment → Build Tools → Gradle에서 Gradle JVM 설정 확인

3. **Spring Boot 실행 실패**
   - Run/Debug Configuration 설정 확인
   - 포트 충돌 여부 확인
4. **윈도우에서 실행되지 않고 멈추는 경우**
   - Win버튼 + S → Windows 보안 → 바이러스 및 위협 방지 → 바이러스 및 위협 방지 설정 관리 → 실시간 보호 끄기





## 7. 추가 설정 및 팁

- **메모리 설정**: Help → Edit Custom VM Options에서 메모리 할당 조정
  - 또는 shift 2번 → Change Memory Settings → 2048 에서 **4096** 으로 변경




## 8. 유용한 자료

- [IntelliJ IDEA 공식 문서](https://www.jetbrains.com/help/idea/getting-started.html)
- [Spring Boot 공식 가이드](https://spring.io/guides)
- [IntelliJ IDEA 팁과 트릭](https://www.jetbrains.com/idea/guide/)





