# 4. 강제변환


## 4.1 값변환


어떤 값을 다른 타입의 값으로 바꾸는 것을 "값 변환"이라고 하고 명시적 변환과 암시적 변환이 있다.


```js
var a = 42;

var b = a + "";			// implicit coercion

var c = String( a );	// explicit coercion
```

<br>

## 4.2 추상연상(추상 동작/Abstract Operation)


추상연상은 ECMAScript language의 일부가 아닙니다. ECMAScript language를 의미론적으로 설명하기 위해 정의하였습니다.

(These operations are not a part of the ECMAScript language; they are defined here solely to aid the specification of the semantics of the ECMAScript language. )


<br>

### 4.2.1 ToString

#### **문자열화**

내장 원시값의 정해진 문자열화 방법 : ex) null -> "null" , undefined -> "undefined" , true -> "true"

숫자는 그냥 문자열로 바뀌고, 너무 작거나 큰 값은 아래 예시와 같이 지수 형태로 바뀐다.

```js
// multiplying `1.07` by `1000`, seven times over
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// seven times three digits => 21 digits
a.toString(); // "1.07e21"
```

일반객체는 *내부 `[[Class]]`* 를 반환한다. ex) `"[object Object]"`.

배열은 모든 원소 값을 콤마(,)로 분리하여 반환한다.

```js
var a = [1,2,3];

a.toString(); // "1,2,3"
```

toString() 메서드는 명시적으로도 호출 가능하다.
문자열 콘텍스트에 문자열이 아닌 값이 있는 경우에도 자동 호출된다.

<br>

#### **JSON 문자열화**

`JSON.stringify()` 메서드가 JavaScript 값이나 객체를 JSON 문자열로 변환할 때도 ToString 추상연상로직 담당한다.

`JSON.stringify()` 은 인자가 undefined, 함수, 심벌 값이면 자동으로 누락, 배열에 포함되어 있으면 null로 바꾼다.



```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (a string with a quoted string value in it)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
```


`JSON.stringify()` 에 참조객체를 `toJSON()`없이 넘기면 에러가 난다.

