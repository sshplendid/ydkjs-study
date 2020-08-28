# :package:객체

## :peach:구문

객체는 선언적형식(리터럴형식)과 생성자 형식으로 정의합니다. 리터럴 형식은 한 번의 선언으로 다수의 키/값 쌍을 프로퍼티로 추가할 수 있지만, 생성자 형식은 한번에 한 프로퍼티만 추가할 수 있다.

**리터럴 형식**
```js
var myObj = {
	key: value
	// ...
};
```

**생성자 형식**
```js
var myObj = new Object();
myObj.key = value;
```


## :peach:타입

객체는 레고 블록과 같고, 자바스크립트 언어 주요 타입 6개는 아래와 같다.

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`

이 중 단순 원시타입(Simple Primitives) 5개(string, number, boolean, null, undefined)는 객체가 아니다. typeof null이 'object'를 반환하는 것은 언어 자체의 버그이다.

단순 원시타입(Simple Primitives)과 대조적으로 복합 원시타입(Complex Primitives)이라고 할 수 있는 객체 하위 타입(Sub Type)이 존재한다. 함수와 배열, 내장 객체가 이에 속한다. 자바스크립트 함수는 일반 객체와 동일하게 취급 가능한 일급(First Class) 객체이다.

<br><br>

### :strawberry:내장객체

내장객체는 자바스크립트 내장 함수로 각각 생성자(new 연산자가 붙은 함수 호출)로 사용되어 주어진 하위타입의 새 객체를 생성한다.

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`


```js
var strPrimitive = "나는 문자열이야!";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "나는 문자열이야" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// 객체 하위 타입을 확인한다.
Object.prototype.toString.call( strObject );	// [object String]
```

`Object.prototype.toString.call( strObject )`은 `toString()` 메서드의 기본 구현체를 빌려 내부 하위 타입을 조사한다. 그 결과 strObject가 String 생성자에 의해 만들어진 객체임을 알 수 있다.


`strPrimitive` 원시값은 객체가 아닌 원시 리터럴이며 불변값이다. 문자별로 접근할 때는 String 객체가 필요하다. 자바스크립트 엔진은 문자열 원시값을 String 객체로 자동 강제변환하므로 생성자 형식은 지양하고 리터럴 형식 사용을 권장한다.


```js
var strPrimitive = "나는 문자열이야!";

console.log( strPrimitive.length );			// 13

console.log( strPrimitive.charAt( 3 ) );	// "m"
```


`42.359.toFixed(2)`와 같은 메서드를 호출할 때, 리터럴 숫자 원시값 `42`와 `new Number(42)`는 동일한 강제변환이 일어나고, `Boolean` 원시값도 `Boolean` 객체도 마찬가지이다.

`null`과 `undefined`은 생성자 형식이 없고, 리터럴형식만 있고, `Date` 값은 리터럴 형식이 없어서 반드시 생성자 형식으로 생성해야 한다.

`Object`, `Array`, `Function`, 그리고 `RegExp`(정규표현식)은 형식과 무관하게 모두 객체이다.

`Error`객체는 생성자 형식으로 생성은 가능하지만 거의 쓸일이 없다.

<br><br>

## :peach:내용

객체의 내용은 우리가 프로퍼티라고 부르는 * 위치 *에 저장된 값(모든 유형)으로 구성된다. 하지만 객체 내부에는 **프로퍼티 값**이 저장되는 것이 아니다. 저장되는 것은 **실제 값이 저장되는있는 위치에 대한 포인터 역할을 하는 프로퍼티 이름**이다. 

```js
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

`myObject` 의 a 위치에 있는 값에 접근하는 방식에는 `.` 연산자를 사용하는 프로퍼티 접근 방식 혹은 `[]` 연산자를 사용하는 키 접근 방식이 있다. 키접근 방식은 UTF-8/유니코드 호환 문자열 모두 프로퍼티명으로 쓸 수 있다는 장점이 있다. 

```js
var wantA = true;
var myObject = {
	a: 2
};

var idx;

if (wantA) {
	idx = "a";
}

// later

console.log( myObject[idx] ); // 2
```

객체 프로퍼티명은 언제나 문자열이고 문자열 이외의 다른 원시 값을 쓰면 문자열로 변환된다.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

<br><br>

### :strawberry:계산된 프로퍼티명

ES6부터 추가된 계산된 프로퍼티명은 아래와 같이 쓸 수 있다.

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```
<br><br>

