# [🔗 1주차: (1~4장) 객체 타입, 타입 확장, 타입 좁히기](https://velog.io/@iberis/14장-객체-타입-타입-확장-타입-좁히기)

> ## 💡용어 정리
**폴리필** : 브라우저가 지원하지 않는 코드를 브라우저가 사용할 수 있는 코드로 변환한 코드 조각이나 플러그인
**슈퍼셋** : 기존 언어에 새로운 기능과 문법을 추가해서 보완하거나 향상하는 것을 말한다. 슈퍼셋 언어는 기존 언어와 호환되며 일반적으로 컴피일러 등으로 기존 언어 코드로 변환되어 실행된다.
**트랜스파일** : 최신 버전의 코드를 구 버전으로 변환하는 과정
**컴파일** : 사람이 이해하는 언어를 컴퓨터가 이해할 수 있는 언어로 변환해주는 과정으로, 서로 다른 수준(고수준-저수준) 간의 코드 변환을 의미
- 타입스크립트의 컴파일 결과물 파일은 자바스크립트 파일이다.
**컴파일 타입** : 컴퓨터가 소스코드를 이해할 수 있도록 기계어로 변환되는 시점
**런타임** : 컴파일 후 변환된 파일이 메모리에 적재되어 실행되는 시점
**웹 애플리케이션** : 사용자와 상호작용하는 쌍방향 소통의 웹사이트 
- cf) 웹사이트 : 단방향 적으로 정보를 제공하는 HTML 에 링크가 연결된 웹 페이지 모음
>
|  | **명목적 타이핑** | **구조적 타이핑** | **덕타이핑** |
| --- | --- | --- | --- |
| 특징 | 명시적인 이름을 가지고 타입을 구분하는 방식 |  어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식 |  |
| 타입 검사 시점 | 컴파일 타임 | 컴파일 타임 | 런타임 |
| 언어 | C++, 자바 | 타입스크립트 | 자바스크립트 |
| 코드 |  |  |  |
>
```java
// 명목적 타이핑
class Cat {
  String name;
    public void hit(){}
}
>
class Arrow {
	String name;
	  public void hit(){}
}
>
class  Main {
    public static void main(){
        Arrow cat = new Cat(); // 🚨 error : incompatible types: Cat cannot be converted to Arrow
        Cat arrow = new Arrow(); // 🚨 error : incompatible types: Arrow cannot be converted to Cat
    }
}
```
>
```tsx
// 구조적 타이핑
class Cat {
	name: string;
    constructor(name: string){
        this.name = name
    }
	public hit(): void{};
}
>
class Arrow {
	name: string;
    constructor(name: string){
        this.name = name
    }
	public hit(): void{};
}
>
const cat: Arrow = new Cat('mewoo');
```
>
<details>
  <summary><h3>구조적 타이핑(덕타이핑)으로 인해 마주하는 이슈</h3></summary>
>
객체의 key 를 `Object.keys()` 와 `.map()` 매서드로 순회하려고 할 때, key 가 객체의 key 로 타입이 좁혀지지 않고, string 으로 정의되는 이슈가 있다.
>
## [](https://5kdk.github.io/blog/2024/04/04/index-signatures-and-duck-typing#%EC%9D%B8%EB%8D%B1%EC%8A%A4-%EC%8B%9C%EA%B7%B8%EB%8B%88%EC%B2%98%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%A0-%EB%95%8C-%EB%A7%88%EC%A3%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%8A%88)
>
```tsx
const student = {
    name: '양해수',
    age: 21
}
>
Object.keys(student).map(key => {
  const value = student[key]; // 🚨 Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ name: string; age: number; }'.  No index signature with a parameter of type 'string' was found on type '{ name: string; age: number; }'.
  return value;
});
```
>
```tsx
const studentKeyType = Object.keys(student); // string[]
```
>
위 문제를 해결하기 위한 방법
1. **Obejct.keys(객체)**를 `as`로 타입 단언    
    ```tsx
    type Student = typeof student;
>    
    (Object.keys(student) as Array<keyof Student>).map(key => {
      const value = student[key];
      return value;
    });
    ```
>    
2. 제네릭을 활용해 **Object.keys**에 대한 **반환 타입을 객체의 key 타입으로 강제하는 함수를 덧씌움**
    - **keysOf** 함수는 객체의 키를 가지고 오면서 동시에 가져온 배열에 대해서도 마찬가지로 타입 단언으로 처리하는 과정을 거친다.