예를 들어,
```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// create a circular reference inside `a`
o.e = a;

// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function() {
	// only include the `b` property for serialization
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

아래를 코드를 보면, 
```js
var a = {
	val: [1,2,3],

	// probably correct!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// probably incorrect!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""
```


<br>

### 4.2.2 ToNumber

ture        --> 1

false       --> 0

undefined   --> NaN

null        --> 0



<br>

### 4.2.3 ToBoolean

* `undefined`
* `null`
* `false`
* `+0`, `-0`, and `NaN`
* `""`


falsy 객체는 유사 객체 ex) `document.all`
truty 값은 위 목록에 없는 모든 값.

<br>

## 4.3 명시적 강제변환
누구나 알 수 있게 대놓고 강제변환


### 4.3.1 문자열 <--> 숫자


`String(..)`, `Number(..)` 함수이용


```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```


```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

`+c`의 "+"는 단항연산자로 피연산자를 숫자로 강제변환한다.

```js
var c = "3.14";
var d = 5+ +c;

d; // 8.14
```

단항 연산자 강제변환은 날짜를 숫자로 강제변환할 때도 쓰인다.
날짜와 시간을 유닉스 타임스탬프 표현형으로 변환.

```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
```


#### **`~` Tilde**


비트 연산자는 피연산자를 암시 적으로 부호있는 32 비트 정수로 변환.

수학에서의 '~' 표기는 (Tilde)라고 부릅니다.  "x ~ y" means "x is equivalent to y".

(https://sites.google.com/site/springofmylifewiki/datastructure/partition-adt/partition-equivalence-relation)
(https://www.it-swarm.dev/ko/javascript/javascript%EC%97%90%EC%84%9C-double-tilde%EC%9D%98-%EA%B8%B0%EB%8A%A5%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9E%85%EB%8B%88%EA%B9%8C/970803933/)


```js
0 | -0;			// 0
0 | NaN;		// 0
0 | Infinity;	// 0
0 | -Infinity;	// 0
```


~는 2의 보수를 구한다.
(▶ 2의 보수 : 1의 보수에 1을 더한 값과 같습니다.​)

~42는 -(42+1)와 같다.
`-(x-+1)`를 -0(falsy)으로 만드는 유일한 값은 -1이다.

```js
~42;	// -(42+1) ==> -43
```

```js
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
	// found it!
}

~a.indexOf( "ol" );			// 0    <-- falsy!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// not found!
}
```

### 4.3.2 비문자열 파싱


`parseInt()` 함수는 문자열 인자의 구문을 분석해 특정 진수(수의 진법 체계에 기준이 되는 값)의 정수를 반환합니다.

`parseInt(string, radix);`

string :분석할 값. 만약 string이 문자열이 아니면 문자열로 변환(ToString 추상 연산을 사용)합니다. 문자열의 선행 공백은 무시합니다.

radix :string이 표현하는 정수를 나타내는 2와 36 사이의 진수(수의 진법 체계에 기준이 되는 값).


```js
parseInt( 1/0, 19 ); // 18
```


```js
parseInt( new String( "42") ); //42
```



```js
var a = {
	num: 21,
	toString: function() { return String( this.num * 2 ); }
};

parseInt( a ); // 42
```


```js
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```



<br>

### 4.3.3 * → 불리언


`Boolean(..)` 


```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```


<br>

## 4.4 암시적 강제변환
### 4.4.1 암시적이란?

만약 y값을 어떤 타입(SomeType)으로 변환할 때, 무조건 어떤 중간 단계를(AnotherType) 거쳐야 한다면, 코드는 아래와 같은 것이다.

```js
SomeType x = SomeType( AnotherType( y ) )
```

하지만, 우리가 쓰는 언어가 아래와 같이 "중간 변환 단계"를 생략할 수 있다면 지저분한 코드를 감추고 가독성을 향상 할 수 있지 않을까?

```js
SomeType x = SomeType( y )
```


<br>

### 4.4.2 문자열 <--> 숫자

#### - 숫자를 문자열로

문자열과 숫자를 암시적 강제변환되게 하는 연산.
`+` 연산자는 '숫자의 덧셈, 문자열 접합' 두 가지 목적으로 오버로드된다.

```js
var a = "42"; //변수 a는 문자열 "42"
var b = "0"; //변수 b는 문자열 "0"

var c = 42; //변수 c는 숫자 42
var d = 0; //변수 d는 숫자 0

a + b; // "420" 문자열 접합
c + d; // 42 숫자의 덧셈
```

연산자 오버로딩은 일반 연산을 위한 연산자가 다른 기능을 하도록 구현하는 것을 말한다. "+" 연산자가 피연산자에 따라 기능이 결정된다고 단순히 생각하지만 실은 그보다 더 복잡하다. 아래 예를 보면 변수 a와 b 모두 **문자열은 아니지만** 둘 다 문자열로 강제변환된 후 접합되었다.


```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

명세에 따르면 `+`알고리즘은 한쪽 피연산자가 문자열이거나 문자열 표현형으로 나타낼 수 있으면 문자열 붙이기를 한다. 따라서 피연산자 중 하나가 객체(배열 포함)라면, 원시값(number, string, default(true/false))으로 추상연산을 수행한다.

(valueOf()를 쓸 수 있고, 반환값이 원시값이면 그대로 강제변환하고 아니면 toString()을 이용해 강제변환 한다. 어찌해도 원시값으로 바꿀 수 없을 때 TypeError가 난다.)

예제의 경우, 단순 원시값을 반환할 수 없어 toString으로 넘어가고 그래서 배열 "1,2"와 "3,4"가 되고 +는 두 문자열을 붙여 최종 결괏값이 "1, 23, 4"가 된다.


대신 빈배열[] + 빈객체{}의 연산 결과는 [object Object]이고, 빈객체{} + 빈배열[]의 연산 결과는 0이다.

Why? 

```js

console.log([] == 0); //true
console.log({} == 0); //false

console.log([].valueOf()); // Array []
console.log({}.valueOf()); // Object {  }

console.log([] + {}); // "[object Object]"
console.log({} + []); // "[object Object]"


```

대표적인 암시적 강제변환은 숫자 + ""(공백문자열)이다. 공백문자열에 숫자를 더하면 문자열로 강제로 변환된다.

```js
var a = 42;
var b = a + "";

b; // "42"
```
문자열 연결 `+`는 대부분 가환적이지 않지만, "" 공백문자열은 특수한 경우라 가환적이다. 

그리고 공백 문자열 `+` 연결로 하는 암시적 강제변환과 String()으로 하는 명시적 강제변환은 둘 다 궁극적으로 문자열을 초래하지만, 일반적인 숫자값을 이용하는 대신 객체를 사용하는 경우 반드시 동일한 결과이 나오는 것은 아니다.  

원시값 추상 연상의 작동 방식 때문에 공백 문자열 연결은 valueOf()를 호출해 그 반환 값을 문자열로 변환하고, String은 toString을 직접 호출한다.

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

#### - 문자열을 숫자로
`-` 연산자는 숫자 뺄셈 기능이 전부이무로 a - 0은 a 값을 숫자로 강제변환한다. 


```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

`a * 1`이나 `a / 1` 연산자 역시 마찬가지다. 

객체 값에 -를 연산하면 문자열로 강제변환된 뒤 숫자열로 강제변환된다. 그리고 `-` 연산을 한다.

```js
var a = [3];
var b = [1];

a - b; // 2
```

```js
var a = [1, 2];
var b = [3, 4];

a - b; // NaN
```



<br>

### 4.4.3 불리언 → 숫자

```js
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```
위의 onlyOne()는 세 인자 중 하나만 true일 떄 `true`를 반환해야 한다. `!부정단항연산자`는 값을 불리언 값으로 명시적 강제변환한다. true인 경우에만 암시적 강제 변환을 하고 다른 부분은 명시적 강제변환을 한다.

이 코드로 여러 인자를 처리하기는 코드가 복잡해진다. 이 때, 불리언 값을 숫자로 변환하면 문제를 해결할 수 있다.

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// falsy 값은 0으로 취급해서 건너뛴다.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );		// true
onlyOne( b, a, b, b, b );	// true

onlyOne( b, b );		// false
onlyOne( b, a, b, b, b, a );	// false
```
 `0 + true` 은 toNumber 추상 연산에 의해서 `0 + 1`이 되기 때문에 `sum += arguments[i];` 에서 암시적 강제변환이 일어난다.

다음은 명시적 강제변환 예제이다.

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

> 저는 명시적 강제 변환이 더 나아 보입니다만....

<br>


### 4.4.4 * → 불리언

다음은 불리언으로 암시적인 강제변환이 일어나는 표현식의 열거이다.

+ if () 문의 조건 표현식
+ for( ; ; )에서 두 번째 조건 표현식
+ while () 및 do...while () 루프의 조건 표현식
+ ? : 삼항 연산 시 첫 번째 조건 표현식
+ || 및 && 의 좌측 피연산자

예제를 보자, 예제의 모든 비 불리언 값은 조건 표현식을 평가하기 위해서 불리언 값으로 강제변환된다.

```js
var a = 42;
var b = "abc";
var c;
var d = null;

if (a) {
	console.log( "넵" );		// 넵
}

while (c) {
	console.log( "절대 실행될 리 없지!" );
}

c = d ? a : b;
c;					// "abc"

if ((a && d) || c) {
	console.log( "넵" );		// 넵
}
```


### 4.4.5 && / ||

논리 연산자 ||(논리 or ) 및 &&(논리 AND) 연산자는 피연자사 값 중 하나를 결괏값으로 반환한다. 


```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

논리 연산자는 우선 첫 번째 피연산자 값을 평가하고 비 불리언이면 ToBloolean으로 강제변환 후 평가를 계속한다.

|| 연산자는 결과가 true이면 첫 번째 피연산자 값을, false이면 두 번째 피연산자값을 반환한다. && 연산자는 true일 때 두 번째 피연산자 값을, false일 때 첫 번째 피연산자값을 반환한다. 


|| 예제)

```js
function foo(a,b) {

	//true이면 첫 번째 피연산자 값을, false이면 두 번째 피연산자값
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

&& 예제)

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

위 예제는 a가 참일 때 foo()를 호출하고, 이런 특성을 가드 연산자라고 한다. 첫 번째 표현식이 두번째 표현식의 가드역할을 하는 것이다. 

논리연산자의 결과가 true/fasle는 아니지만 불리언 타입으로 암시적 강제변환을 사용하기 때문에 조건식을 사용하는데 아무 문제가 없을 수 있었다.


```js
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
	console.log( "넵!" );
}
```


### 4.4.6 심벌의 강제변환
심벌은 암시적 강제변환이 금지되며 시도만 해도 에러가 난다. 하지만 불리언 값으로는 명시적/암시적 강제변환이 가능하다.

## 4.5 느슨한/엄격한 동등 비교
느슨한 동격비교 `==` 는 강제변환을 허용하지만 엄격한 동격비교 `===` 는 강제변환을 허용하지 않는다.

### 4.5.1 비교 성능
타입까지 값은 두 값의 동등 비교라면 두 가지 타입의 동등비교의 알고리즘은 동일하다. 성능에 큰 차이가 없으니 강제변화이 필요하면 느슨한 비교를 아니라면 엄격한 비교를 사용하자.


### 4.5.2 추상 동등 비교
ES5 11.9.3 추상적 동등 비교 알고리즘 명세의 첫째 항에는 다음과 같이 쓰여있다. 비교할 두 값의 타입이 같으면 값을 식별하여 간단히 비교할 수 있지만 예외가 있다.

+ NaN은 그 자신과도 결코 동등하지 않다.
+ +0 와 -0은 동등하지 않다.

마지막 항에는 객체의 느슨한 동등 비교에 대해 두 객체가 정확히 똑같은 값에 대한 레퍼런스일 경우에만 동등하다고 기술되어있다. 

나머지 부분에는 타입이 다른 두 값을 비교 시, 한쪽 또는 양쪽 피연산자를 어떻게 암시적 변화해야하는지 쓰여있다. 결과적으로 두 값을 타입이 일치된 상태에서 간단히 값만 비교하기 위함이다.

그리고 비동등연산자는 동등연산자를 그대로 부정한 값이다.

#### 비교하기 문자열 → 숫자

```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```
이 경우 ES5 11.9.3.4-5 원문은 다음과 같이 설명하고 있다.

+ Type(x) 가 숫자이고 Type(y)과 문자열이면, x == ToNumber(y) 비교값을 반환한다.
+ Type(x) 가 문자열이고 Type(y)과 숫자이면, ToNumber(x) == y 비교값을 반환한다.

#### 비교하기 * → 불리언


```js
var a = "42";
var b = true;

a == b;	// false
```

이 경우 ES5 11.9.3.6-7 원문은 다음과 같이 설명하고 있다.

+ Type(x) 가 불리언이면 ToNumber(x) == y의 비교 결과를 반환한다.
+ Type(y) 가 불리언이면 x == ToNumber(y) 의 비교 결과를 반환한다.


```js
var x = true; //1
var y = "42";

x == y; // false
```
 x는 불리언이므로 숫자로 강제변환되고, 1 == "42"이 된다. 타입이 다르므로 다시 알고리즘을 수행하고. "42"는 42로 바뀌어 결국 값이 false가 된다. 

이번에는 y가 불리언이다.

```js
var x = "42";
var y = false; //0

x == y; // false
```

결론적으로 "42"는 == true 도 == false도 아니게 되는데, 피연산자 한쪽이 불리언값이면 다른 한쪽으로 예외 없이 그 값이 숫자로 강제 변환되므며 == true / == false 같은 코드는 쓰지말아야 한다.


```js
var a = "42";

// bad (will fail!):
if (a == true) {
	// ..
}

// also bad (will fail!):
if (a === true) {
	// ..
}

// good enough (works implicitly):
if (a) {
	// ..
}

// better (works explicitly):
if (!!a) {
	// ..
}

// also great (works explicitly):
if (Boolean( a )) {
	// ..
}
```

#### 비교하기 null → undefined
null과 undefined를 느슨한 동등비교 `==`하면 서로에게 타입을 맞추는 방식으로 암시적 강제변환이 일어난다. 

ES5 11.9.3.2-3 인용.
1. x가 null이고 y가 undefined면 true 를 반환한다.
2. x가 undefined이고 y가 null면 true 를 반환한다.

```js
var a = null;
var b;

a == b;		// true
a == null;	// true
b == null;	// true

a == false;	// false
b == false;	// false
a == "";	// false
b == "";	// false
a == 0;		// false
b == 0;		// false
```

null과 undefined 강제변환은 안전하고 예측가능하며 긍정오류를 할 가능성이 없기 때문에 두 값을 동일 값으로 취급하는 강제변환은 권장하고 싶다.

예를 들면,

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

`a == null`은 `doSomething()`이 null이나 undifined 를 반환하는 경우에만 true이고, 0이나 false, "" 등 다른 falsy한 값이 넘어와도 false이다.

아래 코드보다 위의 코드를 권장한다. 가독성 좋고 안전하게 작동하는 암시적 강제변환의 예이다.

```js
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}
```


#### 비교하기 객체 → 비객체

객체/함수/배열과 단순 스칼라 원시 값(문자열, 숫자, 불리언)의 비교는 ES5 11.9.3.8-9에서 다룬다.

1. Type(x)가 String 또는 Number이고 Type(y)가 객체라면 x == ToPrimitive(y)의 비교 결과를 반환한다.
2. Type(x)가 객체이고  Type(y)가 String 또는 Number라면, ToPrimitive(x) == y 의 비교 결과를 반환한다.


```js
var a = 42;
var b = [ 42 ];

