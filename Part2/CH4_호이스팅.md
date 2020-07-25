### CH4 들어가기 : 이전 시간에 스코프에 대한 이해를 했으리라 생각된다. 자바스크립트에는 선언문이 어디에 위치하느냐에 따라 스코프에 변수가 추가되는 과정에 미묘한 차이가 있다. 이에 대해 알아보는 시간을 가져보자<br>

## 4.1 닭이 먼저냐 달걀이 먼저냐

- 아래 두 결과를 비교해보자. 과연 예상한 대로 나온 것일까? <br>
<b>정답</b> : var a가 먼저 수행되고 a=2가 나중에 수행되기 때문에 아래와 같은 결과가 나온다. 그런데 왜 그런 방식으로 동작할까?

```jsx
a = 2;
var a;
console.log(a);
// 결과값 2
```

```jsx
console.log(a);
var a = 2;
// 결과값 undefined
```
<br>

## 4.2 컴파일러는 두 번 공격한다
- lexical scope의 핵심 : 자바스크립트 엔진이 코드를 인터프리팅 하기 전에 컴파일하고, 컴파일레이션 단계에는 선언문들을 찾아 적절한 스코프에 연결해준다.
- 즉 변수, 함수 선언문 모두 코드 실행전 먼저 처리되어 위치가 재배치된다.
- 그런 이유로 4.1에서의 코드들은 아래와 같이 처리된다.

```jsx
var a;
a = 2;
console.log(a);
```

```jsx
var a;
console.log(a);
a = 2;
```

#### 결국 변수와 함수 선언문은 코드의 꼭대기로 끌어올려지게 되는데, 이렇게 선언문을 끌어올리는 동작을 '호이스팅'(Hoisting) 이라고 한다. 즉, 선언문이 대입문보다 먼저다
<br>

- 함수 역시 호이스팅이 가능하다.
```jsx
foo();

function foo() {
  console.log(a);
  var a = 2;
}
// foo() 가 실행가능해서 console.log(a)로 인해 undefined가 출력된다
```
#### 호이스팅이 스코프별로 작동한다는 점도 중요하다. 글로벌로 끌어올려지는게 아니고, 스코프내 꼭대기로 끌어올려지는 것이다.<br>

- 함수 선언문은 끌어올려지지만, 함수 표현식은 호이스팅이 되지 않는다. 아래에서 var foo 는 호이스팅되어 foo() 호출은 실패하지 않으나, 값을 가지고 있지 않아 호출시 TypeError를 발생시킨다.
```jsx
foo(); // TypeError
var foo = function bar() {
  //...
};
```

- 만약 호출전 호출부 아래에 작성된 함수표현식이나 선언되지 않은 함수를 호출한다면, 해당 스코프에서 찾을 수 없기에 ReferenceError를 발생시킨다.<br>
```jsx
foo(); // TypeError
bar() ; // ReferenceError
var foo = function() {
  var bar = ...self...
  // ...
}
```
<br>

## 4.3 함수가 먼저다
- 호이스팅 순서는 함수가 제일 위로 올라가고, 다음으로 변수가 올려진다.
```jsx
foo(); // 1
var foo;

function foo() {
  console.log(1);
}
foo = function() {
  console.log(2);
}
```
엔진은 위 코드를 아래와 같이 해석하기 때문이다.
```jsx
function foo() {
  console.log(1);
}
foo(); // 1
foo = function() {
  console.log(2);
}
```
- 한가지더, 함수를 중복해서 선언했다면 앞서 선언들을 덮어써버린다. 중복 선언을 사용할때 조심하도록 하자.
```jsx
foo(); // 3
function foo() {
  console.log(1);
}
var foo = function() {
  console.log(2);
};
function foo() {
  console.log(3);
}
```
- 특히 이런 경우 조심하도록 하자. 의도치 않은 호이스팅이 일어나는 사례이다.(저자는 향후 버전에서 바뀔수도 있다고 생각하는듯...)
```jsx
foo(); // "b"

var a = true;
if(a) {
  function foo() { console.log("a");}
} else {
  function foo() { console.log("b");}
}
```
