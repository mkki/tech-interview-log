# React 컴포넌트 라이프사이클

## 전체 흐름 (3단계)

| 단계             | 설명                    | 호출 순서                                                           |
| -------------- | --------------------- | --------------------------------------------------------------- |
| **Mounting**   | 컴포넌트가 DOM에 최초 삽입      | 1. constructor → 2. render → 3. DOM 업데이트 → 4. componentDidMount |
| **Updating**   | props/state 변경 → 리렌더링 | 1. render → 2. DOM 업데이트 → 3. componentDidUpdate                 |
| **Unmounting** | 컴포넌트가 DOM에서 제거        | 1. componentWillUnmount                                         |

## 클래스 컴포넌트 vs 함수형 컴포넌트 대응

### 1. Mounting (마운트)

| 클래스 컴포넌트 | 함수형 컴포넌트 (Hooks) |
|---|---|
| `constructor()` | `useState()` 초기값, `useRef()` 초기값 |
| `static getDerivedStateFromProps()` | 렌더 중 직접 state 계산 |
| `render()` | 함수 본문 자체 |
| `componentDidMount()` | `useEffect(() => {}, [])` |

```tsx
// 클래스
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  componentDidMount() {
    console.log('마운트 완료');
  }
}

// 함수형
const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('마운트 완료');
  }, []);
};
```

### 2. Updating (업데이트)

| 클래스 컴포넌트 | 함수형 컴포넌트 (Hooks) |
|---|---|
| `static getDerivedStateFromProps()` | 렌더 중 직접 계산 |
| `shouldComponentUpdate()` | `React.memo()` |
| `render()` | 함수 본문 자체 |
| `getSnapshotBeforeUpdate()` | 직접 대응 없음 (ref로 유사 구현) |
| `componentDidUpdate()` | `useEffect(() => {}, [deps])` |

```tsx
// 클래스
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    console.log('count 변경됨:', this.state.count);
  }
}

// 함수형
useEffect(() => {
  console.log('count 변경됨:', count);
}, [count]);
```

### 3. Unmounting (언마운트)

| 클래스 컴포넌트 | 함수형 컴포넌트 (Hooks) |
|---|---|
| `componentWillUnmount()` | `useEffect`의 **cleanup 함수** |

```tsx
// 클래스
componentWillUnmount() {
  clearInterval(this.timer);
}

// 함수형
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  return () => clearInterval(timer);
}, []);
```

## useEffect 실행 타이밍 상세

```
렌더링 → DOM 업데이트 → 브라우저 페인트 → useEffect 실행
                  ↑
      useLayoutEffect는 여기서 실행
      (페인트 전, DOM 업데이트 직후)
```

| Hook | 실행 시점 | 용도 |
|---|---|---|
| `useEffect` | 페인트 **후** (비동기) | 데이터 fetching, 구독, 로깅 |
| `useLayoutEffect` | 페인트 **전** (동기) | DOM 측정, 스크롤 위치 조정 |
| `useInsertionEffect` | DOM 삽입 **전** | CSS-in-JS 라이브러리 전용 |

## 함수형 컴포넌트의 실행 순서 정리

```
1. 함수 호출 (렌더)
   ├── useState / useRef 등 초기화
   ├── 함수 본문 실행 (JSX 반환)
   │
2. React가 DOM 커밋
   │
3. useLayoutEffect 실행 (동기, 페인트 전)
   │
4. 브라우저 페인트
   │
5. useEffect 실행 (비동기, 페인트 후)
   │
  ── 리렌더 시 ──
   │
6. 함수 재호출
7. 이전 useLayoutEffect cleanup → 새 useLayoutEffect 실행
8. 브라우저 페인트
9. 이전 useEffect cleanup → 새 useEffect 실행
   │
  ── 언마운트 시 ──
   │
10. useLayoutEffect cleanup
11. useEffect cleanup
```

## 렌더링 스케줄링 & 비동기 처리 (React 19+)

전통적인 Mount/Update/Unmount에는 대응하지 않지만, **렌더링 시점과 우선순위를 제어**하는 현대 React의 핵심 API들이다.