a == b;	// true
```
[ 42 ]는 ToPrimitive 추상연산결과 42가 된다. a, b는 동등하다.

```js
var a = "abc";
var b = Object( a );	// same as `new String( a )`

a === b;				// false
a == b;					// true
```

느슨한 동등 비교의 알고리즘은 b를 언박싱하여 단순 스칼라 원시 값으로 강제변환하고, 따라서 `a == b`는 true가 된다. 하지만 항상 그런 것은 아니다.


```js
var a = null;
var b = Object( a );	// same as `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// same as `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// same as `new Number( e )`
e == f;					// false
```

null과 undefined는 Object()로 해석되어 false이고, NaN은 자기 자신과도 같지 않으므로 false가 된다.

### 4.5.3 희귀사례
#### 알 박힌 숫자값

아래는 내장 네이티브 프로토타입을 변경하면 안되는 예이다.

```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```
`new Number( 2 )` 는 무조건 ToPrimitive 강제변환 후 valueOf()를 호출한다. 따라서  `new Number( 2 ) == 3`는 true가 된다.

아래 예는, `a == 2`가 `a == 3` 보다 먼저 평가되기 때문에 `a == 2 && a == 3` true를 반환한다.

```js
var i = 2;

Number.prototype.valueOf = function() {
	return i++;
};

