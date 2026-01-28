# React Native 상태 기반 복잡한 화면의 컴포넌트 분리 전략

-   등록 상태에 따라 UI가 크게 달라지는 화면 구현
-   조건부 렌더링이 많을 때 컴포넌트를 효과적으로 분리하는 방법
-   400줄 넘는 단일 컴포넌트를 8개의 작은 컴포넌트로 리팩토링
-   부모는 배치만, 자식이 조건 판단하는 구조로 가독성과 유지보수성 향상

---

## 두 가지 접근 방식

### **모든 조건을 부모에서 관리 (안티패턴)**

-   HomeScreen에서 모든 상태 조합을 체크
-   중복 코드가 많고 수정 시 여러 곳을 고쳐야 함
-   파일이 길어져 가독성 저하
-   버그 발생 시 원인 파악 어려움

```tsx
// 400줄 넘는 HomeScreen
{
    !hasDevice && !hasChildProfile && (
        <View>
            <ServiceGuideCard />
            <Button>기기 등록</Button>
            <Button>프로필 설정</Button>
        </View>
    );
}
{
    hasDevice && !hasChildProfile && (
        <View>
            <ServiceGuideCard />
            <DeviceStatusCard />
            <Button>프로필 설정</Button>
        </View>
    );
}
// ... 4가지 케이스 모두 중복 코드
```

<br/>

### **각 섹션을 독립 컴포넌트로 분리 (권장)**

-   조건부 로직을 각 컴포넌트 내부로 이동
-   HomeScreen은 상태 관리와 배치만 담당
-   각 컴포넌트가 자신의 표시 여부 결정
-   파일 크기가 작아져 관리 용이

```tsx
// 120줄로 줄어든 HomeScreen
<HomeHeader nickname={nickname} hasDevice={hasDevice} hasChildProfile={hasChildProfile} />
{!isFullyRegistered && <ServiceGuideCard />}
<RegistrationButtons deviceRegistered={hasDevice} profileRegistered={hasChildProfile} />
{hasDevice && <DeviceStatusCard status={deviceStatus} />}
{hasChildProfile && <ChildProfileCard profile={childProfile} />}
<DailyReportSection isFullyRegistered={isFullyRegistered} />
<TimelineSection isFullyRegistered={isFullyRegistered} />
```

---

## 컴포넌트 분리 전략

### **Step 1: 조건부 렌더링 패턴 파악**

-   어떤 요소가 언제 보이는지 분석
-   유사한 조건을 가진 요소끼리 그룹핑

```typescript
// 등록 완료 전에만 보이는 것들
!isFullyRegistered
  → ServiceGuideCard (안내 카드)
  → RegistrationProgress (진행 단계)

// 기기 등록 시에만 보이는 것
hasDevice
  → DeviceStatusCard

// 프로필 등록 시에만 보이는 것
hasChildProfile
  → ChildProfileCard

// 항상 보이지만 상태에 따라 내용이 달라지는 것
always visible
  → HomeHeader (보조 문구 변경)
  → RegistrationButtons (버튼 조합 변경)
  → DailyReportSection (잠김/활성)
  → TimelineSection (잠김/활성)
```

<br/>

### **Step 2: 각 섹션을 독립 컴포넌트로 추출**

```
HomeScreen.tsx (120줄)
└── components/
    ├── HomeHeader.tsx              # 닉네임 + 상태별 보조문구
    ├── ServiceGuideCard.tsx        # 서비스 안내 카드
    ├── RegistrationProgress.tsx    # 1→2 진행 인디케이터
    ├── RegistrationButtons.tsx     # 상태별 버튼 조합
    ├── DeviceStatusCard.tsx        # 기기 상태 표시
    ├── ChildProfileCard.tsx        # 아이 프로필 표시
    ├── DailyReportSection.tsx      # 탐사 일지 (잠김/활성)
    └── TimelineSection.tsx         # 타임라인 (잠김/활성)
```

<br/>

### **Step 3: 조건 판단 로직을 컴포넌트 내부로**

-   각 컴포넌트가 자신의 표시 여부와 내용을 결정
-   HomeScreen은 단순히 조합만

