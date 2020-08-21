# CH2. this가 이런 거로군

## intro
> this 는 까다로운 면이 있다. 이번장에서는 this 바인딩의 우선 순위와 예시들을 알아보고, 실전에서는 명확한 this를 사용할 것인지 아니면 렉시컬 스코프를 사용할 것인지 코드 스타일을 통일하도록 하자.

## 2.1 호출부
this 바인딩은 오직 호출부(선언이 아님)와 연관되기 때문에 호출 스택에서 진짜 호출부를 찾아내려면 코드를 주의깊게 봐야 한다.
- 호출부란 : 함수를 호출한 지점
- 호출 스택이란 : 현재 실행 지점에 오기까지 호출된 함수들이 쌓여온(stack) 순서

**호출부와 호출 스택의 예제**
```jsx
function baz() {
  // 호출 스택 : 'baz'
  // 호출부는 전역 스코프 내부
  bar(); // bar의 호출부
}

function bar() {
  // 호출스택 : 'baz'->'bar'
  // 호출부는 'baz' 내부
  foo(); // foo의 호출부
}

function foo() {
  // 호출스택 : 'baz' -> 'bar' -> 'foo'
  // 호출부는 'bar'내부
}
baz(); // baz의 호출부
```

## 2.2 단지 규칙일 뿐
this가 무엇을 참조하는지에 대한 4가지 규칙을 소개한다. 이후 규칙이 중복될 때 우선순위도 소개한다.

### 2.2.1 기본 바인딩
- 함수 단독 실행에 관한 규칙
- 나머지 세규칙에 해당하지 않을때에 적용되는 규칙
- this는 전역 객체를 의미하게 된다.

**기본 바인딩 예제**
```jsx
function foo() {
  console.log(this.a); //this가 전역객체와 바인딩되고, a는 전역객체 프로퍼티를 의미하게 된다.
}
var a = 2;

foo(); 
```

다만 엄격 모드에서는 전역 객체가 기본 바인딩 대상에서 제외된다.

**엄격 모드 예제** 
```jsx
function foo() {
  "use strict";
  console.log(this.a);
}
var a = 2;
foo(); // Uncaught TypeError: Cannot read property 'a' of undefined
```

그리고 또 다만, use strict 는 호출부의 엄격 모드에서는 상관없다.

**호출부 엄격 모드 예제**
```jsx
function foo() {
  console.log(this.a);
}
var a = 2;

(function() {
  "use strict";
  foo(); //2
})();
```

- 저자는 엄격 모드와 비엄격 모드를 마구 섞어쓰지 말라고 하고 있고, 3rd party 라이브러리를 사용할 때 mode 체크를 해서 호환성을 확인하라고 조언한다.

### 2.2.2 암시적 바인딩
- 호출부에 콘텍스트 객체가 있다면, 즉 함수 호출 시점에서 객체의 콘텍스트로 함수를 참조하는 식으로 호출한다면(저자는 이를 객체가 호출시점에 함수의 레퍼런스를 '소유'하거나 '포함' 한다고 본다) 객체가 this에 바인딩된다.
```jsx
function foo() {
  console.log(this.a);
}
var obj = {
  a:2, 
  foo:foo
}
obj.foo(); // obj에서 foo 를 참조하여 호출한 것이라, this는 obj가 되어서 결과는 2
```

- 아래와 같이 체이닝된 형태도 참고해 보자
```jsx
function foo() {
  console.log(this.a);
}
var obj2 = {
  a: 42,
  foo: foo
};
var obj1 = {
  a: 2,
  obj2: obj2
};
obj1.obj2.foo(); // foo 호출부인 obj2 가 this가 되어 결과는 42
```

#### 암시적 소실
- 암시적 바인딩이 이루어지지 않는 경우이다. this 바인딩이 헷갈리기 쉬운 경우 중 하나
- 객체의 프로퍼티를 통해 참조하며 함수 호출이 이루어진다면 해당 객체에 바인딩되나, 해당 객체의 레퍼런스를 할당받아 함수를 호출한다면 할당받은 객체의 호출에 의존해 this가 결정된다.

**암시적 소실 예제**
```jsx
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2, // obj의 property
  foo: foo
};
var bar = obj.foo;  //obj.foo 는 global의 foo를 참조하고 있는데 이를 bar가 가르키게 되는 것이다.
var a = "it's global";  //a는 global property
bar();  // it's global 이라는 결과가 나온다. bar의 호출부가 global이기 때문에 this는 global 로 바인딩되었기 때문->기본 바인딩으로 적용됨
```

