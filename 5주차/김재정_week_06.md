# 9장 훅


## 9.1 리액트 훅

리액트 v16.8에서 훅이 추가되기 이전에는 클래스 컴포넌트에서만 상태를 가질 수 있었다.

**클래스 기반 컴포넌트의 단점**

- 컴포넌트 간 상태 로직을 재사용하기 어렵다.
  - 비슷한 형태의 상태 로직을 각 컴포넌트에서 직접 작성해야 했다.
- 생명주기 메서드에서 서로 관련 없는 로직들이 얽혀 코드의 복잡성을 증가시키는 문제가 있었다.
  - `componentDidMount`, `componentDidUpdate`와 같은 생명주기 메서드에서만 상태 업데이트에 대한 사이드 이펙트를 처리할 수 있었다.
- 총평) 프로젝트 규모가 커지면, 상태를 스토어에 연결하거나 비슷한 로직을 가진 상태 업데이트 및 사이드 이펙트 처리가 불편했다.

<br />

리액트 v16.8에서 리액트 훅이 도입되면서 함수 컴포넌트에서도 클래스 컴포넌트와 같이 **컴포넌트의 생명주기에 맞춰 로직을 실행할 수 있게 됐다.**

<br />

**리액트 훅의 장점**

- 비즈니스 로직을 재사용하거나 작은 단위로 코드를 분할하여 테스트하는게 용이해졌다.
- 사이드 이펙트와 상태를 관심사에 맞게 분리하여 구성할 수 있게 되었다.

<br />

### (1) useState

함수 컴포넌트에서 **상태를 관리**하기 위해 사용하는 훅이다.

<br />

**타입 정의**

```tsx
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);

// useState의 반환튜플 첫 번째 요소 : useState로 관리할 상태 타입 S
// 즉, useState는 어떤 타입의 상태든 관리할 수 있음
// useState의 반환튜플 두 번째 요소 : 상태를 업데이트할 함수 Dispatch
```

```tsx
setCount(10); // 상태를 직접 10으로 설정
setCount(prevCount => prevCount + 1); // 이전 상태를 기반으로 1 증가
```

<br />

**useState + typescript 활용 예제**

```tsx
const [memberList, setMemberList] = useState([
  {
    name: "KingBaedal",
    age: 10,
  },
  {
    name: "MayBaedal",
    age: 9,
  },
]);
```

다음과 같이 `memberList`의 초기값에 해당하는 객체를 직접 지정한 상황이다. 객체의 값은 `name`과 `age` 두 개다.

그리고 `setMemberList`를 이용해 `memberList`에 요소를 추가하기 위해 아래와 같은 함수 `addMember()`를 작성하는데... `age`가 아니라 `agee`로 오타를 냈다.

```tsx
const addMember = () => {
  setMemberList([
    ...memberList,
    {
      name: "DokgoBaedal",
      agee: 11, // <-- age 대신에 agee로 오타를 냈다면?
    },
  ]);
};
```

이렇게 되면 아래와 같이 `addMember()`를 실행시키면 `member.age`를 구할 수 없어 `sumAge`가 NaN가 되어버린다.

```tsx
const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);
console.log(sumAge); // ? NaN
```

위와 같은 상황을 예상치 못한 사이드 이펙트가 발생했다고 본다.

이러한 문제가 일어나는 것을 방어하기 위해서 typescript를 활용, 해당 컴포넌트에서 다루는 상태 타입을 지정할 수 있다.

```tsx
// after
import { useState } from "react";

// memberList의 타입 선언
interface Member {
  name: string;
  age: number;
}

const MemberList = () => {
  const [memberList, setMemberList] = useState<Member[]>([]); // 타입 지정

  // member의 타입이 Member 타입으로 보장됨
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);

  const addMember = () => {
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };
  // 🚨 ERROR : Type 'Member | { name: string; agee: number; }' is not assignable to type 'Member'.

  return ( /* ... */ );
};
```

`setMemberList`의 호출 부분에서 추가하려는 새 객체 타입을 확인하여 컴파일타임에 타입 에러를 발견할 수 있다.

<br />

### (2-1) useEffect

**렌더링 이후** 리액트 함수 컴포넌트에 **어떤 일을 수행해야 하는지 알려주기 위해** 사용하는 훅이다.

<br />

**타입 정의**

