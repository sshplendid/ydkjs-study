### CH2 들어가기 : 자바스크립트에는 타입이 없다는 말은 참이 아니라는 점은 이전 챕터에서 알 수 있었다. 값의 종류에 따라 타입이 결정되는데, 각 값들이 다른 언어와 유사하면서도 다른 특징이 있기에 우리를 헷갈리게 하고 자잘하고 미세한 부분에서 버그를 일으키기도 한다. 값들의 성격에 대해서 정확히 알고 실무를 통해 반복하여 값의 특성에 익숙해지는 것이 필요하다.


## 2.1  배열
 - delete 연산자로 배열의 element 를 제거해도 값은 undefined(empty 상태)로 길이는 줄어들지 않는다.

 - 배열도 객체이다. 인덱스는 숫자가 원칙이지만, 키/프로퍼티 문자열 추가 가능 <br>
 <pre><code>
 예) a["foobar"] = 2;  
 </pre></code>
 - 단, 문자열값이 auto casting 되는 경우로, 숫자 인덱스를 사용한것 과 같은 효과가 나타낼 때 주의. <br>
 <pre><code>
 예) a["13"] = 42;  <br>
     a.length;  // 하면 14 가 출력
 </pre></code>

### 2.1.1  유사 배열
 - 일련의 argument로 넘어온 값들을 배열로 만들어주는 배열로 만들어준다 // ??사실 어떤 경우에 써야 할지 잘 모르겠음
 - ES6부터는 기본 내장함수 Array.from(arguments); 로 사용하기를 추천
<br>

## 2.2  문자열
 - 문자열은 문자 배열과 같지 않다.
 1) 문자열은 불변(Immutable)값이나, 문자 배열은 가변(Mutable)값이므로 아래와 같은 경우가 발생한다.<br>
 <pre><code>
 예)  a="foo";    a[1] = "0";	// a는 "foo" <br>
      b=["f","o","o"];    b[1] = "0";	// b는 ["f","0","o"] <br>
 </pre></code>      
 ※ 단 여기서 문자열을 배열의 index참조처럼 접근하면 ie구버전은 문법 에러로 인식하기에 a.charAt(1) 로 접근 사용하길 권장한다.

 2) 문자열 메서드는 문자열이 불변값이라는 성질 때문에 내용을 변경하여 저장하지 않고 새로운 문자열을 생성후 반환한다. <br>
  <pre><code>
 예) c = a.toUpperCase(); // 이런 경우 a="foo", c="FOO" 에다가 a===c; 는 false 값을 반환하는 다른 값으로 생성된다.
  </pre></code>

 3) 문자열을 쉽게 처리하기 위해 문자 배열로 전환하여 처리 후 다시 문자열로 변환하는 방법을 사용할 수도 있다. (배열에 유용한 함수들이 많기 때문) <br>
  <pre><code>
 예) reverse 구현할때... <br>
      var c = a.split("").reverse().join("");  // a는 "foo였다면" c는 "oof"가 된다. split은 배열로 분할, reverse는 배열순서 뒤집기, join은 배열을 합쳐서 문자열로 되돌림. <br>
   </pre></code>    
 ※ 단 복잡한(유니코드) 문자가 섞인 경우 이 방법은 통하지 않는다고 한다.
<br>

