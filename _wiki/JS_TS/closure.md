---
layout  : wiki
title   : JavaScript 클로저 (Closure) 이해하기 
summary : 
date    : 2023-12-09 22:14:31 +0900
updated : 2023-12-10 23:48:05 +0900
tag     : js 
resource: 3B/7E6078-6F7B-4335-A9ED-1669C35C5348
toc     : true
public  : true
parent  : [[/JS_TS]] 
latex   : false
---
* TOC
{:toc}

## 클로저란?

> 클로저는 함수와 그 함수가 선언된 렉시컬 환경과의 조합이다. [^1]

MDN에서 제시하는 예시 코드를 한 번 살펴보자.

```
function init() {
  var name = "Mozilla"; // name은 init에 의해 생성된 지역 변수
  function displayName() {
    // displayName()은 내부 함수이며, 클로저
    console.log(name); // 부모 함수에서 선언된 변수를 사용
  }
  displayName();
}
init();
```

init()은 지역 변수 name과 함수 displayName()을 생성을 하고, displayName()은 init() 안에 정의된 내부 함수로 init() 함수 본문에서만 사용할 수 있다.

displayName() 내부엔 자신만의 지역 변수가 없다. 그러나, 내부 함수에서 외부 함수의 변수에 접근할 수 있기 때문에, displayName() 역시 부모 함수 init()에서 선언된 변수 name에 접근할 수 있다.

이처럼 클로저는 어떤 함수(outer) 내부에 선언된 함수(inner)가 바깥 함수(outer)의 지역변수(outerVariable)를 참조하는 것이 함수(outer)가 종료 후에도 유지되는 현상이다.

### 원인 

클로저가 발생하는 원인은 JS의 스코프와 관련이 있다.

JS는 정적 스코프(렉시컬 스코프)이기 때문에 함수는 생성되자마자 상위 스코프가 결정된다. 이는 언제나 상위 스코프를 알 수 있게 된다는 것이다. 

JS에서 함수를 호출하면 크게 다음과 같은 단계를 거치게 된다.

```
함수 호출 -> 실행 컨텍스트 생성 -> 실행 컨텍스트 스택에 푸시 -> 렉시컬 환경 생성
```

호출된 함수의 실행 컨텍스트를 생성하고 이를 실행 컨텍스트 스택에 push한다.
 
렉시컬 환경을 생성한다는 것은 함수 본인 내부의 식별자 그리고 식별자의 바인딩 등 값 등을 기록하고 있는 하나의 자료 구조를 말한다.

그리고 코드의 실행이 끝나면 해당 컨텍스트에서 pop하여 제거한다.

클로저는 본인의 상위 스코프에서 현재 참조하는 식별자만을 기억하고, 하나의 state가 의도치 않게 변경되지 않도록 state를 안전하게 은닉할 수 있다.

### 스코프 체인

```
// 전역 스코프
var globalVariable = 'I am global';

function outerFunction() {
  // outerFunction 스코프
  var outerVariable = 'I am outer';

  function innerFunction() {
    // innerFunction 스코프
    var innerVariable = 'I am inner';

    console.log(globalVariable);   // 전역 변수에 접근 가능
    console.log(outerVariable);    // 외부 함수의 변수에 접근 가능
    console.log(innerVariable);    // 현재 함수의 변수에 접근 가능
  }

  innerFunction();
}
outerFunction();
```

innerFunction은 세 개의 스코프를 가지고 있다.

먼저, 자신의 스코프에 있는 innerVariable에 직접 접근할 수 있고, 외부 함수의 스코프에 있는 outerVariable에 접근할 수 있다. 마지막으로, 전역 스코프에 있는 globalVariable에도 접근할 수 있다.

스코프 체인은 변수 또는 함수를 찾을 때, 현재 스코프부터 시작하여 상위 스코프로 이동하면서 검색을 진행하고, 만약 변수나 함수를 찾지 못하면 에러가 발생한다.

종합하면 JS 엔진은 스코프 안에 참조하는 식별자 정보가 없다면 함수 평가 전에 수집했던 바로 바깥 스코프로 가서 식별자를 찾고, 바로 바깥 스코프에도 찾는 식별자가 없다면, 그 다음 스코프로 가서 찾고, 마지막엔 전역 스코프까지 가서 찾는데 이때도 존재하지 않는다면 참조에러를 발생시킨다고 요약할 수 있다.

이렇게 스코프가 체인처럼 연결 되어있는 것을 스코프체인이라고 한다.

자 이제 처음에 봤던 코드를 다시 보자.

```
function init() {
  var name = "Mozilla"; 
  function displayName() {
    console.log(name);
  }
  displayName();
}
init();
```

displayName 함수 내에서 console.log(name);이 실행될 때, name 변수를 참조하고 있다.

이때 displayName 함수는 자신이 정의된 스코프 외부에 있는 init 함수의 스코프를 기억하고 있어야 한다. 이렇게 외부 함수의 변수에 접근할 수 있는 함수가 클로저가 된다.

displayName 함수가 init 함수 내에서 선언되었으므로 displayName 함수는 init 함수의 스코프를 기억하며, name 변수에 접근할 수 있게 된다.

결과적으로, init 함수가 호출되면 displayName 함수가 실행되면서 console.log(name);이 실행되고, 출력 결과로 "Mozilla"가 나온다.

이때의 클로저는 displayName 함수가 init 함수의 변수에 접근할 수 있는 기능을 제공한다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 모던 자바스크립트 Deep Dive - 24장 클로저
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures - MDN

## 주석

[^1]: A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment)
