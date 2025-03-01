---
layout: post
title: "React Advanced"
date: 2022-11-29
categories:
  - React
tags:
  - Javascript
  - React
  - Virtual DOM
  - Document Object Model
  - Diffing Algorithm
  - key
  - hooks
  - useMemo
  - useCallback
  - Custom Hooks
  - useFetch
  - static import
  - dynamic import
  - react.lazy
  - react.suspense
---

그러니까 공부하자 잡념은 꿈에서나 하고..

그래서 양들의 침묵은 멈추었는가??

거진 3일간 진행된 custom hook 관련 내용

cpu랑 ram 올리면서 조금 소홀했는데 작업이 끝나고, window11로 올라왔다. 제대로합시다

---

## 1. React advanced

### 1.1. Virtual DOM

#### 1.1.1. 배경

Real(↔Virtual) DOM의 lite copy같다.

- DOM: Document Object Model: 객체 문서보멜, 브라우저가 만든 트리구조의 객체모델 -> document.getElementById(id)
  - DOM은 UI상태가 변할때마다 업데이트가 된다. DOM으로 자주 조작 시 성능이 저하된다.
- 저하원인: 트리구조는 탐색하는건 빠르지만, 업데이트 시 reflow를 한다. 해당 요소와 자식에 의해 dom트리를 재구축하여 ui를 업데이트 해야된다. 레이아웃 및 페인트에 해당되는 재연산을 하게되어 성능이 저하된다. -> 해당요소만 다시 그릴순 없을까?

#### 1.1.2. 개념

DOM객체와 동일한 속성을 가졌지만 더 가벼운 복사본이다. UI요소를 메모리에 유지하고, ReactDOM과같은 라이브러리를 통해 DOM과 동기화한다.

React는 새로운 요소가 UI에 추가되면 가상의 DOM을 생성한다. 상태가 변경이되면 새로운 가상 DOM트리를 생성하고, 이전의 가상DOM과 이후의 가상DOM의 차이를 비교하여 실제 DOM에 반영한다. 최소한의 작업으로 렌더링을 하게되고 업데이트 비용을 줄인다.

#### 1.1.3. 형태

```json
// json으로 표현한것이다. json이 아니다.
html{
  head{
    lang:"en"
  },
  body{
    main{
      ul{
        class:"list",
        li{
          class:"item",
          CustomItem{
            ...
          }
        }
      }
    }
  }
}
```

JS객체로 표현이 가능하다.

```javascript
const virtualDom = {
  tagName:"html",
  children:[
    {tagName:"head"},
    {tagName:"body",
      children:[
        tagName:"ul",
        attributes:{"class":"list"},
        children:[
          {
            tagName:"li",
            attributes:{"class":"item"},
            ...
          }
        ]
      ]
    }
  ]
};
```

DOM과 같이 virtualDOM도 HTML document를 문서 객체 기반으로 한다. 일반 JS객체이고, DOM을 직접조작하지 않고 virtualDOM으로 제어할 수 있다.

### 1.2. React Diffing Algorithm

하나의 트리를 다른 트리로 변형시키는 알고리즘의 시간복잡도는 O(n^3)이다. 반복문을 3번돌린다면 ^3의 비싼 비용을 지불해야한다. 이에 Heuristic Algorithm을 구현했다.

- 가정
  1. 각기 서로 다른 두 요소는 다른 트리를 구축한다.
  2. 게발자가 제공하는 key property를 갖고, 여러번 렌더링을 거쳐도 변경되지 말아야되는 자식요소가 무엇인지 알수있다.

-> Diffing Algorithm이 React에서 사용된다.

기존트리와 새로운 트리를 비교 시 트리의 레벨을 기준으로하는 BFS의 일종으로 순회한다.

#### 1.2.1. 다른 타입의 DOM element일 경우

DOM트리의 tag규칙을 어기는 경우부모의 태그가 달라지면 React는 이전 트리를 버리고 새로운 트리를 생성한다.

#### 1.2.2. 같은 타입의 DOM element일 경우

최대한 렌더링을 하지 않는 방향으로 최소한의 변경사항만 업데이트한다.업데이트 할 내용이 있을 시 virtual dom 내부 property만 수정하여 모든 노드를 거친 뒤 업데이트가 끝나면 DOM 렌더링을 시도한다.

이전 element와 새 element를 비교하여 변화된 property만 수정하고, 한 DOM 노드를 처리하고, React는 해당 노드들의 밑의 자식들을 순차적으로 동시에 순회하며 차이가 발견될때마다 변경한다. 이 과정은 재귀로 처리된다.

#### 1.2.3. 자식 엘리먼트의 재귀적 처리

