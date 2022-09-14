# Route & React **Navigation**

`React Navigation`은 여러 화면을 보여주는 앱에서 화면 이동을 도와주는 라이브러리입니다.

![](https://velog.velcdn.com/images/rhfovk/post/b778bb6f-7bda-4597-8a69-d77d470df41e/image.png)

Navigation은 **stack 구조**로 되어 있어서 화면을 이동하면 **이전 화면을 stack에 쌓아가는 구조**로 되어있습니다.

```jsx
// Navigation.js

// 앱이 각 화면이 전환될 수 있는 기본 틀을 제공
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

const Navigation = () => {
	return (
 {/* 네비게이션 트리를 관리해주는 컴포넌트 */}
			<NavigationContainer>
      {/* 네비게이션 기본틀의 스택을 생성 */}
         <Stack.Navigator>
          {/* 해당스택에 들어갈 화면 요소를 넣어줌 */}
             <Stack.Screen name="HomeScreen" component={HomeScreen}/>
         </Stack.Navigator>
			</NavigationContainer>
    )
};

export default App;
```

### `Navigator`

네비게이션 스택을 생성

### `Screen`

스택안에 들어갈 화면 요소를 설정 (name = title명, componenet = 실제 화면 컴포넌트)

## Navigation props

### `navigate`

stack 안에 있는 화면 컴포넌트로 이동, 인자에 이동하고 싶은 화면의 name을 입력

```jsx
navigation.navigate("HomeScreen");
```

또한 route를 navigate할 때 인자도 같이 보낼 수 있음

```jsx
navigation.navigate("HomeScreen", { paramName: "value" });
```

### `goBack`

stack에 저장된 바로 전 화면으로 이동

```jsx
navigation.goBack();
```

> **🤔 navigation.pop 과 goBack의 차이**

        **pop**: stack navigator 안에서만 해당하고 push된 화면 수와 같은 인수를 받을 수 있음(stack 안에서 여러 화면 앞으로 이동이 가능)
        **goBack**: 어떤 navigator에서 사용 가능(stack, tabs, drawer)

### `reset`

화면 새로고침 (상태 초기화, route 초기화 or 변경)

```jsx
navigation.reset({
  index: 0,
  routes: [{ name: "HomeScreen" }],
});
```

### `push`

navigate와 같은 기능이지만 자신 → 자신 페이지로 이동 가능

### `popToTop`

첫번째 페이지로 이동

### **`setParams`**

마운트된 현재 화면 컴포넌트에서 setParams를 사용하여 navigation params를 업데이트 가능
(react의 setState() 유사)

```jsx
navigation.setParams({ paramName: "Updated" });
```