var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "이런, 정말 되는구만!" );
}
```


#### Falsy 비교
아래는 falsy값 비교에 관한 희귀 사례 목록이다.

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false
```
UH OH! 주석이 붙은 7개의 비교는 결과가 false일 것 같지만 모두 true를 반환하는 긍정오류이다. 

#### 말도 안되는..(The Crazy Ones)

```js
[] == ![];		// true
```
![]는 ! 단항 연산자로 인해 false로 바뀌고, `[] == false`되어 true를 반환한다. 


```js
2 == [2];		// true
"" == [null];	// true
```

우변의 `[2]`와 `[null]`은 ToPrimitive가 강제 변환을 하여 2와 ""로 바뀌고, 배열의 valueOf()는 자신의 배열을 반환하므로 강제변환 시 배열을 문자열화한다.

다음 사례도 유명하다.

```js
0 == "\n";		// true
```
공백 문자가 ToNumber를 경유하여 0으로 강제 변환된다.

```js
42 == "43";							// false
"foo" == 42;						// false
"true" == true;						// false

42 == "42";							// true
"foo" == [ "foo" ];					// true
```

위의 코드를 24개의 falsy비교의 희귀사례 목록과 대보해조면 전부 설명 가능하다는 것을 알 수 있다.

#### 근본부터 따져보자

