# FlatList

---

**많은 양의 스크롤이 필요한 리스트 아이템을 보여주는 RN 컴포넌트입니다.(JS의 `map` 함수와 비슷)**

- 🤔 **ScrollView** 와의 차이
  - 두 컴포넌트 모두 데이터가 화면을 벗어났을 때 scroll이 생기는 공통점이 있음.
  - 하지만 **ScrollView**는 단순히 scroll을 통해 사용자에게 벗어난 화면을 보여주는 목적!
    (한 번에 렌더를 전부 처리하게 되어 출력되는 데이터가 고정적이고 양이 많지 않은 경우에 적당함)

---

## 기본 속성(props)

### `data`

보여주려는 데이터

### `renderItem`

[data]로 받은 각각의 데이터를 item으로 렌더링

### `keyExtractor`

JS에서의 key와 같은 요소로 지정된 항목의 고유의 값을 통해 각 item을 구별하게 함

```jsx
// 예시

import React from "react";
import { FlatList } from "react-native";

const FlatListScreen = () => {
  const renderItem = ({ item, index }) => {
    return <Beat item={item} index={index} />;
  };

  return (
    <CommonLayout.Container>
      <NavigationBar title={t("explore:tab_title_hot_beat")} />
      <FlatList
        data={data}
        renderItem={renderItem}
        keyExtractor={(item) => item.id} // 각 데이터마다의 고유 id값으로 구별
      />
    </CommonLayout.Container>
  );
};

export default FlatListScreen;
```

---

## 추가 속성

### `getItemLayout`

item의 레이아웃 크기를 매번 계산하지 않아도 되기 때문에 성능 개선에 효과적
항목의 크기(width or height)를 미리 알고 있는 경우 쓸 수 있는 최적화 속성

```jsx
const videoHeight = 200;

const getItemLayout = useCallback(
  (data, index) => ({
    length: videoHeight,
    offset: videoHeight * index,
    index,
  }),
  []
);

<FlatList
  data={DATA}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  getItemLayout={getItemLayout}
/>;
```

### `ListHeaderComponent`

데이터를 보여주는 리스트 뷰 위의 컴포넌트

### `ListFooterComponent`

데이터를 보여주는 리스트 뷰 아래의 컴포넌트

### `ListEmptyComponent`

데이터가 없을 때 보여주는 컴포넌트

### `numColums`

나눠지는 열의 개수

```jsx
numColumns={3}
```

![](https://velog.velcdn.com/images/rhfovk/post/b78b7711-2c07-4b9c-91ff-fbc0771486db/image.png)

---

## 페이지네이션(무한 스크롤)

FlatList에서 보이는 부분 외에 실제로 많은 양의 데이터를 불러오기 위해서 페이지네이션이 필요합니다. 한꺼번에 모든 데이터를 가져오게 되면 성능 하락 뿐 아니라 UX도 나빠집니다.

### `onEndReached`

유저가 리스트 끝에 도달했을때(혹은 지정된 위치에 도달했을때) 새로운 데이터를 호출하게 함

```jsx
const onEndReached = () => {
  // 끝에 도달했을 때 실행할 함수 내용
};

<FlatList
  data={DATA}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  onEndReached={onEndReached}
  onEndReachedThreshold={0.5}
/>;
```

### `onEndReachedThreshold`

스크롤이 설정한 값에 도달하면 onEndReached 메서드가 실행 됨. [data]로 받은 마지막 데이터가 보이는 가장 끝부분에서의 화면 비율 값. 값이 0.5인 경우 스크롤이 마지막 화면 길이의 절반 내에 들어오면 onEndReached 메서드가 실행 됨.

> **_onEndReachedThreshold={0}_** ⬇️
> ![](https://velog.velcdn.com/images/rhfovk/post/13cf2bec-d919-43bc-9683-bbda9dca4774/image.gif)

> **_onEndReachedThreshold={0.5}_** ⬇️
> ![](https://velog.velcdn.com/images/rhfovk/post/8439ba24-554e-4c05-9b72-ecee302e0ec7/image.gif)

---

## ScrollView → scroll-event

### `onScroll`

화면을 스크롤하는 동안 실행되는 메서드. 프레임 당 최대 한 번 실행됨. scrollEventThrottle 메서드로 제어가 가능.

### `scrollEventThrottle`

스크롤하는 동안 onScroll 메서드가 실행되는 횟수를 제어함. 숫자가 낮을수록 위치 추적 및 스크롤 이벤트에 대한 정확도가 올라가지만 전송되는 데이터에 따라 성능이 저하될 수 있음.
