---
layout: post
title: "[JavaScript] 차세대 JavaScript for React"
date: 2024-02-23 15:00:00
categories: JavaScript, React
---

---

## let & const

`let`과 `const`는 기본적으로 `var`를 대체한다.

## var와 let, const 비교

`var`와 `let`, `const`의 주요 차이점은 변수의 스코프와 변수 재할당에 있다.

`var`는 <span class="important">함수 스코프</span>를 가지고 있어 함수 내에서 선언된 변수는 함수 내에서만 유효하다. 또한 var로 선언된 변수는 <span class="important">재선언이 가능</span>하다. 반면 `let`과 `const`는 <span class="important">블록 스코프</span>를 가지고 있어 블록 내에서 선언된 변수는 블록 내에서만 유효하며, `let`으로 선언된 변수는 <span class="important">재할당이 가능하지만 재선언은 불가능</span>하다. `const`로 선언된 변수는 한 번 <span class="important">할당된 값은 변경할 수 없다</span>.

이러한 차이점으로 `let`과 `const`는 더 엄격한 변수 선언 규칙을 가지고 있고, 블록 스코프를 가지므로 예기치 않은 오류를 방지하는 데 도움이 된다. `const`는 한 번 할당된 값을 변경할 수 없으므로 불변성을 유지하는 데 사용된다.

### Code Example

```js
// var로 선언된 변수는 함수 스코프를 갖는다.
function varExample() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10
}
varExample();

// let으로 선언된 변수는 블록 스코프를 가지고 있어 블록 내에서만 유효하다.
function letExample() {
  if (true) {
    let y = 20;
  }
  console.log(y); // ReferenceError: y is not defined
}
letExample();

// const로 선언된 변수는 재할당이 불가능하다.
const PI = 3.14;
PI = 3.14159; // TypeError: Assignment to constant variable.
```

---

## ES6 화살표 함수(Arrow Functions)

<span class="important">화살표 함수</span>(Arrow Functions)는 JavaScript에서 함수를 생성하는 다른 방식이다. 짧은 문법 외에도, `this` 키워드의 스코프를 유지하는 데 장점을 제공한다.

화살표 함수 문법은 이상하게 보일 수 있지만 사실 간단하다.

### Code Example

```js
// 일반 함수
function greet(name) {
  console.log("안녕하세요, " + name + "님!");
}

// 화살표 함수로 변환
const greetArrow = (name) => {
  console.log("안녕하세요, " + name + "님!");
};

// 더 간단한 형태로 화살표 함수 사용
const greetShortArrow = name => console.log("안녕하세요, " + name + "님!");

// 함수 호출
greet("JINJIN");
greetArrow("JINJIN");
greetShortArrow("JINJIN");
```

중요! 📝
```js
// 인자가 없는 경우, 빈 괄호를 사용하여야 한다.
const sayHello = () => {
  console.log('안녕하세요!');
};

// 하나의 인자를 가질 때는 괄호를 생략할 수 있다!
const greetPerson = name => {
  console.log('안녕하세요, ' + name + '님!');
};

// 값만 반환할 때, 다음과 같이 짧게 표현할 수 있다.
const doubleNumber = num => num * 2; // return을 사용하지 않아도 된다.

// 함수 호출
sayHello();
greetPerson('JINJIN');
console.log(doubleNumber(5)); // 10 출력
```

---

## Exports & Imports

React 프로젝트(그리고 실제로 모든 최신 JavaScript 프로젝트에서)에서는 코드를 여러 개의 JavaScript
파일, 즉 <span class="important">모듈</span>로 나눈다. 이렇게 하는 이유는 각 파일/모듈에 집중하고 관리할 수 있게 하기 위해서이다. 

다른 파일에서 기능에 접근하려면 해당 기능을 사용 가능하게 만들기 위한 `export` 및 해당 기능에 접근하기 위한 `import` 문이 필요하다.

---

## default export & named export

### default export

- `default export`는 한 파일에서 하나만 가능하다.
- 파일에서 기본적으로 내보내는 것으로, 이름이 없는 export이다.
- 다른 파일에서 해당 모듈을 import할 때, 이름을 원하는 대로 지정할 수 있다.
- import할 때 중괄호 `{}`를 사용하지 않는다.

```js
// utils.js
const add = (a, b) => a + b;
export default add;

// main.js
import myAddFunction from './utils.js'; // 이름을 원하는 대로 지정하여 가져올 수 있음
console.log(myAddFunction(3, 5)); // 8 출력
```

### named export

- 한 파일에서 여러 개의 named export를 사용할 수 있다.
- 각각의 export는 이름이 있으며, import할 때 해당 이름을 사용해야 한다.
- import할 때 중괄호 `{}`를 사용하여 명시적으로 해당 이름을 가져와야 합니다.

```js
// utils.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// main.js
import { add, subtract } from './utils.js'; // 각각의 이름으로 명시적으로 가져와야 함
console.log(add(3, 5)); // 8 출력
console.log(subtract(10, 5)); // 5 출력
```

---

## 클래스(Classes)

<span class="important">클래스</span>는 기본적으로 생성자 함수와 프로토타입을 대체하는 기능이다. 이를 통해 자바스크립트 객체에 대한 블루프린트를 정의할 수 있다.

🙅🏻‍♀️ 다음은 오래된 구문 예시이다.

```js
class Person {
  constructor () {
    this.name = 'JINJIN';
  }
}

const person = new Person();
console.log(person.name); // 'JINJIN'
```

위의 예에서는 클래스뿐만 아니라 해당 클래스의 속성(=> `name`)도 정의되어 있다. 여기에 보이는 구문은 프로퍼티를 정의하는 "오래된" 구문이다. 최신 자바스크립트 프로젝트에서는 다음과 같은 더 편리한 방법으로 클래스 프로퍼티를 정의할 수 있다.