```tsx
function useEffect(effect: EffectCallback, deps?: DependencyList): void;

type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

useEffect의 첫 번째 인자 effect의 타입

- 아무것도 반환하지 않거나(void) 클린업 함수(Destructor)를 반환한다.
- Promise는 반환하지 않는다
- 즉, **useEffect의 콜백 함수에 비동기 함수가 들어갈 수 없다**
- useEffect에서 비동기 함수를 호출할 수 있다면 경쟁 상태를 불러일으킬 수 있기 때문이다.
  - 경쟁 상태(Race Condition): 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제. 이 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원하지 않는 방향으로 흐를 수 있게 된다.

useEffect의 두 번째 인자 deps의 타입

- 옵셔널함(`deps?`)
- effect가 수행되기 위한 조건을 나열한 배열
- **dep 배열의 원소가 변경되면 실행한다는 식으로 사용된다.**

<br />

**의존성 배열**

useEffect는 의존성 배열(`deps`)의 원소가 변화하면 실행한다. 이 때 deps가 변경되었는지를 **얕은 비교**로만 판단한다. 실제 객체 값이 바뀌지 않았더라도 객체의 참조 값이 변경되면 콜백 함수가 실행된다.

때문에 부모에서 받은 인자를 직접 deps로 작성한 경우 원치 않는 렌더링이 반복될 수 있다. 이를 방지하기 위해 **실제로 사용하는 값을 deps에서 사용해야 한다.**

<br />

**동작**

- deps가 빈 배열이라면
  - useEffect의 콜백 함수가 컴포넌트가 처음 렌더링 될 때만 실행됨
  - Destructor(클린업 함수)는 컴포넌트가 마운트 해제될 때 실행됨
- deps 배열이 존재한다면
  - 배열의 값이 변경될 때마다 Destructor가 실행됨

<br />

**useEffect + typescript 활용 예제**

```tsx
type SomeObject = {
  name: string;
  id: string;
};

interface LabelProps {
  value: SomeObject;
}

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    /* 수행할 일 */
  }, [value]);
};
```

위의 경우가 부모에게서 받은 인자 `value`를 직접 deps로 작성한 경우로, 원치 않는 렌더링이 반복될 수 있다. 이를 방지하기 위해 실제로 사용하는 `name`, `id`를 deps로 사용한다.

```tsx
// after
const Label: React.FC<LabelProps> = ({ value }) => {
  const { id, name } = value;
  useEffect(() => {
    /* 수행할 일 */
  }, [id, name]); // value.name과 value.id 대신 name, id 직접 사용
};
```

<br />

### (2-2) useLayoutEffect

useEffect와 비슷한 역할을 하는 훅이다.

useEffect는 `componentDidUpdate`와 같은 기존 생명주기 함수와는 다르게, 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행된다.

그에 반해 useLayoutEffect는 **화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행한다.**

<br />

**타입 정의**

```tsx
// useEffect와 타입 정의가 동일하다
function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;

type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

<br />

**useLayoutEffect + typescript 활용 예제**

```tsx
useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래 setName()을 실행한다고 가정
  setName("배달이");
}, []);

return <div>{`안녕하세요, ${name}님!`}</div>;
```

만일 위처럼 useEffect를 이용한다면 name이 빈칸으로 렌더링되다가 오랜 시간 이후에 "배달이"가 렌더링될 것이다. 사용자는 빈 이름을 오랫동안 보게 된다.

```tsx
useLayoutEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래 setName()을 실행한다고 가정
  setName("배달이");
}, []);

return <div>{`안녕하세요, ${name}님!`}</div>;
```

대신에 useLayoutEffect를 이용한다면 화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행하기 때문에 첫 번째 렌더링 때 빈 이름이 뜨는 경우를 방지할 수 있다.

<br />

### (2-3) useMemo와 useCallback

**이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성해주지 않도록** 해주는 훅이다. 어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용한다.

useEffect와 비슷하게 의존성 배열을 갖고 있으며, 배열 요소가 변화하면 값을 다시 계산한다.

불필요한 곳에 과도하게 메모이제이션하면 컴포넌트의 성능 향상이 보장되지 않을 수 있으니 주의해서 사용해야 한다.

<br />

**타입 정의**

```tsx
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList
): T;

//factory: 값을 생성하는 함수입니다. 
//useMemo는 이 함수의 결과값을 기억하고, 의존성 배열에 포함된 값들이 변경되지 않는 한 
// 이전에 계산된 결과값을 반환합니다.
```

```tsx
const MyComponent = ({ items }) => {
  // items의 총합을 계산하는 함수, 이 계산이 무겁다고 가정
  const total = useMemo(() => items.reduce((sum, item) => sum + item.price, 0), [items]);

  // 클릭 핸들러, items가 변경되지 않으면 함수도 재생성되지 않음
  const handleClick = useCallback(() => {
    console.log("Total clicked:", total);
  }, [total]);

  return <button onClick={handleClick}>Total: {total}</button>;
};

```

