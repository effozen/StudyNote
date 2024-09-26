# 010 CreateElement 만들기

## 🎯 목표

> 목표 1. createElement를 구현해서 JSX 코드를 작성하면 내가 만든 createElement를 이용해서 reactElement를 생성할 수 있게 하자

<br/>

> 목표 2. 기존의 react 시스템과 호환이 되도록 만들어보자.


<br/><br/><br/>

## 🚀 `createElement`를 만들게 된 계기

| 계기                        | 설명                                                                                                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| JSX 트랜스파일러를 만들어보고자 시도한 경험 | ▸ `JSX` -> `JS`로 바꾸는 트렌스파일러를 만들려다가 시간 문제로 실패했다.<br/><br/>▸ 그 대신에 최소한 `createElement`를 만들어서 `reactElement`형태의 객체로 만들고, 이를 활용해보는 것 정도는 해보고자 한다.                                                           |
| `Virtual DOM`을 위한 밑작업     | ▸ `Virtual DOM`은 리엑트에서 사용하는 `DOM` 조작 방법 중 하나이다.<br/><br/>▸ `DOM`은 기본적으로 자바스크립트 객체이다.<br/>▸ `Vitual DOM`도 `JS 객체`이다.<br/><br/>▸ 그 객체를 구성하는게 `reactElement`이다.<br/><br/>▸ 세부적인 구현을 위해서 필연적으로 이를 구현하게 되었다. |
<br/><br/>

- JSX 트랜스파일을 하고 나면 `XML-liked` 요소가 JS로 `react.createElement()`로 변경이 된다.
- 이에 따라서, `react.createElement()`를 구현하면 `Virtual DOM`에 대한 진입점인, `reactElement`로의 생성을 이룰 수 있을 것 같았다.

<br/><br/><br/>

## 🚀 createElement의 정의

```javascript
function createElement(type, props, ...children) {
	...
	return reactElement;
}
```

위와 같은 형태로 되어 있다.<br/>
`return` 값이 `reactElement`라는 데 주목할 필요가 있다.<br/><br/>

