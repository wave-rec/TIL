# React Native SafeAreaView & Insets 완벽 가이드

-   iPhone 노치/펀치홀 영역에 UI가 가려지는 문제 해결
-   기본 SafeAreaView의 한계와 react-native-safe-area-context 도입 이유
-   useSafeAreaInsets로 세밀한 레이아웃 제어하기
-   실전에서 바로 쓸 수 있는 패턴 정리

---

## 왜 SafeAreaView가 필요한가?

iPhone X 이후 노치가 등장하면서, 앱 UI가 물리적으로 가려지는 영역이 생겼다.

```typescript
// ❌ SafeArea 미적용 시
→ 상단 헤더가 노치에 가려짐
→ 하단 버튼이 홈 인디케이터에 묻힘
→ 가로모드에서 좌우 버튼이 라운드 코너에 숨음

// ✅ SafeArea 적용 시
→ 콘텐츠가 안전 영역 내에만 배치됨
→ 모든 방향에서 UI가 정상적으로 보임
```

---

## 두 가지 접근 방식

### **React Native 기본 SafeAreaView (비추천)**

iOS에서만 작동하고 유연성이 떨어진다.

```jsx
import { SafeAreaView } from 'react-native';

// ❌ 문제점
function Screen() {
    return (
        <SafeAreaView style={{ flex: 1, backgroundColor: '#007AFF' }}>
            <Text>콘텐츠</Text>
        </SafeAreaView>
    );
}

/**
 * 문제점:
 * 1. Android에서는 아무 효과 없음
 * 2. flex: 1 없으면 작동 안함
 * 3. 중첩 시 예상과 다르게 동작
 * 4. 배경색이 safe area 밖에는 안 먹힘
 * 5. 방향별 제어 불가능 (항상 4방향 전부 적용)
 */
```

<br/>

### **react-native-safe-area-context (권장)**

크로스 플랫폼 지원과 세밀한 제어가 가능하다.

```jsx
import { SafeAreaView, useSafeAreaInsets } from 'react-native-safe-area-context';

// ✅ 장점
function Screen() {
    const insets = useSafeAreaInsets();

    return (
        <SafeAreaView edges={['top', 'left', 'right']}>
            <Text>콘텐츠</Text>
        </SafeAreaView>
    );
}

/**
 * 장점:
 * 1. iOS + Android 모두 지원
 * 2. edges prop으로 방향별 제어
 * 3. useSafeAreaInsets로 수치 직접 활용
 * 4. React Navigation과 완벽 호환
 * 5. 펀치홀, 폴더블 기기도 대응
 */
```

---

## Step 1: 라이브러리 설치 및 설정

### **설치**

```bash
npm install react-native-safe-area-context
# 또는
yarn add react-native-safe-area-context

# iOS
cd ios && pod install
```

<br/>

### **앱 최상위에 Provider 설정**

SafeAreaProvider는 **딱 한 번**, 앱의 루트에서만 감싼다.

```jsx
// App.tsx
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function App() {
    return (
        <SafeAreaProvider>
            <NavigationContainer>{/* 나머지 앱 */}</NavigationContainer>
        </SafeAreaProvider>
    );
}
```

---

## Step 2: SafeAreaView로 전체 화면 보호

### **기본 사용 - 4방향 모두 적용**

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';

function ProfileScreen() {
    return (
        <SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
            <Text>프로필 화면</Text>
        </SafeAreaView>
    );
}

// → top, right, bottom, left 모두 safe area 적용
```

<br/>

### **edges prop으로 선택적 적용**

특정 방향만 safe area를 적용할 수 있다.

```jsx
// 상단만 (탭바가 있는 화면)
<SafeAreaView edges={['top']}>
  <Text>컨텐츠</Text>
</SafeAreaView>

// 상단 + 좌우 (하단은 탭바가 처리)
<SafeAreaView edges={['top', 'left', 'right']}>
  <Text>컨텐츠</Text>
</SafeAreaView>

// 전체 배경은 꽉 채우되 콘텐츠만 안전 영역
<View style={{ flex: 1, backgroundColor: '#007AFF' }}>
  <SafeAreaView edges={['top', 'bottom']}>
    <Text>컨텐츠</Text>
  </SafeAreaView>
