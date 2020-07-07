# Ch3. 네이티브

일반적으로 아래와 같은 내장함수를 `네이티브`라 한다.

- String()
- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol()

`내장함수 == 네이티브` 이다.

> 네이티브란 특정 환경(브라우저, 클라이언트 등의 환경)에 종속되지 않은 ECMAScript에 명시된 내장함수를 의미함.
window나 Button같은것은 네이티브가 아님.

내장함수로 실행해서 얻은 결과물은 object 타입이다.

```jsx
var a = new String("test");
typeof a;
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fa7ceed-514c-4745-86a8-ea1decce92be/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fa7ceed-514c-4745-86a8-ea1decce92be/Untitled.png)

실행 결과

object내부에서 각 index 별로 한글자씩 들어간것을 확인할 수 있다.

단,`__proto__`는 String 인걸 볼 수 있다.

## 3.1 내부 [[Class]]

---

typeof 가 "object"인 값에는 [[Class]] 라는 내부 프로터피를 가지고있다.

직접 접근은 불가능하고, Object.prototype.toString() 으로 존재를 확인할 수 있다.

그럼 위에 실행결과 스크린샷에서 보이는 [[PrimitiveValue]] 는 뭐지?

[[Class]]는 보통 내장 네이티브 생성자를 가리키지만 아닌 경우도 있다.

원시값의 내부에 [[Class]]는 두가지 타입으로 나뉜다.

Null & Undefined

boolean & number & string

자동으로 Boxing 과정을 거치게 됨

??? 아니 근데 null & undefined랑 나머지랑 무슨 차이가 있다는거지?
똑같은거 아닌가...??

## 3.2 래퍼 박싱하기

---

원시값에는 프로퍼티나 메서드가 없다. 따라서 이론상 아래와 같은 코드는 동작할 수 없다. 

```jsx
var myString = "I am a frontend developer".
myString.length;
```

왜냐하면 myString 은 원시값이기 때문이다.

하지만 문제없이 동작된다. 왜냐하면 자바스크립트는 알아서 박싱(래핑) 하기 때문이다.

오래전부터 브라우저가 위와같은 래핑작업을 스스로 최적화 해왔다.
개발자가 직접 최적화를 한다면 오히려 더 느려질 수 있다. 

따라서 new String("my String"); 과 같이 코드를 짜기보단 "my String" 을 사용하여 브라우저가 최적화 하도록 맡기는것이 좋다.

그럼 왜 오히려 더 느려질 수 있는거지??
크게 다른건 없는것같은데??

래핑해서 사용할때 Boolean은 조심하는것이 좋다.

```jsx
var isSuccess = new Boolean(false);

if (!isSuccess) {
	console.log('Error!');
}
```

isSuccess는 Boolean으로 랩핑한 object 이기 때문엔 true가 되기 때문에 조건문에서 오류를 범하게 된다.

박싱하려면 Object() 를 활용하는것이 좋다. 

```jsx
var a = 'my string';
var b = new String(a);
var c = Object(a);

typeof a;  // "string"
typeof b;  // "object"
typeof c;  // "object"

b instanceof String;  // true
c instanceof String;  // true

Object.prototype.toString.call(b);  // [object String]
Object.prototype.toString.call(c);  // [object String]
```

근데 뭐가 다르길래 Object()를 활용하는게 좋다고 하는거지?? 

걍 하지 말라면 하지 말자....
이런거 별로 좋지 않다ㅠ 

## 3.3 언박싱

---

언박싱은 valueOf() 를 사용한다. 

```jsx
var a = new String("abc");
var b = new Number(28);
var c = new Boolean(true);

a.valueOf();  // "abc";
b.valueOf();  // 28;
c.valueOf();  // true;
```

다른 타입과 연산할 때 암묵적으로 형변환이 발생한다.

```jsx
var a = new String("abc");
var b = a + "def";

typeof a;  // "object";
typeof b;  // "string";
```

a + "def" 를 연산할때 a 는 object가 아닌 string 으로 계산되기 때문이다.

## 3.4 네이티브, 나는 생성자다.

---

결론적으로 리터럴 형태로 생성하는것이 좋다. 

하지만 결과적으론 리터럴 형태와 같은 객체를 생성한다. (래핑되지 않은 값은 없다.)

> 앞에서 살펴본 다른 네이티브도 그랬듯이, 확실히 필요해서 쓰는게 아니라면 **생성자는 가급적 쓰지 않는 편이 좋다.** 나중에 별별 오류와 함정에 빠져 고생하고 싶지 않으면 말이다.

~~⇒ 뒤지기 싫으면....?~~ 

⇒ 생성자는 가급적 쓰지 않는 편이 좋다.

### Array

```jsx
var a = new Array(1, 2, 3);
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

Array() 생성자 앞에 붙은 new가 있건 없건 동일하게 동작한다. 
Array() 와 new Array() 는 동일하다.

그럼 new 는 뭘 의미하는거지?

Array의 생성자는 두개인데, 하나는 위에 예제처럼 여러개의 인자를 주면 각각의 element가 되지만 number타입 하나만 인자로 넘기면 해당 크기만큼의 size로 하는 배열이 생성된다.

**구멍난 배열 - Sparse Array**