## 2.3   숫자
 - number라는 타입에 기존의 정수, 부동 소숫점 숫자를 모두 아우른다. IEEE754 표준(부동 소수점 표준)을 따르며, 64비트 바이너리인 'Double precision' 표준 포맷을 사용한다고 한다. <br>다만 이 숫자들이 메모리에 저장되는 방식과 그렇게 설계된 의미까지 이해하지 않아도 개발은 가능하니 이에 대한 설명은 생략한다.

 - 기본적으로 10진수 리터럴이고

 - 소수점 앞 정수가 0 이면 생략 가능하며 <br>
  <pre><code>
   예) var b = .42;  // 0.42이다.
 </pre></code>
 - 소수점 이하가 0일떄도 생략 가능하다. <br>
  <pre><code>
   예) var b = 42.; // 42.0과도 같다. 단 일반적인 표기법이 아니니 자제하자.
 </pre></code>
 - 소수점 이하가 0으로 도배되면 떼고 출력한다. <br>
  <pre><code>
   예) var a = 42.300; //  a를 출력하면 42.3이 나온다 <br>
       var b = 42.0 ;   //  b를 출력하면 42가 나온다
 </pre></code>
 - 매우 큰 숫자는 지수형으로 표기하고, toExponential() 메서드의 결과와 같다. <br>
 <pre><code> 
   예) var a = 5E10;  // a를 출력하면 50000000000 가 나온다 <br>
        a.toExponential(); // 이렇게 출력하면  "5e+10" 이 나온다 <br>
       b = a*a;  // 이렇게 매우 큰 숫자를 만들면 2.5e+21 이 출력되고 <br>
       c = 1/a;  // 이렇게 매우 작은 숫자를 만들면 2e-11 이 출력된다.
 </pre></code>
 - 지정한 소수점 이하 자릿수까지를 나타내는 메소드는 toFixed이다. 문자열 형태로 반환된다. <br>
 <pre><code> 
    예) var a =42.59; <br>
         a.toFixed(0); // "43" <br>
         a.toFixed(1); // "42.6" <br>
         a.toFixed(2); // "42.59" <br>
         a.toFixed(3); // "42.590" <br>
         a.toFixed(4); // "42.5900"
 </pre></code>
 - 유효 숫자의 개수를 지정하는 메소드는 toPrecision이다.  <br>
 <pre><code>
   예) var a = 42.59; <br>
             a.toPrecision(1); // "4e+1" <br>
             a.toPrecision(2); // "43" <br>
             a.toPrecision(3); // "42.6" <br>
             a.toPrecision(4); // "42.59" <br>
             a.toPrecision(5); // "42.590"
 </pre></code>
 - toFixed와 toPrecision 은 아래와 같이 잘못 사용될수도, 올바르게 사용될수도 있다. <br>
 <pre><code>
   잘못된 예)  42.toFixed(3);  // SyntaxError <br>
   올바른 예)  (42).toFixed(3);  // "42.000" <br>
                   0.42.toFixed(3); // "0.420" <br>
                   42..toFixed(3);  // "42.000" <br>
                   42 .toFixed(3); // "42.000"
 </pre></code>
 - 다른 진법으로 표기함을 나타내기(소문자 표기를 추천) <br>
 <pre><code> 
 예) 0xf3 은 243의 16진수이고, 0o363은 243의 8진수이며, 0b11110011은 243의 이진수이다. <br>
 1. 16진수 : 0x  혹은 0X <br>
 2. 8진수 : 0o  혹은 0O <br>
 3. 2진수 : 0b  혹은 0B 
 </pre></code>

### 2.3.2  작은 소수 값
 - 이진 부동 소수점 숫자의 부작용 문제  <br>
 <pre><code> 
 예) 0.1 + 0.2 === 0.3 ; // 이게 false로 나온다.
 </pre></code>

 - 해결책 : 공식 허용 오차 로 처리하여 사용 <br>
ES6부터 정의된 Number.EPSILON (2의 -52제곱승으로 매우 작은 값이고, 이것보다 작은 오차는 허용가능하다고 보면된다)
 <pre><code>
 해결예) 아래와 같은 메소드를 만들어 0.1+0.2를 n1에, 0.3을 n2에 넣어 비교한다.  <br>
  function numberCloseENoughToEqual(n1, n2) { <br>
    return Math.abs(n1 - n2) < Number.EPSILON; <br>
  }
 </pre></code>
 - 부동 소수점 최대값 : 약 1.798e+308 (Number.MAX_VALUE 로 정의됨) <br>
   부동 소수점 최소값 : 약 5e-324 (거의 0 인 양수이고 Number.MIN_VAUE로 정의됨)


### 2.3.3  안전한 정수 범위
 - ES6에서 Number.MAX_SAFE_INTEGER == 2^53-1  (9007199254740991 대략 9천조넘음) <br>
               Number.MIN_SAFE_iNTEGER == -9007199254740991 

 - 이런 매우 큰 숫자를 사용할때는 DB등에서 64비트 ID를 처리할때가 대부분임. 64비트 숫자는 숫자 타입으로 정확하게 표시할 수 없으므로 string 타입으로 저장해야함.

 - 더 큰 숫자를 사용해야한다면 Big Number 유틸 라이브러리를 사용하는것을 권장