**콜백 전달로 인한 암시적 소실**
```jsx
function foo() {
  console.log(this.a);
}
function doFoo(fn) {
  fn(); // 이곳이 호출부이다. 만약 여기 바로위에 a 프로퍼티를 선언했으면 해당 프로퍼티가 출력될 것이다.
}
var obj = {
  a: 2,
  foo: foo
};
var a = "it's global";
doFoo(obj.foo); // it's global 이라는 결과가 나온다.
```
- 암시적 this바인딩을 사용할 때는 콜백 과정에서 this 행방이 묘연해지는 경우 (예 : this가 이벤트를 유발한 DOM 요소를 가리키도록 강제하는 경우) 가 있으니 주의하도록 한다. (구체적인 사례는 모르겠음..)
- 그래서 함부로 this를 바꾸지 못하게 고정하는 명시적 방법 및 하드바인딩을 사용할 수 있음.

### 2.2.3 명시적 바인딩
- 자바스크립트 함수의 call() 과 apply() 메소드를 이용하여 어떤 객체를 this 바인딩에 이용하겠다는 의지를 명확히 표현하는 방법
- 여기서 파라미터로 객체 대신 단순 원시값을 넣으면 원시 값에 대응되는 객체로 래핑(new String(), new Boolean(), new Number())되는데 이 과정을 박싱이라고 한다.

**명시적 바인딩 예제**
```jsx
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2
};
foo.call(obj);  // 2 가 출력됨
```

#### 하드 바인딩
- 명시적 바인딩에서 call(obj) 와 같이 오브젝트를 this에 매핑하는 부분을 하드코딩하여 새로운 함수로 만들어, 해당 새 함수를 호출하면 무조건 해당 obj 객체에 this가 매핑되도록 하는 것을 하드 바인딩이라고 한다.

**하드 바인딩 예제**
```jsx
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2
};

var bar = function() {
  foo.call(obj);
};
bar(); // 이는 당연히 2를 출력하겠으나
setTImeout(bar, 100); // 내장함의 콜백함수로 bar를 실행시키도록 해도 2가 나오고
bar.call(window); // 명시적으로 window에 this로 바인딩 하려고 하지만 이는 bar의 호출부에 해당할뿐, 이미 foo에 하드 바인딩 되어있음므로 2가 나온다.
```

- 하드 바인딩으로 함수를 감싸는 형태의 코드는 인자를 넘기고 반환값을 돌려받는 창구가 필요할 때 주로 쓰인다.

**예제**
```jsx
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}
var obj = {
  a: 2
}
var bar = function() {
  return foo.apply(obj, arguments); // obj 를 this에 바인딩하고, foo 의 argument들을 arguments에 받는다. 커링기법이 들어감
}
var b = bar(3); // 2 3 이 출력됨. a는 obj에 바인딩된 값이고, 3은 넘겨진 인자값임
console.lob(b); //5 가 출력됨. 결국 bar는 foo 가 obj와 바인딩되어 3을 넘겨받아 계산한 결과값을 리턴받기 때문
```

- bind 헬퍼를 작성해 재사용하는것도 예시로 나와있기도하다.(책 49page 참조)
- 그리고 하드 바인딩은 자주쓰는 패턴이라 ES5 내장 유틸리티 Function.prototype.bind 가 구현되어있어 함수에서 객체를 인자로 받아 this 콘텍스트로 원본 함수를 호출하도록 바인딩후 하드코딩된 새 함수를 반환하여 파라미터까지 전달받을 수 있다.(49page~50page 참조)

#### API 호출 콘텍스트
- 많은 라이브러리 함수와 자바스크립트 언어 및 호스트 환경에 내장된 여러 함수는 대게 콘텍스트라 불리는 선택적인 인자를 제공한다. 이는 bind()로 콜백 함수의 this를 지정할 수 없을 경우를 대비한 일종의 예비책이다.
- 아래의 예제는 call() 이나 apply()를 대신해서 명시적 바인딩을 해주는 API 라고 보면 된다.

**예제**
```jsx
function foo(el) {
  console.log(el, this.id);
}
var obj = {
  id:"아름다운"
};
[1,2,3].forEach(foo,obj); // 개인적으로 forEach에 foo 의 this를 obj로 바인딩해주는 기는이 있다는 것으로 이해함... 출력결과는 1 아름다운 2 아름다운 3 아름다운
```