### :strawberry:프로퍼티 vs 메서드

기술적으로 함수는 객체에서 속하지 않고, 따라서 객체 참조로 접근할 수 있는 함수를 메서드라 말하는건 그 의미를 지나치게 확대 해석 한 것이다.

다른 언어에서 객체(클래스)에 부속된 함수를 메서드라고 부르고 자바스크립트 함수 역시 객체의 부속물이라 생각하기에 '프로퍼티 접근'에 대비되는 용어로 '메서드 접근'이라는 말을 종종 사용한다. 하지만 기술적으로 함수는 객체에서 속하지 않고, 객체 참조로 접근할 수 있는 함수를 메서드라 하는건 그 의미를 무리하게 해석 한 것이다.


```js
function foo() {
	console.log( "foo" );
}

var someFoo = foo;	// `foo`에 대한 변수 레퍼런스

var myObject = {
	someFoo: foo
};

foo;				// function foo(){..}

someFoo;			// function foo(){..}

myObject.someFoo;	// function foo(){..}
```

위 예제에서 `someFoo`나 `myObject.someFoo`는 동일한 함수에 대한 각각의 참조일 뿐이며, 특별히 다른 것은 없다. 만약 `foo()`내부에 `this` 참조가 포함하도록 정의된 경우 `myObject.someFoo`에서 발생할 암시적 바인딩이 유일한 차이점이다. 

객체 리터럴의 일부로 함수 표현식을 선언하더라도 함수가 객체에 속하는 것은 아니고, 여전히 동일한 함수 객체에 대한 여러 참조 일뿐이다. 

가장 안전한 결론은 자바스크립트에서 함수와 메서드는 같은 의미를 가진다고 보는 것이다.

<br><br>

### :strawberry:배열

배열도 `[]` 연산자로 접근하지만 값을 저장하는 방법과 장소가 더 체계적이다. 배열은 인덱스라는 양수로 표기된 위치에 값을 저장하고, 배열 자체가 객체이기 때문에 프로퍼티를 추가하는 것도 가능하다. 하지만 키/값 저장소로는 객체, 숫자 인덱스 저장소로는 배열을 쓰는게 좋다.

배열에 프로퍼티를 추가할 때  프로퍼티명이 숫자와 유사하면 숫자 인덱스로 잘못 해석되어 배열 내용이 달라질 수 있으니 주의하자.

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```


<br><br>

### 객체 복사

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// 카피가 아닌 레퍼런스다!
	c: anotherArray,	// 역시 레퍼런스다!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

`myObject`의 얕은 복사(Shallow Copy)는 a 프로퍼티는 원래 값 2가 그대로 복사되고, b, c, d 는 원본 객체의 참조와 동일한 위치에 대한 참조가 된다. 깊은 복사(Deep Copy)를 하면 `myObject`는 물론 `anotherObject`와 `anotherArray`까지 모두 복사한다. 여기서 문제는 `anotherArray`가 `anotherObject`와 `myObject`에 대한 참조를 가지고 있기 때문에 순환 참조로 인해 `anotherArray`가 무한 반복되는 것이다.

이 문제를 해결하기 위해서는 객체를 JSON 문자열로 변환후 JSON.parse()를 이용해 다시 자바스크립트 객체로 만들어주면 깊은 복사가 된다.

```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

ES6에서는 Object.assign() 메서드를 사용해서 얕은 복사를 할 수 있다. 이 메서드의 첫번째 인자는 타깃 객체이고 두번째 인자는 소스 객체이다.

```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

<br><br>

### :strawberry:프로퍼티 서술자

ES5부터 모든 프로퍼티는 프로퍼티 서술자로 표현된다.

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

프로퍼티 `a`의 프로퍼티 서술자를 조회하면 `writable`, `enumerable`, `configurable` 의 특성값을 확인할 수 있다. `Object.defineProperty` 메서드를 이용하면 프로퍼티를 추가하거나 수정할 수 있다.

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```
<br><br>

### :strawberry:불변성

프로퍼티/객체를 불변 객체로 바꾸는 방법을 소개한다.

<br>

#### 객체 상수

