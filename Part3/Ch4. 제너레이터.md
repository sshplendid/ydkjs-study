# 제너레이터

Callback의 단점은 다음과 같다.

- 머릿속으로 계획하는 작업의 단계와 맞지 않는다.
- 제어의 역전 문제로 신뢰성이 떨어진다.

⇒ Promise는 이러한 제어의 역전 문제를 해결하였다.

⇒ Promise가 Callback 지옥까지 해결한것은 아니다.

제너레이터까지 사용하면 비동기 흐름을 제어함에 있어 순차적/동기적으로 표현할 수 있다. 

Generator에 대해 알아보자.

## 4.1 완전-실행을 타파하다

---

Javascript는 기본적으로 함수를 실행하면 완료될때까지 중간에 다른 코드가 끼어들지 못한다.

하지만 Generator는 다르다.

```jsx
var x = 1;

function foo() {
	x++;
	bar();
	console.log("x: " + x);
}

function bar() {
	x++;
}

foo();
bar();
```

위 코드에서 foo 함수가 실행되는 중간에 bar가 실행되지 않는다.

⇒ 함수의 처음과 끝 사이에 다른 외부 작업이 간섭하지 않는다는 것을 보장할 수 있다. 

⇒ 완전-실행

선점형 언어? 멀티쓰레드 언어?

선점이란 어떤 작업을 실행하다가 나중에 다시 실행할 의도를 갖고 강제로 멈추게 하는것이다. 
우선순위가 높은 작업을 우선 처리하는것으로, 작업이 전환되는것을 Context Change 라고 한다.

선점 ↔협동

선점은 협동할 생각이 없는, 특권을 가진 독재자가 마음대로 실행중인 작업을 다른 작업으로 갈아 치우는 등의 스케줄링을 하는것이다. 

협동은 작업들 사이에 시스템 자원을 쓰지 않을때 다른 작업이 사용할 수 있게 제어권을 넘긴다. 

ES6를 통해 협동적 동시성을 갖춘 예제를 보자

```jsx
var x = 1;

function *foo() {
	x++;
	yield;
	console.log("x: " + x);
}

function bar() {
	x++;
}

var it = foo();
it.next();
x; // 2

bar();
x; // 3

it.next();  // x: 3
```

1. foo(); 를 호출하면 foo함수가 실행된게 아니라 제네레이터를 제어할 이터레이터를 반환한다.
2. it.next() 호출시 foo()가 실행되며 yield 전까지 실행되고 멈춘다.
3. bar() 호출시 x값이 1 증가한다.
4. it.next()를 다시 호출하면 yield에서 다시 실행되어 x를 출력하도 끝난다. 

제너레이터에 를 function에 붙일지, name에 붙일지는 컨벤션에 문제이다.
다만  *foo 를 추천하는데, 이유는 제너레이터의 위임 때문이다.
위임은 다음과 같이 사용하기 때문이다. `yield *foo()`

- function* foo(); 
- function *foo();

### 4.1.1 입력과 출력

제너레이터는 새로운 처리 모델에 기반을 둔 특별한 함수이다.

제네레이터를 실행하면 함수가 실행되는것이 아니라 이터레이터가 반환된다. 

next의 결괏값은 *foo() 가 반환한 값을 value 속성에 저장한 객체이다. 

### 반복 메시징

yield와 next를 통해서 서로 인자를 주고받을 수 있다. 

```jsx
function *foo() {
	var y = x * (yield "hello");
	return y;
}

var it = foo(6);
var res = it.next();
res.value;  // hello

res = it.next(7);
res.value; // 42
```

yield 의 다음값을 next() 호출의 value 값으로 전달하며

next의 인자를 멈췄던 yield 자리에 치환한다. 

### 4.1.2 다중 이터레이터

제너레이터는 각각 독립적으로 사용 가능하다.

자세한건 예제 참조

### 인터리빙

제너레이터를 통해서 완전-실행을 보장할 수 없게 되었다. 

next의 호출 순서에 따라서 공유자원이 어떻게 바뀔지 알 수 없게 되었다. 

