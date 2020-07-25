- Scope 동작 방식
    - Lexical-scope
    - Dynamic-scope

    ```jsx
    function foo() { 
    	console.log( a ); 
    }
    // lexical:2, dynamic:3

    function bar() { 
    	var a = 3;
    	foo(); 
    }

    var a = 2;
    bar();
    ```

## 1. Lex-time

- compile 첫 단계, 소스코드를 의미적으로 할당, 상태를 parsing하여 token을 만든다.
- lexical scope는 이 단계에서 정해진다. 즉 소스코드를 작성하는 시간에 정해진다.

```jsx
function foo(a) { 
	var b = a * 2;

	function bar(c) {
	  console.log(a, b, c);
	}

	bar(b * 3);
}

foo( 2 ); // 2, 4, 12
```

- 모든 scope들은 하나의 부모를 가진다.

## 2. Look-ups

- scope3의 console.log(a, b, c)에서 a,b,c를 찾는다.
    - a: scope3에 없음 → scope2에 있음!(foo()의 argument)
    - b: scope3에 없음 → scope2에 있음(var b = a * 2)
    - c: scope3에 있음(bar()의 argument)

- **Shadowing**
    - scope look-up은 첫번째 매치가 이루어지면 더이상 진행하지 않는다. 그래서 상위 scope에 같은 identifier name이 있더라도 사용하지 않고 가려진다. 이를 Shadowing이라고 한다.
    - Q. c가 bar(..)에도 있고, foo(..)에도 있다고 책에 나와 있는데, bar(b * 3)을 말하는건가?
    - global object 같은 경우에 "window.a"와 같이 golobal object의 property 로 접근해서  shadowing으로 접근 불가능한 상위 scope의 identifier에 접근할 수 있다.
    - 이와 같이 Lexical scope는 함수가 선언이 될 때 정의 된다. 그리고 lexical-scope look-up 프로세스는 `first-class-identifiers` 에만 적용된다.
    - first-class-citizens

        > 1. 모든 요소는 함수의 실제 매개변수가 될 수 있다.
        > 2. 모든 요소는 함수의 반환 값이 될 수 있다.
        > 3. 모든 요소는 할당 명령문의 대상이 될 수 있다.
        > 4. 모든 요소는 동일 비교의 대상이 될 수 있다.

    - foo.bar.baz 라는 코드가 있다면 lexical scope look-up은 `foo` identifier를 찾는데 까지 적용 된다.
    - bar, baz를 찾는데는 object-property access rule이 적용 된다.

    cf) dynamic-scope

## 3. Cheating Lexical

- runtime 때 Lexical scope를 변경하는 방법에 대해 소개
- **eval()**
    - eval의 argument의 string을 eval문이 선언된 곳에서 author-time(a.k.a lexical) 인 것처럼 다룬다.

    ```jsx
    function foo(str, a) {
    	eval( str ); // cheating!
      console.log( a, b );
    }

    var b = 2;

    foo("var b = 3;", 1); // 1, 3
    ```

    - b = 3이 출력, `eval("var b = 3;")` 이 runtime때 lexical-scope를 변경한다.

        그래서 global의 2가 아닌 3이 출력.(global b shadowing, lexical-scope cheating)

    - 위 예제는 단순히 eval에 static한 string을 넣었지만, dynamic하게 코딩을 하여 복잡하게 lexical-scope를 변경할 수 있다. 하지만 이는 authoring code directly 하는 것 외의 장점이 없다.(author-time인 것처럼)

    ```jsx
    function foo(str) {
    	"use strict";
    	eval( str );
    	console.log( a ); // ReferenceError: a is not defined
    }

    foo( "var a = 2" );
    ```

    - lexical-scope내 `use strict`를 사용하면, lexical-scope 변경을 막아준다.
    - eval(..), setTimeout(..), setInterval(..)
        - 첫번째 argument에 string code를 작성하면 lexical-scope을 변경할수 있다.
        - [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/eval](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/eval)
    - new Function(..)
        - 마지막 argument(리턴값 정의)에 string code를 작성하면 lexical-scope 변경할 수 있다.
        - [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function)

- with - now deprecated!

```jsx
// with 문이란???
var obj={ 
	a: 1,
	b: 2,
  c: 3,
}

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) { 
	a = 3;
	b = 4;
	c = 5; 
}
```

```jsx
function foo(obj) {
  // var a = 99; -> foo() scope!!!
	with (obj) {
		a = 2; 
	}
}

var o1 = { 
	a:3
};

var o2= { 
	b:3
};

foo(o1);
console.log(o1.a); // 2

foo(o2);
console.log(o2.a); // undefined 
console.log(a); // 2 — Oops, leaked global!
```

- with문에서 받은 object가 독립된 lexical-scope를 가지고 있는 것처럼 다룬다.(function처럼)

    그래서 obejct의 property들을 lexically defined identifier로 다룬다.

- 그래서 object o2에 a가 없음 → foo에서 찾아도 없음 → global에서 a 만들어서 할당(`non-strict mode`)
- `but` with block에 정의된 var 변수는 function scope에 포함된다.

    위 예시에서는 var a 는 foo function scope에 포함된다.

## 4. Performance

- tokenize/lexing을 하여 code를 실행하기 위해 최적화를 정적으로 진행하는데, eval이나 with문은 runtime 떄 새로운 lexical scope를 생성하기 때문에 이를 방해한다. 그래서 퍼포먼스가 떨어진다. 쓰지맙시다!