key값이 없는 경우 이전 노드와 새로운 노드가 동일함을 알 지못하고 새로 렌더링해버리기에 이전 노드와 새로운 노드를 비교할 시 key값으로 비교하며, 이에 key값이 없는 리스트이 아이템(ex: li, ui 등)은 warning을 throw한다.

#### 1.2.4. 키(key)

키는 전역적으로 유일할 필요는 없으나, 형제 element사이에서는 유일해야한다. 정 안될경우는 배열의 index를 key값으로 사용할 수 있다. 하지만 배열이 다르게 정렬되버리면 index는 그대로 유지되므로 배열 전체가 변경된 것으로 인지해 새로 그리게 되므로, key값은 우리가 지정하는 것이 더 낫다.

## 2. React Hooks

React 16.8부터 추가된 기능이다.

### 2.1. Component & Hooks

Hook은 함수 컴포넌트에서 사용하는 메서드로 함수 컴포넌트 이전에는 class 컴포넌트로 사용했다.

다른 객체 프로그램들과 같이 class중심의 로직은 this지옥과 고도화될 수록 다중 상속, 등 문법을 숙지하지 않으면 사용할 수 없는 등의 문제를 가졌고, 함수의 경우에는 상태값을 최적화할 수 있는 기능이 클래스 방식보다는 미흡했으나, Hook이 나오며 개선된다.

class를 작성하지 않고도 state와 다른 React의 기능을 사용할 수 있게 해주는 편리한 메서드이다. 함수로만 React를 동작시키기에 class에서는 동작하지 않는다.

#### 2.1.1. 사용규칙

1. 리액트 함수의 최상위에서만 호출해야된다. 중첩된 함수에서 실행 시 발견하기 힘든 오류가 발생한다.
2. 오직 리액트 함수 내에서만 사용되야한다. 일반 JS함수안에서도 호출하면 안된다.

> 상기 규칙 위반 시 compiler가 error throw

### 2.2. useMemo

컴포넌트는 상태가 변겨오디거나 부모 컴포넌트가 렌더링 될때마다 리렌더링되는 구조이다. 잦은 리렌더링은 앱성능에 좋지 않다.  
<span style="color:red"> -> 에러구문: Error:Too many re-renders React limits the number of renders to prevent an infinite loop</span>

#### 2.2.1. useMemo란?

특정값(value)를 재사용할때 사용하는 hook이다.

