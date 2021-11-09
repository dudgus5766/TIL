## Immutable Object

---

<br>

> 도서 **코어자바스크립트**를 읽고 준비한 발표자료를 블로그 글로 정리한다!

### 불변 객체를 만드는 간단한 방법

1. **불변 객체란?**

- 불변 객체란 어떤 객체 내부의 프로퍼티들을 변경할 수 없도록 되어 있는 객체를 일컫는다.

2. **불변 객체가 필요한 이유**

- 값으로 전달받은 객체에 변경을 가하더라도 원본 객체는 변하지 않아야 하는 경우

  - 문제상황

  ```js
  let people1 = {
  		name : "A",
  		gender : "male",
  		age : 10,
  		... // 더 많은 프로퍼티가 있다고 가정
  }
  ```

  예를 들어 위와 같이 복잡한 데이터 구조를 가지는 people 객체가 존재한다.
  그리고 이때 이름만 다르고 나머지 속성들은 모두 같은 people의 복제 객체가 3개가 더 필요하다면?
  만약, 기존 객체를 이용하여 이름만 바꾸게 된다면?

  ```js
  const changeName = function (people, newName) {
    let newPeople = people;
    newPeople.name = newName;
    return newPeople;
  };

  let people2 = changeName(people1, "B");
  let people3 = changeName(people1, "C");

  console.log(people1.name); // C
  console.log(people2.name); // C
  console.log(people3.name); // C
  ```

  people1~3까지 모든 변수는 같은 주솟값을 바라보고, 그 주솟값 중 name 프로퍼티가 가리키는 데이터를 변경했기 때문에, **모든 변수에 담긴 name 프로퍼티 값이 변경**된다.

  - 문제 해결
    **문제 해결을 위해서는 윈본 객체 내부의 프로퍼티들을 복사한 새로운 객체를 생성하여 변수에 할당해주어야 한다.**

  ```js
  const changeNum = function(people, newName) {
  	return {
  			name : newName,
  			gender : people.gender,
  			age : people.age,
  			key : people ... // 나머지 프로퍼티 등
  	}
  }

  people2 = changeName(people1, "B");
  people3 = changeName(people1, "c");

  console.log(people1.name); // A
  console.log(people2.name); // B
  console.log(people3.name); // C
  ```

  → 이러한 하드 코딩이 수고로우니 모든 프로퍼티를 복사하는 함수를 만드는 편이 낫겠다!

  ```js
  const copyObject = function (target) {
    const newObject = {};

    for (let prop in target) {
      newObject[prop] = target[prop];
    }

    return newObject;
  };
  ```

  또한 이렇게 불변 객체를 만들어 변경을 방지하고, 변경이 필요한 경우에는 **방어적 복사**를 통해 새로운 객체를 생성한 후 기존 참조를 새로운 객체에 대한 참조로 완전히 갈아끼워주는 방식을 사용할 수 있다.

  - **Object.assign**
  - **Spread operator**
  - '**Immutable.js**' 라이브러리

3. **우리가 쓰는 리액트에서 왜 참조형 데이터의 가변성이 문제가 될까?**

- 만약 A에서 A2로 객체를 변경한다고 다시 생각해본다면, 리액트에서는 A에서 A2로 무언가 값이 변했다는 걸 파악한 후 다시 렌더링을 하게 된다.
- 그런데 위의 경우처럼 같은 주소값을 바라보고 있는 A와 A2에 대해서는 그런 변화를 감지할 수 없게 된다. 즉, 기존의 데이터(A)와 새로운 데이터(A2)가 같이 바뀌게 되니 기존의 데이터가 사라졌다고 할 수 있는 것이다.

---

<br>

### 얕은 복사와 깊은 복사

- 얇은 복사 (shallow copy) : 바로 아래 단계의 값만 복사하는 방법
- 깊은 복사 (deep copy) : 내부의 모든 값들을 하나하나 찾아서 전부 복사하는 방법

1. **중첩된 객체에 대한 깊은 복사 예시 1**

```jsx
const copyObject = function (target) {
  let result = {};
  for (const prop in target) {
    result[prop] = target[prop];
  }
  return result;
};

let user = {
  name: "younghyun",
  urls: {
    github: "http://github.com/younghyun",
    blog: "http://velog.io/@younghyun",
  },
};

let user2 = copyObject(user);
user2.urls = copyObject(user.urls);

user2.name = "jaewon";
console.log(user.name === user2.name); // false

user.urls.blog = "https://medium.com/@jaewon";
console.log(user.urls.blog === user2.urls.blog); // false

user2.urls.github = "";
console.log(user.urls.github === user2.urls.github); // false
```

- user만 복사하는 것이 아니라 urls도 별도로 복사

2. **객체의 깊은 복사를 수행하는 범용 함수 예시**

```jsx
const copyObjectDeep = function (target) {
  let result = {};
  if (typeof target === "object" && target !== null) {
    for (const prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
};

let user = {
  name: "younghyun",
  urls: {
    github: "http://github.com/younghyun",
    blog: ["http://velog.io/@younghyun", "https://medium.com/@younghyun"],
  },
};

let user2 = copyObjectDeep(user);

user2.name = "jaewon";
console.log(user.name === user2.name); // false

user2.urls.blog[1] = "https://steemit.com/@jaewon";
console.log(user.urls.blog[1] === user2.urls.blog[1]); // false

user2.urls.github = "";
console.log(user.urls.linkedin === user2.urls.linkedin); // false

console.log(user);
/*
{
	name: 'younghyun',
  urls: {
    github: 'http://github.com/younghyun',
    blog: ['http://velog.io/@younghyun','https://medium.com/@younghyun']
  }
}
*/

console.log(user2);
/*
{
	name: 'jaewon',
  urls: {
    github: '',
    blog: {
      '0': 'http://velog.io/@younghyun',
      '1': 'https://steemit.com/@jaewon'
    }
  }
*/
```

- 재귀적으로 함수를 호출하며 내부의 모든 객체들을 복사

3. **JSON을 활용한 간단한 깊은 복사 예시**

```jsx
const copyObjectViaJson = (target) => {
  return JSON.parse(JSON.stringify(target));
};
// ... 중략
```

- 객체를 JSON 문법으로 표현 된 문자열로 전환했다가 다시 JSON 객체로 전환하면 새로운 객체가 생성
- **다만 숨겨진 프로퍼티인 proto 와 getter/setter 등과 같이 JSON으로 변경할 수 없는 프로퍼티는 무시 됨**
- httpRequest로 받은 데이터를 저장한 객체를 복사할 때 등 순수한 정보만 다룰 때 유용
- Function은 복사가 안됨

<br>

### 출처 📚

---

도서 코어자바스크립트
https://galid1.tistory.com/663