24개 말이 안되는 7개는 아래와 같다.

```js
"0" == false;			// true -- UH OH!
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

처음 4개는 `==  false` 비교와 연관된다. 나머지 3개는 발생할 가능성이 아주 희박하다.


#### 암시적 강제변환의 안전한 사용법

+ 피연산 중 하나가 true/false일 가능성이 있으면 절대로 `==` 연산자를 쓰지 말자.
+ 피연산 중 하나가 [], "", 0이 될 가능성이 있으면 가급적 `==` 연산자를 쓰지 말자.


typeof 연산자는 항상 7가지 문자열 중 하나를 반환하므로 값의 타입을 체크한다고 암시적 강제변환과 문제될 일은 전혀 없다. 따라서 `typeof x == "function"` / `typeof x === "function"` 둘다 안전하다. 

<br>

## 4.6 추상 관계 비교
`a < b` 비교는 우선 두 피연산자의 원시값을 도출한 후 어느 한쪽이라도 문자열이 아는 경우 양쪽 모두 숫자로 강제변환하여 비교한다. 


```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

그러나 비교 대상이 모두 문자열이면 문자를 알파벳 순서로 비교한다. 문자 단위로 4와 0을 비교하면 비교는 처음부터 false를 반환하고 끝난다.


```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```


```js
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];

a < b;	// false
```

변수가 객체인 경우는 아래 a/b 둘다 `[object object]` 이기 때문에 비교가 불가능하다.

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false : 별개의 객체의 이기 때문에 완전 같지 않다. 
a > b;	// false

a <= b;	// true
a >= b;	// true
```
`a <= b`  는 `b < a` 결과를 부정하도록 명세에 기술되어 있기 때문에 `b < a`가 false이므로 `a <= b` 는 true가 된다. 

안전한 관계 비교라면 비교시 암시적 강제변환을 고려하지 않고 사용해도 좋지만, 조심해야할 경우라면 미리 타입을 변환해두고 사용하자.

```js
var a = [ 42 ];
var b = "043";

a < b;						// false -- string comparison!
Number( a ) < Number( b );	// true -- number comparison!
```


<br>

## 4.7 정리하기
명시적 강제변환은 변환 의도가 확실한 코드, 암시적 강제변환은 숨겨진 로직에 의한 부수적인 변환이 처리되는 과정이다. 암시적 강제변화은 코드 가독성 향상에 도움이 되어 사용해도 좋지만, 어떤 변환 과정을 거치는 알고 사용해야한다.