```typescript
// RegistrationButtons.tsx
export function RegistrationButtons({ deviceRegistered, profileRegistered, onDevicePress, onProfilePress }) {
    // 둘 다 등록되면 아예 렌더링 안함
    if (deviceRegistered && profileRegistered) {
        return null;
    }

    return (
        <View>
            {!deviceRegistered && <Button onPress={onDevicePress}>기기 등록</Button>}
            {!profileRegistered && <Button onPress={onProfilePress}>아이 프로필 설정</Button>}
        </View>
    );
}
```

```typescript
// HomeHeader.tsx
export function HomeHeader({ nickname, hasDevice, hasChildProfile }) {
    // 보조 문구 결정 로직을 내부에서 처리
    const getSubtitleText = () => {
        if (hasDevice && hasChildProfile) return null; // 둘 다 완료
        if (!hasDevice && !hasChildProfile) return '정보를 등록해주세요.';
        if (hasDevice && !hasChildProfile) return '아이 프로필도 등록해주세요.';
        if (!hasDevice && hasChildProfile) return '기기도 등록해주세요.';
    };

    const subtitle = getSubtitleText();

    return (
        <View>
            <Text>안녕하세요, {nickname}님</Text>
            {subtitle && <Text>{subtitle}</Text>}
        </View>
    );
}
```

---

## 실제 구현 예시: 4-State 홈 화면

### **상태별 UI 변화**

```typescript
// 1. 둘 다 미등록
!hasDevice && !hasChildProfile
→ 보조문구: "정보를 등록해주세요"
→ 서비스 안내 카드 표시
→ 진행 단계: ●━━○
→ 두 버튼 모두 표시
→ 모든 섹션 잠김

// 2. 기기만 등록
hasDevice && !hasChildProfile
→ 보조문구: "아이 프로필도 등록해주세요"
→ 서비스 안내 카드 표시
→ 진행 단계: ✓━━○
→ 프로필 버튼만 표시
→ 기기 상태 카드 표시
→ 나머지 섹션 잠김

// 3. 프로필만 등록
!hasDevice && hasChildProfile
→ 보조문구: "기기도 등록해주세요"
→ 서비스 안내 카드 표시
→ 진행 단계: ○━━✓
→ 기기 버튼만 표시
→ 프로필 카드 표시
→ 나머지 섹션 잠김

// 4. 둘 다 등록 (완료!)
hasDevice && hasChildProfile
→ 보조문구 없음 (깔끔!)
→ 안내 카드 숨김
→ 진행 단계 숨김
→ 등록 버튼 숨김
→ 기기 상태 카드 표시
→ 프로필 카드 표시
→ 모든 기능 활성화
```

<br/>

### **HomeScreen 최종 구조**

```typescript
export function HomeScreen({ nickname, hasDevice, hasChildProfile, deviceStatus, childProfile }) {
    const [dailyReport, setDailyReport] = useState(null);
    const isFullyRegistered = hasDevice && hasChildProfile;

    return (
        <SafeAreaView>
            <ScrollView>
                {/* 헤더 - 상태별 보조문구 자동 변경 */}
                <HomeHeader nickname={nickname} hasDevice={hasDevice} hasChildProfile={hasChildProfile} />

                {/* 등록 완료 전에만 표시 */}
                {!isFullyRegistered && <ServiceGuideCard />}
                {!isFullyRegistered && (
                    <RegistrationProgress deviceRegistered={hasDevice} profileRegistered={hasChildProfile} />
                )}

                {/* 상태별 버튼 조합 - 내부에서 판단 */}
                <RegistrationButtons
                    deviceRegistered={hasDevice}
                    profileRegistered={hasChildProfile}
                    onDevicePress={handleDeviceRegistration}
                    onProfilePress={handleChildProfileSetup}
                />

                {/* 조건부 섹션 */}
                {hasDevice && deviceStatus && <DeviceStatusCard status={deviceStatus} />}
                {hasChildProfile && childProfile && <ChildProfileCard profile={childProfile} />}

                {/* 항상 표시, 상태에 따라 잠김/활성 */}
                <DailyReportSection
                    isFullyRegistered={isFullyRegistered}
                    report={dailyReport}
                    onGenerate={handleCreateDailyReport}
                />
                <TimelineSection isFullyRegistered={isFullyRegistered} />
            </ScrollView>
        </SafeAreaView>
    );
}
```

---

## 주의사항

### **컴포넌트가 너무 많아지면**