### 2.3.4  정수인지 확인
 - ES6부터는 Number.isInteger() 로 정수 여부를 확인. 폴리필의 내용을 보면 타입이 number이고 %1로 연산시 0이 나온 경우로 정의됨.
 - 사용하기 안전한 정수 여부는 Number.isSafeInteger() 로 체크. 여기에 NUMBER.MAX_SAFE_INTEGER 보다 큰 값을 넣으면 false뜸.


### 2.3.5  32비트(부호 있는)정수 // ?? 언제 사용할지 이해를 잘 못함
 - 정수의 안전범위는 대략 9천조(53비트) 지만, 32비트 숫자에만 가능한 연산이 있으므로 실제 범위는 훨씬 줄어든다(bit operation?)
 - 따라서 정수의 안전범위는 Math.pow(-2, 31) (약 -21억) 에서 Math.pow(2,31)-1 (약 21억) 까지다.
 - a | 0 과 같이 쓰면 ' 숫자값 -> 32비트 부호 있는 정수' 로 강제변환을 한다. (32비트 상위 비트는 소실됨)
<br>

## 2.4  특수 값
 타 언어와는 다른 개념을 지닌 값들이 많다. 조심해서 사용해야 할 것이다.  <br>
 <pre><code> 
 예) undefined, null, NaN, Infinity, -Infinity, 0, -0
 </pre></code>

### 2.4.1  값 아닌 값
 - 1장에서 보았듯이 undefined타입의 값은 undefined밖에 없고, null 타입도 null값 뿐이다. 이둘은 타입과 값이 항상 같다.
 - 이 둘은 의미가 조금 유사한거 같지만 다르므로 헷갈리기도 한다. 아래와 같이 정의 해 볼수 있을거 같다. <br>
  1) null : 빈 값,  undefined : 실종된 값 <br>
  2) null : 예전에 값이 있었거나 있을수 있었으나 지금 없는 상태, undefined 아직 값을 가지지 않은 것.


### 2.4.2  Undefined
 - 왜 그렇게 만든지는 모르지만 undefined라는 이름의 변수를 생성 가능하고, 그곳에 값을 할당 가능하다. 그러나 비추한다. 
 - void 라는 연산자에 무엇을 넣든지 무효로 만들어 undefined로 반환을 시킨다. <br>
 <pre><code> 
 예) 책의 52~53page 는 APP.ready 가 되지 않으면 doSometing을 무효화시키는 로직으로 해석된다.
 </pre></code>

### 2.4.3  특수 숫자
 - 숫자와 숫자가 아닌 값이 연산되면? 결과는 NaN으로 출력된다. 이는 Not A Number의 준말인데 오해의 소지가 다분한 글이다. <br>
 <pre><code> 
 예) var a = 2 / "foo"  //이 결과로 a 에는 NaN이 저장된다. <br>
     type of a === "number";  // 그런데 NaN의 타입은 number이다. -_-
 </pre></code>
 - 어떤 식의 결과값이 NaN인지 알아보기 위해서 비교 연산자 == 이나 === 를 사용할 수 없다. NaN은 어떤 NaN과도 바로 비교는 불가. <br>
 ※ 단, window.isNaN() 으로 확인 가능하나, 이는 일반 문자열을 넣어도 true를 반환해주는 멍청한 기능을 가졌기에 NaN이라 오해하게된다. 이는 찾기힘든 버그를 발생시키는 데 한몫을 하게 되는 사례이다. <br>
     ES6부터 Number.isNaN() 으로 이 문제를 해결하였으니 해당 내장함수를 사용하길 바란다. 

 - 무한대 개념
  javascript에서는 다른 언어와는 달리 0으로 나누는 연산이 예외를 발생시키지 않는다. 무한이라는 결과값이 있기 때문이다. <br>
 그 뿐 아니라, Number.MAX_VALUE 보다 매우 큰 값(대략 Math.pow(2, 970) 이라고 한다) 을 더하면 Infinity 가 된다. <br>
  1) 양수를 0으로 나누면 -> Infinity (Number.POSITIVE_INFINITY) <br>
  2) 음수를 0으로 나누면 -> -Infinity (Number.NEGATIVE_INFINITY) <br>
  3) 무한을 무한으로 나누면 -> NaN

 - 0 개념
  1) javascrpt에서 0을 양수로 나누거나 곱하면 양의 0이, 0을 음수로 나누거나 곱하면 음의 0 (-0) 이 나오게 된다. 단 브라우저에 따라 그냥 0으로 표기되는 경우도 있다.

  2) -0을 문자열화 하면 항상 "0"으로 된다. <br>
  <pre><code> 
  예) var a = 0/-3; <br>
      a.toString(); <br>
      a+""; <br>
      String(a); <br>
      JSON.stringify(a);
 </pre></code>
   3)"-0"을 숫자로 바꾸면 -0으로 된다. <br>
   <pre><code> 
   예) +"-0"; <br>
        Number("-0"); <br>
        JSON.parse("-0"); <br>
    </pre></code>     
   ※ JSON.parse("-0") 은 -0 이나, JSON.stringify(-0) 은 "0" 임에 주의한다. 

   4) 비교연산에서는 -0과 0은 true라는 같은 값이라는 결과가 나온다. 구분하고 싶다면 59page의 isNegZero(n) 와 같은 메소드를 만들어 써야 한다.

   5) -0의 등장이유 : 벡터에서 영감을 받은듯이, 값의 크기로 어떤 정보(애니메이션 프레임당 넘김 속도)와 그 값의 부호로 다른 정보(넘김 방향) 을 동시에 나타내야 하는 경우 사용된다고 한다.