useMemo와 useCallback의 차이점 요약
useMemo: 값을 메모이제이션합니다. 계산 결과가 변경되지 않았으면 이전 값을 재사용합니다.
useCallback: 함수를 메모이제이션합니다. 함수 참조값이 변경되지 않았으면 이전 함수를 재사용합니다.


useMemo: items 배열이 변경될 때만 total을 다시 계산하고, 변경되지 않으면 이전에 계산된 값을 사용

useCallback: total이 변경될 때만 handleClick 함수를 다시 생성하고, 변경되지 않으면 이전 함수를 재사용

<br />

### (3) useRef

리액트 애플리케이션에서 `<input />`요소에 포커스를 설정하거나 특정 컴포넌트의 위치로 스크롤을 하는 등 **DOM을 직접 선택해야 하는 경우**에 사용한다.

<br />

**타입 정의**

useRef는 세 종류의 타입 정의를 갖고 있다. 인자의 타입에 따라 반환되는 타입이 달라진다.

```tsx
// (1) 제네릭으로 HTMLInputElement | null 사용시 아래 타입 반환
function useRef<T>(initialValue: T): MutableRefObject<T>;
// (2) 제네릭으로 HTMLInputElement, 인자로 null 사용시 아래 타입 반환
function useRef<T>(initialValue: T | null): RefObject<T>;
// (3) 초기값 없이 useRef 사용시 아래 타입 반환
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T; // current값 변경 가능
}

interface RefObject<T> {
  readonly current: T | null; // readonly로 인해 current값 변경 불가능
}
```

<br />

**useRef + typescript 활용 예제 (1) input DOM 포커스 설정**

```tsx
import { useRef } from "react";

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null);
  // ref가 null로 초기화됐다가 <input ref={ref} />를 통해 이 <input>을 가리키게 된다
  const onClick = () => {
    ref.current?.focus(); // <button> 클릭 시 <input>에 포커스 설정
  };
  return (
    <>
      <button onClick={onClick}>ref에 포커스!</button>
      <input ref={ref} />
    </>
  );
};

export default MyComponent;
```

<br />

**useRef + typescript 활용 예제 (2) 자식 컴포넌트에 ref 전달하기**

`<button />`이나 `<input />`과 같은 기본 HTML 요소가 아닌, 리액트 컴포넌트를 ref로 전달할 수도 있다.

그러나 이때 ref를 일반적인 props로 넘겨주는 방식으로 전달하면 브라우저에서 경고 메세지를 띄운다.

```tsx
import { RefObject, useRef } from "react";

const Component = () => {
  const ref = useRef<HTMLInputElement>(null);
  return <MyInput ref={ref} />;
};

interface Props {
  ref: RefObject<HTMLInputElement>;
}

const MyInput = ({ ref }: Props) => {
  return <input ref={ref} />;
};
```

```shell
🚨 RunTime WARNING : MyInput: `ref` is not a prop.
Trying to access it will result in `undefined` being returned.
If you need to access the same value within the child component, you should pass it as a different prop.
(https://reactjs.org/link/special-props)
```

ref라는 속성의 이름이 "DOM 요소 접근"이라는 특수한 목적으로 사용되기 때문이다.
해결 방법으로 **`forwardRef`** 를 사용하거나 `ref`가 아닌 `inputRef` 등의 다른 이름을 사용하는 방법이 있다. 아래 코드는 전자에 해당한다.

```tsx
// after
import { forwardRef, useRef } from "react";

const Component = () => {
  const ref = useRef<HTMLInputElement>(null);
  return <MyInput ref={ref} name="인풋이름" />; // 아래 Props에서 정의한 필수 속성 name
};

// 꼭 받을 속성을 추가로 정의하기 위한 타입 정의
interface Props {
  name: string;
}

const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  // forwardRef의 두 번째 인자에 ref를 넣어 자식 컴포넌트로 ref를 전달
  // forwardRef의 타입 정의로 인해 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계없이 자식 컴포너트가 해당 ref를 수용할 수 있다.
  return (
    <div>
      <label>{props.name}</label>
      <input ref={ref} />
    </div>
  );
});
```

<br />

**useRef의 여러가지 특성**

useRef는 자식 컴포넌트를 저장하는 변수로 활용할 수 있을 뿐만 아니라 다른 방식으로도 유용하게 사용할 수 있다.

- useRef로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않는다.
  - 상태가 변경되더라도 불필요한 렌더링을 피할 수 있게 된다.
