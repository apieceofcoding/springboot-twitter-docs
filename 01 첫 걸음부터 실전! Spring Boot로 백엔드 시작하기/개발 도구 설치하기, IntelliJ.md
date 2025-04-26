

# IntelliJ IDEA 설치하기

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

1. File → New → Project 선택
2. Spring Initializr 선택 (IntelliJ Ultimate)
   - 강의자료: [무료로 내 첫 스프링부트 프로젝트 만들어보기](무료로 내 첫 스프링부트 프로젝트 만들어보기.md) (Intellij Community Edition)
3. 프로젝트 설정
   - Java 버전 선택
   - Spring Boot 버전 선택
   - 프로젝트 메타데이터 입력
   - 의존성 선택



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