-   관련된 컴포넌트끼리 폴더로 그룹핑
-   `index.ts`로 export 정리

```typescript
// components/index.ts
export { HomeHeader } from './HomeHeader';
export { ServiceGuideCard } from './ServiceGuideCard';
export { RegistrationProgress } from './RegistrationProgress';
// ...

// HomeScreen.tsx에서
import { HomeHeader, ServiceGuideCard, RegistrationProgress } from './components';
```

<br/>

### **props가 너무 많아지면**

-   관련된 props를 객체로 묶어서 전달
-   TypeScript interface로 타입 명확히

```typescript
// ❌ props 지옥
<HomeHeader
    nickname={nickname}
    hasDevice={hasDevice}
    hasChildProfile={hasChildProfile}
    isOnline={isOnline}
    lastSync={lastSync}
/>;

// ✅ 객체로 그룹핑
interface RegistrationStatus {
    hasDevice: boolean;
    hasChildProfile: boolean;
}

<HomeHeader nickname={nickname} registrationStatus={{ hasDevice, hasChildProfile }} />;
```

<br/>

### **조건부 렌더링 중첩 피하기**

```typescript
// ❌ 가독성 나쁨
{
    !isFullyRegistered && (hasDevice ? <View>...</View> : <View>...</View>);
}

// ✅ 컴포넌트로 분리
{
    !isFullyRegistered && <RegistrationSection hasDevice={hasDevice} hasChildProfile={hasChildProfile} />;
}
```

<br/>

### **컴포넌트 파일명은 역할이 명확하게**

```typescript
// ❌ 모호함
Section1.tsx;
Container.tsx;
Item.tsx;

// ✅ 역할이 명확
DeviceStatusCard.tsx;
RegistrationButtons.tsx;
DailyReportSection.tsx;
```

---

## 분리 시점 가이드

-   조건부 렌더링 `{조건 && <JSX>}` 가 **3개 이상** → 분리 고려
-   한 컴포넌트가 **200줄 이상** → 분리 고려
-   같은 JSX 패턴이 **2번 이상 반복** → 컴포넌트 추출
-   한 화면에서 **4가지 이상 상태 조합** → 각 섹션 분리
-   테스트하기 어렵다고 느껴지면 → 분리 신호

---

## 추가: 헤더 통일 전략

여러 화면에서 헤더 위치와 스타일이 제각각이었던 문제를 공통 컴포넌트로 해결

### **문제점**

```typescript
// LoginScreen: 중앙 정렬, padding 50/20
// ChildProfileScreen: 왼쪽 치우침, padding 24
// DeviceRegistrationScreen: absolute 위치
```

<br/>

### **해결: ScreenHeader 컴포넌트**

```typescript
// ScreenHeader.tsx
export function ScreenHeader({
  title,
  showBackButton = true,
  onBackPress
}) {
  return (
    <View style={styles.container}>
      <View style={styles.header}>
        {showBackButton && (
          <TouchableOpacity style={styles.backButton} onPress={onBackPress}>
            <Ionicons name="arrow-back" size={24} color={COLORS.primary} />
          </TouchableOpacity>
        )}
        <Text style={styles.title}>{title}</Text>
        {showBackButton && <View style={styles.placeholder} />}
      </View>
    </View>
  );
}

// 사용
<ScreenHeader title="로그인" showBackButton={false} />
<ScreenHeader title="아이 프로필" showBackButton={true} />
```

**통일된 스펙:**

-   iOS: paddingTop 50, Android: paddingTop 20
-   제목 항상 중앙 정렬
-   뒤로가기 버튼 왼쪽 절대 위치
-   대칭을 위한 오른쪽 투명 placeholder

---

## 핵심 용어

-   **조건부 렌더링**: 특정 조건에 따라 컴포넌트를 보여주거나 숨기는 패턴
-   **컴포넌트 분리**: 큰 컴포넌트를 작은 단위로 나누어 역할을 명확히 하는 리팩토링
-   **오케스트레이터**: 여러 컴포넌트를 조합하고 배치만 담당하는 부모 컴포넌트
-   **early return**: 조건에 맞지 않으면 빨리 null을 반환하여 불필요한 렌더링 방지
-   **props drilling**: props를 여러 단계에 걸쳐 전달하는 현상 (과도하면 컨텍스트 고려)
