# 스코트 클로저

## 요약

> `클로저`란 함수가 속한 렉시컬 스코프를 기억하며 함수가 렉시컬 스코프 밖에서 실행될 때에도 이 스코프에 접근할 수 있게 하는 기능을 뜻함

## 5.1 깨달음

---

...

## 5.2 핵심

---

**클로저 예제1**

```jsx
function foo() {
	var a = 2;

	function bar() {
		console.log(a);
	}

	bar();
}

foo();
```

위 예제는 단순 렉시컬 스코프 규칙에 따라 RHS 참조를 통해 a를 불러와 출력해줄 뿐이다.

클로저의 규칙중 일부이므로 클로저라고 할 순 있다.

bar() 는 foo() 스코프에 대한 클로저를 가진다.

스코프가 bar는 foo안에 중첩되어 있기 때문

**클로저 예제2**

```jsx
function foo() {
	var a = 2;

	function bar() {
		console.log(a);
	}

	return bar;
}

var baz = foo();
baz();  // 2
```

bar()는 foo()의 렉시컬 스코프에 접근할 수 있고, bar() 함수 자체를 외부에 공개한다.

baz를 통해 bar를 받고 이를 호출하면 foo의 렉시컬 스코프에 접근하여 a를 출력할 수 있게 된다.

foo 실행 후에 foo의 스코프는 사라졌다고 생각되지만 사실은 사라지지 않는다. 

bar() 가 사용중이기 때문이다.

**클로저 예제3**

```jsx
function foo() {
	var a = 2;
	function baz() {
		consoele.log(a);
	}

	bar(baz);
}

function bar(fn) {
	fn();
}
```

bar 함수에 baz 함수를 인자로 넘겼다.

baz 함수는 외부의 bar에서 호출되지만 여전히 foo 스코프를 볼 수 있다. (⇒ 클로저)

**클로저 예제4**

```jsx
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log(a);
	}

	fn = baz;
}

function bar() {
	fn();
}

foo();
bar();  // 2
```

마찬가지로 foo 함수를 실행하여 전역번수 fn에 baz 를 할당해 준다.

baz는 외부에서도 여전히 foo 스코프를 볼 수 있다. (⇒ 클로저)

⇒ 결론적으로 어떤 방식으로든 내부 함수를 자신이 속한 렉시컬 스코프 밖으로 수송하든 함수는 처음 선언된 곳의 스코프에 대한 참조를 유지한다는 것이다.

어디에서 해당 함수를 실행하든 클로저가 작용한다.

## 5.3 이제 나는 볼 수 있다.

---

**setTimeout 예제**

```jsx
function wait(message) {
	setTimeout(() => {
		console.log(message);
	});
}

wait('Hello, Closure!');
```

setTimeout 의 첫번째 인자 함수는 wait 스코프에 대한 클로저를 갖고 있으므로, message를 유지할 수 있다.

wait 함수가 끝난 이후엔 스코프가 사라져야 할 것 같지만 사라지지 않는 이유다.

**jQuery 예제**

```jsx
function setupBot(name, selector) {
	$(selector).click(function activator() {
		console.log('Activating: ' + name)
	});
}

setupBot('closure bot 1', '#bot_1');
setupBot('closure bot 2', '#bot_2');
```

click 이벤트 내부에 name 변수는 여전히 setupBot의 스코프를 보고있기 때문에 참조 가능하다.

**IIFE 패턴 예제**

```jsx
var a = 2;

(function IIFE() {
	console.log(a);
})();
```

이 예제는 엄격하게 얘기하면 클로저는 아니다.

IIFE 함수가 외부 스코프에서 동작한게 아니기 때문이다.

a 를 스코프 규칙에 의해서 가져옴

클로저는 기술적으론 선언할때 발생하지만, 바로 관찰할 수 있는것은 아니다.

## 5.4 반복문과 클로저

---

```jsx
for (var i = 1; i <= 5; i++) {
	setTimeout(function timer() {
		console.log(i);
	}, i * 1000);
}
```

- setTimeout은 for문이 끝나고 callback이 호출된다.
- i를 바라보고 있음

### 해결책

1. 새로운 스코프를 생성
2. 스코프에 새로운 변수를 추가

**해결책 1**

```jsx
for (var i = 1; i <= 5; i++) {
	(function() {
			var j = i;
			setTimeout(function timer() {
				console.log(j);
			}, i * 1000);
	})();
}
```

- 새로운 스코프를 생성하고, 값을 잡아둘 새로운 변수를 생성하여 사용

**해결책 2**