```jsx
var a = new Array(3);
var b = [undefined, undefined, undefined];
var c = [];
c.length = 3;

a;
b;
c;
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab5b06ef-041a-42d0-9da9-285f1c42d53e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab5b06ef-041a-42d0-9da9-285f1c42d53e/Untitled.png)

실행 결과

각각 브라우저에서 표기되는 내용은 다르다.

과거에는 [undefined x 3] 이라고 표기되었지만 지금은 [empty x 3] 라고 표기되는듯 하다.

실행 결과를 보면 알 수 있지만 b만 [undefined, undefined, undefined] 로 표기된다.

⇒ 결과적으론 이런 Sparse Array는 사용하지 말아야 한다.

- Sparse Array의 TMI

**빈 배열로 채워서 생성하는 방법**

```jsx
var a = Array.apply(null, {length: 3});
a;  // [undefined, undefined, undefined]
```

Apply  에 대한 자세한 내용은 아래 링크 참조

[call, apply 그리고 bind](https://www.notion.so/call-apply-bind-182276d09e0643f0953ab93b05c99069)

### 3.4.2 Object(), Function(), and RegExp()

위 3개도 선택적으로 생성자를 사용할 수 있다.

```jsx
var c = new Object();
c.foo = 'bar';
c;  // { foo: 'bar' }

var d = { foo: 'bar' };
d;  // { foo: 'bar' }

var e = new Function('a', 'return a * 2');
var f = function(a) { return a * 2; };
function g(a) { return a * 2; };

var h = new RegExp('^a*b+', 'g');
var i = /^a*b+/g;
```

**Object** 

생성자가 있긴 하지만 의미가 없음.

차라리 리터럴 형태로 생성하는게 더 좋음

**Function**

매우 드문 경우로 생성자를 사용할 때가 있다고 함.

절대 eval() 처럼 생각하고 사용하면 안됨.

**RegExp**

정규식은 성능상 이점이 있어 리터럴 형태를 적극 권장한다.

엔진이 실행전에 정규 표현식을 미리 컴파일 한 후 캐시한다. 

### 3.4.3 Date() and Error()

Date와 Error는 리터럴 형태를 지원하지 않는다.

date는 new Date() 로 생성한다. 인자로는 날짜/시각을 받는다.

unix 타임 스탬프 값을 얻는 용도로 많이 사용한다.

Error는 앞에 new 가 있던 없던 동일하다.

주로 사용용도는 현재의 실행 스텍 콘텍스트를 포착하여 객체에 담는것이다. 

위에가 무슨말을 하는건지 좀더 자세히 알아보자...

함수 호출 스택과 error 객체가 만들어진 줄 번호등 디버깅에 도움될만한 정보를 담고 있다.

```jsx
function foo(x) {
	if (!x) {
		throw new Error('x is undefined');
	}
}
```

Error 내부에는 message 프로퍼티와 type등의 다른 프로퍼티가 있다. 

- 추가적인 Error 타입

### 3.4.4 Symbol()

Symbol 은 ES6에 추가되어 충돌 염려없이 객체 프로퍼티로 사용 가능한 유일값이다. 

하지만 절대적으로 유일함이 보장되지는 않는다.

⇒ 무엇??

미리 정의된 Symbol이 있는데, Symbol.create, Symbol.iterator 이다.

```jsx
obj[Symbol.iterator] = function() {};
```

근데 언제 쓰는거지?

새롭게 정의하려면 new를 빼고 네이티브 함수를 호출하면 된다. 

new를 붙이면 에러가 나는 유일한 네이티브다.

```jsx
var mySymbol = Symbol('my symbol');
mySymbol;

mySymbol.toString();
typeof mySymbol;

var a = {};
a[mySymbol] = 'foo';

Object.getOwnPropertySymbols(a);
```

Symbol은 private 프로퍼티는 아니지만 암묵적으로 내부 프로퍼티로 사용한다. 과거 __value 과 동일한 의미이다.

Symbol은 객체가 아닌 primitive value 이다.

### 3.4.5 네이티브 프로토타입

내장 네이티브 생성자는 각각 .prototype 객체를 가진다.

.prototype 객체에는 해당 객체의 하위 타입별로 고유한 로직이 있다.

예를 들면 아래와 같은것들이 있다.

- String#indexOf()
- String#charAt()
- String#substr(), String#substring(), String#slice()
- String#toUpperCase(), String#toLowerCase()
- String#trim

`Prototype delegation` 덕분에 모든 문자열이 이 메서드를 같이 사용할  수 있다. 

prototype delegation 이란?

**프로토타입은 디폴트다.**

변수에 적당한 값이 할당되지 않은 상태에서 프로토타입은 디폴트 값이 들어가있다.

Function → 빈 함수

Regexp → 빈 정규식

Array → 빈 배열

프로토타입으로 값을 세팅하면 성능적 이점이 있다.

프로토타입은 내장된 값이라서 한번만 생성한다. 하지만 따로 디폴트 값을 표기한다면 새롭게 생성하므로 성능이 떨어질 수 있다.

⇒ 하지만 이게 엄청 미미할텐데....ㄷㄷ

## 3.5 정리하기

---

자바스크립트는 원시값을 감싸는 객체 래퍼, 네이티브를 제공한다.

네이티브에 정의된 메서드를 사용할 수 있다.

```jsx
"abc".length;
```

위 코드에서 "abc"는 boxing 되어 String 이 되고, String.prototype에 정의된 프로퍼티에 접근할 수 있다.

개인적인 생각으론 네이티브 프로토타입을 절대 변경하지 말자.

어떤 사이드 이펙트가 생길지 모른다.