- useRef로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.
  - 리액트 컴포넌트의 상태가 상태 변경 함수를 호출하고 렌더링 이후에 업데이트된 상태를 조회할 수 있는 점과 상반된다.

<br />

**useRef + typescript 활용 예제 (3) 현재 상태를 ref로 확인하기**

```tsx
import { useRef } from "react";

type BannerProps = {
  autoplay: boolean;
};

const Banner: React.FC<BannerProps> = ({ autoplay }) => {
  const isAutoPlayPause = useRef(false);
  if (autoplay) {
    // keepAutoPlay 같이 isAutoPlay가 변하자마자 사용해야 할 때 쓸 수 있다
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;
    // ...
  }
  return (
    <>
      {autoplay && (
        <>
          <button
            aria-label="자동 재생 일시 정지"
            onClick={() => {
              isAutoPlayPause.current = true;
            }}
          ></button>
          {/* isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 주므로 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없다 */}
        </>
      )}
    </>
  );
};
```

위에서 `isAutoPlayPause`는 현재 자동 재생이 일시 정지 되었는지를 확인하는 ref다. 이 변수는 렌더링에 영향을 미치지 않고, 값이 변경되더라도 다시 렌더링을 기다릴 필요 없이 사용할 수 있어야한다. 때문에 예시는 `isAutoPlayPause.current`에 null이 아닌 값을 할당해서 마치 변수처럼 활용할 수 있게 했다.

<br />

**훅의 규칙**

리액트 훅을 안전하게 사용하기 위한 2가지 규칙

- **훅은 항상 최상위 레벨에서 호출되어야 한다.**
  - 조건문, 반복문, 중첩 함수, 클래스 등의 내부에서는 훅을 호출하지 않아야 한다.
  - 반환문으로 함수 컴포넌트가 종료되거나, 조건문 또는 변수에 따라 반복문 등으로 훅의 호출 여부가 결정되어서는 안된다.
  - useState, useEffect가 여러 번 호출되더라고 훅의 상태를 올바르게 유지하기 위함이다.
- **훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.**

위 규칙을 지켜야 컴포넌트의 모든 상태 관련 로직을 좀 더 명확하게 드러낼 수 있다. 리액트에서 훅은 호출 순서에 의존하기 때문에 항상 동일한 컴포넌트 렌더링을 보장해야만 한다.

<br />

## 9.2 커스텀 훅

### (1) 나만의 훅 만들기

리액트 기본 훅 useState, useRef 등의 훅에 더해 **사용자 정의 훅을 생성하여 컴포넌트 로직을 함수로 뽑아내 재사용**할 수 있다. 이를 **커스텀 훅(Custom Hook)** 이라고 명한다.

커스텀 훅은 리액트 컴포넌트 내에서만 사용할 수 있으며, 이름은 반드시 use로 시작해야 한다.

<br />

예제로 가장 유명한 커스텀 훅 useInput을 확인해보자.

```jsx
// hooks/useInput.jsx
import { useState, useCallback } from "react";

const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue); // 인자로 받은 값을 초기값으로 지정하여 관리
  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);
  // input의 값 value와
  // 해당 값을 수정할 수 있는 함수 onChange를 반환하는 훅
  return { value, onChange };
};

export default useInput;
```

```jsx
// App.jsx
import useInput from "./hooks/useInput";

function App() {
  const { value, onChange } = useInput("");
  return (
    <>
      <h1>{value}</h1>
      <input onChange={onChange} value={value} />
    </>
  );
}

export default App;
```

<br />

### (2) 타입스크립트로 커스텀 훅 강화하기

타입스크립트에서 위 코드를 실행하면 에러가 발생한다.

```tsx
// hooks/useInput.tsx
import { useState, useCallback } from "react";

const useInput = (initialValue) => {
  // 🚨 ERROR : Parameter 'initialValue' implicitly has an 'any' type.
  const [value, setValue] = useState(initialValue);
  const onChange = useCallback((e) => {
    // 🚨 ERROR : Parameter 'e' implicitly has an 'any' type.
    setValue(e.target.value);
  }, []);
  return { value, onChange };
};

export default useInput;
```

위 에러를 해결하기 위해 타입을 지정해주자.

```tsx
// after
import { ChangeEvent, useCallback, useState } from "react";

const useInput = (initialValue: string) => {
  // 인자 타입 추가
  const [value, setValue] = useState(initialValue);
  const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    // 이벤트 타입 추가
    setValue(e.target.value);
  }, []);
  return { value, onChange };
};

export default useInput;
```