### 2.2.4 new 바인딩
- 한가지 오해 : 전통적인 클래스 지향 언어의 생성자는 클래스 인스턴스 생성시 new 연산자로 호출되지만, 자바스크립트는 이와 달리 클래스 지향과는 관계가 없다.
- 자바스크립트에서의 new : 생성자라 불리우는건 같으나 앞에 new 연산자가 있을떄 호출되는 일반 함수이고, 자동으로 붙들려 실행되는 평범한 함수이다. 다만 새 객체가 만들어진다.
- 함수 앞에 new를 붙여 생성자 호출을 하면 아래와 같은 일들이 저절로 일어난다.
```
1. 새 객체가 툭 만들어진다.
2. 새로 생성된 객체의 [[Prototype]]이 연결된다. 
3. 새로 생성된 객체는 해당 함수 호출 시 this로 바인딩된다.  // 여기서는 이것이 중요. 즉 new는 함수 호출시 this를 새 객체와 바인딩하는 방법이 되는 것이다.
4. 이 함수가 자신의 또 다른 객체를 반환하지 않는 한 new와 함께 호출된 함수는 자동으로 새로 생성된 객체를 반환한다.
```
1,3,4 관련 아래 예제를 보면 이해가 쉽다

**예제**
```jsx
function foo(a) {
  this.a = a;
}
var bar = new foo(2); // foo를 본딴 객체가 새로 만들어지고 그 객체는 this에 바인딩되며 인자로 2가 들어가며, 이는 bar에 할당된다.
console.log(bar.a);  // 2가 출력된다.
```

## 2.3 모든 건 순서가 있는 법
- 일단 결과부터 말하자면 우선순위로 기본 바인딩이 제일 낮고, new 바인딩이 가장 높다. 그 사이에 명시적이 암시적보다 우선한다.
(new -> 명시적 -> 암시적 -> 기본. ※ 단 하드 바인딩은 오버라이딩 불가. 책53p~54p참고)
- bind의 폴리필을 구현하여 하드 바인딩조차 new로 오버라이딩을 가능케 할 수 있다. 이런 방식의 폴리필은 아래 예제와 같이 함수 인자를 일부 미리 세팅할때 유용하게 사용된다.(책 55p~56p참고) 

**예제**
```jsx
function foo(p1,p2) {
  this.val = p1 + p2;
}
var bar = foo.bind(null,"p1"); // null이 아니라 아무객체나 넣어도 상관없음. 어차피 아래 new에서 this바인딩이 오버라이딩됨.
var baz = new bar("p2");
baz.val; // p1p2 라는 결과값 출력
```

### 2.3.1 this 확정 규칙
- 위에서 결론으로 언급한대로의 순서를 다시 정리해본다.
1. new 로 함수를 호출(new 바인딩) 했는가? -> 맞으면 새로 생성된 객체가 this이다.
```jsx
var bar = new foo();
```
2. call과 apply로 함수를 호출(명시적 바인딩), 이를테면 bind하드 바인딩 내부에 숨겨진 형태로 호출했는가? -> 맞으면 지정된 객체가 this
```jsx
var bar = foo.call(obj2);
```
3. 함수를 콘텍스트(암시적 바인딩), 즉 객체를 소유 또는 포함하는 형태로 호출했는가? -> 맞으면 바로 이 콘텍스트 객체가 this이다.
```jsx
var bar = obj1.foo();
```
4. 그 외의 경우에 this는 기본값(엄격 모드는 undefined, 비엄격 모드는 전역 객체)으로 세팅된다. 기본 바인딩이다.
```jsx
var bar = foo();
```

## 2.4 바인딩 예외
- 소제목은 바인딩 예외이지만, 개인적으로 기본 바인딩 관련 알아야 할 사항이라고 생각한다.

### 2.4.1 this 무시
- call, apply, bind 메서드의 첫 인자로 null 또는 undefined를 넘기면 기본 바인딩 규칙이 적용된다.

**예제**
```jsx
function foo() {
  console.log(this.a); // 단 여기 부분 위에 a=3을 할당했으면 맨아래 결과는 3으로 달라진다.
}
var a = 2;
foo.call(null); // 결과가 2로 나온다.
```
- 커링이나 펼침 연산자 전달을 위한 null바인딩. 저자는 null같은 값으로 this 바인딩을 하는 이유중 하나로 설명한다.

**예제**
```jsx
function foo(a,b){
  console.log("a:" + a + ", b:" + b);
}
foo.apply(null, [2,3]); // 이때 this는 global, a에는 2, b에는 3이 할당

var bar = foo.bind(null,2); // 위와 같은 방법을 커링으로 구현. 일단 a에 2가할당
bar(3); // b에 3이 할당
```

- 저자는 null을 애용하는 것이 리스크가 있다면서 자제하라고 한다.(예 : 어떤 3rd 파티 라이브러리 함수 호출시 null을 전달했는데, 해당 함수가 this 를 레퍼런스로 참조한다면 예기치 못한 일이 발생가능)
- 위와 관련하여 저자는 공집합을 가르키는 객체(텅빈 객체 혹은 DMZ객체라고 한다)를 생성해서 쓰길 권장한다. 

