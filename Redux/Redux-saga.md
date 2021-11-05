# Redux-saga

## 미들웨어

![](https://images.velog.io/images/rhfovk/post/82160492-20d7-4ae7-8c4e-553220422af7/image.png)
미들웨어는, 액션이 dispatch 되어서 리듀서에서 이를 처리하기 전에 사전에 지정된 작업들을 설정한다.

액션과 리듀서 사이의 중간자라고 이해하면 쉽다! 액션을 검증하거나 **필터링, 모니터링, 외부 API와의 연동, 비동기 처리** 등을 추가적으로 수행할 수 있도록 한다.

---

## Redux-saga

그래서 미들웨어 중 `saga`에 대해 알아보자!

![](https://images.velog.io/images/rhfovk/post/b27e6cc0-553e-4ec0-80c4-256c13f821a0/image.png)

redux-saga는 **비동기적**으로 dispatch를 사용할 수 있고, 내부 메소드를 활용하여 사용자의 여러번 requset 중 가장 최근 or 가장 마지막 request의 response만 받아오도록 하는 기능도 있다.

### Layer 구성

![](https://images.velog.io/images/rhfovk/post/13137d95-9130-4c8c-920f-e1d58b22a84d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-05%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.36.57.png)

`view layer`에는 보편적으로 **리액트**가,
`state layer`에는 **리덕스**,
`side effect layer`에는 **리덕스 사가**를 쓰인다.

> **Side Effect?**
> 여기서 말하는 사이드 이펙트란 외부세계에 영향을 주거나 받는 것.
> 외부로부터 데이터를 불러와서 결과값에 따라 다르게 보여 주거나 또는 기존 데이터와 비교를 해서 업데이트 유무를 판단해야 하는 상황 등이 있다.
> 이럴때 Redux-Saga를 사용하게 된다.

### Generator

일단 saga를 이해하기 위해서는 그 전에 **Generator 문법**을 알아야 한다.

제너레이터 함수를 만들 때에는 `function*` 이라는 키워드를 사용한다.
제너레이터 함수를 호출했을때는 한 객체가 반환되는데, 이를 **제너레이터**라고 부른다.

```js
function* getNumbers() {
  console.log("첫번째");
  yield 10;
  console.log("두번째");
  yield 20;
  console.log("세번째");
  return "finished";
}
const gen = f1();
```

이 제너레이터 객체에는 `next` 라는 함수가 있다. 이것을 총 3번 호출해보자!
그러면 `yield` 라고 표시되어있는 부분을 기준으로 구간이 나눠지고, 한번 호출할 때마다 `yield` 가 있는 구간까지 실행이 된다. 반환 객체는 `yield` 오른쪽 값이 반환되고, done 이라는 속성에 f1 함수가 전부 완료되었는지 상태여부가 boolean 타입으로 리턴됩니다.

```js
console.log(gen.next());
/*
  실행구간
  console.log('첫번째');
  yield 10;

  반환객체
  { value: 10, done: false }
*/

console.log(gen.next());
/*
  실행구간
  console.log('두번째');
  yield 20;

  반환객체
  { value: 20, done: false }
*/

console.log(gen.next());
/*
  실행구간
  console.log('세번째');
  yield 30;

  반환객체
  { value: 'finished', done: true }
*/
```

### Effect

Effect는 **미들웨어에 의해 수행되는 명령을 담고 있는 자바스크립트 객체**라고 생각하면 된다.

Effect에는 여러가지 유틸함수들이 있다. 리덕스 사가에서는 아래와 같이 `put`, `call`, `all` 이라는 부수효과 함수를 주로 사용한다.

> `put` 이라는 것은 리덕스 액션을 발생시키는 것.
> `call` 은 서버의 API 호출 함수가 인수로 들어간다.(예외도 있음)

여기에서는 REQUEST_LIKE 액션이 실행될 때마다 fetchData 아래에 들어있는 로직이 실행이 되는 것입니다. 여기서 takeLeading 의 역할은 우리가 만약 좋아요를 한번 눌렀을때, fetchData 함수가 아직 진행중에 있다면, 그 사이에 들어온 액션은 무시하고 처음에 들어온 액션을 우선으로 처리하도록 관리해줍니다.

반대의 기능을 하는 것은 takeLatest 라는 함수가 있습니다. 이것은 처음에 요청했던 부분은 무시하고 최근에 요청한 액션을 우선적으로 처리합니다. 처리중인 것을 취소시키고 새로 들어온 것을 다시 처리하는 역할을 합니다.

---

## 날씨 API를 사용한 리덕스 사가 실습

플랜즈 커피에서 사용하는 리덕스 사가를 익히기 위해 오픈 api인 날씨 api를 사용해 서울의 날씨를 가져오는 앱을 만드는 실습을 진행했다.

**components/Weather.js **

```js
import { call, put, take } from "redux-saga/effects";
import { eventChannel, END } from "redux-saga";

// 이벤트 채널을 만들어 setinterval 함수를 통해 1초 마다 api 호출을 하는 함수를 만들었다.
// 10초를 제한으로 둬서 10초 동안 1초씩 api 호출을 한다.
function countdown(secs) {
  return eventChannel((emitter) => {
    const iv = setInterval(async () => {
      secs -= 1;
      if (secs > 0) {
        const result = await axios.get(
          "http://api.openweathermap.org/data/2.5/weather?q=Seoul&units=metric&APPID=e75a0a68adc50371c5898d8d43931062"
        );
        emitter(result);
      } else {
        emitter(END);
      }
    }, 1000);
    return () => {
      clearInterval(iv);
    };
  });
}

// 사가에서는 channel로 countdown함수의 결과값을 받는다.
// yield로 channel을 받아서 result에 값을 넣고
// yield로 액션을 put한다.
export function* weatherSaga() {
  const channel = yield call(countdown, 10);

  while (true) {
    const { data } = yield take(channel);
    const result = {
      area: data.name,
      lowTemp: Math.floor(data.main.temp_min),
      highTemp: Math.floor(data.main.temp_max),
    };
    yield put(getWeatherSuccess(result));
  }
}
```

<br>

### 출처 📚

---

https://mskims.github.io/redux-saga-in-korean/
https://velopert.com/3401
날씨api : https://www.apistore.co.kr/generalApi/generalApiView.do?general_service_seq=84