### 2.4.4  특이한 동등 비교
 - ES6부터 어떤값이 NaN인지 0인지 -0인지 와같은 것까지 고려해서 비교해 주는 유틸리티를 지원한다.
 - Object.is(a, b); <br>
 <pre><code> 
  예)  var a = 2 / "foo"; // NaN <br>
        var b = -3 * 0;  // -0 <br>
        Object.is(a, NaN); // true <br>
        Object.is(b. -0); //  true <br>
        Object.is(b, 0); // false <br>
 </pre></code>
 ※ 단 Object.is 는 비교 효율이 떨어지기 때문에 안전한 비교가 확실하다면 일반 ==나 ===연산을 사용하는게 좋다.
<br>

## 2.5 값 vs 레퍼런스
  - 자바스크립트에서는 포인터 개념이 없고, 값 또는 레퍼런서의 할당 및 전달을 표시 및 제어하는 구문 암시(Syntactic Hint)가 없으며, 값의 타입만으로 값-복사, 레퍼런스-복사 둘 중 하나가 결정된다. 이것은 개발자의 의사와 관계없이 엔진의 재량으로 결정된다.

  - null, undefined, string, number, boolean, symbol 같은 단순 값(sclalar primitives) 은 언제나 값-복사 방식으로 할당/전달되고
  - 객체(배열과 박싱된 객체 래퍼 전체) 나 함수 등 합성값(compound values)은 할당/전달시 레퍼런스 사본을 생성한다.

  - 만약 배열같은 합성값을 값-복사에 의해 효과적으로 전달하려면 손수 값의 사본을 만들어 전달하여 레퍼런스가 원본을 가르치지 않게 한다. <br>
 <pre><code>  
  예) foo( a.slice() );  // slice()를 하면 새로운 배열의 얕은 복사(shallow copy)에 의해 사본을 만든다.
 </pre></code>
  - 반대로 원시값을 레퍼런스처럼 값을 바꿀 수 있게 하려면 다른 합성값(객체, 배열) 로 감싸야한다. <br>
  <pre><code> 
  예) function foo(wrapper) { <br>
         wrapper.a = 42; <br>
       } <br>
       var obj = { a: 2}; <br>
       foo(obj); <br>
       obj.a;  // 42가 출력된다. <br>
 </pre></code>
   ※ 단 Number 객체로 래핑된 스칼라 원시 값은 불변이라서(문자열, 불리언도 마찬가지) 해당 객체가 다른 원시값을 가지도록 변경할 수 없다. 다만 Number 도 객체이기에 내부 원시 값 변경이 아닌 새로운 프로퍼티를 추가해 간접적으로 추가된 프로퍼티를 통해 정보를 바꾸고 교환할 수 있다. 이것은 일반적인 Number 객체의 사용과는 거리가 있으므로 좋은 개발 습관은 아니다.