**텅빈 객체 바인딩 예제**
```jsx
function foo(a,b) {
  console.log("a:" + a + ", b" + b);
}
var ∮ = Object.create(null); //DMZ 객체 생성
foo.apply(∮, [2,3]); // 인자들을 배열 형태로 쭉 펼친다. a:2, b:3

//위랑 같은 내용임
var bar = foo.bind(∮,2);
bar(3); 
```

### 2.4.2 간접 레퍼런스
- 앞에서도 살짝 예시를 보여주었던, 원 함수 객체의 레퍼런스를 할당해와서 호출부가 원함수 호출과 다를바 없어지는 경우이다. 아래 예시를 보면 이해가 쉽다

**간접 레퍼런스 예시**
```jsx
function foo() {
  console.log(this.a);
}
var a = 2;
var o = {a:3, foo:foo};
var p = {a:4};
o.foo(); // 결과는 3 이는 o 에 암묵적 바인딩이 된 것이다.
(p.foo = o.foo)(); // 결과는 2  o.foo가 가르키는것은 결국 그냥 foo이기 때문이다.
```

### 2.4.3 소프트 바인딩
- 저자는 해당 바인딩 기법은 this를 명시적, 암시적 방법으로도 바인딩 할 수 있으면서도 기본 바인딩 규칙이 적용되야 할 떄 다시 되돌릴 수 있다는 의미로 소프트 바인딩이라는 용어를 사용한 것 같다.(추측)
- softBind는 javascript 지원 함수가 아니고 사람들이 폴리필로 고안해낸 유틸리티이다.(p60~61 참조)
- 사실 개인적으로 어떤 점에서 유용하게 쓰일 수 있을지는 잘 모르겠다...아래 예제 코드를 보고 같이 생각해 주세요

**소프트 바인딩 예제**
```jsx
function foo() {
  console.log("name: " + this.name);
}
var obj = { name: "obj"},
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
var fooOBJ = foo.softBind.(obj);
fooOBJ(); // name :obj
obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2 <-주목!
fooOBJ.call(obj3);  // name : obj3 <- 주목!
setTimeout(obj2.foo, 10); // name : obj  <- 소프트 바인딩이 적용됫대(?)
```

## 2.5 어휘적 this(사실 어휘적이라는 말이 나는 어렵다...)

### Arrow function
- 일반적인 this 바인딩 규칙에 적용이 안되는 예외로서 애로 펑션에서 '어휘적 바인딩'(?) 을 하게 되면 이후 절대 오버라이드 할 수 없다. 여기서 어휘적 바인딩의 의미가 무엇인지 아시는분 있으면 알려주세요...
- 화살표 함수는 this를 확실히 보장하는 수단으로 bind() 를 대체할 수 있고 끌리는 구석이 있다. 하지만 저자는 this 체계를 포기하는 형국이라는 this 를 잘 알아야 한다는 무언가의 압박을 주고 있다.
```jsx
function foo() {
  return (a) => { 
    console.log(this.a); // 여기서 'this'는 어휘적으로 'foo()'에서 상속된다.
  };
}
var obj1 = {
  a: 2
};
var obj2 ={
  a: 3
};
var bar = foo.call(obj1);
bar.call(obj2); // 2가 된다. 3이 아님.
```

### this를 어휘적으로 포착
- 아래 예제처럼 self = this 를 사용해서 골치아픈 this 에서 도망치는 꼼수도 있다.

**예제**
```jsx
function foo() {
  var self = this; // this를 어휘적으로 포착한다.
  setTimeout(function(){
          console.log(self.a);
   }, 100);
}
var obj = {
  a: 2
};
foo.call(obj);  //2
```

- 저자가 어휘적인 this에서 말하고 싶은것은 아래와 같다.
```
1. 오직 렉시컬 스코프만 사용하고 가식적인 this 스타일의 코드는 쓰지말자
2. 필요하면 bind()까지 포함하여 완전한 this 스타일의 코드를 구사하되 self = this 나 화살표 함수같은 소위 '어휘적 this'꼼수는 삼가하자 // 무슨말인지 모르겠어요 왜그런지 ㅠ
3. 두 스타일 모두 적절히 혼용하여 효율적인 프로그래밍을 할 수도 있겠찌만, 동일 함수 내에서 똑같은 것을 찾는데 다른 스타일이 섞여있으면 관리가 잘 안되고 이해가 잘 안될 것이다.
```