>    
    ```tsx
    // Object.keys를 대신할 keyOf 함수 생성
    function keysOf<T extends Object>(obj: T): Array<keyof T> {
      return Array.from(Object.keys(obj)) as Array<keyof T>;
    }
>    
    keysOf(student).map(key => {
      const value = student[key];
      return value;
    });
    ```
>    
3. 순회할 **key 의 타입을 as로 단언** 
>    
    ```tsx
    type Student = typeof student;
>    
    Object.keys(student).map(key => {
      const value = student[key as keyof Student];    
      return value;
    });  
    ```
</details>

## 객체 타입일까? { }, object

### `{}` : null 과 undefined 를 제외한 모든 타입에 해당한다.

- 대입은 가능하지만 사용할 수 없다.

```tsx
const str : {} = '문자열';
const num : {} = 123;
const bool : {} = true;
const arr: {} = [1, 2, 3];
const func: {} = () => {};
const obj: {} = {name: '이름'};

const nu: {} = null; // 🚨 Type 'null' is not assignable to type '{}'.
const unde: {} = undefined; // 🚨 Type 'undefined' is not assignable to type '{}'.

// 실제 사용하려고 하면 에러가 발생한다.
arr[0]; // 🚨 Element implicitly has an 'any' type because expression of type '0' can't be used to index type '{}'.  Property '0' does not exist on type '{}'.
func(); // 🚨 This expression is not callable.  Type '{}' has no call signatures.
obj.name; // 🚨 Property 'name' does not exist on type '{}'.
```

- {} 타입에 null 과 undefined 를 합치면 unknown 과 비슷해진다.

```tsx
const unkn : unknown = 'string';
if(unkn){
  unkn // const unkn: {}
} else {
  unkn // const unkn: unknown
}
```

- 빈 객체 타입을 지정하기 위해서는 유틸리티 타입으로 `Record<string, never>` 로 사용하는 것이 바람직하다.

### `object` : 원시타입을 제외한 객체, 배열, 정규 표현식, 함수, 클래스 등과 호환된다.

대입은 가능하지만 사용할 수 없다.

```tsx
// 원시타입에 호환되지 않는다.
const str : object = '문자열'; // 🚨 Type 'string' is not assignable to type 'object'.
const num : object = 123; // 🚨 Type 'number' is not assignable to type 'object'.
const bool : object = true; // 🚨 Type 'boolean' is not assignable to type 'object'.
const nu: object = null; // 🚨 Type 'null' is not assignable to type 'object'.
const unde: object = undefined; // 🚨 Type 'undefined' is not assignable to type 'object'.

const arr: object = [1, 2, 3];
const func: object = () => {};
const obj: object = {name: '이름'};

// 실제 사용하려고 하면 에러가 발생한다.
obj.name; // 🚨 Property 'name' does not exist on type 'object'.
arr[0]; // 🚨 Element implicitly has an 'any' type because expression of type '0' can't be used to index type '{}'.  Property '0' does not exist on type '{}'.
func(); // 🚨 This expression is not callable.  Type '{}' has no call signatures.
```

---

## `type` alias 와 `interface`

### 공통점

### 인덱스 시그니처 : 객체의 key 타입에는 오로지  `string`, `number`, `symbol` 타입만 지정할 수 있다.

- literal type, 제네릭 등은 지정할 수 없다.

> **인덱스 시그니처(Index Signatures)** : 특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고 있을 때 사용하는 문법 `[key: 타입]: 타입` 
**리터럴 타입(literal type)** : string, number, boolean 타입 하위의 구체적인 타입
> 

```tsx
interface IStringArray { [index: number]: string; };

const A_Symbol: unique symbol = Symbol('A-type');

interface ISymbol { [A_Symbol]: number; };
type SymbolType = { [A_Symbol]: number; };

// interface 에서도 같은 에러 발생
type LiteralTypeKey = {
    [key: 'a' | 'b']: string; // 🚨 An index signature parameter type cannot be a literal type or generic type. Consider using a mapped object type instead.
}

// Ok
type LiteralTypeKey = {
  a: string;
  b: strin;
}

```

- 마찬가지로 객체의 key 타입으로 템플릿 리터럴 타입을 직접 사용할 수 없다.
    - 템플릿 리터럴 타입을 타입에 지정 후  맵드 타입(Mapped Type) 정의하여 사용하여야 한다.