출처: [codestates](https://www.codestates.com)

```javascript
const Calculator = {value} => {
  const result = caculate(value);
  return <>
    <div>
      {result}
    </div>
  </>
}
```

상기 컴포넌트를 계속 리렌더링한다고 생각한다면, caculate함수의 실행시간만큼 연산시간이 밀린다. 이에 user는 성능에 문제가 있다고 판단하여 접속을 다시는 안할 수 있다.

```javascript
import { useMemo } from "react";

const Calculator = {value} => {
  const result = usememo(() => caculate(value), [value]);

  return <>
    <div>
      {result}
    </div>
  </>
}
```

value의 값이 바뀌지 않는다면, 리렌더링 될 시에 caculate연산을 리렌더링이 될 때마다 하지 않도록 useMemo hook을 사용하여 값을 어딘가에 저장했다 다시 꺼내어 쓰게 되어있다. 이전 구축된 렌더링과 새로 구축되는 렌더링을 비교해 value값이 동일할 시 이전 렌더링의 value값을 그대로 재활용할 수 있다. Memoization의 개념과 긴밀한 관계가 있다.

> Memoization은 알고리즘에서 자주 나오는 개념으로 기존에 수행한 연산의 결과값을 메모리에 저장해두고, 동일한 입력이 들어올 시 재활용하는 프로그래밍 기법이다. 잉여연산을 막아 앱의 성능을 최적화 할 수 있다.
> useMemo역시 불필요한 잉여연산을 하지 않도록 하여 React 앱의 성능을 최적화한다. 직접 구현해도 되지만, 이미 만들어져있는 기능을 사용하는 것도 생산성에서 굉장한 도움이 된다.

### 2.3. useCallback

출처: [codestates](https://www.codestates.com)

useMemo와 같이 memoization기법을 이용한 hook이다. useMemo는 값의 재사용을 위해 사용한다면, useCallback은 함수를 재사용한다.

```javascript
const Calculator = {x, y} => {
  const add = () => x + y;
  return <>
    {add()}
  </>;
}
```

상기 코드는 useMemo와 동일하게 add() 함수가 렌더링 될때마다 연산된다. 전달되는 값 변경이 없다고한다면, 함수도 메모리에 저장 후 다시 꺼내 쓸 수 있다.

useCallback hook을 사용하면 함수가 의존하는 값이 변경되지 않으면 함수를 계속해서 반환한다.

```javascript
import React, { useCallback } from "react";

const Calculator = {x, y}{
  const add = useCallback(() => x + y, [x, y]);
  return <>
    <div>
      {add()}
    </div>
  </>;
}
```

단순하게 컴포넌트 내에서 함수를 반복해서 생성하지 않기위해 useCallback을 사용하는건 큰 의미가 없거나 더 손해인 경우도 있다. 자식 컴포넌트의 props로 함수를 전달할 때 useCallback은 진가를 보인다.

#### 2.3.1. useCallback과 참조 동등성

참조 동등성에 의존하는 useCallback은 js문법을 따라간다. js에서 function, 함수는 객체이다. 객체는 메모리에 저장할 때 값이 아닌 값의 주소를 저장한다. 반환하는 값이 같아도 일치연산자로 비교하면 false가 나온다.

React는 리렌더링시 함수를 새로 생성하여 호출한다. 호출된 함수는 기존 함수와 같은 함수가 아니다. 그러나 useCallback을 이용해 함수 자체를 저장해서 사용하면 함수의 메모리 주소값을 저장한 후 다시 사용한다. React 컴포넌트 내에서 다른 함수의 인자로 넘기거나 자식 컴포넌트의 prop으로 넘길 때 예상하지 못하는 에러를 막을 수 있다.

### 2.4. Custom Hooks

개발자가 스스로 만든 훅을 의미한다. 주로 여러 url을 fetch할때 여러 input으로 인한 상태변경 등 반복되는 로직을 동일한 함수에서 작동하게 하고 싶을 때 사용한다.

1. 상태관리 로직의 재활용이 가능하다.
2. 클래스 컴포넌트보다 적은 양의 코드로 동일한 로직을 구현할 수 있다.
3. 함수형으로 작성하기에 보다 명료하다.

출처: [React custom hook](https://ko.reactjs.org/docs/hooks-custom.html)

```javascript
//FriendStatus : 친구가 online인지 offline인지 return하는 컴포넌트
function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}

//FriendListItem : 친구가 online일 때 초록색으로 표시하는 컴포넌트
function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? "green" : "black" }}>{props.friend.name}</li>
  );
}
```

- 커스텀 훅을 정의할 시 use를 붙이는 것이 규칙이다.
- 대개의 경우 hooks 폴더내 커스텀 훅을 위치한다.
- 커스텀 훅으로 만들 시 함수는 조건부 함수가 아니어야 한다.(반환 시 조건는 안된다.)

```javascript
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}

function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? "green" : "black" }}>{props.friend.name}</li>
  );
}
```

같은 상태를 공유하는것이 아니라 로직만 공유한다. 상태는 컴포넌트 내에 독립적으로 정의된다.

#### 2.4.1. custom hooks useFetch예시

출처: [codestates](https://www.codestates.com)

##### 2.4.1.1. 여러 url을 fetch할 때 사용할 수 있는 useFetch hook

```javascript
const useFetch = (initialUrl: string) => {
  const [url, setUrl] = useState(initialUrl);
  const [value, setValue] = useState("");

  const fetchData = () => axios.get(url).then(({ data }) => setValue(data));

  useEffect(() => {
    fetchData();
  }, [url]);

  return [value];
};

export default useFetch;
```

##### 2.4.1.2. 여러 input에 의해서 상태 변경을 할 수 있는 useInputs hooks

```javascript
import { useState, useCallback } from "react";

function useInputs(initialForm) {
  const [form, setForm] = useState(initialForm);
  // change
  const onChange = useCallback((e) => {
    const { name, value } = e.target;
    setForm((form) => ({ ...form, [name]: value }));
  }, []);
  const reset = useCallback(() => setForm(initialForm), [initialForm]);
  return [form, onChange, reset];
}

export default useInputs;
```

## 3. React first function

리액트 버전 18로 올라오면서 콘솔경고창에 경고가 보이는것이 추가되었다. 리액트는 메타에의해 지속적으로 업데이트 및 관리되고 있는 라이브러리이다. 하여 현 시점에서 legacy한 코드보다는 최신 기술스택을 알아야 사용할 수 있다(우리가 죽을때까지 공부하는 이유).

### 3.1. Code spliting

리액트앱은 webpack, rollup과 같은 툴을 사용해 bundling한다. 번들링을 하면 html 웹페이지에 js를 쉽게 추가할 수 있다. 번들링 된 앱은 js파일을 한곳에 있고, 과거에는 이런방식이 편리하였다. 하지만 모던 웹으로 넘어오며 코드가 방대해지고, 유지관리의 비용이 늘어남에 따라 성능이슈가 발생되었고, dom을 다루는 정도가 정교해지고 무거워져, 코드를 디코딩하고 컴파일하는 정도가 느려진지 파악해 번들을 나누어 당장 필요한 코드만 불러오고 나중에 필요한 코드는 필요할 시 만 불러오는 방식을 고안하게 된다.

물리적으로 번들을 나누어 런타임시 여러 번들을 동적으로 만들고 불러오는 것으로, 현재 webpack과 rollup과 같은 번들러가 지원하는 기능이다. 이걸 통해 원래 고안의 아이디어를 따라가 성능이슈를 개선하였다.

#### 3.1.1. 분할, 줄이는 방법

```javascript
// 모든걸 가져옴
import example from 'example';

...

example.show();
```

npm을 통한 써드파티 라이브러리도 번들링 시 포함이 된다. 번들링 시 모든 기능을 불러오면 잉여자원이 늘어나 앱의 크기가 커지게 된다.

```javascript
// 필요한걸 가져옴
import show from 'example/show';

find();
```

#### 3.1.2. React code spliting

single page application인 react는 한번에 불러오기 때문에 첫 화면이 렌더링 될때까지 시간이 걸린다. 하여 성능을 위해 코드 분할을 적용했다.

static import: dynamic import를 통해 사용하는 것으로 코드 파일의 가장 최상위에서 import 지시자를 사용해 사용하고자하는 라이브러리 및 파일을 불러오는 방법을 사용했다.

##### 3.1.2.1. Static Import

```javascript
//최상위
import example from "example";

//코드 진행
...

example.show().now();

//코드 진행
...

//코드 끝
```

##### 3.1.2.2. Dynamic Import

```javascript
//코드 진행
...

import(example.show)
  .now();
//dynamic import hello world 출력

...

//코드 끝
...
```

dynamic import를 사용해 필요한것만 불러온다. 가져온 코드에대한 모든 호출은 해당 함수 내부에 있어야한다. 번들링 시 분할된 코드(chunk)를 지연 로딩시키거나 요청 시 로딩할 수 있다.

React.lazy와 함께 사용한다.

##### 3.1.2.3. React.lazy()

dynamic import를 사용해 컴포넌트를 렌더링할 수 있다. single page application이므로 한번에 사용하지 않는 컴포넌트까지 불러오는 단점을 동적으로 import할 수 있어 초기 렌더링 지연시간을 어느정도 줄일 수 있다.

해당 함수로 씌워진 컴포넌트는 단독으로는 못쓰고, React.suspense컴포넌트의 하위에 렌더링해야된다.

```javascript
import Component from './Component';

const Component = React.lazy(()=>import('./Component));
```

##### 3.1.2.4. React.Suspense

Router 분기가 나누어진 컴포넌트를 lazy를 통해 import할 시 해당 path로 이동할때 컴포넌트를 불러오는데 이 과정에서 로딩하는 시간이 생긴다. Suspense는 아직 렌더링이 준비되지 않은 컴포넌트가 있을 시 로딩화면을 보여주고, 로딩이 완료 시 렌더링이 준비된 컴포넌트를 보여주는 기능이다.

Router를 Suspense적용하는것이 간단하다. 라우터가 분기되는 컴포넌트에서 각 컴포넌트에 React.lazy를 사용해 import한다. Route 컴포넌트들을 Suspense로 감싼 후 로딩 화면으로 사용할 컴포넌트를 fallback속성으로 설정하면 된다. 서비스에따라 모든 곳에서 로딩이 불러와져야하는지 설계에서부터 결정해야된다.

```javascript
import {Suspense} from 'react';

const AComponent = React.lazy(()=> import('./AComponent'));
const BComponent = React.lazy(()=> import('./BComponent'));

const SomeComponent = () => {
  return(
    <div>
      <Suspense fallback={<div>불러오는중</div>}>
        <AComponent/>
        <BComponent/>
      </Suspense>
    </div>
  )
}

//with Router
import { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

const Main = lazy(() => import('./routes/Main'));
const Post = lazy(() => import('./routes/Post'));

const App = () => {
  <Router>
    <Suspense fallback={<div>불러오는중</div>}>
      <Routes>
        <Route path="/" element={<Main/>}>
        <Route path="/post" element={<Post/>}>
      </Routes>
    </Suspense>
  </Router>
}
```

---

상당히 미루고미루다 결국한다. 지은님 고마워요

---

## 참조

> [Chulgil Lee: Big-O 쉽게 이해하기](https://blog.chulgil.me/algorithm/)  
> [reactjs: custom hooks](https://ko.reactjs.org/docs/hooks-custom.html)