</View>
```

---

## Step 3: useSafeAreaInsets로 세밀한 제어

SafeAreaView는 자동으로 padding을 적용하지만, 때로는 직접 insets 값을 활용해야 한다.

### **Insets 값 가져오기**

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function MyComponent() {
    const insets = useSafeAreaInsets();

    console.log(insets);
    // {
    //   top: 59,      // 상단 노치 영역
    //   bottom: 34,   // 홈 인디케이터 영역
    //   left: 0,      // 좌측 (세로 모드)
    //   right: 0      // 우측 (세로 모드)
    // }

    return (
        <View
            style={{
                paddingTop: insets.top,
                paddingBottom: insets.bottom,
            }}
        >
            <Text>수동으로 insets 적용</Text>
        </View>
    );
}
```

---

## 실전 패턴: 상황별 적용 방법

### **패턴 1: 고정 헤더 (상단 노치 대응)**

```jsx
function Header() {
    const insets = useSafeAreaInsets();

    return (
        <View
            style={{
                paddingTop: insets.top + 16, // 노치 높이 + 추가 여백
                backgroundColor: '#007AFF',
                paddingHorizontal: 20,
                paddingBottom: 16,
            }}
        >
            <Text style={{ color: 'white', fontSize: 20, fontWeight: 'bold' }}>헤더</Text>
        </View>
    );
}
```

<br/>

### **패턴 2: 하단 버튼 (홈 인디케이터 대응)**

```jsx
function BottomCTA() {
    const insets = useSafeAreaInsets();

    return (
        <View
            style={{
                position: 'absolute',
                bottom: 0,
                left: 0,
                right: 0,
                paddingBottom: insets.bottom + 20, // 홈바 + 여유 공간
                paddingHorizontal: 20,
                backgroundColor: 'white',
                shadowColor: '#000',
                shadowOffset: { width: 0, height: -2 },
                shadowOpacity: 0.1,
                shadowRadius: 4,
            }}
        >
            <TouchableOpacity style={styles.button}>
                <Text style={styles.buttonText}>다음</Text>
            </TouchableOpacity>
        </View>
    );
}
```

<br/>

### **패턴 3: ScrollView에서 활용**

```jsx
function ArticleScreen() {
    const insets = useSafeAreaInsets();

    return (
        <ScrollView
            contentContainerStyle={{
                paddingTop: insets.top,
                paddingBottom: insets.bottom + 40, // 스크롤 끝 여유
                paddingHorizontal: 20,
            }}
        >
            <Text>긴 콘텐츠...</Text>
        </ScrollView>
    );
}
```

<br/>

### **패턴 4: 전체 배경 + 안전 영역 콘텐츠**

배경은 화면 전체를 채우고, 콘텐츠만 안전 영역 내에 배치.

```jsx
function SplashScreen() {
    const insets = useSafeAreaInsets();

    return (
        <View style={{ flex: 1, backgroundColor: '#007AFF' }}>
            {/* 배경색은 노치까지 꽉 채움 */}
            <View
                style={{
                    flex: 1,
                    paddingTop: insets.top,
                    paddingBottom: insets.bottom,
                    justifyContent: 'center',
                    alignItems: 'center',
                }}
            >
                <Text style={{ color: 'white', fontSize: 32 }}>로고</Text>
            </View>
        </View>
    );
}
```

<br/>

### **패턴 5: 조건부 Insets (모달/풀스크린 분기)**

```jsx
function ModalScreen({ isFullScreen }) {
    const insets = useSafeAreaInsets();

    return (
        <View
            style={{
                flex: 1,
                paddingTop: isFullScreen ? insets.top : 0,
                // 풀스크린이 아닌 모달은 insets 불필요
            }}
        >
            <Text>모달 콘텐츠</Text>
        </View>
    );
}
```

---

## 디바이스별 Insets 값 참고

| 디바이스             | 방향 | top | bottom | left | right |
| -------------------- | ---- | --- | ------ | ---- | ----- |
| iPhone 15 Pro        | 세로 | 59  | 34     | 0    | 0     |
| iPhone 15 Pro        | 가로 | 0   | 21     | 59   | 59    |
| iPhone SE (3rd)      | 세로 | 20  | 0      | 0    | 0     |
| Android (대부분)     | 세로 | 24  | 0      | 0    | 0     |
| Android 펀치홀       | 세로 | 30  | 0      | 0    | 0     |
| Galaxy Z Fold (펼침) | 세로 | 36  | 24     | 0    | 0     |

---

## 주의사항 및 팁

### **SafeAreaProvider는 한 번만**

```jsx
// ❌ 여러 번 감싸면 안됨
<SafeAreaProvider>
  <Screen1 />
  <SafeAreaProvider> {/* 중복! */}
    <Screen2 />
  </SafeAreaProvider>
</SafeAreaProvider>

// ✅ 앱 최상위에서 한 번만
<SafeAreaProvider>
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen name="Screen1" component={Screen1} />
      <Stack.Screen name="Screen2" component={Screen2} />
    </Stack.Navigator>
  </NavigationContainer>
</SafeAreaProvider>
```