[참고: 리엑트 공식문서](https://react.dev/reference/react/createElement)

## 🚀 reactElement의 이해
`createElement`는 리턴값이 `reactElement`이다. <br/>
구현에 앞서서 테스트코드를 먼저 작성하고, 기능 구현을 해보고자 했다.<br/><br/>

입력은 명확하다. `type`, `props`, `...children`<br/>
그러면, 출력은?<br/><br/>

이에 대해서 살펴보기 위해서 `react` 리포지토리를 포크떠서 살펴보았다.<br/>
다음과 같은 코드를 발견할 수 있었다.<br/><br/>

```javascript
import type {ReactDebugInfo} from './ReactTypes';  
  
export type ReactElement = {  
  $$typeof: any,  
  type: any,  
  key: any,  
  ref: any,  
  props: any,  
  // __DEV__ or for string refs  
  _owner: any,  
  
  // __DEV__  
  _store: {validated: 0 | 1 | 2, ...}, // 0: not validated, 1: validated, 2: force fail  
  _debugInfo: null | ReactDebugInfo,  
  _debugStack: Error,  
  _debugTask: null | ConsoleTask,  
};
```

`ReactElementType`에 있는 내용이다.<br/>
그러면 저 형태로 출력을 할 수 있게 하면 된다.<br/><br/>

다만 그냥 가져다쓰기만 하는 것은 자존심이 상하니, 각 속성에 대해서 학습해본 내용을 정리해봤다.<br/><br/><br/>

<hr/>

## 🤔 추가 학습 :: React Element의 \$\$typeof 속성 상세 설명

### 1. 기본 정의
- 값: `Symbol.for('react.element')`
- 목적: React Element의 유효성을 식별하는 내부 마커

### 2. 보안 목적
- 주요 기능: XSS(Cross-Site Scripting) 공격 방지
- 작동 원리: JSON으로 직렬화되지 않는 Symbol 타입 사용

### 3. 상세 설명
#### 3.1 Symbol 사용 이유
- `typeof` 대신 `$$typeof`를 사용하는 이유: JavaScript의 예약어와 충돌 방지
- Symbol의 특성:
  - 고유성: 각 Symbol은 유일함
  - JSON 직렬화 불가: `JSON.stringify()`로 변환 시 무시됨

#### 3.2 XSS 공격 방지 메커니즘
1. 서버에서 JSON으로 React 요소를 전송할 경우:
   - `$$typeof`가 포함되지 않음 (Symbol은 JSON으로 직렬화되지 않음)
2. 클라이언트에서 받은 JSON을 React 요소로 사용 시:
   - React는 `$$typeof` 확인
   - `$$typeof`가 없거나 올바르지 않으면 요소로 취급하지 않음

#### 3.3 구현 예시
```javascript
const element = {
  $$typeof: Symbol.for('react.element'),
  type: 'div',
  props: { children: 'hello' }
};

// React는 이를 유효한 요소로 인식
ReactDOM.render(element, container);
```

### 4. 역사적 배경
- 도입 시기: React v0.13
- 도입 이유: 
  - 이전 버전에서는 JSON으로 전달된 객체를 유효한 React 요소로 취급
  - 이로 인해 XSS 취약점 발생 가능성

### 5. 개발자 관점에서의 중요성
- 직접 조작 불필요: React.createElement()가 자동으로 설정
- 디버깅 시 유용: 콘솔에서 React 요소 식별에 도움
- 커스텀 렌더러 개발 시 고려 필요

### 6. 성능 영향
- 미미한 오버헤드: Symbol 비교는 매우 빠름
- 보안 이점이 성능 저하를 크게 상회

### 7. 브라우저 지원
- 모든 최신 브라우저에서 지원
- 구형 브라우저를 위한 폴리필 제공

### 8. 관련 React API
- React.isValidElement(): \$\$typeof를 확인하여 유효한 React 요소인지 판단

이해를 돕기 위한 코드 예시:
```javascript
function isReactElement(obj) {
  return typeof obj === 'object' && obj !== null && obj.$$typeof === Symbol.for('react.element');
}

const validElement = React.createElement('div');
console.log(isReactElement(validElement)); // true

const invalidElement = { type: 'div', props: {} };
console.log(isReactElement(invalidElement)); // false
```


## 🤔 추가 학습 :: `type` 속성에 대한 이해

### 1. 기본 정의
- 목적: React Element가 어떤 종류의 컴포넌트나 DOM 요소로 렌더링될지 지정
- 가능한 값 타입: string | React.ComponentType\<any\> | React.ReactFragment

### 2. 가능한 값의 종류

#### 2.1 문자열 (String)
- 사용: 내장 DOM 요소 지정
- 예시: 'div', 'span', 'p', 'button' 등
- 특징: 소문자로 시작

```javascript
React.createElement('div', null, 'Hello World');
// JSX 등가: <div>Hello World</div>
```

#### 2.2 React 컴포넌트 (React.ComponentType)
- 함수형 컴포넌트
  ```javascript
  function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
  }
  React.createElement(Welcome, { name: 'Sara' });
  // JSX 등가: <Welcome name="Sara" />
  ```

- 클래스형 컴포넌트
  ```javascript
  class Welcome extends React.Component {
    render() {
      return <h1>Hello, {this.props.name}</h1>;
    }
  }
  React.createElement(Welcome, { name: 'Sara' });
  // JSX 등가: <Welcome name="Sara" />
  ```

#### 2.3 React Fragment (React.ReactFragment)
- 사용: 여러 요소를 그룹화할 때 사용
- 값: React.Fragment 또는 단축 문법인 <></>
```javascript
React.createElement(React.Fragment, null, 
  React.createElement('li', null, 'Item 1'),
  React.createElement('li', null, 'Item 2')
);
// JSX 등가: <><li>Item 1</li><li>Item 2</li></>
```

### 3. 동작 원리
1. React는 type 값을 확인하여 렌더링 방식을 결정
2. 문자열인 경우: 해당 DOM 요소 생성
3. 함수/클래스인 경우: 해당 컴포넌트 실행/인스턴스화
4. Fragment인 경우: 자식 요소들만 렌더링

### 4. 특별한 type 값들

#### 4.1 React.lazy()
- 동적 임포트를 위한 특별한 type
- 코드 분할(Code-splitting)에 사용
```javascript
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

#### 4.2 React.memo()
- 성능 최적화를 위한 고차 컴포넌트
```javascript
const MemoizedComponent = React.memo(function MyComponent(props) {
  // ...
});
```

### 5. type 검사 및 처리
- React 내부에서 `typeof type`을 사용하여 처리 방식 결정
- 문자열: DOM 요소로 처리
- 함수: 함수형 컴포넌트 또는 클래스형 컴포넌트로 처리
- 객체 (클래스 인스턴스): 클래스형 컴포넌트로 처리

### 6. 개발 시 주의사항
- 컴포넌트 이름은 항상 대문자로 시작 (React가 DOM 요소와 구분하기 위함)
- type으로 동적 값 사용 시 주의 필요 (렌더링 최적화에 영향)

### 7. 성능 고려사항
- type이 매 렌더링마다 변경되면 불필요한 리렌더링 발생 가능
- 가능하면 type을 상수나 컴포넌트 외부에 정의하여 안정성 확보

### 8. TypeScript에서의 사용
```typescript
type ReactElementType = 
  | string 
  | React.FunctionComponent<any> 
  | React.ComponentClass<any> 
  | React.ReactFragment;

const element: React.ReactElement<any, ReactElementType> = 
  React.createElement('div', null, 'Hello');
```

이해를 돕기 위한 코드 예시:
```javascript
function renderElement(type, props, ...children) {
  if (typeof type === 'string') {
    // DOM 요소 렌더링
    return document.createElement(type);
  } else if (typeof type === 'function') {
    // 함수형 또는 클래스형 컴포넌트 렌더링
    return type(props);
  } else if (type === React.Fragment) {
    // Fragment 렌더링
    return children;
  }
  // 다른 특수한 경우 처리...
}
```


## 🤔 추가 학습 :: React Element의 key 속성 상세 설명

> [Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state) :: 여기서 key의 이유 확인 가능

### 1. 기본 정의
- 타입: string | number | null
- 목적: 동적인 리스트에서 각 항목을 고유하게 식별
- 기본값: null

### 2. 주요 기능
- 리스트 렌더링 최적화
- 컴포넌트 상태 유지
- 불필요한 DOM 조작 최소화

### 3. 작동 원리
1. React는 렌더링 시 각 요소의 key를 비교
2. key가 같으면 기존 컴포넌트 재사용
3. key가 다르면 컴포넌트를 새로 생성하고 이전 컴포넌트 제거

### 4. 올바른 key 사용법

#### 4.1 고유성
- 형제 요소 사이에서만 고유하면 됨
- 전체 애플리케이션에서 고유할 필요는 없음

#### 4.2 안정성
- 렌더링 간에 변경되지 않아야 함
- 배열의 인덱스를 key로 사용하는 것은 권장되지 않음 (항목 순서 변경 시 문제 발생)

#### 4.3 예측 가능성
- 데이터의 고유 ID를 사용하는 것이 가장 이상적
  ```jsx
  const todoItems = todos.map((todo) =>
    <li key={todo.id}>
      {todo.text}
    </li>
  );
  ```

### 5. key 사용 시 주의사항

#### 5.1 인덱스를 key로 사용하는 경우
- 리스트가 정적이고 변경되지 않을 때만 사용
- 동적 리스트에서는 성능 저하와 버그 발생 가능성 있음
  ```jsx
  // 권장되지 않는 방식 (동적 리스트의 경우)
  const items = someItems.map((item, index) =>
    <li key={index}>{item.name}</li>
  );
  ```

#### 5.2 key 생성 시 주의점
- Math.random()이나 uuid를 사용하여 동적으로 생성하지 말 것
- 렌더링마다 새로운 key가 생성되어 성능 저하 발생

#### 5.3 컴포넌트에서 key 접근
- key는 props로 전달되지 않음
- 필요한 경우 다른 prop 이름으로 명시적 전달 필요
  ```jsx
  const Item = (props) => <li>{props.id}</li>;
  
  const List = () => (
    <ul>
      {items.map(item => <Item key={item.id} id={item.id} />)}
    </ul>
  );
  ```

### 6. key의 성능 영향
- 올바른 key 사용: 렌더링 성능 향상
- 잘못된 key 사용: 불필요한 리렌더링, DOM 조작 증가

### 7. 특수한 경우의 key 사용

#### 7.1 다중 렌더링 요소
```jsx
function ListItem(props) {
  return (
    <li key={props.id}>
      <h3>{props.name}</h3>
      <p>{props.description}</p>
    </li>
  );
}
```

#### 7.2 동적 컴포넌트 전환
```jsx
function App() {
  const [view, setView] = useState('list');
  
  return (
    <div>
      {view === 'list'
        ? <ListView key="list" />
        : <GridView key="grid" />}
    </div>
  );
}
```

### 8. React DevTools에서의 key 확인
- React DevTools를 사용하여 key 속성 확인 가능
- 누락된 key나 중복된 key 문제 식별에 유용

### 9. key와 관련된 React 내부 동작
```javascript
function updateChildren(prevChildren, nextChildren) {
  prevChildren.forEach((prevChild, index) => {
    const nextChild = nextChildren[index];
    if (prevChild.key === nextChild.key) {
      updateElement(prevChild, nextChild);
    } else {
      removeElement(prevChild);
      createNewElement(nextChild);
    }
  });
}
```

이 예시 코드는 React의 내부 로직을 간단히 표현한 것으로, 실제 구현은 더 복잡합니다.


## 🤔 추가 학습 : React Element의 props 속성 상세 설명

### 1. 기본 정의
- 타입: Object
- 목적: 부모 컴포넌트에서 자식 컴포넌트로 데이터 전달
- 특성: 읽기 전용 (Read-only)

### 2. props의 주요 특징

#### 2.1 불변성 (Immutability)
- 컴포넌트 내부에서 props를 직접 수정해서는 안 됨
- 새로운 props를 받아 리렌더링하는 방식으로 업데이트

#### 2.2 단방향 데이터 흐름
- 부모에서 자식으로만 데이터가 전달됨
- 자식 컴포넌트에서 부모의 상태를 직접 변경할 수 없음

#### 2.3 타입
- 모든 JavaScript 값 타입 가능 (문자열, 숫자, 객체, 함수 등)
- JSX를 통해 전달 시 문자열은 따옴표, 그 외는 중괄호 사용

### 3. props 사용 방법

#### 3.1 함수형 컴포넌트
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// 사용
<Welcome name="Sara" />
```

#### 3.2 클래스형 컴포넌트
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

#### 3.3 비구조화 할당
```jsx
function Welcome({ name, age }) {
  return <h1>Hello, {name}. You are {age} years old.</h1>;
}
```

### 4. 특별한 props

#### 4.1 children
- 컴포넌트의 여는 태그와 닫는 태그 사이의 내용
```jsx
function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

// 사용
<Layout>
  <h1>Welcome</h1>
  <p>This is my app</p>
</Layout>
```

#### 4.2 key (리스트 렌더링 시)
- 리스트의 각 항목을 고유하게 식별
```jsx
const items = ['apple', 'banana', 'orange'];
return (
  <ul>
    {items.map((item, index) => (
      <li key={index}>{item}</li>
    ))}
  </ul>
);
```

#### 4.3 className (HTML class 속성 대신 사용)
```jsx
<div className="my-class">Content</div>
```

### 5. props의 기본값 설정

#### 5.1 함수형 컴포넌트
```jsx
function Button({ text = 'Click me' }) {
  return <button>{text}</button>;
}
```

#### 5.2 클래스형 컴포넌트
```jsx
class Button extends React.Component {
  static defaultProps = {
    text: 'Click me'
  };

  render() {
    return <button>{this.props.text}</button>;
  }
}
```

### 6. props 검증

#### 6.1 PropTypes 사용
```jsx
import PropTypes from 'prop-types';

function User({ name, age }) {
  return <p>{name} is {age} years old</p>;
}

User.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number
};
```

### 7. 고급 props 패턴

#### 7.1 Render Props
```jsx
<Mouse render={mouse => (
  <Cat mouse={mouse} />
)}/>
```

#### 7.2 Higher-Order Components (HOC)
```jsx
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    // ... 로직 구현
    render() {
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

### 8. props와 성능 최적화

#### 8.1 React.memo
```jsx
const MyComponent = React.memo(function MyComponent(props) {
  // ... 컴포넌트 로직
});
```

#### 8.2 useMemo 훅
```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### 9. TypeScript에서의 props 사용
```typescript
interface WelcomeProps {
  name: string;
  age?: number;
}

const Welcome: React.FC<WelcomeProps> = ({ name, age }) => {
  return <h1>Hello, {name}. You are {age || 'unknown'} years old.</h1>;
};
```

이 예시는 TypeScript에서 props의 타입을 정의하고 사용하는 방법을 보여줍니다.

## 🤔 추가 학습 : React Element의 ref 속성 상세 설명

### 1. 기본 정의
- 타입: 
  - 함수: `(instance: any) => void`
  - 객체: `{ current: any }`
- 목적: DOM 요소나 클래스 컴포넌트 인스턴스에 직접 접근
- 기본값: null

### 2. 주요 기능
- DOM 요소 직접 조작
- 클래스 컴포넌트의 메서드 호출
- 컴포넌트 생명주기 외부에서 상태 관리

### 3. ref 생성 및 사용 방법

#### 3.1 createRef 사용 (클래스 컴포넌트)
```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
  componentDidMount() {
    console.log(this.myRef.current);
  }
}
```

#### 3.2 useRef 훅 사용 (함수형 컴포넌트)
```jsx
function MyComponent() {
  const myRef = useRef(null);
  
  useEffect(() => {
    console.log(myRef.current);
  }, []);

  return <div ref={myRef} />;
}
```

#### 3.3 콜백 ref
```jsx
function CustomTextInput() {
  const textInput = useRef(null);
  
  function handleClick() {
    textInput.current.focus();
  }

  return (
    <>
      <input type="text" ref={textInput} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

### 4. ref 사용 시 주의사항

#### 4.1 함수형 컴포넌트에 ref 전달
- 기본적으로 함수형 컴포�트는 인스턴스가 없어 ref를 가질 수 없음
- forwardRef를 사용하여 내부 DOM 요소나 클래스 컴포넌트로 ref 전달 가능
```jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 사용
const ref = useRef(null);
<FancyButton ref={ref}>Click me!</FancyButton>
```

#### 4.2 ref와 생명주기
- ref는 componentDidMount 또는 componentDidUpdate 이후에 설정됨
- 언마운트 시 null로 설정됨

#### 4.3 남용 주의
- ref의 과도한 사용은 React의 선언적 패러다임을 해칠 수 있음
- 가능한 한 상태와 props를 통한 제어를 우선적으로 고려

### 5. ref의 고급 사용법

#### 5.1 ref 속성 이름 변경
```jsx
const FancyInput = React.forwardRef((props, ref) => (
  <input {...props} ref={ref} />
));

// 사용
<FancyInput inputRef={inputRef} />;
```

#### 5.2 조건부 ref
```jsx
const MeasureExample = () => {
  const [height, setHeight] = useState(0);
  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
};
```

### 6. ref와 성능

#### 6.1 장점
- DOM 조작이 필요한 경우 효율적인 접근 제공
- 불필요한 리렌더링 방지 가능

#### 6.2 단점
- 과도한 사용 시 컴포넌트 간 결합도 증가
- 테스트와 디버깅 복잡성 증가 가능

### 7. ref 관련 Best Practices

1. 꼭 필요한 경우에만 사용 (예: 포커스, 텍스트 선택, 미디어 재생 제어 등)
2. 상태 관리에는 가능한 useState 사용
3. 레거시 코드나 비-React 라이브러리 통합 시 유용하게 활용
4. 성능 최적화를 위해 불필요한 리렌더링을 방지할 때 고려

### 8. TypeScript에서의 ref 사용
```typescript
import React, { useRef, RefObject } from 'react';

const MyComponent: React.FC = () => {
  const inputRef: RefObject<HTMLInputElement> = useRef(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
};
```

이 예시는 TypeScript에서 ref를 타입 안전하게 사용하는 방법을 보여줍니다.

<hr/><br/><br/><br/>

## 🧑‍💻 어떻게 구현해볼 것인가?

> 모든 구현은 함수형을 기준으로 진행한다.

### 📚 속성의 단순화

위에 보면 알겠지만 개발이나 `React`등에서 사용하는 속성들이 존재했다.<br/>
그리고 그 중에서 필수 속성을 뽑아내면 다음과 같다.<br/><br/>

|  필수 속성  |
| :-----: |
| `type`  |
| `props` |
| `keys`  |
|  `ref`  |

<br/><br/>

### 🧪 테스트 코드의 작성

위의 명세만 놓고 보았을 때 구현이 복잡해질 수도 있겠다는 생각이 들었다.<br/>
그래서 어떻게 할 지 고민하던 도중, 입력과 출력에 대한 명세가 너무 명확하다는 생각이 들었다.<br/><br/>

그래서, 테스트 코드를 작성해보기로 했다.<br/><br/>

테스트작성에 있어서 고려해야하는 상황은 다음과 같다. <br/><br/>

| 테스트해야하는 경우                                                  | 기타                                                                              |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 1. `type` 에 대한 테스트 :: `String` 태그가 들어왔을 경우                  | `string`                                                                        |
| 2. `type` 에 대한 테스트 :: `함수형` 태그가 들어왔을 경우                     | `React.FunctionComponent<any>`<br/><br/>▸ 단, 클래스형은 고려하지 않기로 해서.. 어떻게 할 지는 고민이다. |
| 3. `type` 에 대한 테스트 :: `클래스형` 태그가 들어왔을 경우                    | `React.ComponentClass<any>`                                                     |
| 4. `type` 에 대한 테스트 :: `Fragment`가 들어왔을 경우                   | `React.ReactFragment`                                                           |
| 5. `props`에 대한 테스트 :: `key`와 `ref` 속성이 들어올 경우               | 별도로 `key`와 `ref` 속성으로 처리                                                        |
| 6. `props`에 대한 테스트 :: `children`이 들어온 경우                    | 별도로 재귀 수행                                                                       |
| 7. `props`에 대한 테스트 :: `className`이 들어온 경우                   | 별도로 처리                                                                          |
| 8. `props`에 대한 테스트 :: `<div>텍스트</div>`와 같은 형태에서 `텍스트` 부분 처리 | `children:'텍스트'`와 같이 처리                                                         |

<br/><br/><br/>

### 🧑‍💻 타입스크립트를 활용한 테스트 코드 작성에 대한 고민

위와 같이 생각을 하고 타입스크립트를 본격적으로 사용을 해보기로 했다.<br/>
타입스크립트의 핵심은 타입 추론이다. 추론이 가능하면, 타입을 명세할 필요가 없지만, 그게 아니라면 타입을 명세해줘야 한다.<br/><br/>

타입스크립트를 사용해서 코딩을 하려고 보니, `reactElement`의 타입이 명확하지 않았다.<br/><br/>

|  필수 속성  |
| :-----: |
| `type`  |
| `props` |
|  `key`  |
|  `ref`  |

그래서 다시 처음으로 돌아갔다. 너무 많은 것을 하려고 해서 그랬다는 생각이 들었기 때문이다.<br/><br/>

[참고 링크 : 리액트 공식 문서](https://ko.react.dev/reference/react/createElement)

| ![](리액트공식문서_createElement_정의.png) |
| :-------------------------------: |
|             리액트 공식 문서             |

필수 속성에 대한 타입을 명세해보자.<br/><br/>

|  필수 속성  | 타입                                                                     |
| :-----: | ---------------------------------------------------------------------- |
| `type`  | string \| Function \| generic<br/><br/>▸ `Fragment`는 지금 단계에서 고려하지 않는다. |
| `props` | object \| null                                                         |
|  `key`  | string \| number \| null                                               |
|  `ref`  | string \| number \| null                                               |
