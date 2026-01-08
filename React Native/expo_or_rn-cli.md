# Expo vs React Native CLI 정리

React Native 앱을 시작할 때 보통 두 가지 선택지가 존재.

-   **Expo (Expo SDK + Expo CLI)**
-   **React Native CLI (공식 RN CLI로 iOS/Android 네이티브 프로젝트 포함)**

둘 다 “React Native”로 앱을 만들지만, **개발/빌드 방식과 네이티브 접근 방식**이 달라서 상황에 따라 선택이 달라진다.

---

## 1) 개념 한 번에 이해하기

### Expo란?

React Native 앱 개발을 쉽게 시작할 수 있게 해주는 **플랫폼/툴체인**

-   기본 세팅(네이티브 설정, 빌드 설정 등)을 대부분 Expo가 대신 해줌
-   카메라/푸시/위치 등 자주 쓰는 기능은 **Expo SDK**로 바로 사용 가능
-   iOS/Android 네이티브 폴더 없이도 시작 가능(Managed Workflow)

> “RN 개발을 편하게 해주는 올인원 패키지”

---

### React Native CLI란?

React Native 공식 CLI로 앱을 생성하면 **iOS/Android 네이티브 프로젝트가 함께 생성**

-   Xcode/Android Studio 기반 네이티브 설정을 직접 만질 수 있음
-   네이티브 라이브러리 연결/수정이 자유로움

> “네이티브까지 전부 직접 관리하는 정통 방식”

---

## 2) Expo vs RN CLI 핵심 차이

| 구분               | Expo                              | React Native CLI                 |
| ------------------ | --------------------------------- | -------------------------------- |
| 시작 난이도        | 매우 쉬움                         | 상대적으로 어려움                |
| 네이티브 설정      | 대부분 숨겨져 있음(자동)          | 직접 설정/관리                   |
| 개발 속도          | 빠름                              | 비교적 느림(세팅 시간)           |
| 네이티브 기능 확장 | 제한적일 수 있음(단, 많이 개선됨) | 거의 무제한                      |
| 빌드/배포          | Expo 빌드(EAS)로 간단             | Xcode/Gradle 직접 관리           |
| 팀/프로젝트 규모   | 소~중규모에 매우 좋음             | 중~대규모/복잡한 네이티브에 좋음 |

---

## 3) 장단점 정리

### Expo 장점

-   초기 세팅이 거의 필요 없음 (바로 개발 가능)
-   개발 경험이 편함 (디버깅/테스트/빌드 툴 잘 되어 있음)
-   Expo SDK로 카메라, 위치, 알림 등 구현이 쉬움
-   EAS(Build/Submit)로 배포 파이프라인도 비교적 단순

### Expo 단점

-   “원하는 네이티브 기능”이 Expo SDK에 없으면 난감할 수 있음  
    (다만 **Prebuild / Dev Client / Config Plugin** 등으로 많이 해결 가능)
-   특정 네이티브 커스터마이징이 많은 프로젝트는 불리

---

### React Native CLI 장점

-   iOS/Android 네이티브 코드를 직접 수정 가능 (자유도 최고)
-   네이티브 SDK/라이브러리(지도, 결제, 보안, BLE 등) 연결이 강함
-   복잡한 빌드 설정/환경 분리(Flavor, Scheme 등)에서 유리

### React Native CLI 단점

-   개발 시작부터 환경 세팅이 빡셈
    -   iOS: Xcode, CocoaPods
    -   Android: JDK, Android Studio, Gradle, SDK 등
-   라이브러리 연결/빌드 에러 등으로 헤맬 가능성 농후

---

## 4) 상황별 추천

### Expo 추천 상황

-   React Native를 **처음 시작**한다
-   MVP / 사이드 프로젝트 / 포트폴리오를 빠르게 만들고 싶다
-   카메라/위치/푸시 등 일반적인 기능 위주다
-   “네이티브 커스텀”이 거의 없다
-   배포/빌드 파이프라인을 간단히 가져가고 싶다

> 처음 RN 공부 + 대부분의 일반 앱 개발은 Expo로 시작하는 게 효율적

---

### React Native CLI 추천 상황

-   네이티브 SDK 연동이 확실히 필요하다
    -   특정 결제 SDK, BLE/IoT, 고급 지도, 보안 모듈, 사내 SDK 등
-   기존 네이티브 앱(iOS/Android)에 RN을 일부 붙이는 경우
-   빌드 커스터마이징이 많은 팀 프로젝트(Flavor/Scheme 등)
-   “Expo에서 막히는 요구사항”이 명확히 있다

> 네이티브 제어가 중요한 프로젝트라면 처음부터 RN CLI가 안정적

---

## 5) 용어 메모 (헷갈리기 쉬운 것들)

-   **Managed Workflow(Expo)**: 네이티브 폴더를 직접 관리하지 않는 방식
-   **Bare Workflow(Expo/RN CLI)**: 네이티브 폴더(iOS/Android)를 직접 관리
-   **EAS**: Expo Application Services (빌드/배포/업데이트 도구 모음)
-   **Prebuild(Expo)**: Expo 프로젝트에서 iOS/Android 네이티브 폴더를 생성하는 과정
-   **Dev Client(Expo)**: Expo Go 대신 “내 네이티브 모듈 포함”한 개발용 앱을 만들어 테스트하는 방식