이렇게 해서 얻는 이득이 있을까?
오히려 손해인것같은데, 단순 개념 이해하는 정도로는 괜찮을 순 있겠지만...

## 4.2 값을 제너레이팅

---

제너레이트는 뭔가 만들어내는 장치 라는 의미로 유래된 것이다. 

### 4.2.1 제조기와 이터레이터

클로저를 구현하여 제너레이터를 따라할 수 있다.

```jsx
var something = (function() {
	var nextVal;

	return {
		[Symbol.iterator]: function() { return this; },
		next: function() {
			if (nextVal === undefined) {
				nextVal = 1;
			} else {
				nextVal = 3 * nextVal + 6;
			}

			return {
				done: false,
				value: nextVal
			}
		}
	}
});
```

```jsx
for (var v of something) {
	console.log(v);  // 1, 9, 33, 105 .....
}
```

Symbol.iterator 를 정의해주었으므로 for...of 를 사용할 수 있다. (무한루프 주의)

### 4.4.2 이터러블

next() 메서드로 인터페이스하는 객체를 이터레이터 라고 한다. 

순환 가능한 이터레이터를 포함한 객체.

ES6 이후부턴 Symbol.iterator 을 사용하여 정의하고 있다. 

for...of 루프는 Symbol.iterator를 사용하여 이터레이터를 생성한다. 

### 4.4.3 제너레이터 이터레이터

제네레이터는 next 함수를 통해 값을 찍어내는 공장이라고 생각해도 될듯 하다.

```jsx
for (var v of something()) {
	console.log(v);
}
```

- something() 을 호출해야만 이터레이터가 생성된다.
- 제너레이터의 이터레이터도 이터레이터다.

### 제너레이터 멈춤

제너레이터의 이터레이터를 통해 종료시킬 수 있다. 

```jsx
function *something() {
	try {
		var nextVal; 

		while (true) {
			if () {
				...
			} 

			yield nextVal;
		}

	} finally {
		console.log('종료');
	}
}
```

위의 제너레이터가 있을때 it.return() 을 통해서 종료시킬 수 있다. 

비정상정인 신호에 의해서도 finally를 통해 종료될 수 있다. 

## 4.3 제너레이터를 비동기적으로 순회

---

이전 비동기 콜백식 코드를 제너레이터를 사용해 동기적으로 바꿔보자

```jsx
function foo(x, y) {
	ajax('http://~~~~.com/api/~~~', function (error, data) {
		if (error) {
			it.throw(error);
		} else {
			it.next(data);
		}
	})
}

function *main() {
	try {
		var response = yield foo(1, 2);
		console.log(response);
	} catch(error) {
		console.error(error);
	}
}

var it = main();
it.next();
```

yield를 통해 동기적으로 비동기를 표현할 수 있다.

여기서 yield를 메시지 전달 수단이 아니라 멈춤 / 중단을 위한 흐름 제어 수단으로 사용한 것이다. 

⇒ 비동기성을 순차적, 동기적 방향으로 표현할 수 있게 되었다.

### 4.3.1 동기적 에러 처리

위의 코드는 it.throw를 통해서 catch에서 받아서 처리 가능하게한다. 

비동기 코드에서 에러를 동기적인 모양새로 처리할 수 있어서 코드 가독성, 추론성 면에서 큰 장점이다.

Promise에선 try ~ catch로 에러를 잡을 수 없는데, 이를 해결해준다. 

```jsx
function *main() {
	var x = yield "Hello World";

	yield x.toLowerCase();
}

var it = main();
it.next().value;  // "Hello World"

try {
	it.next(42);
  // or it.throw('Error!');
} catch(error) {
	console.error(error);
}
```

위와 같이 예외처리가 가능하고, 이터레이터의 throw를 이용해 처리할 수도 있다.

## 4.4 제너레이터 + 프라미스

---

이제 프라미스의 믿음성과 조합해보자.