| Hook | 역할 | 라이프사이클 관점 |
|---|---|---|
| `useTransition` | state 업데이트를 **낮은 우선순위**로 표시 | 긴급 렌더와 비긴급 렌더를 분리 |
| `useDeferredValue` | 값의 업데이트를 **지연**하여 UI 응답성 유지 | 리렌더 시점을 React가 판단 |
| `useOptimistic` | async action 중 **낙관적 UI** 즉시 반영 | 실제 결과 전에 임시 렌더 삽입 |
| `use()` | 렌더 중 Promise/Context를 **직접 unwrap** | Suspense를 트리거하여 렌더 일시 중단 |

### useTransition

```tsx
const [isPending, startTransition] = useTransition();

const handleSearch = (query: string) => {
  startTransition(() => {
    setSearchResults(filterItems(query)); // 낮은 우선순위로 처리
  });
};
```

- `startTransition` 안의 state 업데이트는 긴급 업데이트(입력, 클릭)에 양보한다
- `isPending`으로 전환 중 로딩 UI를 표시할 수 있다

### useDeferredValue

```tsx
const deferredQuery = useDeferredValue(query);

// query가 바뀌어도 deferredQuery는 React가 여유 있을 때 갱신
// → 타이핑 중 목록 리렌더가 지연되어 입력이 끊기지 않음
```

### useOptimistic

```tsx
const [optimisticLikes, addOptimisticLike] = useOptimistic(
  likes,
  (current, newLike) => [...current, newLike]
);

const handleLike = async () => {
  addOptimisticLike({ id: 'temp', user: 'me' }); // 즉시 UI 반영
  await saveLikeToServer();                       // 서버 응답 후 실제 값으로 교체
};
```

### use()

```tsx
const DataComponent = ({ dataPromise }: { dataPromise: Promise<Data> }) => {
  const data = use(dataPromise); // 렌더 중 Promise를 직접 읽음
  return <div>{data.name}</div>; // Suspense boundary가 fallback 처리
};
```

- 조건문/반복문 안에서도 호출 가능 (일반 Hook 규칙 예외)
- `Suspense`와 함께 동작하여, Promise가 resolve될 때까지 렌더를 중단한다

### 렌더링 흐름에서의 위치

```
                    ┌─ use() → Suspense fallback (Promise pending)
                    │
1. 함수 호출 (렌더)  ──┤
                    │  ┌─ useTransition: 낮은 우선순위 업데이트 예약
                    └──┤
                       └─ useOptimistic: 낙관적 값 즉시 반영
                              │
2. DOM 커밋 ───────────────────┤
3. useLayoutEffect ───────────┤
4. 브라우저 페인트 ───────────────┤
5. useEffect ─────────────────┘
   │
   └─ useDeferredValue: 지연된 값이 이 시점쯤 갱신 → 재렌더 트리거
```

## 2026년 기준 추가 사항

**`useEffectEvent`** (React 19.2+)는 라이프사이클 단계에 속하지 않는다. effect 내부에서 호출되는 "이벤트 핸들러"로, 의존성 추적에서 제외되는 별도 개념이다.

**React Compiler** (v1.0)를 사용하면 `shouldComponentUpdate` / `React.memo` / `useMemo` / `useCallback`에 해당하는 최적화를 컴파일러가 자동 처리하므로, 업데이트 단계의 수동 최적화 코드가 크게 줄어든다.

## 면접 포인트

- **"왜 `componentWillMount`가 없어졌나요?"** -- React 16.3에서 deprecated, 18에서 제거. 비동기 렌더링(Concurrent Mode)에서 여러 번 호출될 수 있어 부작용 발생 위험
- **`useEffect`는 `componentDidMount + componentDidUpdate + componentWillUnmount`를 합친 것인가요?** -- 개념적으로는 맞지만, 실행 타이밍이 다르다. `componentDidMount/DidUpdate`는 페인트 전 동기 실행이고 `useEffect`는 페인트 후 비동기 실행이라, 정확한 대응은 `useLayoutEffect`이다
- **cleanup은 언제 실행되나요?** -- 언마운트 시 + 의존성 변경으로 재실행되기 직전
