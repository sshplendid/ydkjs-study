# 3. Promise

- 콜백의 결함: 순차성, 신뢰성 결여
- 콜백의 믿음

Promise

> 프로그램의 진행을 다른 파트에 넘겨주지 않고도 언제 작업이 끝날지 알 수 있고, 그 다음에 무슨 일을 해야 할지 스스로 결정할 수 있다.

- [콜백 HELL](https://miro.medium.com/max/721/1*zxx4iQAG4HilOIQqDKpxJw.jpeg)은 가라
- 자바스크립트의 최신 비동기 API는 대부분 프라미스 기반

## 3.1. 프라미스란

### 3.1.1. 미랫값

- 시나리오: 성공
- 시나리오: 실패

**지금값과 나중값**

- 코드를 작성할 때 값이 지금 존재하는 구체적인 값이라는 근원적인 가정
- 아래 코드에서 `x + y` 연산을 할 때 `x`와 `y` 모두 이미 세팅된 값이라고 가정 (Resolved)
- `+` 연산자가 x, y 값의 상태를 감지하다가 덧셈연산을 해줄 수는 없음
- 어떤 구문은 지금 실행되고 어떤 구문은 나중에 실행? → 프로그램은 혼돈의 늪으로

```jsx
var x, y = 2;

console.log(x + y);
```

- 두 구문 중 한쪽 (혹은 둘다) 아직 실행 중이라면?
- 콜백 패턴의 비동기 코드
    - x, y는 모두 미래값으로 취급 (언젠간 값이 할당 될 것이다)
    - 현재 x,y의 상태는 모두 관심 밖 (현재는 undefined)

```jsx
function add(getX, getY, cb) {
	var x, y;
	getX(function(xVal) {
		x = xVal;
		if(y != undefined) { // getY의 실행이 종료됐다면
			cb(x + y);
		}
	});
	getY(function(yVal) {
		y = yVal;
		if(x != undefined) { // getX의 실행이 종료됐다면
			cb(x + y);
		}
	});
}
```

**프라미스 값**

- fetchX, fetchY의 반환값(Promise)을 add()에 전달한다.
- 시점에 관계없이 각 프로미스가 같은 결과를 내게끔 정규화한다.
- x, y는 시간 독립적으로 추론 가능

```jsx
function add(xPromise, yPromise) { // 비동기 처리 결과
	return Promise.all([
		xPromise,
		yPromise,
	]).then(function(values) {
		return values[0] + values[1];
	});
}

add(fetchX(), fetchY())
.then(function(sum) {
	console.log(sum);
});
```

- 프로미스는 이룸(Fulfillment) or 거부(Rejection)으로 귀결된다.
    - 그 전에 대기(Pending) 상태가 존재한다, 결과를 기다리는 중
- 거부의 경우, 프로그램 로직에 따라 세팅되거나, 런타임 예외에 의해 암시적으로 발생하기도 한다.
- `then()` 함수는 Fulfillment 함수를 첫번째 인자로 Rejection 함수를 두번째 인자로 받는다.
- fetch 동작에 문제가 있거나 덧셈 연산이 실패하면, 프라미스 결과값은 버려지고 에러 처리 콜백이 버림 값을 받는다.

```jsx
add(fetchX(), fetchY())
.then(function(sum) { // Fulfillment
	console.log(sum);
},
function(err) { // Rejection
	console.error(err);
});
```

- 프라미스는 시간 의존적 상태를 외부로부터 캡슐화하기 때문에 **시간 독립적**이다
- 프라미스는 귀결된 후 상태가 그대로 유지된다. 즉 **불변값**이다.

### 3.1.2. 완료 이벤트

- 프라미스의 귀결(Resolved)은 비동기 작업의 여러 단계를 **흐름 제어**하기 위한 체계
- `foo()`라는 함수의 작업 내용은 알수 없고 언제 끝날지도 모른다.
    - 단지 언제 다음 단계로 넘어갈 수 있을지만 알면 된다. (작업의 완료상태 알림을 받고싶다)
- 전통적인 자바스크립트 사고방식
    - 알림 자체를 하나의 이벤트로 보고 리스닝
    - foo를 호출한 뒤 이벤트 리스너를 설정

```jsx
foo(x) {
	// do something...
}

var event = foo(42);
event.on('complete', function() { // 이벤트 리스너
	// do next job...
});
event.on('error', function() {
	// something's happen
});
```

- 프라미스에서는 관계가 역전되어 `foo()` 에서 이벤트를 리스닝하고, 알림을 받으면 다음으로 진행
    - **제어의 비역전 (un-inversion of control)**
    - 관심사의 분리

```jsx
var event = foo(42);

onCompleteBar(event); // 완료 이벤트를 리스닝

onCompleteBaz(event); //완료 이벤트를 리스닝
```

**프라미스 '이벤트'**

- 위 이벤트 구독기는 프라미스와 유사함
- 프라미스 식의 예제
    - **프라미스는 일단 귀결(resolve)되면 똑같은 결과(fulfillment or rejection)를 항상 유지한다**
    - 몇번이고 계속 꺼내 쓸 수 있음

```jsx
function foo(x) {
	// do something...

	return new Promise(function(resolve, reject) { // 생성자 노출 패턴
		// on complete
		resolve()
		// on failure
		reject();
	}
}

var promise = foo(42);

bar(promise)
baz(promise) // 이 코드는 promise.then(bar).then(baz)와 완전히 다르다!!

function bar(promise) {
	promise.then(
		() => {
			// resolve, on complete
		},
		() => {
			// reject, on failure
		}
	);
};
```

- 간단 promise

    ```jsx
    function fetchSomething(name) {
        // 5초가 소요되는 fetch...
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                console.log('fetch done.')
                if(!name) {
                    reject(new Error('Name is empty!'))
                }
                resolve(`hello ${name}!`)
            }, 5000)   
        })
    }

    console.log('first try');
    fetchSomething('shawn')
    .then((result) => {
        console.log(`then: ${result}`)
    })

    console.log('second try');
    fetchSomething('')
    .then((result) => {
        console.log(`then: ${result}`)
    }, err => {
        console.error(`reject:`, err)
    })
    ```

## 3.2. 데너블 덕 타이핑

- 어떤 값이 진짜 프라미스일까?
- 프라미스처럼 작동하는 값이 맞을까?
- 프라미스인지 확인하는 방법 →  `p instanceof Promise`
    - 인스턴스 체크만으로 불충분
    - 서드파티 라이브러리는 ES6 Promise가 아닌 자체 구현 프라미스가 존재할 수 있다.
    - IE8 같은 구식 브라우저는?
- 프라미스의 규정: `then()` 메서드를 가진 데너블(thenable) 객체 혹은 함수를 정의하여 판별하는 것으로 규정
    - 데너블에 해당하는 값은 무조건 프로미스 객체이다! (duck typing)
    - `then()` 함수를 가진 객체는 JS엔진이 데너블로 인식하여 특별한 규칙을 적용한다.
    - `then` 이라는 키워드를 쓸 때는 조심해야 한다!

```jsx
// 프라미스 판별 코드
if( p !== null &&
		(
			typeof p === 'object' ||
			typeof p === 'function'
		) &&
		typeof p.then === 'function
	) {
		// Thenable!
} else {
		// Not Thenable!
}
```

- 이것도 프로미스라고?

```jsx
function thenable() {
    return {
        then(resolve, reject) {
            resolve('이것도 프로미스?')
        }
    }
}

var promise = Promise.resolve(thenable())

promise.then((msg) => {
    console.log('프로미스 완료')
    console.log(msg)
});

console.log('promise instance of Promise? ', promise instanceof Promise)
```

## 3.3. 프라미스 믿음

- 프로미스는 아래와 같은 상황에서 사용가능한 해결책을 제시
    - 너무 일찍 콜백 호출
    - 너무 늦게 콜백 호출 or 호출하지 않음
    - 너무 적게, 많이 콜백 호출
    - 필요한 환경, 인자를 콜백에 전달 못함
    - 에러/예외 무시

### 3.3.1. 너무 빨리 호출

- 프로미스는 바로 귀결(resolve)되어도 비동기로 처리된다.
- 이미 귀결된 이후라 해도 `then()` 에 건낸 콜백은 항상 비동기적으로 호출한다.

```jsx
function thenable() {
    return {
        then(resolve, reject) {
            resolve('이것도 프로미스?')
        }
    }
}

var promise = Promise.resolve(thenable())

promise.then((msg) => {
    console.log('프로미스 완료')
    console.log(msg)
});

console.log('promise instance of Promise? ', promise instanceof Promise)

promise.then((msg) => {
    console.log('다시 한번 호출해본다. ', msg)
})

console.log('이 텍스트가 먼저 호출된다.')
// promise instance of Promise?  true
// 이 텍스트가 먼저 호출된다.
// 프로미스 완료
// 이것도 프로미스?
// 다시 한번 호출해본다.  이것도 프로미스?
```

### 3.3.2. 너무 늦게 호출

- `then()` 에 등록한 콜백은 새 프로미스가 생성되면서 `resolve()` , `reject()` 중 어느 한쪽은 호출하도록 스케쥴링 됨
- 프로미스가 귀결되면 다음 비동기 처리 기회가 왔을 때 순서대로 실행되며 콜백 내부에서 다른 콜백의 호출에 영향을 주거나 지연시킬 일은 없다.

```jsx
function getPromise(msg) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(msg)
        }, 1000)
    })
}

var p = getPromise('약속')

p.then(() => {
    p.then(() => {
        console.log('C')
    })
    console.log('A')
})

p.then(() => {
    console.log('B')
})

// A
// B
// C
```

**프라미스 스케줄링의 기벽**

- 별개의 두 프로미스에서 연쇄된 콜백 사이의 상대적인 실행 순서는 장담할 수 없다?
- 일반적인 인식
    1. `p1`이 귀결되고
    2. `p2`가 귀결됐으니
    3. 당연히 A, B 순서로 나와야 하는거 아냐?

```jsx
var p3 = new Promise( function(resolve,reject){
	resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){
	resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){
	resolve( "A" );
} );

p1.then( function(v){
	console.log( v );
} );

p2.then( function(v){
	console.log( v );
} );

// A B  <-- not  B A  as you might expect
```

- 실제 처리과정
    1. p3: 언젠간 처리됨
    2. p1: 언젠간 처리됨
    3. p2: 언젠간 처리됨
    4. p1.then: 귀결된 결과가 프로미스 객체다. unwrapping을 위해 다시 콜백 큐로 던진다.
    5. p2.then: 귀결된 결과가 즉시값(immediate value)이므로 로그를 찍는다. → A
    6. unwrapping된 p3 객체의 즉시값을 로그로 찍는다 → B
- 순서/스케줄링에 의존하지 말자
- 순서가 문제를 일으키지 않는 방향으로 코딩하는 것이 바람직하다.

### 3.3.3. 한번도 콜백을 호출하지 않음

- 프로미스 귀결 사실을 알리지 못하게 할 방법은 없음
- 만약 프로미스가 귀결되지 않는다면? 경합(race)을 이용하여 해결할 수 있다.
    - 아주 오래 걸리는 작업이라 언젠간 결과가 오지만, 귀결되지 않는것 처럼 보일 수 있음

```jsx
function getPromise(msg, timeout) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(msg)
        }, timeout || 1000)
    })
}

function timeoutPromise() {
    return getPromise('타임아웃', 3000)
}

Promise.race([
    getPromise('오래 걸리는 작업', 5000), // 이 작업은 빨리 끝날수도, 오래 걸릴수도 있다.
    timeoutPromise(),
])
.then((msg) => {
    console.log(msg)
}, (err) => {
    console.log(err)
})

// 타임아웃
// 타임아웃 이전에 getPromise가 귀결된다면? 그 결과가 전달될 것이다.
```

### 3.3.4. 너무 가끔, 너무 종종 호출

- 콜백의 호출 횟수는 오직 **한 번**
- 너무 가끔? → 0 번, 3.3.3. 의 내용
- 너무 종종? 최초 한번만 귀결하고 나머지는 무시한다.

### 3.3.5. 인자/환경 전달 실패

- 귀결값은 딱 하나
- 명시적인 값으로 귀결되지 않으면 그 값은 `undefined`
- resolve, reject 함수를 호출할 때 두번째 이후 인자는 그대로 무시한다. → SPEC!!
- 값을 여러개 넘기고 싶다면 배열, 객체를 사용하자
- 혹은 closure를 이용

### 3.3.6. 에러/예외 삼키기

- 어떤 이유로 프로미스를 버리면 그 값은 reject로 전달
- then에서 에러가 발생한다면?
    - 이미 귀결된 불변 값이기때문에 reject는 호출되지 않음
    - 대신 `catch()` 로 에러핸들링이 가능하다.

```jsx
var p = getPromise('abc')
p.then((msg) => {
    undeclared.foo()
}, err => {
    // 이미 귀결되었기 때문에 이 부분은 호출되지 않는다.   
})
.catch(err => {
    console.log('이렇게 에러처리 가능', err)
})
```

### 3.3.7. 미더운 프라미스?

- 프로미스는 콜백을 완전히 없애기 위한 장치가 아님
- 콜백을 넘겨주는 위치만 달라질 뿐
- 못미더운 프라미스? thenable

```jsx
var p = {
	then: function(cb,errcb) {
		cb( 42 );
		errcb( "evil laugh" );
	}
};

p
.then(
	function fulfilled(val){
		// 여기도 실행되고
		console.log( val ); // 42
	},
	function rejected(err){
		// 여기도 실행됨
		console.log( err ); // evil laugh
	}
);
```

- `Promise.resolve()`를 사용하자
- 정규화된 프로미스 객체로 변환하므로, 안전한 결과를 기대할 수 있다.

```jsx
var p = {
	then: function(cb,errcb) {
		cb( 42 );
		errcb( "evil laugh" );
	}
};

Promise.resolve(p) // 안전한 프로미스 객체
.then(
	function fulfilled(val){
		// 여기도 실행되고
		console.log( val ); // 42
	},
	function rejected(err){
		// 실행되지 않음
		console.log( err );
	}
);
```

### 3.3.8. 믿음 형성

- 프로미스 없이 비동기 코딩이 가능할까? 20년 동안 콜백으로 작성해옴
- 하지만 콜백은 불안함 (2장)

## 3.4. 연쇄 흐름

- 프로미스는 일련의 비동기 단계를 나타낼 수 있다.
    - `then()`을 호출할 때마다 생성하여 반환되는 새 프로미스를 연쇄(chaining)할 수 있다.
    - `then`의 resolve 콜백이 반환한 값은 어떤 값이든 체이닝되서 세팅된다.

```jsx
var p = Promise.resolve(10)

p
.then((n) => n*2) // 20
.then(n => n + 5) // 25
.then(n => console.log(n)) // 콘솔 출력
```

- resolve, reject 함수가 명시되있지 않으면 다음처럼 처리된다.

```jsx
var p = Promise.resolve(10)

p.then(
	function fullfilled() { }, // resolve, 귀결값을 폐기한다.
	function error(err) { throw err; } // reject, 에러 값을 던진다.
)
```

- 프로미스 고유 특징
    - then을 호출하면 그 결과 자동으로 새 프로미스를 생성하여 반환
    - resolve/reject 안에서 어떤 결과를 반환하거나 에러를 던지면 새 프로미스가 귀결된다.
    - resolve/reject에서 반환한 프로미스는 풀린(unwrapping) 상태이고 연쇄 프로미스의 귀결값이 된다.

### 3.4.1. 용어 정의: 귀결, 이룸, 버림

- 귀결(resolve): 프로미스의 최종 값/상태를 지정한다. 이룸/버림 두 상태 중 하나로 결정
- 이룸(fulfill): 프로미스가 이행된 상태
- 버림(reject): 프로미스가 버려진 상태

## 3.5. 에러 처리

- `try ... catch` : 일반적인 에러처리 패턴
    - 동기적으로만 사용 가능
- 비동기의 에러 처리 패턴
    - 에러 우선 콜백(error first callback): 콜백 지옥을 피하기 힘듬

```jsx
function fetch(callback) {
    setTimeout(() => {
        try {
            var x = baz.bar();
            callback(null, x); // resolved!
        } catch(e) {
            callback(e); // rejected!
        }
    }, 1000)
}

fetch((err, data) => {
  if(err) {
      console.error('에러 처리:', err);
      return;
  }  

  console.log(data)
})
```

- 프로미스 에러 처리의 미묘한 문제
    - 이미 fulfill 상태에서 에러가 발생한다면? 프로미스는 에러를 포착하지 못함

```jsx
Promise.resolve(42)
.then(val => {
	console.log(val.toLowerCase());
}, error => {
	console.error('에러 발생: ', error);
})
```

### 3.5.1. 절망의 구덩이

- 프로미스는 에러가 발생해도 무시할 수 있다고 본다.
    - 프로미스 연쇄 끝에 `catch` 를 달아야하나?
    - catch에서 에러가 발생한다면?

### 3.5.2. 잡히지 않은 에러 처리

- 일부 프로미스 라이브러리는 전역 미처리 버림(global unhandled rejection) 처리기 같은 것을 등록하여 이 메서드가 대신 호출되도록 해놓음
    - 타임아웃(e.g. 3초)을 걸어 에러처리가 안되면 앞으로도 처리기를 등록하지 않겠다고 간주
    - 대부분의 경우 잘 통함
    - 버림상태를 유지해야 할 경우에는?
- `done()`을 쓰자! (아마 `finally()` 얘기인듯)
    - ECMA 표준에 없음 ([finally는 후에 ECMA 2018로 발표됨](https://github.com/tc39/proposals/blob/master/finished-proposals.md))

### 3.5.3. 성공의 구덩이

- 기본적으로 프로미스는 이벤트 루프 틱 시점에 에러 처리기가 등록되어 있지 않은 경우 모든 버림을 (콘솔창에) 알리도록 되어있음
- 감지되기 전까지 프로미스 버림상태를 유지하려면 `defer()`를 호출해서 해당 프로미스에 관한 알림 기능을 끈다
    - [defer는 더 이상 사용되지 않음](https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Promise.jsm/Deferred)
- 긍정오류를 제어하는건 개발자의 몫

## 3.6. 프라미스 패턴

### 3.6.1. `Promise.all( [ ] )`

- 여러 비동기 작업을 **병렬**로 수행하고 싶을 때 사용
- 모든 작업이 끝나야 다음 작업이 가능함
- 예제: 비동기 요청 두 번을 수행한 뒤 세 번째 비동기 요청 작업
- 하위 프로미스가 모두 resolve되어야한다.
- 하나라도 reject되면 다른 프로미스도 무효화된다
- 프로미스마다 버림/에러 처리기를 붙여놔야 함

```jsx
var url1 = 'https://www.google.com/search?q=promise'
var url2 = 'https://www.google.com'

Promise.all([
	fetch(url1),
	fetch(url2)
]).then(([res1, res2]) => { // 프로미스 결과를 배열로 받는다.
	if(res1.status !== 200 || res2.status !== 200) {
		// 병렬 작업의 결과를 검증하고 다음 작업을 이어갈지 결정한다.
		console.error('뭔가 잘못됨')
		return
	}
	console.log('다음 작업 수행')
	return fetch('https://www.google.com')
		
}).then(response => {
	console.log(response.status);
}).catch(error => {
    console.error('에러 발생', error)
})
```

### 3.6.2. `Promise.race( [ ] )`

- 여러 프로미스 중에 최초로 resolve된 프로미스만 인정하고 나머지는 무시해야 할 때 사용
    - 타임아웃 구현
    - Latch 패턴이라고도 부름
    - 하나라도 reject되면 프로미스가 reject됨

    ```jsx
    var url1 = 'https://www.google.com/search?q=promise'
    var url2 = 'https://www.google.com'

    Promise.race([
    	fetch(url1),
    	fetch(url2)
    ]).then((res) => { // 프로미스 결과를 배열로 받는다.
    	console.log('승자는? ', res.url)		
    }).catch(error => {
        console.error('에러 발생')
    })
    ```

**타임아웃 경합**

```jsx
var getTimeout = (milli) => {
    return new Promise((res, rej) => {
        setTimeout(() => {
            rej(`Timeout: ${milli} milliseconds over...`)
        }, milli)
    })
}
Promise.race([
	fetch(url1),
	getTimeout(5),
]).then((res) => { // 프로미스 결과를 배열로 받는다.
	console.log('승자는? ', res.url)
	return fetch('https://www.google.com')
		
}).catch(error => {
    console.error('에러 발생: ', error)
})
```

**결론**

- 폐기/무시되는 프로미스는?
    - 취소가 안됨
    - 그냥 묻힘 (실행은 됨)
- 폐기된 프로미스에서 어떤 작업을 해야할때?
    - `finally()`를 사용
- 헬퍼 유틸리티

```jsx
Promise.observe = (promise, callback) => {
	promise.then(message => {
		Promise.resolve(msg).then(callback)
	},
	error => {
		Promise.reject(error).then(callback)
	}
	
	return promise
}
```

### 3.6.3. all([])/race([])의 변형

- none
- any
- first
- last: 작성해본 코드

```jsx
var fetchT = (time) => {
	// {time} 값만큼의 시간이 걸리는 fetch함수, 결과값으로 name 쿼리스트링을 리턴함
  const url = `https://some.url.com/v1/time/${time}?name=john${time}`
  console.log(`Trying to fetch '${url}'`)
  return fetch(url)
}

Promise.prototype.last = (promises) => {
    return new Promise((res, rej) => {
        const size = promises.length
        let count = 0
        promises.forEach((promise) => {
            promise.then(value => {
                count += 1
                if(count === size) {
                    res(value) // 마지막 이룸이면 프로미스를 이룸처리한다.
                }
            }).catch(err => {
                count += 1 // reject 되어도 카운트를 올린다.
            })
        })
    })
}

Promise.prototype.last([
    fetchT(1000),
    fetchT(3000),
    fetchT(2000),
]).then(res => {
    console.log('최후의 승자는?', res.url) // john3000
})
```

### 3.6.4. 동시 순회

- `map`, `some`, `every`, `forEach`, ...
- 비동기 버전의 유틸리티를 사용하자
- 결국 `Promise.all`

## 3.7. 프라미스 API 복습

### 3.7.1. `new Promise()`

- 생성과 함께 동기적으로 즉시 호출할 콜백 함수를 전달해야 함
- 보통 resolve, reject라고 명명
- reject는 프로미스를 버리지만 resolve는 이룸/버림 한가지로 처리함
- 즉시값(혹은 데너블이 아닌 값)이 넘어오면 프로미스는 그 값으로 이룸처리(resolved)
- 프로미스/데너블 값이 전달되면 재귀적으로 풀어보고 최종값이 프로미스의 귀결상태가 됨

### 3.7.2. `Promise.resolve()` / `Promise.reject()`

- reject는 버려진 프로미스를 생성하는 지름길
- resolve는 이루어진 프로미스를 생성하는 용도
    - 데너블 값을 재귀적으로 풀어 최종값을 반환

### 3.7.3. `then()` / `catch()`

- 프로미스 인스턴스엔 then, catch 메서드가 들어있음
- 프로미스가 귀결되면 두 처리기 중 하나만 비동기적으로 호출됨
- then은 resolve/reject 콜백을 인자로 받고 어느 한쪽이 누락되거나 함수가 아닌 인자가 들어오면 기본 콜백으로 대체됨
- catch는 버림 콜백 하나만 받음
- 둘 다 새로운 프로미스를 만들어서 반환하기 때문에 체이닝이 가능함
- 콜백 안에서 예외가 발생하면 반환된 프로미스는 버림

### 3.7.4. `Promise.all` / `Promise.race`

- ES6 프로미스 API의 헬퍼 유틸리티
- 프로미스 배열에 따라 귀결이 좌우됨
- all()은 모든 프로미스들이 이뤄져야 메인 프로미스도 이뤄짐
    - gateway
- race()는 최초로 귀결된 프로미스만 귀결됨
    - winner

## 3.8. 프라미스 한계

### 3.8.1. 시퀀스 처리

- 프로미스 체이닝은 전체 체이닝을 하나의 '뭔가'로 가리킬 개체가 마땅치않음
- 에러 처리기가 없는 프로미스는 어딘가에서 감지될 때까지 쭉 하위로 전파됨
- 예외가 잡혀도 묻힐 가능성이 있음

### 3.8.2. 단일값

- 프로미스는 하나의 이룸/버림만을 가짐
- 여러 값을 사용하려면? object 혹은 array
- 체인에서 계속 풀고 묶어야할까?

**값을 분할**

- 2개 이상의 프로미스로 분할하면?
- 결국 값 배열을 프로미스 배열로 바꾼거 아님?
- 결국 다 꼼수...
- ES6 destructing문법을 사용하자

### 3.8.3 단일 귀결

- 프로미스는 최초 1회만 귀결됨    중요!!!!!!
- 이벤트/스트림 같은 비동기 모델에서는? 프로미스가 잘 동작할지 확실하지 않음

### 3.8.4. 타성

- 기존 콜백 코드를 프로미스로 바꾸라고?
- 굳은 결심을 하자...
- 헬퍼 유틸리티를 만든다면?

### 3.8.5. 프로미스는 취소 불가

- 일단 프로미스를 생성하면 외부에서 진행을 멈출 방법이 없음
- 프로미스 추상화 라이브러리를 쓰자

### 3.8.6. 프로미스 성능

- (콜백보다) 프로미스가 좀 더 느린건 사실
- 신뢰성 보장을 위해서 그정도는 감수할 수 있지않나?
- 무조건 빨라야해서 프로미스를 쓰지말자고?

## 3.9. 정리하기

- 프라미스는 시간 의존적 상태를 외부로부터 캡슐화하기 때문에 **시간 독립적**이다
- 프라미스는 귀결된 후 상태가 그대로 유지된다. 즉 **불변값**이다.
- 훌륭하다
- 마음껏 쓰자
- 비동기 흐름을 순차적으로 표현하는 더 나은 방법!