```js
class Person {
  name = 'JINJIN';
}

const person = new Person();
console.log(person.name); // 'JINJIN'
```

다음과 같이 메서드를 정의할 수도 있다.

```js
class Person {
  name = 'JINJIN';
  printMyName() {
    console.log(this.name);
  }
}

const person = new Person();
person.printMyName(); 
```

또는 이렇게도 가능하다.

```js
class Person {
  name = 'JINJIN';
  printMyName = () => {
    console.log(this.name);
  }
}

const person = new Person();
person.printMyName(); 
```

클래스를 사용할 때 <span class="important">상속</span>을 사용할 수도 있다.

```js
class Human {
  species = 'human';
}

class Person extends Human {
  name = 'JINJIN';
  printMyName = () => {
    console.log(this.name);
  }
}

const person = new Person();
person.printMyName(); 
console.log(person.species); // human
```

---

## Spread & Rest 연산자

Spread 연산자와 Rest 연산자는 모두 같은 구문인 세 개의 점(`...`)을 사용한다. 연산자의 사용 방법에 따라 스프레드 Spread 또는 Rest 연산자로 작동한다.

### Spread 연산자

- `...` 구문을 사용하여 반복 가능한(iterable) 항목(예: 배열 또는 객체)을 개별 요소로 확장할 때 Spread 연산자로 작동한다.
- 기존의 배열, 문자열 또는 객체를 변경하지 않고 새로운 배열, 문자열 또는 객체를 생성한다.
- 배열 확장: 배열을 확장하여 다른 배열에 요소를 추가하거나 병합할 수 있다.
  ```js
  const arr1 = [1, 2, 3];
  const arr2 = [4, 5, 6];

  // 배열 확장 예시
  const mergedArray = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]
  ```
- 함수 매개변수: 함수 호출 시에 배열의 요소를 분리하여 함수의 매개변수로 전달할 수 있다.
  ```js
  function sum(a, b, c) {
    return a + b + c;
  }

  const numbers = [1, 2, 3];

  // 함수 호출 시에 스프레드 연산자를 사용하여 배열의 요소를 전달
  const result = sum(...numbers); // 6
  ```
- 배열 복사: spread 연산자를 사용하여 배열을 복사할 수 있다.
  ```js
  const originalArray = [1, 2, 3];
  const copiedArray = [...originalArray]; // [1, 2, 3]
  ```
- 문자열 확장: 문자열을 문자 단위로 분리하여 배열로 확장할 수 있다.
  ```js
  const str = "hello";
  const strArray = [...str]; // ['h', 'e', 'l', 'l', 'o']
  ```
- 객체 병합: 객체를 병합하여 새로운 객체를 생성할 수 있다. (ES2018부터 지원)
  ```js
  const obj1 = { a: 1, b: 2 };
  const obj2 = { c: 3, d: 4 };

  const mergedObj = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }
  ```

### Rest 연산자

- `...` 구문을 사용하여 여러 요소를 하나의 배열 또는 객체로 수집할 때, 일반적으로 함수 매개변수나 비구조화 할당에서 사용될 때 Rest 연산자로 작동한다.
- 함수 매개변수: 함수 정의 시에 rest 연산자 매개변수를 사용하여 가변적인 수의 인수를 받을 수 있다.
  ```js
  function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
  }

  console.log(sum(1, 2, 3, 4, 5)); // 15
  ```
- 배열 디스트럭처링: rest 연산자를 사용하여 나머지 요소를 추출할 수 있다.
  ```js
  const [first, ...rest] = [1, 2, 3, 4, 5];
  console.log(first); // 1
  console.log(rest); // [2, 3, 4, 5]
  ```

---

## Destructuring (비구조화)

- 비구조화는 배열이나 객체의 값을 분해하여 개별 변수에 할당하는 것을 말한다. 이를 통해 코드를 간결하고 가독성 있게 작성할 수 있다.
- 배열 비구조화: 배열에서 값을 추출하여 개별 변수에 할당한다.
  ```js
  const numbers = [1, 2, 3];

  // 비구조화를 사용하여 배열에서 값을 추출하여 변수에 할당
  const [a, b, c] = numbers;

  console.log(a); // 1
  console.log(b); // 2
  console.log(c); // 3
  ```
- 객체 비구조화: 객체에서 속성 값을 추출하여 개별 변수에 할당한다.
  ```js
  const person = { name: 'JINJIN', age: 20 };

  // 비구조화를 사용하여 객체에서 속성 값을 추출하여 변수에 할당
  const { name, age } = person;

  console.log(name); // 'JINJIN'
  console.log(age); // 20
  ```
- 기본값 할당: 비구조화 할당시 기본값을 지정할 수 있다.
  ```js
  const numbers = [1, 2];

  // 배열 비구조화 할당시 기본값을 지정
  const [a, b, c = 3] = numbers;

  console.log(c); // 3 (numbers 배열에 세 번째 요소가 없으므로 기본값인 3이 할당됨)
  ```
- 나머지 요소: 배열이나 객체에서 나머지 요소를 수집할 수 있다.
  ```js
  const numbers = [1, 2, 3, 4, 5];

  // 나머지 요소를 수집하여 배열에 할당
  const [first, ...rest] = numbers;

  console.log(first); // 1
  console.log(rest); // [2, 3, 4, 5]
  ```
- 비구조화를 사용하여 함수의 매개변수로 전달된 객체에서 특정 속성을 추출하고 출력하는 예시
  ```js
  // 함수 정의
  function printName({ name }) {
    console.log(name);
  }

  // 함수 호출 시 객체를 전달
  const person = { name: 'JINJIN', age: 20 };
  printName(person); // JINJIN
  ```


### 참고
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)