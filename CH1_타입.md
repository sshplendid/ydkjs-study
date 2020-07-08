- javascript에서 타입은 dynamic하게 부여된다. -> 타입이 없는 것이 아니다.

## **1 . A Type by Any Other Name...**

- rough하게 정의하자면 value를 구분하기 위한 용도이다.(ex 42 와 '42')
- type을 이해하는 것은 형변환을 이해하는 것과 같다고 볼 수 있다.
- 명시적이나 묵시적으로 형변환이 일어나는데 이것을 올바르게 다루는 것이 중요하다.  완벽하게 이해한다면 강력하고 유용하다.

## **2. Built-in Types**

- built-in types(primitives): null, undefined, boolean, number, string, object, symbol
- typeof로 null을 제외한 나머지 타입은 1:1로 매칭, null은 object로 매칭

      (typeof null === "null" => false) --> it's buggy

- test for null

```jsx
(!a && typeof a === "object"); // true
```

- function: subtype of object(callable object)) [[Call]] property
- function은 callable object라서 length property를 가지고 있어 유용하게 쓸 수 있다. (argument의 길이 반환)
- array는 object!(Native chapter 참조)

## **3. Values as Types**

- 변수가 타입을 가지는 것이 아니라 값이 타입을 가진다. 변수는 언제나 모든 값을 가질 수 있다.
- JS는 타입 강제성이 없다. 변수가 특정 타입으로 초기화 될 필요가 없다.

      그래서 처음에 string을 할당하고 다음에 number나 다른 타입을 할당할 수 있다.

- typeof 를 쓸 때 "변수가 무슨 타입이지?" 가 아니라 변수가 어떤 타입의 값을 가지고 있는지를 생각해야 한다.
- typeof는 string Return한다.

```jsx
typeof 42 // 'number'
typeof typeof 42 // 'string'
```

## **4. undefined Versus "undeclared"**

- 값을 할당하지 않은 변수는 undefined
- undefined: 접근 가능하도록 선언되었으나 값이 할당되지 않음
- undeclared: 접근 가능하도록 선언이 되지 않음(not defined 에러로 뜸 헷갈림 !== undefined)
- typeof는 undeclared 한 변수도 "undefined"를 리턴한다.(safe guard in the behavior of typeof)

## **5. typeof Undeclared**

- debug 모드를 전역 변수로 관리할 때,  DEBUG 변수가 선언되었는지  상관없이 체크 가능

```jsx
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

- var를 쓰면 if 조건을 만족하지 않아도 atob를 hoisting하여 atob를 글로벌로 만든다. 이미 global로 atob가 있다면 충돌이나서 에러가 날 수 있다.

```jsx
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}
```

- global 변수는 브라우저에서 global object인 window의 property로 존재한다. 그래서 'window.'으로 접근해서 check 가능. 없는 property 접근해도 undefined를 return하지 오류가 나지 않는다.

```jsx
if (window.DEBUG) {
// ..
}
if (!window.atob) {
// ..
}
```

- node에서는 global object가 window가 아니기 때문에 주의해서 써야한다.
- typeof를 통해서 선언을 파악할 수도 있고, DI pattern을 통해서도 파악할 수 있다.

```jsx
// global에 FeatureXYZ가 있으면 가져다 쓰고 없으면 아래 선언된 걸 쓴다.
function doSomethingCool() {
        var helper =
            (typeof FeatureXYZ !== "undefined") ?
            FeatureXYZ :
            function() { /*.. default feature ..*/ };
        var val = helper();
// .. }
```

```jsx
// FeatureXYZ가 글로벌은 아니지만 아래처럼 선언을 체크할 수 있다.
// an IIFE (see the "Immediately Invoked Function Expressions"
    // discussion in the Scope & Closures title in this series)
(function(){
    function FeatureXYZ() { /*.. my XYZ feature ..*/ }
    // include `doSomethingCool(..)`
    function doSomethingCool() {
        var helper =
            (typeof FeatureXYZ !== "undefined") ?
            FeatureXYZ :
            function() { /*.. default feature ..*/ };
        var val = helper();
				// .. 
		}
    doSomethingCool();
})();
```

```jsx
//DI Pattern
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
  function() { /*.. default feature ..*/ };
  var val = helper();
	// .. 
}
```
