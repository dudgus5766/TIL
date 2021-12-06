# 비동기와 Promise

> '동기? 비동기? promise?'
> api호출을 할 때 오류를 피하기 위해 썼던 promise나 async.
> 왜 써야하는지 좀 더 자세하게 찾아보고 싶어 이렇게 글을 쓴다.

## 비동기 처리

자바스크립트의 비동기 처리란 **특정 코드의 연산이 끝날 때까지 코드의 실행을 멈추지 않고 다음 코드를 먼저 실행하는 자바스크립트의 특성**을 의미한다.

---

## Promise

프로미스는 자바스크립트 **비동기 처리에 사용되는 객체**이다. 여기서 자바스크립트의 비동기 처리란 **‘특정 코드의 실행이 완료될 때까지 기다리지 않고 다음 코드를 먼저 수행하는 자바스크립트의 특성’**을 의미한다.(원래 자바스크립트는 싱글스레드(single thread)로 동기적으로 움직이기 때문!)

#### 기본 코드

```js
function getData(callback) {
  // new Promise() 추가
  return new Promise(function (resolve, reject) {
    $.get("url 주소/products/1", function (response) {
      // 데이터를 받으면 resolve() 호출
      resolve(response);
    });
  });
}

// getData()의 실행이 끝나면 호출되는 then()
getData().then(function (tableData) {
  // resolve()의 결과 값이 여기로 전달됨
  console.log(tableData); // $.get()의 reponse 값이 tableData에 전달됨
});
```

---

## Promise 3가지 상태

- **Pending(대기)** : 비동기 처리 로직이 아직 완료되지 않은 상태
- **Fulfilled(이행)** : 비동기 처리가 완료되어 프로미스가 결과 값을 반환해준 상태
- **Rejected(실패)** : 비동기 처리가 실패하거나 오류가 발생한 상태

```js
new Promise();
```

먼저 아래와 같이 new Promise() 메서드를 호출하면 **대기(Pending) 상태**가 된다.

```js
new Promise(function (resolve, reject) {
  // ...
});
```

`new Promise()` 메서드를 호출할 때 콜백 함수를 선언할 수 있고, 콜백 함수의 인자는 `resolve`, `reject`이다.

```js
new Promise(function (resolve, reject) {
  resolve();
});
```

여기서 콜백 함수의 인자 `resolve`를 위와 같이 실행하면 **이행(Fulfilled) 상태**가 된다.

```js
function getData() {
  return new Promise(function (resolve, reject) {
    var data = 100;
    resolve(data);
  });
}

// resolve()의 결과 값 data를 resolvedData로 받음
getData().then(function (resolvedData) {
  console.log(resolvedData); // 100
});
```

그리고 이행 상태가 되면 아래와 같이 `then()`을 이용하여 처리 **결과 값**을 받을 수 있다.

```js
function getData() {
  return new Promise(function (resolve, reject) {
    reject(new Error("Request is failed"));
  });
}

// reject()의 결과 값 Error를 err에 받음
getData()
  .then()
  .catch(function (err) {
    console.log(err); // Error: Request is failed
  });
```

그리고, 실패 상태가 되면 실패한 이유(실패 처리의 결과 값)를 `catch()` 로 받을 수 있다.

> ![](https://images.velog.io/images/rhfovk/post/cb81f762-adbb-4153-808d-db0a0ad5a1a6/image.png)프로미스 처리 흐름 - 출처 : MDN

<br>

### 출처 📚

---

도서 코어자바스크립트

https://joshua1988.github.io/web-development/javascript/promise-for-beginners/

https://medium.com/@la.place/async-await%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94%EA%B0%80-fa08a3157647
