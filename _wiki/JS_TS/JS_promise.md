---
layout  : wiki
title   : JavaScript promise 객체
summary : 
date    : 2023-04-18 23:04:46 +0900
updated : 2023-04-18 23:27:37 +0900
tag     : js
resource: 0A/4E8DEA-F57C-4550-8D48-83E179AC3665
toc     : true
public  : true
parent  : [[/JS_TS]]
latex   : false
---
* TOC
{:toc}

## Promise 객체

> 비동기 작업이 맞이할 미래의 완료 또는 실패와 그 결과 값을 나타낸다.

- Javascript는 특정 코드의 연산이 끝날 때까지 코드의 실행을 멈추지 않고 다음 코드를 먼저 실행하는 **비동기 연산**을 한다.
- promise는 비동기 작업이 완료되었을 때 미래의 값을 나타내는 대리인 객체로, 작업이 완료되면 해당 값 또는 에러를 처리할 수 있다.
- 세 가지 상태(states)를 가진다.
    - 대기(pending): 초기 상태로, 비동기 작업이 아직 완료되지 않은 상태
    - 이행(fulfilled): 비동기 작업이 성공적으로 완료되어 결과 값을 가진 상태
    - 거부(rejected): 비동기 작업이 실패하거나 에러가 발생한 상태

## 작동 흐름 및 코드 구현

![alt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/promises.png)

```js
// pending
new Promise(resolve, reject);
```
new Promise() 메서드를 호출하면 대기(Pending) 상태가 되며,

new Promise() 메서드를 호출 시 callback 함수를 선언할 수 있고, 콜백 함수의 인자는 resolve, reject이다.

```js
// fulfilled
new Promise(function(resolve, reject) {
  resolve();
});

function getData() {
  return new Promise(function(resolve, reject) {
    var data = 100;
    resolve(data);
  });
}

// resolve()의 결과 값 data를 resolvedData로 받음
getData().then(function(resolvedData) {
  console.log(resolvedData); // 100
});
```

callback 함수의 인자 resolve를 실행하면 이행(Fulfilled) 상태가 된다.

이행 상태가 되면 아래와 같이 then()을 이용하여 처리 결과 값을 받을 수 있다.

```js
// reject
new Promise(function(resolve, reject) {
  reject();
});

function getData() {
  return new Promise(function(resolve, reject) {
    reject(new Error("Request is failed"));
  });
}

// reject()의 결과 값 Error를 err에 받음
getData().then().catch(function(err) {
  console.log(err); // Error: Request is failed
});
```

new Promise()로 프로미스 객체 생성 시, callback 함수 인자로 resolve와 reject를 사용할 수 있다.
reject를 호출 시 실패(Rejected) 상태가 되고 ,실패 상태가 되면 실패한 이유(실패 처리의 결과 값)를 catch()로 받을 수 있다.

### 응용

```js
// Promise 객체 생성
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const randomNum = Math.random();
    if (randomNum > 0.5) {
      resolve(`성공: ${randomNum}`); // 성공 시 resolve 호출
    } else {
      reject(`실패: ${randomNum}`); // 실패 시 reject 호출
    }
  }, 1000);
});

// Promise 사용
myPromise.then((result) => {
  console.log(result); // 이행된 경우 출력
}).catch((error) => {
  console.error(error); // 거부된 경우 출력
});
```

myPromise는 1초 뒤에 무작위로 성공 또는 실패하는 비동기 작업을 수행하는 Promise 객체이다.

then 메소드는 Promise가 이행되었을 때 호출되는 callback 함수를 등록하는데,
catch 메소드는 Promise가 거부되었을 때 호출되는 콜백 함수를 등록한다.

이러한 방법을 통해 비동기 작업의 결과 값을 처리할 수 있다.

## 참고자료
- https://joshua1988.github.io/web-development/javascript/promise-for-beginners/ - 캡틴판교님의 글
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise
