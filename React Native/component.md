# React Native 컴포넌트 (Components and APIs)
- React Native는 앱을 만들 때 사용할 수 있는 다양한 내장 컴포넌트를 제공함
- 이러한 컴포넌트들은 크로스 플랫폼(iOS, Android)에서 동작하며, 각 플랫폼에 맞게 최적화된 네이티브 UI로 렌더링됨
- React Native 컴포넌트는 웹의 HTML 태그와 유사하지만, 모바일 환경에 최적화되어 있다는 차이점이 있음

```jsx
import React from 'react';
import {View, Text, Image, ScrollView} from 'react-native';

const App = () => {
    return (
        <ScrollView>
            <View>
                <Text>안녕하세요!</Text>
                <Image source={{uri: 'https://example.com/image.png'}} />
            </View>
        </ScrollView>
    );
};

export default App;
```

---

## 기본 컴포넌트 (Basic Components)
대부분의 앱에서 자주 사용하게 되는 핵심 컴포넌트들

### **View**
- UI를 구성하는 가장 기본적인 컴포넌트
- 웹의 `<div>` 태그와 유사한 역할
- 레이아웃, 스타일링, 터치 이벤트 등을 처리하는 컨테이너 역할
```jsx
<View style={{flex: 1, backgroundColor: '#fff'}}>
    <Text>View 안에 들어가는 내용</Text>
</View>
```
<br/>

### **Text**
- 텍스트를 표시하는 컴포넌트
- React Native에서는 모든 텍스트가 반드시 Text 컴포넌트 안에 있어야 함
- 중첩 가능하며, 스타일과 터치 이벤트를 지원
```jsx
<Text style={{fontSize: 20, fontWeight: 'bold'}}>
    Hello React Native!
</Text>
```
<br/>

### **Image**
- 이미지를 표시하는 컴포넌트
- 로컬 이미지와 네트워크 이미지 모두 지원
- `source` prop을 통해 이미지 경로를 지정
```jsx
// 네트워크 이미지
<Image source={{uri: 'https://example.com/logo.png'}} 
       style={{width: 100, height: 100}} />

// 로컬 이미지
<Image source={require('./assets/logo.png')} />
```
<br/>

### **TextInput**
- 사용자로부터 텍스트 입력을 받는 컴포넌트
- 키보드를 통해 텍스트를 입력할 수 있음
- placeholder, onChangeText 등 다양한 prop 지원
```jsx
<TextInput
    placeholder="이름을 입력하세요"
    onChangeText={(text) => setText(text)}
    value={text}
    style={{borderWidth: 1, padding: 10}}
/>
```
<br/>

### **ScrollView**
- 스크롤 가능한 컨테이너를 제공하는 컴포넌트
- 화면에 표시할 내용이 화면 크기를 초과할 때 사용
- 내부에 여러 컴포넌트와 뷰를 포함할 수 있음
```jsx
<ScrollView>
    <Text>긴 콘텐츠 1</Text>
    <Text>긴 콘텐츠 2</Text>
    <Text>긴 콘텐츠 3</Text>
</ScrollView>
```
<br/>

### **Pressable**
- 다양한 press 인터랙션을 감지할 수 있는 래퍼 컴포넌트
- onPress, onPressIn, onPressOut, onLongPress 등 다양한 이벤트 지원
- 터치 피드백을 커스터마이징할 수 있음
```jsx
<Pressable onPress={() => alert('눌렸습니다!')}>
    <Text>눌러보세요</Text>
</Pressable>
```

---

## 사용자 인터페이스 컴포넌트 (User Interface)
플랫폼에 관계없이 일관된 UI를 제공하는 컴포넌트들

### **Button**
- 터치를 처리하는 기본 버튼 컴포넌트
- 플랫폼에 맞는 기본 스타일이 적용됨
- `title`과 `onPress` prop이 필수
```jsx
<Button 
    title="클릭하세요"
    onPress={() => console.log('버튼 클릭!')}
    color="#841584"
/>
```
<br/>

### **Switch**
- boolean 값을 입력받는 토글 스위치 컴포넌트
- On/Off 상태를 표현할 때 사용
```jsx
<Switch
    value={isEnabled}
    onValueChange={() => setIsEnabled(!isEnabled)}
/>
```

---