```jsx
for (var i = 1; i <= 5; i++) {
	(function(j) {
			setTimeout(function timer() {
				console.log(j);
			}, i * 1000);
	})(i);
}
```

- 새로운 스코프를 생성하고 인자값으로 넘겨줌

### 5.4.1 다시 보는 블록 스코프

**해결책 3**

```jsx
for (let i = 1; i <= 5; i++) {
	setTimeout(function timer() {
		console.log(i);
	}, i * 1000);
}
```

- let은 하나의 블록을 닫을 수 있는 스코프로 만든다.
- for 안에 let은 반복할때마다 새롭게 변수가 생성된다.

## 5.5 모듈

---

### 모듈 패턴

```jsx
function CoolModule() {
	var something = 'cool';
	var another = [1, 2, 3];

	function doSomething() {
		console.log(something);
	}

	function doAnother() {
		console.log(another.join(', '));
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	}
}

var coolModule = CoolModule();
coolModule.doSomething();
coolModule.doAnother();
```

- CoolModule은 함수일 뿐이지만 모듈 인스턴스를 생성하려면 반드시 호출해야 함.
- CoolModule은 객체를 반환한다. 이때 내장 데이터는 반환하지 않는다.

모듈 패턴을 사용하려면 다음 조건이 필수적인다.

1. 하나의 최외곽 함수가 존재, 이 함수가 최소 한번은 호출되어야 함
2. 최외곽 함수는 최소 한번은 하나의 내부 함수를 반환해야 한다. 
그래야 내부 함수가 비공개 스코프에 대한 클로저를 가져 비공개 상태에 접근하고 수정할 수 있다.

### 싱글톤

하나의 인스턴스를 갖는 싱글톤

```jsx
var foo = (function CoolModule() {
	... do something...
})();

foo.doSomething();
foo.doAnother();
```

IIFE로 즉시 실행하여 반환값을 하나의 모듈 인스턴스에 대입.

### 5.5.1 현재의 모듈

많은 모듈 의존성 로더와 관리자는 모듈페턴으로 감싸고 있다.

```jsx
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i = 0;; i++) {
			deps[i] = modules[deps[i]];
		}

		modules[name] = impl.apply(impl, deps);
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	}
})();
```

`modules[name] = impl.apply(impl, deps);` 이부분이 핵심이다.

모듈에 대한 정의 래퍼 함수를 호출하여 반환값인 모듈 API를 이름으로 정리된 내부 모듈 리스트에 저장한다.

```jsx
MyModules.define("bar", [], function () {
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	}
});

MyModules.define("foo", ["bar"], function () {
	var hungry = "hippo";

	function awesome(who) {
		console.log(bar.hello(hungry).toUpperCase());
	}

	return {
		awesome: awesome
	}
});
```

### 5.5.2 미래의 모듈

최신 문법을 추가했다.

ES6에서는 파일을 개별 모듈로 처리한다. 

ES6 모듈은 inline 형식을 지원하지 않고, 개별 파일에 정의되어야 함

브라우저와 엔진은 기본 모듈 로더를 가진다.

함수 기반 모듈은 정적으로 알려진 패턴이 아니다. 실제로 호출되기 전까진 해석되지 않는다.
반면, ES6모듈의 API는 정적이다. 컴파일레이션 중에 불러온 모듈의 API 멤버를 미리 알 수 있다. 

```jsx
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

```jsx
import hello from 'bar';

var hungry = "hippo";
function awesome() {
	console.log(hello(hungry).toUpperCase());
}

export awesome;
```

```jsx
module foo from 'foo';
module bar from 'bar';

console.log(bar.hello("rhino"));
foo.awesome();
```

`import` 는 모듈 API에서 하나 이상의 맴버를 불러와 특정 변수에 묶어 현재 스코프에 저장한다.

`module`은 모듈 API 전체를 불러와 특정 변수에 묶는다.

`export`는 확인자를 현재 모듈의 공개 API로 내보낸다.

앞서 살펴본 함수 - 클로저 모듈처럼 모듈 파일의 내용은 스코프 클로저에 감싸진 것으로 처리된다.

## 5.6 정리하기

---

클로저란 함수를 렉시컬 스코프 밖에서 호출해도 

함수는 자신의 렉시컬 스코프를 기억하고 접근할 수 있는 특성을 말한다.

모듈은 두가지 특징을 갖어야 한다.

- 최외곽 래퍼 함수를 호출하여 외곽 스코프를 생성한다.
- 래핑 함수의 반환 값은 반드시 하나 이상의 내부 함수 참조를 가져야 하고, 그  배부 함수는 래퍼의 비공개 내부 스코프에 대한 클로저를 가져야 한다.