<br/>

### **SafeAreaView 중첩 주의**

```jsx
// ❌ insets가 두 번 적용되어 공간 낭비
<SafeAreaView edges={['top']}>
  <SafeAreaView edges={['top']}>
    <Text>콘텐츠</Text>
  </SafeAreaView>
</SafeAreaView>

// ✅ 필요한 곳에만 한 번
<SafeAreaView edges={['top']}>
  <View>
    <Text>콘텐츠</Text>
  </View>
</SafeAreaView>
```

<br/>

### **flex: 1 꼭 넣기**

```jsx
// ❌ flex 없으면 SafeAreaView가 제대로 작동 안함
<SafeAreaView style={{ backgroundColor: '#fff' }}>
  <Text>콘텐츠</Text>
</SafeAreaView>

// ✅ flex: 1로 화면 전체 채우기
<SafeAreaView style={{ flex: 1, backgroundColor: '#fff' }}>
  <Text>콘텐츠</Text>
</SafeAreaView>
```

<br/>

### **React Navigation 사용 시**

React Navigation은 자동으로 safe area를 처리해준다.

```jsx
// Screen 옵션에서 headerShown: false로 커스텀 헤더 쓸 때만 직접 처리
<Stack.Screen name="Home" component={HomeScreen} options={{ headerShown: false }} />;

// HomeScreen.tsx에서
function HomeScreen() {
    return (
        <SafeAreaView edges={['top']}>
            <CustomHeader />
            <Content />
        </SafeAreaView>
    );
}
```

---

## 언제 무엇을 사용할까?

```typescript
// SafeAreaView
→ 화면 전체를 감싸고 싶을 때
→ 간단하게 safe area 적용
→ edges로 방향 선택 가능

// useSafeAreaInsets
→ 특정 요소에만 적용
→ 계산이 필요한 경우 (insets.bottom + 20)
→ 조건부로 insets 적용
→ 애니메이션과 결합

// 수동 padding
→ 복잡한 레이아웃
→ ScrollView contentContainerStyle
→ Animated View와 함께 사용
```

---

## 실제 앱에서의 적용 사례

### **홈 화면 (탭바 포함)**

```jsx
function HomeScreen() {
    return (
        <SafeAreaView edges={['top', 'left', 'right']} style={{ flex: 1 }}>
            {/* bottom은 탭바가 처리하므로 제외 */}
            <ScrollView>
                <Header />
                <Content />
            </ScrollView>
        </SafeAreaView>
    );
}
```

<br/>

### **로그인 화면 (풀스크린)**

```jsx
function LoginScreen() {
    const insets = useSafeAreaInsets();

    return (
        <View style={{ flex: 1, backgroundColor: '#007AFF' }}>
            <View
                style={{
                    flex: 1,
                    paddingTop: insets.top + 40,
                    paddingBottom: insets.bottom + 40,
                    paddingHorizontal: 24,
                }}
            >
                <Logo />
                <LoginForm />
            </View>
        </View>
    );
}
```

<br/>

### **상세 페이지 (커스텀 헤더)**

```jsx
function DetailScreen() {
    const insets = useSafeAreaInsets();
    const navigation = useNavigation();

    return (
        <View style={{ flex: 1 }}>
            {/* 커스텀 헤더 */}
            <View
                style={{
                    paddingTop: insets.top + 12,
                    paddingBottom: 12,
                    paddingHorizontal: 16,
                    backgroundColor: 'white',
                }}
            >
                <TouchableOpacity onPress={() => navigation.goBack()}>
                    <Ionicons name="arrow-back" size={24} />
                </TouchableOpacity>
            </View>

            {/* 콘텐츠 */}
            <ScrollView>
                <Text>상세 내용</Text>
            </ScrollView>
        </View>
    );
}
```

---

## 핵심 용어

-   **Safe Area**: 노치, 상태바, 홈 인디케이터 등으로 가려지지 않는 안전한 영역
-   **Insets**: 안전 영역까지의 거리 (top, right, bottom, left)
-   **edges**: SafeAreaView에서 어느 방향에 safe area를 적용할지 지정하는 prop
-   **SafeAreaProvider**: 앱 전체에 safe area 정보를 제공하는 Context Provider
-   **useSafeAreaInsets**: safe area insets 값을 가져오는 React Hook