## 리스트 뷰 컴포넌트 (List Views)
대량의 데이터를 효율적으로 표시하기 위한 컴포넌트들
- ScrollView와 달리 현재 화면에 보이는 요소만 렌더링하여 성능이 우수함
- 긴 목록을 표시할 때 반드시 사용해야 함

### **FlatList**
- 성능이 우수한 스크롤 가능한 리스트 컴포넌트
- 대량의 데이터를 효율적으로 렌더링
- `data`와 `renderItem` prop이 필수
```jsx
<FlatList
    data={[
        {id: '1', title: '첫 번째 항목'},
        {id: '2', title: '두 번째 항목'},
        {id: '3', title: '세 번째 항목'},
    ]}
    renderItem={({item}) => <Text>{item.title}</Text>}
    keyExtractor={item => item.id}
/>
```
<br/>

### **SectionList**
- FlatList와 유사하지만 섹션별로 데이터를 그룹화할 수 있음
- 헤더를 가진 리스트를 만들 때 유용
```jsx
<SectionList
    sections={[
        {title: '과일', data: ['사과', '바나나']},
        {title: '채소', data: ['당근', '양배추']},
    ]}
    renderItem={({item}) => <Text>{item}</Text>}
    renderSectionHeader={({section}) => <Text>{section.title}</Text>}
/>
```

---

## 기타 유용한 컴포넌트 (Others)

### **ActivityIndicator**
- 로딩 중임을 표시하는 원형 인디케이터
```jsx
<ActivityIndicator size="large" color="#0000ff" />
```
<br/>

### **Modal**
- 현재 화면 위에 콘텐츠를 표시하는 모달 컴포넌트
```jsx
<Modal visible={modalVisible} animationType="slide">
    <View>
        <Text>모달 내용</Text>
    </View>
</Modal>
```
<br/>

### **KeyboardAvoidingView**
- 키보드가 올라올 때 자동으로 뷰를 이동시켜주는 컴포넌트
- 입력 폼이 키보드에 가려지는 것을 방지
```jsx
<KeyboardAvoidingView behavior="padding">
    <TextInput placeholder="입력하세요" />
</KeyboardAvoidingView>
```
<br/>

### **StatusBar**
- 앱의 상태 바를 제어하는 컴포넌트
```jsx
<StatusBar barStyle="dark-content" backgroundColor="#fff" />
```
<br/>

### **Alert**
- 지정된 제목과 메시지로 경고 다이얼로그를 표시
```jsx
Alert.alert(
    '알림',
    '정말 삭제하시겠습니까?',
    [
        {text: '취소', style: 'cancel'},
        {text: '확인', onPress: () => console.log('삭제됨')}
    ]
);
```
<br/>

### **RefreshControl**
- ScrollView 내부에서 pull-to-refresh 기능을 추가할 때 사용
```jsx
<ScrollView
    refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
    }
>
    <Text>당겨서 새로고침</Text>
</ScrollView>
```

---

## 플랫폼별 컴포넌트

### **Android 전용**
**BackHandler**
- 안드로이드 하드웨어 뒤로가기 버튼 감지 및 제어
<br/>

**DrawerLayoutAndroid**
- 안드로이드의 DrawerLayout을 렌더링
<br/>

**PermissionsAndroid**
- 안드로이드 M에 도입된 권한 모델에 대한 액세스 제공
<br/>

**ToastAndroid**
- 안드로이드 토스트 알림 생성
```jsx
ToastAndroid.show('저장되었습니다!', ToastAndroid.SHORT);
```

### **iOS 전용**
**ActionSheetIOS**
- iOS 액션 시트 또는 공유 시트를 표시하는 API
<br/>

**InputAccessoryView**
- 키보드 위에 사용자 정의 뷰를 표시
<br/>

**SafeAreaView** (deprecated)
- iOS의 안전 영역 경계 내에서 콘텐츠를 렌더링
- 현재는 `react-native-safe-area-context` 라이브러리 사용 권장

---

## 컴포넌트 선택 가이드
- 간단한 텍스트 표시 → **Text**
- 레이아웃 컨테이너 → **View**
- 버튼이 필요할 때 → **Button** 또는 **Pressable**
- 사용자 입력 → **TextInput**
- 긴 콘텐츠 → **ScrollView** (정적) 또는 **FlatList** (동적/대량 데이터)
- 로딩 표시 → **ActivityIndicator**
- 이미지 표시 → **Image**
- 섹션별 리스트 → **SectionList**
- 모달 화면 → **Modal**
- 토글 스위치 → **Switch**

