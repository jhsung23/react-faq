# React FAQ

React를 기반으로 개발하며 **한 번이라도 떠올렸었던 질문들**에 대한 답을 정리한 레포지토리입니다.
길고 상세하게 보다는 핵심 위주로 짧고 간단하게 작성했고.... 그저 공유 목적으로 올렸으니.... 누군가에게 도움이 되길....바라요 🍀

(오류가 있는 경우 언제든 Issue를 통해 리포트 부탁드립니다. 미리 감사합니닷)

## Index

1. [원리](#원리)
   - [render](#render)
   - [재조정 (reconciliation)](<#재조정-(reconciliation)>)
   - [가상 DOM (virtual DOM)](<#가상-DOM-(virtual-DOM)>)
2. [주요 개념](#주요-개념)
   - [JSX](#JSX)
3. [Hook](#Hook)
   - [useState](#useState)
   - [useEffect](#useEffect)
   - [useReducer](#useReducer)
   - [useMemo](#useMemo)
   - [useCallback](#useCallback)
   - [useRef](#useRef)
   - [useContext](#useContext)
   - [useLayoutEffect](#useLayoutEffect)
4. [React 18](#React-18)
   - [자동 배치 (Automatic Batching)](<#자동-배치-(Automatic-Batching)>)
   - [동시성 (Concurrent Feature)](<#동시성-(Concurrent-Feature)>)
   - [useTransition](#useTransition)
   - [useDeferredValue](#useDeferredValue)

## 원리

### render

[목차로 이동하기](#Index)

#### ⚪️ 리액트에서 “rendering”의 의미

- 컴포넌트를 호출하는 것
- initial render에서는 root 컴포넌트를 호출하고, 이후 render에서는 상태가 업데이트된 함수 컴포넌트를 호출함
- 위 과정은 재귀적으로 이루어짐
  - 업데이트된 컴포넌트가 다른 컴포넌트를 리턴한다면, 리액트는 다음으로 그 컴포넌트를 render하는 식으로 진행됨
  - 위 과정은 더 이상 중첩된 컴포넌트가 없어 리액트가 화면에 무엇을 표시할지 정확히 알아낼 때까지 계속됨

#### ⚪️ 리액트는 언제, 왜 컴포넌트를 render하는가?

- app을 시작해서 첫 rendering을 해야 하는 경우, 컴포넌트의 상태가 업데이트된 경우 rendering됨

#### ⚪️ 리액트의 Rendering 중에는 무슨 일이 일어나는가?

- 이전 render와 비교했을 때 변경이 있는지 계산함
- 다음 단계(commit) 전까지는 계산한 정보로 아무런 작업도 수행하지 않음 (그저 계산만 할 뿐)

#### ⚪️ “Rendering은 순수 계산이어야 한다”는 것의 의미

- 동일한 입력값에는 동일한 출력값(jsx)이 나와야 함
- Rendering 전에 존재했던 객체나 변수들을 변경해서는 안됨

#### ⚪️ 왜 rendering은 항상 DOM 업데이트로 이어지지 않을 수 있는지?

- 리액트는 rendering 간 차이가 있는 경우에만 DOM을 변경하기 때문
- 부모의 props가 변경됐더라도, 자식 컴포넌트가 변경된 props를 사용하지 않는다면 업데이트되지 않음

#### ⚪️ 리액트 app의 화면 업데이트 단계

- 3단계로 이루어짐
  1. trigger: rendering을 유발하는 것 (첫 렌더링 - createRoot().render(), 상태 업데이트)
  2. rendering: 이전 렌더링과 비교했을 때 변경 사항을 계산하는 것 (concurrent하게 실행됨)
  3. commit: 계산 결과 변경 사항이 존재한다면 DOM을 업데이트하는 것 (synchronous하게 실행됨)

#### ⚪️ 리액트의 rendering vs 브라우저의 rendering

- 리액트의 rendering: 변경 사항을 계산하는 것
- 브라우저의 rendering: 브라우저가 변경된 요소를 화면에 그리는 작업

### 재조정 (reconciliation)

[목차로 이동하기](#Index)

#### ⚪️ 재조정이란?

- virtual DOM을 바탕으로 UI의 이상적인 표현을 메모리에 저장하고 변경된 부분을 실제 DOM과 동기화하는 과정

#### ⚪️ 스택 재조정

- 재조정 작업을 하나의 큰 태스크라는 단위로 동기적 실행하는 것
- 현재 상태의 트리와 작업 중인 트리를 DFS(재귀적) 탐색하며, 이 작업은 일시 중지되거나 취소될 수 없음
- 따라서 이 콜 스택이 전부 처리되기 전까지는 다른 작업을 할 수 없고 앱은 일시적으로 무반응 상태가 됨(또는 버벅거림)

#### ⚪️ diffing 알고리즘

- 재조정 과정에서 virtual DOM을 비교하여 변경 사항을 찾아내는 데에 사용되는 알고리즘

#### ⚪️ diffing 알고리즘의 전제 조건과 전제 조건이 필요한 이유

- 전제 조건
  - 서로 다른 `type`의 두 element는 서로 다른 트리를 만든다.
  - 개발자가 `key` prop을 통해 여러 렌더링 사이에서 어떤 자식 element가 변경되지 않아야 하는지 표시해줄 수 있다.
- 이유
  - 하나의 트리를 가지고 다른 트리로 변환하기 위한 최소한의 연산 수를 구하는 알고리즘은 O(n^3)의 시간 복잡도를 가지는 매우 비싼 연산임
  - 따라서 시간복잡도를 줄이고자 위 2가지 조건을 전제로 하는 휴리스틱 알고리즘을 구현하게 됨

#### ⚪️ 파이버(fiber)란?

- 컴포넌트 트리에 대한 추가 정보를 포함하고 있는 내부 객체이자 **작업의 단위**
- Virtual DOM 구현의 일부

#### ⚪️ 파이버가 등장한 이유

- 기존의 재조정 과정은 모든 작업을 동기적으로 하나의 큰 태스크로 실행하기 때문에 빠르고 반응적인 업데이트가 필요한 UI 업데이트에 충분히 대응하지 못했음
- 이 문제점을 개선하고자 파이버라는 단위로 작업을 쪼개어 작업의 우선순위에 따라 concurrent하게 UI 업데이트를 할 수 있게 함
- 즉, 리액트가 작업의 우선 순위를 따져 스케줄링을 활용할 수 있도록 하기 위함

#### ⚪️ 파이버 재조정

- Render 작업을 제어 가능한 파이버 단위로 쪼개어 작업을 concurrent하게 실행하는 재조정 방법
- 스택 재조정의 취약점을 보완하기 위해 react16에 도입됨

#### ⚪️ key prop을 사용하는 이유

- 재조정 과정에서 key prop을 비교하여 어떤 요소에 변화가 있는지 빠르게 알아낼 수 있음

#### ⚪️ 참고 문서들

- [[React 공식 문서] 코드 구조 개요 - 스택 재조정자, 파이버 재조정자 설명](https://ko.legacy.reactjs.org/docs/codebase-overview.html#stack-reconciler)
- [[콴다 기술 블로그] React Deep Dive - Fiber](https://blog.mathpresso.com/react-deep-dive-fiber-88860f6edbd0)
- [[acdlite] React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)
- [[React legacy 공식 문서] 재조정](https://ko.legacy.reactjs.org/docs/reconciliation.html)
- [[Naver D2] React 파이버 아키텍처 분석](https://d2.naver.com/helloworld/2690975)

### 가상 DOM (virtual DOM)

[목차로 이동하기](#Index)

#### ⚪️ Virtual DOM이란?

- 실제 DOM의 복사본으로, DOM의 속성을 가지고 있는 자바스크립트 객체
- 다음 문제를 해결하기 위한 프로그래밍 개념
  - DOM 조작에 의한 브라우저 렌더링 비용(비효율) 문제
  - SPA 특징으로 인한 DOM 조작 및 복잡도 증가에 따른 최적화, 유지보수 문제

#### ⚪️ Virtual DOM의 등장 배경

- SPA는 잦은 DOM 조작이 발생하며 DOM이 조작되면 이를 업데이트하기 위해 브라우저의 렌더링 과정을 거치게 됨
- 그런데 브라우저의 렌더링 과정 중 레이아웃을 재연산하는 과정(reflow)과 화면에 그려내는 과정(repaint)은 무거운 연산이 될 수 있음
- 이러한 무거운 연산의 반복을 줄이고자 등장한 개념

#### ⚪️ 동작 방식

- 변경 이전의 내용을 담은 virtual DOM과 변경 이후의 내용을 담은 virtual DOM을 diffing 알고리즘으로 비교함
  - 위 연산은 자바스크립트 연산이므로 브라우저 렌더링 과정에 영향을 미치지 않아 렌더링 비용이 발생하지 않음

#### ⚪️ Virtual DOM을 사용했을 때의 장점 (왜 virtual DOM을 사용하는가)

- 메모리 상에서 변경 사항이 있는 부분을 찾아 한 번에 업데이트하기 때문에 DOM 조작 횟수를 줄여 브라우저 렌더링 비용을 줄일 수 있는 장점이 있음

#### ⚪️ Virtual DOM은 항상 효율적일까?

- 잦은 DOM 조작이 발생한다면 효율적
- 그렇지 않다면 직접 DOM을 조작하는 것이 효율적일 수 있음
  - Virtual DOM을 사용하면 각 요소를 모두 비교해야 하는 오버헤드가 있기 때문

## 주요 개념

### JSX

[목차로 이동하기](#Index)

#### ⚪️ JSX란?

- 자바스크립트를 확장한 문법 (그러나 ECMAScript 표준의 일부는 아님)
  - 자바스크립트의 표준이 아니기 때문에 바로 실행하면 실행되지 않고 에러가 발생함
- 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데에 도움을 주는 새로운 문법

#### ⚪️ JSX를 사용하는 이유 (역할)

- 다양한 트랜스파일러에서 다양한 속성을 가진 트리 구조를 토큰화해 ECMAScript로 변환하기 위한 목적

#### ⚪️ JSX 어트리뷰트에 카멜케이스(camelCase) 네이밍을 사용하는 이유

- JSX는 HTML보다 자바스크립트에 가깝기 때문에 React DOM은 HTML 어트리뷰트 이름 대신 카멜케이스 프로퍼티 네이밍 규칙을 사용함
- class → className, tabindex → tabIndex

#### ⚪️ JSX → Javascript

- JSX는 자바스크립트 표준이 아니기 때문에 트랜스파일러를 거쳐야 됨 (feat. babel)

#### ⚪️ 리액트를 사용할 때 JSX는 필수인가?

- 아님
- JSX는 React.createElement를 호출하기 위한 문법적 설탕임
- 즉, JSX로 할 수 있는 모든 것은 순수 자바스크립트로도 가능함

## Hook

### useState

[목차로 이동하기](#Index)

#### ⚪️ useState의 정의

- 함수형 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리하게 해 주는 훅

#### ⚪️ useState를 const로 선언할 수 있는 이유

- 상태가 변경된다는 것은 state 변수를 직접 수정하는 것이 아니라, 컴포넌트 함수가 재실행되는 것이기 때문.
- [[참고] React Hook의 동작 원리](https://hewonjeong.github.io/deep-dive-how-do-react-hooks-really-work-ko/)

#### ⚪️ setState로 업데이트해야 하는 이유 (state를 직접 수정하면 안되는 이유)

- state는 컴포넌트를 렌더링하는 데 사용되는 데이터이기 때문에, 업데이트된 값으로 화면을 렌더링하기 위해 렌더링을 트리거하는 setState로 업데이트해야 함
- 직접 변경할 경우 렌더링이 일어나지 않으며 상태 변경을 추적하기 어려워짐

#### ⚪️ lazy initialization이란?

- useState를 호출할 때 변수 대신 함수를 인수로 넘기는 것
- 게으른 초기화 함수는 state가 처음 만들어질 때만 사용되기 때문에 이후 리렌더링이 발생하면 이 함수의 실행은 무시됨
- 따라서 초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하기 적합

#### ⚪️ 동일한 값으로 setState를 호출하는 경우 리렌더링이 발생할까?

- 리액트는 `Object.is`를 호출하여 현재 상태와 새로운 상태를 비교한 후, 동일하다면 렌더링을 건너뛰도록 최적화되어 있기 때문에 리렌더링이 발생하지 않음
- [[참고] state는 Object.is로 비교 후 렌더링 여부를 결정한다](https://hewonjeong.github.io/deep-dive-how-do-react-hooks-really-work-ko/)

#### ⚪️ 하나의 핸들러에서 setState를 여러 번 호출한 경우, 호출한 횟수만큼 리렌더링이 발생할까?

- 상태 업데이트는 하나의 핸들러 실행이 끝난 후 배치 처리 되므로 리렌더링은 한 번만 발생함
- 단, 비동기 함수에서 상태 업데이트는 각각 리렌더링을 일으킴. 그러나 **18버전**에서는 자동 배치가 도입되어 비동기 함수에서도 배치 처리될 수 있음
- [[참고] React18에 업데이트된 자동 배치](https://github.com/reactwg/react-18/discussions/21)
- [[참고] queueing a series of state updates](https://react.dev/learn/queueing-a-series-of-state-updates)

#### ⚪️ 상태를 read-only로 다뤄야 하는 이유

- 상태를 직접 변경하면 리렌더링이 발생하지 않기 때문에 상태 자체는 읽기 전용으로 다루고, 변경하려면 setState 함수를 호출해야 함

#### ⚪️ 변수와 상태의 차이

- 변수: 렌더 간 유지되지 않고 리렌더링을 일으키지 않음
- 상태: 렌더 간 유지되며 리렌더링을 일으킴

#### ⚪️ 의존성 배열에 setState를 넣은 경우 리렌더링될까?

- setState는 리렌더링에 관계없이 참조가 동일한 함수이기 때문에 의존성 배열에 넣지 않아도 안전함
- [[참고] React guarantees that setState function identity is stable and won’t change on rerenders](https://react.dev/learn/queueing-a-series-of-state-updates)

#### ⚪️ 배열 또는 객체인 상태를 직접 수정하면 안되는 이유 (불변성을 지켜야하는 이유)

- 렌더링 간 state가 어떻게 변경됐는지 명확하게 파악하기 어려워 디버깅과 최적화에 이슈가 생길 수 있음

#### ⚪️ 객체인 상태를 업데이트하는 방법

- 객체의 사본을 만들어 사본을 수정해야 함. 또는 immer 라이브러리를 사용

### useEffect

[목차로 이동하기](#Index)

#### ⚪️ useEffect의 정의

- 리액트 컴포넌트를 외부와 동기화하기 위한 부수 효과를 다루는 훅

#### ⚪️ Effect란 무엇이고 Event handler와는 무엇이 다른가?

- Event handler는 특정 상호작용에 대한 응답으로 실행되는 것
- Effect는 동기화가 필요할 때 실행되는 것
- Effect는 Event handler와 다르게 특정 상호작용에 의해 실행되는 것이 아님

#### ⚪️ 클린업 함수의 기능과 역할

- 컴포넌트가 리렌더링됐을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는 함수
  - 말 그대로 이전 상태를 청소해 주는 개념
- 등록했던 이벤트 핸들러를 삭제하는 역할을 많이 함

#### ⚪️ 의존성 배열 작성 케이스

- 작성하지 않은 경우: 매 렌더링 직후마다 실행됨
- 빈 배열인 경우: 최초 렌더링 직후에 실행된 다음부터는 더 이상 실행되지 않음
- 배열에 값이 채워진 경우: 의존성이 변경된 경우 실행됨

#### ⚪️ 리액트는 의존성 배열이 변경됐다는 것을 어떻게 알까?

- `Object.is`를 기반으로 얕은 비교를 수행하고, 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 부수 효과 콜백을 실행함
- `**useEffect(()⇒console.log(’log’)}`와 `console.log(’log’)`의 차이 (useEffect vs 직접 실행)\*\*
  - useEffect는 클라이언트 사이드에서만 실행됨을 보장하므로 window 객체에 접근하는 코드를 작성할 수 있지만, 직접 실행한다면 서버에서도 실행되기 때문에 window에 접근할 수 없음
  - useEffect는 컴포넌트의 렌더링이 완료된 이후에 실행되지만, 직접 실행은 컴포넌트가 렌더링되는 도중에 실행되어 성능에 영향을 미침

#### ⚪️ useEffect의 경쟁 상태

- useEffect 내부에서 호출한 요청이 서로 경쟁하여 예상과 다른 순서로 응답이 도착하는 경우
- useEffect로 데이터를 페칭할 때에는 오래된 응답을 무시하도록 클린업 함수를 구현해야 함

#### ⚪️ useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?

- useEffect에서 비동기로 함수를 호출할 경우 경쟁 상태가 발생할 수 있음
- useEffect의 인수로 비동기 함수가 사용 가능하다면 비동기 함수의 응답 속도에 따라 결과가 이상하게 나타날 수 있음
- 또한 cleanup 함수의 실행 순서를 보장할 수 없게 됨

#### ⚪️ 그렇다면 비동기 함수를 실행할 수 없는 것인가?

- useEffect의 인수로 비동기 함수를 지정할 수 없는 것이지, 비동기 함수 실행 자체가 문제가 되는 것은 아님
- useEffect 내부에서 비동기 함수를 선언해 실행하거나, 즉시 실행 비동기 함수를 만들어 사용할 수 있음

### useReducer

[목차로 이동하기](#Index)

#### ⚪️ useReducer의 정의

- 많은 이벤트 핸들러에 분산된 상태 업데이트 로직을 통합하기 위한 훅
- 컴포넌트의 상태 업데이트 로직을 컴포넌트에서 분리시킬 수 있음

#### ⚪️ reducer의 의미

- 상태 업데이트 로직을 갖고 있는 순수 함수
- 현재 상태와 액션 객체를 파라미터로 받고 새로운 상태를 반환함
- 상태 로직의 복잡성과 양을 줄이기 위해 사용됨 (상태 업데이트 로직을 컴포넌트 바깥에 작성할 수 있음)

#### ⚪️ reducer가 순수 함수여야 하는 이유

- reducer는 렌더링 중에 실행되기 때문
- 감속기는 순수해야 한다. 상태 업데이터 기능과 마찬가지로, 감속기는 렌더링 중에 실행됩니다! (액션은 다음 렌더링까지 대기열에 있습니다.) 이것은 감속기가 순수해야 한다는 것을 의미합니다. 동일한 입력은 항상 동일한 출력을 초래합니다. 그들은 요청을 보내거나, 시간 초과를 예약하거나, 부작용(구성 요소 외부에 영향을 미치는 작업)을 수행해서는 안 됩니다. 그들은 돌연변이 없이 객체와 배열을 업데이트해야 한다.

#### ⚪️ reducer 내부 action의 의미

- 사용자 인터렉션 단위
- 이 작업의 종류에 따라 reducer에서 새로운 상태를 반환하게 됨

#### ⚪️ useState와 비교했을 때 차이점

- 코드양: useState는 작성해야 하는 코드의 양이 적지만, useReducer는 reducer 함수와 dispatch 액션들을 모두 작성해야 함. 그렇지만, 많은 이벤트 핸들러에서 비슷한 방식으로 상태를 업데이트할 경우 useReducer를 사용하면 코드를 줄일 수 있음
- 가독성: useState는 아주 읽기 쉽고 상태 업데이트가 간단하지만, 복잡해질 수록 코드를 읽기 어려워짐. useReducer를 사용하면 이벤트 핸들러에서 상태 업데이트 로직을 분리해낼 수 있다는 장점
- 디버깅: useState를 사용할 때 버그가 발생한 경우, 상태가 어느 곳에서 잘못 업데이트됐는지 알아채기 어려움. 그러나 useReducer를 사용하면 reducer에 console log를 추가하여 어디서 잘못 업데이트됐는지 확인하기가 쉬움. 단, 작성해야 할 코드가 많음
- 테스팅: reducer는 순수 함수기 때문에 컴포넌트에 의존하지 않아 독립적으로 테스트가 가능함
- 개인 취향: 누구는 reducer를 선호하지만, 누구는 그렇지 않음 (선호의 문제)

#### ⚪️ useState 대신 useReducer를 사용하면 좋은 상황

- 잘못된 상태 업데이트로 인해 버그가 자주 발생하여 구조화된 코드 작성이 필요한 경우 도입하기 좋음

#### ⚪️ initialState에 함수 실행 결과를 전달하는 것의 문제점

- 초기값을 반환하는 함수는 오직 첫 번째 렌더를 위해 실행되지만, 실제로 매 렌더링마다 호출되기 때문에 비효율적
- 이를 최적화하기 위해서는 세 번째 인수로 함수 자체를 넘기면 됨
  - `useReducer(reducer, 이니셜_함수에_전달할_인수, 이니셜_함수)`
  - 이니셜 함수가 아무런 인수를 받지 않는 경우, useReducer의 두 번째 인수에 null을 전달
- [[참고] Avoiding recreating the initial state](https://react.dev/reference/react/useReducer#avoiding-recreating-the-initial-state)

#### ⚪️ 동일한 상태로 변경하는 경우 리렌더링이 발생할까?

- 현재 상태와 다음 상태를 Object.is로 비교하고, 동일하다면 업데이트를 건너뜀

#### ⚪️ dispatch 함수의 역할

- 상태를 업데이트하고 리렌더를 트리거하는 함수

### useMemo

[목차로 이동하기](#Index)

#### ⚪️ useMemo의 정의

- 리렌더 간 계산 결과를 캐싱할 수 있는 훅

#### ⚪️ useMemo의 용도

- 비싼 연산을 스킵하고자 할 때
- 컴포넌트의 리렌더링을 스킵하고자 할 때
- 다른 훅의 의존성을 메모하고자 할 때
- 함수를 메모하고자 할 때

#### ⚪️ useMemo의 calculateValue(첫 번째 인자)는 언제 실행될까?

- Initial render 때 호출하고, 다음 render에서는 의존성 배열에 변경이 없다면 같은 값을 리턴함

#### ⚪️ 의존성 배열이 변경됐다는 것을 어떻게 알까?

- 각각의 의존성을 이전 값과 Object.is() 함수로 비교함

### useCallback

[목차로 이동하기](#Index)

#### ⚪️ useCallback의 정의

- 리렌더 간 함수 정의를 캐싱할 수 있는 훅

#### ⚪️ useCallback의 용도

- 컴포넌트의 리렌더링을 스킵하고자 할 때
- 지나치게 자주 발생하는 effect를 차단하기 위해
  - 의존성 배열에 함수가 들어간 경우, 리렌더링이 발생할 때마다 함수의 참조가 새롭게 생성되어 의존성 배열에 변경을 발생시켜 effect가 자주 발생하게 되는 상황이 있음
- 커스텀 훅을 최적화하기 위해

#### ⚪️ 함수를 캐싱하는 이유

- 리렌더 간 함수의 참조는 달라지기 때문
- 렌더 단계에서 업데이트가 필요한 컴포넌트를 호출하면 그 내부에 있는 객체는 새로 생성되기 때문에 내용은 동일한 객체더라도 참조값이 달라져 해당 객체를 참조하는 컴포넌트에 불필요한 렌더링이 발생할 수 있음

#### ⚪️ 의존성 배열이 변경됐다는 것을 어떻게 알까?

- 각각의 의존성을 이전 값과 Object.is() 함수로 비교함

#### ⚪️ useCallback과 useMemo의 관계

- 두 훅은 무언가를 메모(캐싱)할 수 있게 해주며 모두 자식 컴포넌트를 최적화하고자 할 때 유용함
- 차이점
  - useMemo는 함수를 호출한 결과를 캐싱함 (리액트가 rendering 동안에 결과 계산을 위해 함수를 호출함)
  - useCallback은 함수 자체를 캐싱함 (리액트가 함수를 호출하지 않음)

### useRef

[목차로 이동하기](#Index)

#### ⚪️ useRef의 정의

- Rendering에 필요하지 않은 값을 참조할 수 있는 훅

#### ⚪️ useRef가 반환하는 것

- current 프로퍼티가 있는 단일 객체

#### ⚪️ useRef의 용도

- 컴포넌트의 시각적인 아웃풋에 영향을 주지 않는 값을 저장하기에 적합 (ex. interval ID)

#### ⚪️ state와 ref 객체의 차이점

- state를 변경하는 것은 re-render를 트리거하지만, ref는 그렇지 않음
- state를 변경하기 위해서는 setState 함수를 호출해야 하지만, ref는 current 프로퍼티를 직접 변경하면 됨

#### ⚪️ 변수와 ref 객체의 차이점

- 변수는 render마다 새롭게 생성되지만, ref는 render 간 유지됨

#### ⚪️ useRef는 매 render 마다 실행되는가?

- 실행은 되지만, 이전 render에서 얻은 리턴값과 동일한 객체를 반환함

#### ⚪️ ref.current를 rendering 중에 읽거나 쓰면 안 되는 이유

- 컴포넌트는 순수 함수여야 하기 때문
  - input이 같으면 항상 동일한 jsx를 리턴해야 하고, 다른 순서로 호출해도 호출 결과에 영향이 있으면 안 됨

#### ⚪️ 그럼 ref는 어디서 읽고 쓸 수 있는가

- 이벤트 핸들러나 effect 내부에서 사용할 수 있음

#### ⚪️ useRef(getRef())이 비효율적인 이유 (함수를 호출하여 초깃값을 셋팅하는 것)

- getRef는 매 렌더링마다 호출되는데, 함수를 호출한 값은 첫 render에만 사용되므로 불필요한 함수 호출임
- null 체크문을 작성하여 최적화할 수 있음
  - 렌더링 중에 ref.current를 변경하는 것은 허용되지 않으나, 이러한 경우 실행 결과는 항상 동일하고 충분히 예측 가능하기 때문에 괜찮음

### useContext

[목차로 이동하기](#Index)

#### ⚪️ useContext의 정의

- 컴포넌트에서 context를 읽고 구독할 수 있도록 해 주는 훅

#### ⚪️ useContext가 반환하는 것

- context value를 반환함
- 반환된 context는 항상 최신 값임 (up-to-date)

#### ⚪️ useContext가 참조하는 context

- 현재 레벨에서 부모 방향으로 useContext를 호출할 때 넣어준 context의 가장 가까운 Provider를 탐색함

### useLayoutEffect

[목차로 이동하기](#Index)

#### ⚪️ useLayoutEffect의 정의

- 브라우저의 리페인트 전에 실행되는 useEffect

#### ⚪️ useLayoutEffect의 용도

- 브라우저가 리페인트를 수행하기 전에 레이아웃을 측정하기 위함
- 대부분의 컴포넌트가 자신이 스크린에서 보여지는 위치나 크기를 알 필요가 없지만, 가끔 필요한 경우가 있음
- ex) 툴팁의 높이 파악

#### ⚪️ setup 함수와 cleanup 함수가 실행되는 타이밍

- 컴포넌트가 DOM에 추가되기 전에 실행됨
- 매 render가 실행된 다음, setup 함수의 cleanup 함수가 먼저 실행(old value 사용)되고 그 다음 setup 함수가 실행(new value 사용)됨
- 컴포넌트가 DOM에서 제거되기 전 cleanup 함수가 실행됨

#### ⚪️ useLayoutEffect의 성능 이슈

- useLayoutEffect 내부의 코드는 브라우저의 리페인팅을 블로킹함
- 따라서 과도하게 사용할 경우 app을 느리게 하는 원인이 되기 때문에 가능하다면 useEffect를 사용하는 것이 좋음

## React 18

### 자동 배치

[목차로 이동하기](#Index)

#### ⚪️ Before 18

- 하나의 핸들러 내부의 여러 개의 상태 업데이트를 하나의 리렌더링으로 처리했음
- 그러나 비동기 함수의 경우 여러 개의 상태 업데이트 각각을 리렌더링으로 처리함

#### ⚪️ After 18

- 비동기 함수의 여러 개의 상태 업데이트도 하나의 리렌더링으로 처리함 (배치)

### 동시성 (Concurrent Feature)

[목차로 이동하기](#Index)

#### ⚪️ 동시성 개념

- 동시에 여러 버전의 UI를 리액트가 준비할 수 있도록 하는 메커니즘

#### ⚪️ Suspense란

- 데이터가 준비되기 전 사용자에게 보여주고 싶은 컴포넌트를 먼저 렌더링하는 기능

#### ⚪️ Transition이란

- 리액트로 하여금 긴급한 업데이트와 긴급하지 않은 업데이트를 구별하도록 하는 개념
- `Urgent updates`: 타이핑, 클릭과 같이 사용자 인터렉션에 대한 즉각적인 반영
- `Transition updates`: 하나의 뷰에서 다른 뷰로 UI를 전환하는 것

### useTransition

[목차로 이동하기](#Index)

#### ⚪️ useTransition의 정의

- UI를 차단하지 않고 상태를 업데이트할 수 있는 훅

#### ⚪️ useTransition이 반환하는 것

- `isPending`: transition이 pending 상태임을 알려주는 플래그
- `startTransition`: 상태 업데이트를 transition으로 표시할 수 있는 함수

#### ⚪️ useTransition의 용도

- 즉각적으로 UI를 업데이트하고 싶은 상태 업데이트가 있을 때 사용
- startTransition으로 래핑한 상태 업데이트가 트리거되면 기존에 진행되던 렌더링을 버림

#### ⚪️ startTransition 사용법

- setState를 호출하는 함수(scope라 칭함)를 인수로 전달
- 리액트는 scope를 호출하고 동기적으로 예정된 모든 상태 업데이트를 transition으로 표시
  - 이는 차단되지 않고 로딩을 표시하지 않음
- 예를 들어 유저가 탭을 클릭하고 이어서 바로 다른 탭을 클릭했을 때 처음 탭을 클릭한 상태의 UI를 표시하기까지 기다리지 않고 바로 다른 탭을 클릭한 UI를 그림 (클릭한 탭 상태 업데이트를 startTransition으로 래핑한 경우)

#### ⚪️ isPending 용도

- Transition 중이라는 시각적인 상태를 표현하고자 할 때 사용

### useDeferredValue

[목차로 이동하기](#Index)

#### ⚪️ useDeferredValue의 정의

- UI의 일부 업데이트를 연기할 수 있는 훅

#### ⚪️ useDeferredValue가 반환하는 것

- 첫 렌더링이라면 훅 호출 시 전달한 값과 동일한 값을 리턴
- 업데이트 하는 동안에는 먼저 이전 값으로 리렌더링을 하고(이전 값 리턴), 새로운 값으로 백그라운드에서 또 다른 리렌더를 시도함(업데이트된 값 리턴)