```tsx
type AorB = 'A' | 'B';
type TempLiter = `${AorB}-type`;

//Ok
type Example = {
  [key in TempLiter]: number;
};

// Error 1
type Wrong1 = {
  `${AorB}-type`: number; // 🚨 'AorB' only refers to a type, but is being used as a value here.
}

// Error 2 computed property name 에는 표현식 또는 symbol 타입만 가능하다.
type Wrong2 = {
  [`${AorB}-type`]: number; // 🚨 A computed property name in a type literal must refer to an expression whose type is a literal type or a 'unique symbol' type.
}
interface IWrong2 {
  [`${AorB}-type`]: number; // 🚨 A computed property name in a type literal must refer to an expression whose type is a literal type or a 'unique symbol' type.
}

// Error 3 'TempLiter' 라는 문자열 키로 해석된다.
type WrongKey = {
 TempLiter: number;
}

const wrong: WrongKey = {
  TempLiter: 123,
  ["A-type"]: 456 // 🚨 Object literal may only specify known properties, and '["A-type"]' does not exist in type 'WrongKey'.
}
```

### 차이점

### 1. Mapped Types ( computed property name)

 type alias 은 **computed property name** 을 사용한 타입 선언이 가능하지만 interface 에서는 불가능하다.

> **computed property name** : 표현식(expression)을 이용해 객체의 key 값을 정의하는 문법
  - `[key: T]` 대괄호(`[]`) 안에 동적으로 이름을 정의하는 방식
> 

```tsx
type fruite = 'apple' | 'banana' | 'grape';

type fruitePrice = {[key in fruite]: number};

interface IFruitePrice {
	[key in fruite]: number; // 🚨 A mapped type may not declare properties or methods.
}
```

```tsx
type Mapped = {
  num: number;
  str: string;
  bool: boolean;
}

type MappedType = {
  [K in keyof Mapped]: string;
}
 
interface IMapped {
  [K in keyof Mapped]: string; // 🚨 Member '[K in keyof' implicitly has an 'any' type.
} // 🚨 'Mapped' only refers to a type, but is being used as a value here.
// 🚨 'string' only refers to a type, but is being used as a value here.

/* interface 내의 인덱스 시그니처에서는 keyof Mapped 를 하나의 타입으로 인식하지 못한다
* type MappedKeys = keyof Mapped
* interface IMapped {[K in MappedKeys]: string; } // 🚨 A mapped type may not declare properties or methods.
*/
```

### 2. interface의 경우 동일한 이름으로 다시 interface를 정의해 확장하는 것이 가능하지만 type은 동일한 이름으로 다시 선언할 수 없다.

- 단 interface 간의 속성이 겹치는데 타입이 다를 경우에는 에러가 발생한다
- 인터페이스 간의 이름이 겹쳐서 의도하지 않게 병합되거나 에러가 발생하지 않도록 하기 위해서 **namespace**  를 사용할 수 있다.

```tsx
interface Person {
  name: string;
};

interface Person {
  age: number;
}

const person: Person = {
  name: '김지수',
  age: 20
}

// 🚨 Subsequent property declarations must have the same type.  Property 'age' must be of type 'number', but here has type 'string'
interface Person {
  age: string;
}
```

```tsx
type Student = {
  name: string
}

type Studnet = {
  score: number
}

const studnet: Student = { // type Student = { name: string; }
  name: '이수민',
  score: 100 // 🚨 Object literal may only specify known properties, and 'score' does not exist in type 'Student'.
}
```

### extends : 서브 타입에 슈퍼 타입과 같은 속성 이름이 있으면 해당 속성을 덮어씌운다.

- 슈퍼 타입의 속성값 타입보다 서브타입 속성값 타입의 범위가 같거나 작아야 덮어씌울 수 있다.

```tsx
interface IPerson {
  phone: string | number;
};

interface IStudent extends IPerson {
  phone: number;
}

const student:IStudent = {
  phone: 123456 // IStudent.phone: number
}

/* 서브 타입의 속성 타입이 더 넓은 경우 타입 에러가 발생한다.
🚨 Interface 'IPerson2' incorrectly extends interface 'IStudent'.
  Types of property 'phone' are incompatible.
    Type 'string | number' is not assignable to type 'number'.
      Type 'string' is not assignable to type 'number'.
*/
interface IPerson2 extends IStudent {
  phone: number | string;
}
```

- 다 중 상속

여러 개의 슈퍼 타입을 확장하려는 경우, 슈퍼 타입 간의 속성이 동일해야 다 중 상속이 가능하다.
슈퍼 타입 간에는 속성의 타입이 좁혀지지 않는다.

