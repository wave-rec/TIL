# React Native 커스텀 폰트 적용 (Pretendard)
- React Native에서 커스텀 폰트를 전역으로 적용하는 방법
- 기본 방식과 커스텀 Text 컴포넌트 방식 두 가지 존재
- 프로젝트 규모와 유지보수 요구사항에 따라 적절한 방식 선택 필요

---

## 두 가지 적용 방식

### **기본 방식 (직접 fontFamily 지정)**
- 각 컴포넌트의 StyleSheet에 fontFamily를 직접 지정
- 간단하고 빠른 적용이 가능하지만 관리가 어려움
- 컴포넌트마다 일일이 폰트를 지정해야 함
- 오타 발생 가능성이 높고 일관성 유지가 어려움

```jsx
<Text style={{ fontFamily: 'Pretendard-Bold' }}>제목</Text>
<Text style={{ fontFamily: 'Pretendard-Medium' }}>라벨</Text>
```
<br/>

### **커스텀 Text 컴포넌트 방식 (권장)**
- 전역에서 사용할 커스텀 Text 컴포넌트를 만들어 폰트 자동 적용
- fontWeight prop으로 폰트 굵기 제어
- 폰트 적용 로직을 한 곳에서 관리
- 일관성과 유지보수성 향상

```jsx
<Text fontWeight="bold">제목</Text>
<Text fontWeight="medium">라벨</Text>
<Text fontWeight="regular">본문</Text>
```

---

## 커스텀 폰트 적용 단계

### **Step 1: 폰트 파일 추가**
```
src/
  assets/
    fonts/
      Pretendard-Regular.otf
      Pretendard-Medium.otf
      Pretendard-Bold.otf
```
<br/>

### **Step 2: 폰트 상수 정의**
- 폰트 이름을 상수로 관리하여 오타 방지
- TypeScript로 타입 안정성 확보

```typescript
// src/constants/fonts.ts
export const FONTS = {
  regular: 'Pretendard-Regular',
  medium: 'Pretendard-Medium',
  bold: 'Pretendard-Bold',
};
```
<br/>

### **Step 3: App.tsx에서 폰트 로딩**
- `useFonts` 훅을 사용하여 폰트를 비동기로 로드
- 폰트 로딩이 완료될 때까지 대기 화면 표시

```jsx
// App.tsx
import { useFonts } from 'expo-font';

export default function App() {
  const [fontsLoaded] = useFonts({
    'Pretendard-Regular': require('@/assets/fonts/Pretendard-Regular.otf'),
    'Pretendard-Medium': require('@/assets/fonts/Pretendard-Medium.otf'),
    'Pretendard-Bold': require('@/assets/fonts/Pretendard-Bold.otf'),
  });

  if (!fontsLoaded) {
    return <SplashScreen />; // 폰트 로딩 중
  }

  return <AppNavigator />;
}
```
<br/>

### **Step 4: 커스텀 Text 컴포넌트 구현**
- 기본 Text 컴포넌트를 래핑하여 폰트를 자동 적용
- fontWeight prop으로 간편하게 폰트 굵기 제어

```typescript
// src/components/common/Text.tsx
import { FONTS } from '@/constants/fonts';
import React from 'react';
import { Text as RNText, TextProps as RNTextProps, StyleSheet } from 'react-native';

interface TextProps extends RNTextProps {
  fontWeight?: 'regular' | 'medium' | 'bold';
}

export function Text({ style, fontWeight = 'regular', ...props }: TextProps) {
  const fontFamily = FONTS[fontWeight] || FONTS.regular;
  
  return (
    <RNText
      style={[
        styles.default,
        { fontFamily },
        style,
      ]}
      {...props}
    />
  );
}

const styles = StyleSheet.create({
  default: {
    // 기본 스타일
  },
});
```

---

## 주의사항

### **폰트 로딩 완료 대기**
- 폰트가 로드되기 전에 렌더링하면 폰트가 적용되지 않음
- 반드시 `fontsLoaded` 상태를 확인

```jsx
if (!fontsLoaded) {
  return <SplashScreen />;
}
```
<br/>

### **웹/네이티브 환경별 처리**
```jsx
// 웹에서는 document.fonts.ready 사용
if (Platform.OS === 'web' && typeof document !== 'undefined') {
  await document.fonts.ready;
}
```
<br/>

### **스타일시트에서 fontFamily 제거**
- 커스텀 Text 컴포넌트 사용 시 StyleSheet에서 fontFamily, fontWeight 제거
- 모든 폰트 관련 로직은 컴포넌트에서 처리

```jsx
// ❌ 잘못된 방식
const styles = StyleSheet.create({
  title: {
    fontSize: 20,
    fontFamily: 'Pretendard-Bold', // 제거 필요
    fontWeight: 'bold', // 제거 필요
  },
});

// ✅ 올바른 방식
const styles = StyleSheet.create({
  title: {
    fontSize: 20,
    // fontFamily, fontWeight는 Text 컴포넌트에서 처리
  },
});

<Text style={styles.title} fontWeight="bold">제목</Text>
```
<br/>

### **폰트 파일 확장자**
- iOS/Android: `.otf`, `.ttf` 모두 가능
- 웹: `.otf` 권장 (더 나은 호환성)

---

## 방식별 선택 가이드
- 프로토타입/빠른 테스트 → **기본 방식**
- 단기 프로젝트/한두 곳에서만 사용 → **기본 방식**
- 프로덕션 앱 개발 → **커스텀 Text 컴포넌트**
- 팀 프로젝트 → **커스텀 Text 컴포넌트**
- 유지보수 기간이 긴 프로젝트 → **커스텀 Text 컴포넌트**
- 폰트 변경 가능성이 있는 경우 → **커스텀 Text 컴포넌트**

---

## 핵심 용어
- **useFonts**: Expo에서 폰트를 비동기로 로드하는 훅
- **fontFamily**: React Native에서 폰트를 지정하는 스타일 속성
- **fontWeight**: 폰트 굵기 (커스텀 폰트에서는 별도 파일 필요)
- **FONTS 상수**: 폰트 이름을 상수로 관리하여 오타 방지
- **커스텀 Text 컴포넌트**: 기본 Text를 래핑하여 폰트를 자동 적용하는 컴포넌트