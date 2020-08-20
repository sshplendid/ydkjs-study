# CH1. this라나 뭐라나

## intro
> 숙련된 개발자도 헷갈려하는 javascript 의 this. 이것은 모 언어처럼 객체 자신을 가르키는 개념이 아니고, 함수 객체의 스코프를 가르킨다는 개념도 아니다. 그렇다면 그 개념은 어떻게 정의될 수 있을까?

## 1.1 this를 왜?
아래 두 코드를 비교해 보자. 어떤 코드가 더 깔끔해 보이는가? 
저자는 암시적인 객체 레퍼런스를 넘기는 this 체계가 (위의 코드가) API 설계상 더 깔끔하고 명확하며 재사용하기 쉽다고 한다. 
여러분들의 생각은 어떠한가?

**예제1**
```jsx
function identify() {
  return this.name.toUpperCase();
}

function speak() {
  var greeting = "Hello, I'm "+ identify.call(this);
  console.log(greeting);
}

var me = {
  name: "Reader"
};

identify.call(me); //KYLE
identify.call(you); //READER

speak.call(me); // Hello, I'm KYLE
speak.call(you); // Hello, I'm READER
```

**예제2**
```jsx
function identify(context) {
  return context.name.toUpperCase();
}

function speak(context) {
  var greeting = "Hello, I'm " + identify(context);
  console.log(greeting);
}

identify(you);  //READER
speak(me);  //Hello, I'm KYLE
```
## 1.2 헷갈리는 것들
this 에 대한 오해에 대해 소개한다.

### 1.2.1 자기자신
모 언어처럼 객체(함수) 그 자체를 가리킨다는 오해가 있다. 하지만 이는 아래 예로서 아니라는 것을 알 수 있다.<br>

**반례**
```jsx
function foo(num) {
  console.log("foo: "+num);
  this.count++;
}
foo.count = 0;
var i;
for(i = 0; i<10; i++) {
  if(i>5) {
    foo(i);
  }
}
```
console.log(foo.count); // this가 함수를 가르켜서 count가 함수의 프로퍼티였다면 4가 나오핬어야 할 것이나, 값이 0이 나온다.

이와 관련 저자는 count프로퍼티를 다른 객체로 옮기는 등의 우회책을 사용하는 것으로 문제를 해결하는 것도 제안하면서, 이는 this를 잘 알지 못하고 선택하는 해결법이라면 발전이 없다며 쓴소리를 한다.
외려 아래와 같은 foo 함수 객체를 직접 가리키도록 강제하는 방법을 소개한다(...)<br>

**소개 코드**
```jsx
function foo(num) {
  console.log("foo: "+num);
  this.count++;
}
foo.count = 0;
var i;

for(i=0; i<10; i++) {
  if(i>5) {
    foo.call(foo, i); // call() 함수로 호출하여 this를 명시적으로 foo로 가리키게 한다.
  }
}
console.log(foo.count); // 결국 4번 호출된거로 뜬다.
```
### 1.2.2 자신의 스코프
아래 예제를 보면 적어도 this 가 함수 자신의 스코프를 가르키지 않는 다는 것을 알 수 있다.<br>

**예제**
```jsp
function foo() {
  var a = 2;
  this.bar(); // 저자는 여기서 this.bar() 라고 하지말고 bar() 로 호출이 자연스럽다고 지적.
}
function bar() {
  console.log(this.a); // 만약 this가 bar 함수 스코프를 가리킨다면 foo의 a를 참조 할 수 있어야함. 외려 a=2를 global 하게 선언하면 참조가능!
}
foo();  // ReferenceEror : a is not defined
```

## 1.3 this는 무엇인가?
- this는 작성 시점이 아닌 런타임 시점에 바인딩되며 함수 호출 당시 상황에 따라 콘텍스트가 결정된다.
- 함수 선언 위치와 상관없이 this 바인딩은 함수 호출 방법, 위치에 따라 정해진다.
- 어떤 함수를 호출하면 활성화레코드(Activation Record), 즉 실행 콘텍스트(Execution context) 가 만들어진다. 여기엔 함수가 호출된 근원(Call-stack) 과 호출 방법, 전달된 인자 등의 정보가 담겨있는데, this 레퍼런스는 그 중 하나로 함수가 실행되는 동안 이용할 수 있다.