```tsx
interface IPerson {
  phone: string | number;
};

interface IStudent {
  phone: number;
}

interface Phone extends IStudent, IPerson {} 
/* 🚨 Interface 'Phone' cannot simultaneously extend types 'IPerson' and 'IStudent'.
     Named property 'phone' of types 'IPerson' and 'IStudent' are not identical. */
     
/*
{ phone: number } 로 좁혀질 것 같지만, 좁혀지지 않고 에러 발생
*/
```

### `&` intersection

값을 기준으로 연산에 사용된 두 타입을 모두 포함한다. (교집합)
intersection type의 경우 무조건 상속이 성공하고 에러가 발생하지 않으므로 전체 타입 검사를 해야 이를 알아낼 수 있다.
따라서 두 타입을 intersection 할 때는 두 타입의 **속성 값이 교집합을 가지는지 주의하며 사용**해야한다.

```tsx
type PersonType = {
  phone: string;
}

type StudentType = PersonType & {
  phone: number;
}

// string 타입이면서 number 타입은 없으므로 phone 은 never 타입이 된다.
const student : StudentType = {
    // phone: '123456' // Type 'string' is not assignable to type 'never'.
    phone: 123456 // Type 'number' is not assignable to type 'never'. (property) phone: never
}
```

---

참고 
우아한 타입스크립트 1 ~ 4장
타입스크립트 교과서
[[TypeScript] 여러 타입 선언 방법과 interface & type Alias 비교](https://velog.io/@skawnkk/interface-type-Alias)
[[TypeScript] 인터페이스: 확장과 교차](https://velog.io/@violetwhenisawu/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4-%EC%86%8D%EC%84%B1-6wc1ad68)
[타입스크립트의 인덱스 시그니처와 Object.keys 그리고 덕 타이핑](https://5kdk.github.io/blog/2024/04/04/index-signatures-and-duck-typing)

---


> ### 💡 배열의 요소 타입 조회 : 인덱스드 엑세스 타입(Indexed Access Types)
>
- 책 102 페이지 오류 : 제네릭 위치에 타입이 아닌 값을 사용 함
- T 가 배열 타입이어야 T[number] 가 성립할 수 있음
>
> `infer` 키워드는 타입 추론에 사용되며, 조건부 타입 내에서 타입을 추론하는 변수로 사용될 수 있다.
>
```tsx
// 수정 1
type ElementOf<T extends readonly any[]> = T[number];
>
const studentList = [{name: '김지수', class: 2}, {name: '이현빈', class: 1}];
type Student = ElementOf<typeof studentList>; // type Student = {name: string; class: number};
```
>
```tsx
// 수정2
type ElementOf<T> = T extends readonly (infer U)[] ? U : never;
>
const studentList = [{name: '김지수', class: 2}, {name: '이현빈', class: 1}];
type Student = ElementOf<typeof studentList>; // type Student = {name: string; class: number}; 
```

## Object.prototype.toString.call(…)

`Object.prototype.toString.call()`은 자바스크립트 객체의 내부 `[[Class]]` 속성을 이용하여 객체의 원형 타입을 정확하게 반환한다. 반환 값의 `[object Type]` 형식에서 `object`는 기본적으로 모든 데이터가 객체라는 것을 나타내고, `Type`은 해당 객체의 구체적인 타입을 나타낸다.

```tsx
console.log(Object.prototype.toString.call({})); // [object Object]

function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType([]));        // "Array"
console.log(getType({}));        // "Object"
console.log(getType(123));       // "Number"
console.log(getType(123n));       // "BigInt"
console.log(getType('abc'));     // "String"
console.log(getType(true));     // "Boolean"
console.log(getType(null));      // "Null"
console.log(getType(undefined)); // "Undefined"
console.log(getType(function() {})); // "Function"
console.log(getType(new Date())); // "Date"
console.log(getType(new Error())); // "Error"
console.log(getType(Symbol('sym'))); // "Symbol"
console.log(getType(/abc/)); // "RegExp"
```

- 복잡한 내부 동작 과정으로 typeof 보다는 성능 저하 문제가 발생할 수 있다.
    - 원시 값을 다룰 때 **박싱(래핑) 비용**
    - `toString()` 메서드를 찾기 위해 객체의 프로토타입 체인 탐색 비용
    - 타입을 `[object Type]` 형식의 문자열로 반환하기 위해,  메모리 할당을 요구하며, 문자열 연산이 포함된다
    - `call()` 메서드는 명시적으로 `this` 값을 설정한 후 함수를 호출하기 때문에, 함수 호출에 대한 추가 비용이 발생

<details>
<summary><h3>💡객체.toString() 으로 바로 사용하지 않고 `Object.prototype.toString.call(…)` 을 사용하는 이유</h3></summary>

### Object.prototype
자바스크립트의 모든 객체는 `Object`의 인스턴스이다. `Object.prototype`은 모든 객체의 부모 객체이며, 여기에 정의된 메서드나 속성은 모든 객체가 상속받는다. 모든 객체는 `Object`의 프로토타입 체인을 따르므로, 객체에서 메서드를 직접 정의하지 않더라도 `Object.prototype`에 정의된 메서드(ex. `toString()`)를 상속받아 사용할 수 있다.

### .toString()

- 객체를 문자열로 변환하는 기능을 한다.
- 모든 객체는 이 메서드를 상속받으므로, 각 객체는 `toString()`을 호출하여 자신을 문자열로 표현할 수 있습니다.
- **기본 동작**: 기본적으로 `Object.prototype.toString()`은 객체의 타입 정보를 `"[object Type]"` 형식으로 반환한다. 이 메서드는 객체가 `Array`, `Function`, `Date` 등의 구체적인 타입 정보를 반환하도록 설계되어 있다.
- **오버라이딩 가능**: 객체가 자신만의 `toString()` 메서드를 정의하면, 기본`Object.prototype.toString()` 대신 객체 자체의 메서드가 호출된다.
    
 ```tsx
    const obj = {};
    console.log(obj.toString()); // [object Object] // Object.prototype.toString()을 호출
    
    const arr = [];
    console.log(arr.toString()); // "" // Array.prototype.toString()을 호출
 ```
    
### 함수.call(객체)

 - 자바스크립트의 모든 함수에 정의된 메서드로, 함수에 특정한 `this` 값을 설정한 뒤 그 함수를 호출할 수 있게 해준다.  첫 번째 인수로 전달된 객체를 `this`로 설정하고, 그 이후의 인수는 호출되는 함수의 인수로 전달됩니다.
   - 쉽게 말하자면,  객체의 임시 메서드로 함수를 맵핑해준다.
  - **`Object.prototype.toString.call()`에서 `call()`을 사용하는 이유**는, `toString()` 메서드를 객체의 컨텍스트에 맞게 명시적으로 호출하기 위해서 이다. `toString()` 메서드는 객체의 프로토타입 체인에서 사용될 때 특정 객체에 바인딩된 채로 호출되므로, `call()`을 이용해 `this` 값을 강제로 다른 객체로(호출하는 객체로) 바꿔서 사용할 수 있다. 

   ```tsx
        const obj = {};
        console.log(Object.prototype.toString.call(obj)); // [object Object]
        
        const arr = [];
        console.log(Object.prototype.toString.call(arr)); // [object Array]
  ```   

  >💡  **`.call()`** 메서드와 **`.apply()`** 는 첫 번째 인자로 객체를 받아서 같은 동작을 한다. call() 과의 차이는 두 번째 인자로 값이 들어오는 지, 배열이 들어오는 지 차이가 있다.
</details>
    
<details>
  <summary><h3>`Object.prototype.toString.call(객체)` 에서 객체 대신 원시타입을 전달해도 원시타입의 Class 가 나오는 이유 </h3></summary>

- 자바스크립트는 원시 타입을 객체처럼 다룰 수 있도록 자동으로 **래핑(wrapping)**하는 과정을 거친다. 이를 **박싱(boxing)**이라고 한다.
- 원시 타입은 객체가 아니지만, 자바스크립트는 원시 값을 일시적으로 객체처럼 다룰 수 있도록 **박싱한**다. 따라서 `Object.prototype.toString.call()` 함수는 박싱된 객체의 내부 `[[Class]]` 값을 사용해 해당 원시 타입의 객체 버전을 반환하는 것이다.

### 박싱(자동 래핑)의 동작

자바스크립트는 원시 타입을 객체처럼 사용할 때, 일시적으로 해당 값을 감싸는 객체를 생성한다. 예를 들어, `123` 같은 숫자는 원래 원시 값이지만, 객체의 메서드처럼 다룰 때 자바스크립트는 자동으로 숫자 원시 값을 `Number` 객체로 래핑한다.

```tsx
const num = 123
console.log(num.toString());  // "123" (Number 객체의 메서드를 사용함)
```

### `Object.prototype.toString.call(123)`의 동작

`Object.prototype.toString.call(123)`을 실행하면 내부적으로 자바스크립트 엔진이 숫자 원시 값을 `Number` 객체로 래핑하므로 결과값은 `"[object Number]"`가 된다.

```tsx
console.log(Object.prototype.toString.call(123));  // [object Number]
```

이는 원시 값 `123`을 **일시적으로** `Number` 객체로 변환해서 해당 객체의 `[[Class]]` 속성을 참조하기 때문에 `[object Number]`가 반환된다. 

- 하지만 이는 박싱된 객체일 뿐, 원시 값 자체가 객체로 변환된 것은 아니다.
</details>
---

## 추가적인 타입 검사

### satisfies

- 타입스크립트 4.9 버전에 추가된 연산자
- 타입 추론을 그대로 활용하면서 추가로 타입 검사를 하고 싶을 때 사용할 수 있다.

```tsx
type Universe = {
  [key in 'sun' | 'sirius' | 'earth']: string | {type: string; parent: string};
}

// sirius 속성이 s`ii`rius 로 오타가 났을 때,
// 타입으로 바로 지정해주면 에러는 잡을 수 있지만, 다른 타입의 속성값에 접근할 때 문제가 생긴다
// Problem
const universe1:Universe = {
  sun: 'star',
  siirius: 'star', // 🚨 Object literal may only specify known properties, but 'siirius' does not exist in type 'Universe'. Did you mean to write 'sirius'?
  earth: {type: 'planet', parent: 'sun'}
}

// universe.earth 도 string | {type: string; parent: string} 타입으로 추론되므로 earth 의 속성에 접근할 수 없는 문제가 생긴다.
universe.earth.parent // 🚨 Property 'parent' does not exist on type 'string | { type: string; parent: string; }'.  Property 'parent' does not exist on type 'string'.

// Ok
const universe2 = {
  sun: 'star',
  sirius: 'star',
  earth: {type: 'planet', parent: 'sun'}
} satisfies Universe;

/* universe2 의 타입 
const universe: {
    sun: string;
    sirius: string;
    earth: {
        type: string;
        parent: string;
    };
}
*/
```

---

## 타입 좁히기

### 1. 타입스크립트에도 자바스크립트의 문법을 사용하므로, 타입 좁히기에 꼭 typeof 를 사용할 필요는 없다.

```tsx
/* undefined, string, null 을 제대로 구분하지 못한 예 */

function typeCheck(param: string|null|undefined){
  if(typeof param === 'undefined'){
    console.log(param, 'is undefined');
  } else if(param){ // typeof null 은 object 
    console.log(param, 'is string');
  }else{
    console.log(param, 'is string or null');
  }
}

/* 자바스크립트 문법을 활용해서 구분한 예 */
function typeCheck(param: string|null|undefined){
  if(param === undefined){
    console.log(param, 'is undefined');
  } else if(param === null){
    console.log(param, 'is string');
  }else{
    console.log(param, 'is string');
  }
}
```

### 2. 타입 좁히기는 자바스크립트에서도 실행할 수 있는 코드여야 한다.

```tsx
interface X {
  x: number;
}

interface Y {
 y: number;
}

// Error1. if 문은 자바스크립트에서 실행되는 코드인데, 타입스크립트의 인터페이스를 사용해서 타입을 좁히려고 하면 에러가 발생한다.
function XorY(param: X | Y){
  if(param instanceof X){ // 🚨 'X' only refers to a type, but is being used as a value here.
    return param // (parameter) param: X | Y
  }
}

// Error2. 아직 타입 좁히기가 이루어지기 전 속성에 접근하면 에러가 발생한다
function XorY2(param: X | Y){
  if(param.x){ // 🚨 Property 'x' does not exist on type 'X | Y'.  Property 'x' does not exist on type 'Y'.
  	 return param // (parameter) param: X | Y
  }
}

// Ok
function XorY3(param: X | Y){
  if('x' in param){ 
    return param // (parameter) param: X 
  }
}
```
  
---
참고
 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gad_source=1&gclid=Cj0KCQjw3bm3BhDJARIsAKnHoVVvEm8AqscCY-aLy0EgZIFvKy0gml4t-osNfNr-2qC3shany_VtVlEaAo1aEALw_wcB)