```jsx
function foo(x, y) {
	return request('http://abc.com');
}

foo(1, 2).then(
	function(response) {
		// 성공
	},
	function(error) {
		// Error 처리
	}
)
```

위의 일반적인 Promise 기반의 비동기 통신 예제가 있다.

여기에 이터레이터는 Promise가 귀결되길 기다리고 있다가 resolve나 reject 되었을때 처리하도록 한다. 

```jsx
function foo(x, y) {
	return request('http://abc.com');
}

function *main() {
	try {
		var text = yield foo(1, 2);
		console.log(text);
	} catch(error) {
		console.error(error);
	}
}

var it = main();
it.next().value;

p.then()
```

위와 같이 제너레이터 함수로 한번 감싸서 비동기처리할 수 있다.

### 4.4.1 Promise-인식형 제너레이터 실행기

```jsx
코드는 322p ~ 323p 참조
```

위 코드는 제너레이터를 인자로 받아 완전이 끝날때까지 실행시켜주는 실행기이다. 

꽤 유용하게 사용된다.

### ES7: async와 await

ES7에 추가되는 async와 await을 통해 async 함수가 완료될때까지 기다린다. 

따라서 4.4.1에서 사용했던 라이브러리는 사용할 필요가 없어진다.

### 4.4.2 제너레이터에서의 프라미스 동시성

비동기적 작업을 하나만 사용한다면 큰 문제 없겠지만 여러 요청을 어떠한 순서에 맞게 보낼지에 따라 성능을 최적화 할 수 있다.

```jsx
function *foo() {
	var r1 = yield request('http://request.com/1');
	var r2 = yield request('http://request.com/2');
	var r3 = yield request('http://request.com/' + r1 + r2);

	console.log(r3);
}

run(foo);
```

위 코드는 r1을 끝낸 이후에, r2를 보내고, r2를 끝낸 뒤 r3를 보낸다. 

즉, 동시에 작업을 처리하는것이 아닌 하나씩 동기적으로 처리하는것이다. 

성능에 최적화 되지 않으므로 다음과 같이 짤 수 있다.

```jsx
function *foo() {
	var p1 = request('http://request.com/1');
	var p2 = yield request('http://request.com/2');

	var r1 = yield p1;
	var r2 = yield p2;
	var r3 = yield request('http://request.com/' + r1 + r2);

	console.log(r3);
}

run(foo);
```

이렇게 하면 r1과 r2를 동시에 요청하고 각각 응답이 왔을때 r3를 요청할 것이다.

Promise.all[...] 과 동일한 효과를 줄 수 있다.

### Promise 숨김

내부에 얼마만큼의 Promise로직을 넣을지 신중히 판단해야 한다.

제네레이터에선 Promise가 얼마나 있는지, 세부 로직은 숨기는게 가독성면에서 더 좋다.

추론성과 유지보수적으로도 더 좋은 코드이다.

세부 로직(Promise) 과 제네레이터 코드를 분리
⇒ 관심사 분리

## 4.5 제너레이터 위임

---

```jsx
function *foo() {
	console.log("*foo 시작");
	yield 3;
	yield 4;
	console.log("*foo 끝");
}

function *bar() {
	yield 1;
	yield 2;
	yield *foo();
	yield 5;
}

var it = bar();

it.next().value; // 1
it.next().value; // 2
it.next().value; // foo 시작 3
it.next().value; // 4
it.next().value; // foo 끝  5
```

위와 같이 위임할 수 있다. `yield *foo();`에서 bar에서의 제어권이 foo로 이동하게 된다.

### 4.5.1 왜 위임을?

위임의 목적은 주로 코드를 조직화 하고, 그렇게 해서 일반 함수 호출과 맞추기 위함이다. 

즉, 의미상으로 함수를 만들기 위함인데, 다음과 같은 장점이 있다.

- 가독성
- 유지보수성
- 디버깅

### 4.5.2 메시지 위임

양방향 메시징도 동일하게 동작한다.