`writable:false`와 `configurable:false`를 같이 쓴다.

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```
<br>

#### 확장 금지

프로퍼티를 추가할 수 없게 하고 싶을 때 `Object.preventExtensions()`를 호출한다.

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

<br>

#### 봉인

`Object.seal()`는 봉인 객체를 생성한다.

<br>

#### 동결

`Object.freeze()`는 고정된 객체를 생성한다. 

<br><br>

### :strawberry:[[Get]]

`myObject.a` 퍼로퍼티 접근은 `myObject`에 대한 `[[Get]]` 연산을 한다.

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

프로퍼티 값을 찾을 수 없으면 `[[Get]]` 연산은 undefined를 반환한다.

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

<br><br>

### :strawberry:[[Put]]

`[[Put]]` 연산의 알고리즘은 이미 존재하는 프로퍼티에 대해 다음과 같다.

1. 프로퍼티가 접근 서술자인가? 맞으면 세터를 호출한다.
2. 프로퍼티가 `writable:false`인 데이터 서술자인가? 맞으면 비엄격 모드에서 조용히 실패하고 엄격 모드는 TypeError가 발생한다.
3. 이외에는 프로퍼티에 해당 값을 세팅한다.

<br><br>

### :strawberry:게터와 세터

`[[Put]]`과 `[[Get]]` 기본 연산은 이미 존재하거나 전혀 새로운 프로퍼티에 값을 세팅하거나 기존 프로퍼티로부터 값을 조회하는 역할을 한다. ES5부터는 게터/세터를 통해 프로퍼티 수준에서 이런 로직을 오버라이드할 수 있다.


```js
var myObject = {
	// `a`의 게터를 정의한다.
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// 타겟
	"b",		// 프로퍼티 명
	{			// 서술자
		// `b`의 게터를 정의한다.
		get: function(){ return this.a * 2 },

		// 'b'가 객체 프로퍼티로 확실히 표시되게 한다.
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

아래 예는 a 게터가 정의되어 있으므로 값을 세팅하려고 하면 무시된다.

```js
var myObject = {
	// `a`의 게터를 정의한다.
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

게터와 세터는 항상 둘 다 선언하는 것이 좋다. 어느 한쪽만 선언하면 예상외의 결과가 나올 수 있다.

```js
var myObject = {
	// `a`의 게터를 정의한다.
	get a() {
		return this._a_;
	},

	// `a`의 세터를 정의한다.
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```



<br><br>

### :strawberry:존재 확인

프로퍼티 접근 시 결과값이 `undefined`면 원래 값이 `undefined`거나 해당 객체에 프로퍼티가 없다는 의미다. 이 두 경우를 구분할 수 있을까?

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

`in` 연산자는 어떤 프로퍼티가 해당 객체에 존재하는지 아닌지, 상위 단계에 존재하는지 확인한다. `hasOwnProperty()`는 프로퍼티가 객체에 있는지만 확인하고 상위 단계는 찾지 않는다. 그리고 아래와 같이 열거 가능성을 불가능하게 세팅을 하면 `for..in` 루프에서는 없어져 버리지만, `in` 연산자, `hasOwnProperty()`로 존재 확인이 가능하다.


```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` NON-enumerable
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```



<br><br>

## :peach:순회

ES5에서는 `forEach()`, `every()`, `some()` 등 배열 관련 순회 헬퍼가 도입됐다. `forEach()` 는 배열 전체를 순회하고 콜백함 수 반환값을 무시한다. `every()` 콜백함수가 falsy값을 반환할 때까지, `some()` 은 truthy 값을 반환할 때까지 순회한다. `for..in` 루프를 이용한 객체 순회는 열거 가능한 프로퍼티만 순회하고 그 값을 얻으려면 일일이 프로퍼티에 접근해야하므로 간접적인 값 추출이다.


배열 값을 직접 순회하려면 ES6의 `for..of` 구문을 사용할 수 있다.

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```
 `for..of`는 순회당 한번씩 next() 메서드를 호출해 연속적으로 반환 값을 순회한다. ES6부터는 `Symbol.iterator` 심볼로 객체 내부 프로퍼티 @@iterator(기본 내부 함수)에 접근할 수 있다.

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

순회하려는 객체의 기본 @@iterator를 커스텀할 수도 있다.

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// `myObject`를 수동으로 순회한다.
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// `myObject` 를 `for..of` 루프로 순회한다.
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

<br><br>
