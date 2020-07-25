# 3. 함수 vs 블록 스코프

- 스포프는 컨테이너 또는 바구니 구실을 하는 일련의 '버블'이다.
- 함수만 버블을 만들까? 자바스크립트의 다른 구조는 스코프 버블을 생성하지 못할까?

## 3.1. 함수 기반 스코프

- 자바 스크립트는 **함수 기반 스코프**를 사용
- 함수는 저마다의 버블을 생성
- 다른 어떤 자료구조도 자체적인 스코프를 생성하지 않음

→ ⚠️**사실이 아니다!!**

- 아래 코드에서 `foo()`의 스코프 버블은 `a`, `b`, `c`, `bar`를 포함한다.
- 선언문이 어디에 있는지는 중요하지 않다.
- 스코프 안에 있는 변수와 함수는 스코프 버블에 속한다.

```jsx
function foo(a) {
	var b = 2;
	// ...
	function bar() {
		// ...
	}
	// ...
	var c = 3;
}
```

- `bar()`는 자체 스코프 버블이 있다.
- 글로벌 스코프도 마찬가지다.
- 글로벌 스코프에는 하나의 함수, `foo`가 있다.
- `bar` 함수 스코프에서 a, b, c에 대해 접근 가능하다. (동일한 이름의 변수를 선언하지 않았을 경우)

## 3.2. 일반 스코프에 숨기

- 함수의 전통적 개념
    - 함수를 선언하고 그 안에 코드를 넣는다.
    - 작성한 코드에서 임의 부분을 함수 선언문으로 감싼다. (해당 코드를 숨기는 효과)

- 해당 코드 주위에 새로운 스코프 버블이 생성
- 감싸진 코드 안의 변수 혹은 함수는 새로운 함수 스코프에 묶임
- 왜 '숨기는' 테크닉(함수 스코프)을 사용할까?

> 최소 권한의 원칙
모듈/객체의 API와 같은 소프트웨어를 설계할 때 필요한 것만 최소한으로 남기고 나머지는 '숨겨야' 한다.

- 모든 변수와 함수가 글로벌 스코프에 존재한다면,
    - 어디서든 이들에게 접근할 수 있음
    - 접근할 필요가 없거나 비공개로 남겨둬야 할 변수와 함수를 노출하게 됨
    - → 사이드 이펙트 발생

```jsx
function doSomething(a) {
	b = a + doSomethingElse(a * 2);
	console.log(b * 3);
}

function doSomethingElse(a) {
	return a - 1;
}

var b;
doSomething(2); // 15
```

- 이 코드에서 `b`와 `doSomethingElse`는 가려도 무방함
- 사실, 접근하도록 내버려두는건 **위험함**
- 의도치않게 잘못된 방법으로 사용 가능

```jsx
// Good
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}
	
	var b;

	b = a + doSomethingElse(a * 2);
	console.log(b * 3);
}

doSomething(2); // 15
```

- `b`와 `doSomethingElse`는 외부에서 접근 불가
- 기능과 결과는 달라지지 않음

### 3.2.1. 충돌 회피

- 동일한 이름을 가진 객체가 충돌하는 것을 피할 수 있음

```jsx
function foo() {
	function bar(a) {
		i = 3;  // 이를 포함하는 스코프의 변수 i 값을 변경한다.
		console.log(a + i);
	}

	for(var i = 0; i < 10; i++) {
		bar(i * 2); // 무한 루프
	}
}

foo();
```

- 해결 방법? `var i = 3;`
- 혹은 변수 명을 변경

**글로벌 네임스페이스**

- 보통 라이브러리는 글로벌 스코프에 하나의 고유 이름을 가진 객체 선언문을 생성
- 고유 이름은 이후 라이브러리의 **네임스페이스**로 이용됨 (e.g. `$.ajax()`)

**모듈 관리**

- 의존성 관리자를 이용한 **모듈** 접근법이 존재함
- 글로벌 스코프를 추가할 필요 없이 의존성 관리자를 이용해서 명시적인 방법으로 가져와 사용 가능
    - [import](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/import), [export](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/export)

## 3.3. 스코프 역할을 하는 함수

```jsx
var a = 2;

function foo() {
	var a = 3;
	console.log(a); // 3
}

foo();

console.log(a); // 2
```

- foo라는 함수를 선언해야 함
- 그 함수를 직접 호출해야 함 → 함수를 선언하고 자동으로 실행한다면 이상적 ??

```jsx
var a = 3;

(function foo() {
	var a = 3;
	console.log(a); // 3
})();

console.log(a); // 2
```

- [즉시 실행 함수](https://developer.mozilla.org/ko/docs/Glossary/IIFE): 함수 표현식을 즉시 실행함
- 함수 선언문과 표현식의 차이: 구문의 시작위치에 `function`이 있으면 선언문, 다른 경우는 표현식
- 함수 이름 `foo`는 함수 자신의 내부 스코프에 묶임 ????
- 함수를 둘러싼 스코프를 오염시키지 않음

### 3.3.1. 익명 vs 가명

- 콜백 인자로 사용하는 함수 표현식 사례를 **익명 함수 표현식**이라고 부름
    - 함수의 이름이 없다
    - 빠르고 쉽게 입력할 수 있다??
    - 디버깅 어려움
    - 함수 재귀호출이 어려움
    - 함수 이름이 없어 코드 이해 어려움

```jsx
setTimeout(function timeoutHandler() {
	console.log('1초 기다림');
}, 1000);
```

- IIFE 사용 예: window 객체를 인자로 넘겨, 글로벌 참조와 함수 스코프 참조의 차이를 만듬

```jsx
var a = 2;

(function iife(global) {
	var a = 3;
	console.log(a); // 3
	console.log(global.a); //2
})(window);

console.log(a); // 2
```

## 3.4. 스코프 역할을 하는 블록

- 

### 3.4.1. with

- with문 안에서 생성된 객체는 바깥 스코프에 영향을 주지 않는다.

### 3.4.2. try ... catch

- catch 블록에서 선언된 변수는 catch 블록 스코프에 속한다.

### 3.4.3. let

- 변수를 선언하는 다른 방식
- 블록 스코프
- 호이스팅 효과 없음

**가비지 컬렉션**

- 블록 스코프는 메모리를 회수하기 위한 클로저, 가비지 컬렉션과 관련있음

```jsx
function process(data) {
	// do something with data
}

var bigData = { ... };
process(bigData);

var btn = document.querySelector('#myButton');
btn.addEventListener('click', function click(event) {
	console.log('button clicked');
}, /*capturing phase*/ false);
```

- click 함수는 `bigData` 변수가 필요없음
- 그러나 click함수가 해당 스코프의 클로저를 가지고 있기 때문에 엔진은 데이터를 남겨둠

```jsx
// Good
function process(data) {
	// do something with data
}

{ // bigData는 블록이 끝나면 사라짐
	let bigData = { ... };
	process(bigData);
}

var btn = document.querySelector('#myButton');
btn.addEventListener('click', function click(event) {
	console.log('button clicked');
}, /*capturing phase*/ false);
```

### 3.4.4. const

- 상수
- 블록 스코프

## 3.5. 정리하기

- 함수는 스코프를 이루는 가장 흔한 단위
- 블록 스코프는 임의의 코드 블록에 변수와 함수가 속하는 개념
- try ... catch 의 catch 부분을 블록 스코프를 가짐
- ES6부터 let, const 키워드가 추가되어 코드 블록 안에 변수를 선언할 수 있게됨
- 블록 스코프는 var 함수 스코프를 완전히 대체할 수 없음
    - 적절한 곳에 적절한 기술을 사용해야 함