```jsx
function *foo() {
	console.log("*foo 내부", yield "B");
	console.log("*foo 내부", yield "C");

	return "D";
}

function *bar() {
	console.log("*bar 내부", yield "A");
	console.log("*bar 내부", yield *foo());
	console.log("*bar 내부", yield "E");

	return "F"
}

var it.bar();

it.next().value;
it.next(1).value;
it.next(2).value;
it.next(3).value;
it.next(4).value;
```

외부 이터레이터 it는 원래의 제너레이터나 위임 제너레이터나 다른점이 없다. 

즉, 이터레이터는 제너레이터를 가리키는것이 아니라 해당 지점을 가리키는 것이다. 

제러에티터가 아닌 일반 이터러블에도 동일하게 동작된다.

예외도 위임된다.

### 4.5.3 비동기성을 위임

```jsx
function *foo() {
	var r2 = yield request("");
	var r3 = yield request("");

	return r3;
}

function *bar() {
	var r1 = yield request("");
	var r3 = yield *foo();

	console.log(r3);
}

run(bar);
```

위와 같이 비동기성을 위임하여 처리할 수도 있다. 

### 4.5.4 위임 재귀

```jsx
function *foo(val) {
	if(val > 1) {
		val = yield *foo(val - 1);
	}

	return yield request('http://some.com/' + val);
}

function *bar() {
	var r1 = yield *foo(3);

	console.log(r1);
}

run(bar);
```

## 4.6 제너레이터 동시성

---

```jsx
function response(data) {
	if (data.url === 'https://some.com/1') {
		res[0] = data;
	} else if (data.url === 'https://some.com/2') {
		res[1] = data;
	}
}
```

위와 같은 코드를 제너레이터로 바꾸면 아래와 같은 형태가 될것이다. 

```jsx
var res = [];

function *reqData(data) {
	res.push(yield request(url))
}
```

수동으로 res[0], res[1] 과 같이 넣어주는것보단 res.push로 적절하게 들어가는게 더 좋다. 

만약 각각이 서로 상호작용이 있다고 하면 어떻게 해야할까?

```jsx
var it1 = reqData("http://some.com/1");
var it2 = reqData("http://some.com/2");

var p1 = it1.next();
var p2 = it2.next();

p1.then(function () {
	it1.next(data);
	return p2;
}).then(function (data) {
	it2.next(data);
})
```

위와같이 하면 각각 ajax요청 후 yield에서 멈추고, res 배열에 순서에 맞춰 잘 들어갈 것이다. 

하지만 손이 너무 많이간다.

이를 개선하면 다음과 같아진다. 

```jsx
var res = [];

function *reqData(url) {
	var data = yield request(url);
	yield;
	res.push(data);
}

var it1 = reqData("http://some.com/1");
var it2 = reqData("http://some.com/2");

var p1 = it1.next();
var p2 = it2.next();

p1.then(function(data) {
	it1.next(data);
});

p2.then(function(data) {
	it2.next(data);
});

Promise.all([p1, p2]).then(function () {
	it1.next();
	it2.next();
})

```

이렇게 reqData에서 한번 기다리고 처리하면 가능해진다.

## 4.7 썽크

---

> 썽크란 다른 함수를 호출할 운명을 가진 인자가 없는 함수이다. 

즉, 어떤 함수 정의부를 또 다른 함수 호출부로 감싸 실행을 지연시키는데, 여기서 감싼 함수가 썽크이다.

```jsx
function foo(x, y) {
	return x + y;
}

function fooThunk() {
	return foo(3, 4);
}

console.log(fooThunk());  // 7
```

위와 같이 한번 감싼 함수를 썽크라 한다. 

```jsx
function thunkify(fn) {
	var args = [].slice.call(arguments, 1);
	return function(cb) {
		args(cb);
		return fn.apply(null, args);
	}
}

var fooThunk = thunkify(foo, 3, 4);

fooThunk(function(sum) {
	console.log(sum);
});
```

위와 같이 thunkify의 유틸리티 함수를 만들수도 있다.

### 4.7.1 s / promise / thunk /

제너레이터와 썽크는 직접적인 연관은 없다. 

