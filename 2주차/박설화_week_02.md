# [🔗 2주차: (5장) 타입 활용하기 - 커스텀 유틸리티 타입 PickOne](https://velog.io/@iberis/14장-객체-타입-타입-확장-타입-좁히기)

[커스텀 유틸리티 타입 PickOne](#커스텀-유틸리티-타입-PickOne)\
[책에 나와 있는 PickOne 타입을 활용](#책에-나와-있는-PickOne-타입을-활용)\
[하지만 여기서 문제](#하지만-여기서-문제)\
[직접 PickOne 타입 구현 과정](#직접-PickOne-타입-구현-과정)\
[최종 결과](#최종-결과)\
[번외: 실제 적용한 프로젝트에서는...](#번외-:-실제-적용한-프로젝트에서는...)

5장에서는 조건부타입, 유틸리티 타입 등 타입을 좀 더 활용할 수 있는 방법들에 대한 소개가 나왔다. 

그 중에서도 유틸리티 타입을 좀 더 커스텀해서 활용하는 방법이 흥미로웠다.
특히 책에서 소개된 커스텀 유틸리티 타입 중 PickOne 타입에 대해, 마침 진행 중인 프로젝트에서 필요한 경우가 생겨서 좀 더 연구해보게 되었다.

# 커스텀 유틸리티 타입 PickOne
책 168 페이지

서로 다른 속성을 가지는 두 객체 타입이 있을 때, 두 타입 중 하나의 타입에만 해당하는 타입을 만들어야 한다.
```ts
type IconOption = {name: string};
type TextOption = {children: ReactNode};

// type Option = IconOption 타입 또는 TextOption 타입
```
Option 타입은 IconOption 타입 또는 TextOption 타입이다.
name 이 있는 경우 children 을 가질 수 없고, 반대로 children 이 있는 경우 name 은 가질 수 없는 타입이어야 한다.

`type Option = IconOption | TextOption` 으로 구현하면 name 과 children 을 모두 허용하게 되는 문제가 있다.

## 책에 나와 있는 PickOne 타입을 활용
```ts
// 책 168 페이지

type PickOne<T> = {
[P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>> }[keyof T];


type PickOneOption = PickOne<IconOption | TextOption>

const options1:PickOneOption = [
  {name: 'icon'}, // Ok
  {children: 123}, // Ok
  {name: 'icon', children: 123}, // Error: 'number' 형식은 'undefined' 형식에 할당할 수 없습니다.ts(2322)
];
```
name 과 children 을 둘 다 가지려고 하는 경우 children 에서 에러가 발생하며 타입 가드가 잘 된다.


### 하지만 여기서 문제
```ts
type IconOption = {
 name: string;
  onClick?: (event: React.MouseEvent<HTMLInputElement>) => void;
};

type TextOption = {
  children: ReactNode;
  onClick?: (event: React.MouseEvent<HTMLInputElement>) => void;
};

// onClick 의 타입도 undefined 로 강제된다.
const options3:PickOneOption = [
  {iconName: 'icon', onClick: () => {console.log('icon type')}}  // Error: '() => void' 형식은 'undefined' 형식에 할당할 수 없습니다.
  ];
```

책에서 나와 있는 예시는 정말 각 타입의 객체가 단 1개의 key, value 만 가질 때에만 가능한 PickOne 타입이었다.\
내 프로젝트에서는 IconOption, TextOption 에서 각각 onClick 을 옵셔널한 속성으로 필요했기에, 책의 PickOne 타입은 사용할 수 없었다.

또한 단 한개의 key, value 만 가지는 객체들 중 하나의 타입을 선택하는 일이 그렇게 많지 않을 것 같아서, \
차라리 새롭게 나의 PickOne 커스텀 유틸리티 타입을 만들어 보기로 했다.

## 직접 PickOne 타입 구현 과정

1. 우선 2가지 객체 타입인 경우 부터!\
서로 겹치는 속성이 없는 두 가지 객체 타입을 제네릭으로 받아서, 둘 중 하나의 객체 타입만 선택 가능한 PickOne 타입을 만든다.
```ts
type PickOne<T, U> = (T & Partial<Record<keyof U,undefined>>) | (U & Partial<Record<keyof T, undefined>>);


type IconOption = { name: string; color: string; };
type TextOption = { children: number };
type OptionsType = PickOne<IconOption, TextOption>[]

const option: OptionsType = [
  {name: 'icon', color: '#ffff'}, // Ok
  {children: 123 } // Ok
  {name: 'icon', color: '#ffff', children: 123 } // Error

    // 타입 가드가 잘 된다 🙂
    // Type '{ name: string; color: string; children: number; }' is not assignable to type 'PickOne<IconOption, TextOption> | PickOne<TextOption, IconOption>'.
  //  Types of property 'children' are incompatible.
  //  Type 'number' is not assignable to type 'undefined'.
]
```
2. 3개 이상의 객체 타입을 받는 경우를 생각해 본다면,\
각 타입을 PickOne 으로 연결한 조합을 만들어야 한다.
```ts
type PickOne<T, U> = T & Partial<Record<keyof U,undefined>>;
type IconOption = { name: string; color: string; };
type TextOption = { children: number; };
type ButtonOption = { button: string; onClick: () => void}

type OptionsType = (PickOne<IconOption, TextOption & ButtonOption> 
                    | PickOne<TextOption, IconOption & ButtonOption> 
                    | PickOne<ButtonOption, IconOption & TextOption>)[];
```
만약 전달되는 객체 타입의 개수가 명확하다면,\ 
좀 더 간결히 사용할 수 있도록 한 번 더 제네릭으로 감쌀 수 있다.
```ts
type PickOneType<A, B, C> = (PickOne<A, B & C> | PickOne<B, A & C> | PickOne<C, A & A>)[];

// 사용할 때
type OptionsType = PickOneType<IconOption, TextOption, ButtonOption>;
```

3. 이제, 선택해야하는 객체 타입이 계속 추가되는 경우를 생각해보자\
만약 또 다른 객체 타입들이 추가될 경우 매번 PickOneType 에 각 조합 `| PickOne<추가된 타입, 나머지 타입들>` 을 추가해야한다면 너무 번거로울 것이다.\

이를 위해, 각 객체 타입들을 배열로 타입을 받아서 재귀적으로 배열의 각 요소를 순회하면서 첫 번째 타입에는 자기 자신을 두고, 나머지는 &로 연결하도록 했다.

```ts
// Rest 배열을 &로 연결하는 타입
type Unionize<T extends any[]> = T extends [infer First, ...infer Rest] ? First & Unionize<Rest> : {};

// 재귀적으로 배열의 각 요소를 순회하면서 첫 번째 타입에는 자기 자신을 두고, 나머지는 &로 연결
type PickOneType<T extends any[]> = T extends [infer First, ...infer Rest]
  ? PickOne<First, Unionize<Rest>> | PickOneType<Rest>
  : never;
```

## 최종 결과
```ts
type PickOne<T, U> = T & Partial<Record<keyof U,undefined>>;

// Rest 배열을 &로 연결하는 타입
type Unionize<T extends any[]> = T extends [infer First, ...infer Rest] ? First & Unionize<Rest> : {};

// 재귀적으로 배열의 각 요소를 순회하면서 첫 번째 타입에는 자기 자신을 두고, 나머지는 &로 연결
type PickOneType<T extends any[]> = T extends [infer First, ...infer Rest]
  ? PickOne<First, Unionize<Rest>> | PickOneType<Rest>
  : never;
```
**사용 예시**
```ts
type IconOption = { name: string; color: string; };
type TextOption = { children: number; };
type ButtonOption = { button: string; onClick: () => void}

type OptionsType = PickOneType<[IconOption, TextOption, ButtonOption]>[];

const option: OptionsType = [
  {name: 'icon', color: '#ffff'}, // Ok
  {children: 123 }, // Ok
  {button: 'button',  onClick: () => {}}, // Ok
  {children: 123, button: 'asdf', onClick: () => {}} // Error
]
```

### 번외 : 실제 적용한 프로젝트에서는...
위에서 구현한 PickOneType 은 각 객체에서 서로 겹치는 key 가 하나도 있으면 안된다.
하지만 프로젝트에서는 onClick 속성이 옵셔널한 값으로 겹치기 때문에 적용할 수 없었다.\
(차라리 onClick의 타입이 옵셔널하지 않고, 각 객체마다 다른 값을 가지는 타입이었다면 Discriminated Unions 가 될 수 있도록 구분자 역할을 해줬을 텐데 😂)

구별해야하는 타입이 두 가지 타입이고 내부 속성도 많지 않아서 각 타입에서 들어가면 안되는 타입을 undefined 로 처리해주는 것으로 해결했다. \

```ts
type IconOption = {
  name: string;
  onClick?: () => void;
  children?: undefined;
};

type TextOption = {
  children: ReactNode;
  onClick?: () => void;
 name?: undefined;
};

type UnionOption = (IconOption | TextOption)[];


// 의도했던 대로 타입 가드도 잘 동작한다.
const options:UnionOption = [
  {name: 'icon'}, // Ok
  {children: 123}, // Ok
  {name: 'icon', onClick: () => {console.log('icon type')}}, // Ok
  {children: 123, onClick: () => {console.log('text type')}}, // Ok
  {name:'icon', children: 123, onClick: () => {console.log('text type')}} // Error
]; 
```

추후 객체가 서로 공통된 속성을 옵셔널하게 가질 때도 PickOne 타입 할 수 있는 custom utility 타입을 만들 수 있을 지 좀 더 생각해봐야겠다. 🤔