하지만 어떤 값을 요청하면 비동기적 응답을 받는다는 점은 같다. 

```jsx
var fooThunkory = thunkify(foo);
var fooPromisory = promisify(foo);

var fooThunk = fooThunkory(3, 4);
var fooPromise = fooPromisory(3, 4);

fooThunk(function(error, sum)) {
	...
});

fooPromise.then(
	function (sum) {

	}, function(error) {

	}
);
```

위에서 보이는것처럼 꽤 비슷한 형태를 띄고있다. 

어떤 값을 물어보면, 그에따른 미래값을 받아낸다. 

이런 관점에서 제너레이터는 비동기성 프라미스 대신 비동기성 썽크를 yield 해도 된다.

....?? 해도 됨?

기존에 run 메소드를 조금 건들여서 썽크까지 지원하도록 한다.

## 4.8 ES6 이전 제너레이터

---

브라우저에서 직접적으로 제너레이터를 쓰기엔 부담이 있다. 

모든 브라우저가 ES6를 지원하진 않기 때문이다. 

따라서 새로 확장된 구문중 트랜스파일러를 통해 ES6구문을 ES5 이하에서 동작하도록 바꿔야 한다.

어떤식으로 yield를 바꾸는지 알아보자

### 4.8.1 수동 변환

```jsx
function *foo(url) {
	try {
		console.log("요청중");

		var val = yield request(url);

		console.log(val);
	} catch(error) {
		console.error(error);

		return false;
	}
}

var it = foo("http://some.com/1");
it.next();
```

위의 코드를 바꾸면 어떻게 될까?

```jsx
function foo(url) {
	// ... 
	var state;// 제너레이터의 상태를 관리

	var val;  // 제너레이터의 스코프 변수 선언

	function process(v) {
		switch(state) {
			case 1: 
				console.log("요청중");
				return request(url);
			case 2: 
				val =v;
				console.log(v);
				return;
			case 3:
				var error = v;
				console.error(error);
				return false;
		}
	}

	return {
		next: function(v) {
			if (!state) {
				// 초기 상태인 경우
				state = 1;
				return {
					done: false,
					value: process();
				}
			} else if (state === 1) {
				state = 2;
				return {
					done: false,
					value: process(v)
				}
			} else {
				return {
					done: true,
					value: undefined
				}
			}
		},
		throw: function(e) {
			if (state === 1) {
				state = 3;
				return {
					done: true,
					value: process(e)
				}
			} else {
				throw e;
			}
		}
	}
}

var it = foo("http://some.com/1");
it.next();
```

결과적으로 클로저를 사용해서 위와같이 바꿀 수 있었고, 천천히 위의 코드를 보면 이해될것이다. 

위와같이 수동으로 변환하면 과정이 아주 복잡하고, 다른 제너레이터를 이식하기도 매우 힘들기 때문에 손으로 변환하는건 비현실적이다. 

### 4.8.2 자동 변환

똑똑하신 분들이 잘 만들어놓은 툴을 사용하여 변환할 수 있다. 

자세한 코드는 367p 참조.

이러한 툴 덕분에 제너레이터가 ES6+ 환경만 쓸모있는 장치는 아니다.

다른 브라우저 환경에서도 동작하니, 호환성 도구를 사용하여 즐기자.

## 4.9 정리하기

- 제너레이터는 ES6에서 도입된 새로운 함수이다.
- 일반 함수처럼 완전-실행 하지 않고, 도중에 멈추고 멈춘 시점에서 다시 실행할 수 있게 한다.
- 선점형이 아닌 협동적인 도구이다.  yield 키워드를 사용해 멈추고 이터레이터를 통해 재개할 수 있다.
- yield, next를 통해 서로 양방향 메시징이 가능하다.
- 비동기 흐름 제어에서 제너레이터는 동기적 순차적 형태로 작업 단계를 표현할 수 있게 해준다.
- 비동기 코드의 순차/동기/중단적 패턴을 유지함으로써 가독성 좋은 코드가 가능해졌다.
