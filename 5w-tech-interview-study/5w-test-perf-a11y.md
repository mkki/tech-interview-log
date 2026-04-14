### Q1. 프론트엔드 환경에서 테스트 코드를 작성할 때 얻을 수 있는 진짜 이점은 무엇이며, 반대로 테스트 코드를 짤 때 가장 경계해야 할 안티 패턴(예: 구현 세부사항 테스트)은 무엇이라고 생각하나요?

* 의도: 실용적인 테스트 작성 기준(ROI)과 유지보수하기 좋은 테스트의 특징을 이해하고 있는지 평가
* 키워드: `구현 세부사항(Implementation details)` `행위 기반(Behavior-driven)` `유지보수성` `리팩토링 안정성`

<details>
<summary>답변</summary>

**진짜 이점**은 리팩토링 시 기존 기능이 깨지지 않았음을 보장하는 **안전망(Safety Net)** 역할입니다. 코드를 수정할 때 자신감을 가지고 변경할 수 있고, 버그를 조기에 발견하여 디버깅 시간을 단축합니다. 또한 테스트 코드 자체가 **살아있는 문서** 역할을 하여, 컴포넌트의 의도된 동작을 명시합니다.

**가장 경계해야 할 안티 패턴은 구현 세부사항(Implementation Details) 테스트**입니다:

- **나쁜 예**: 내부 state 변수명, 특정 함수 호출 횟수, CSS 클래스명을 검증하는 테스트
- **좋은 예**: 사용자가 버튼을 클릭하면 화면에 특정 텍스트가 나타나는지(행위 기반) 검증하는 테스트

구현 세부사항에 결합된 테스트는 내부 리팩토링만 해도 깨지기 때문에, 기능은 정상인데 테스트가 실패하는 **거짓 음성(False Negative)**이 빈번해집니다. 결국 개발자들이 테스트를 신뢰하지 않게 되어, "테스트부터 고치자"가 아니라 "테스트를 무시하자"가 되어버립니다. Kent C. Dodds의 원칙처럼 **"사용자가 소프트웨어를 사용하는 방식과 유사하게 테스트할수록 더 높은 신뢰도를 얻는다"**가 핵심입니다.

</details>

### Q2. 단위 테스트(Unit Test)와 통합 테스트(Integration Test)의 역할을 각각 설명해 주시고, 실무에서 어떤 기준으로 두 테스트의 비중을 조절하는 것이 좋다고 생각하시나요?

* 의도: 테스트의 경계를 이해하고, 컴포넌트 간의 상호작용을 검증하는 통합 테스트의 가치를 아는지 확인
* 키워드: `단일 모듈 검증(단위)` `컴포넌트 상호작용 및 상태 흐름 검증(통합)` `테스트 트로피(Testing Trophy)`

<details>
<summary>답변</summary>

**단위 테스트(Unit Test)**:
- 개별 함수, Custom Hook, 유틸리티 등 **단일 모듈을 격리하여 검증**합니다.
- 외부 의존성을 모킹하여 빠르게 실행됩니다.
- 적합: 복잡한 비즈니스 로직, 데이터 변환 함수, 상태 계산 로직.

**통합 테스트(Integration Test)**:
- 여러 컴포넌트가 함께 동작하는 **상호작용과 상태 흐름을 검증**합니다.
- 실제에 가까운 환경에서 테스트하므로 신뢰도가 높습니다.
- 적합: 폼 입력 → 유효성 검증 → 제출 흐름, 리스트 필터링 → 결과 표시.

**테스트 트로피(Testing Trophy)** 전략을 따르는 것이 실무적입니다. Kent C. Dodds가 제안한 이 모델은 기존 테스트 피라미드와 달리 **통합 테스트에 가장 큰 비중**을 둡니다:

```
    E2E (소량 - 핵심 시나리오만)
   통합 (가장 많이 - 비용 대비 효과 최대)
  단위 (적당히 - 복잡한 로직 위주)
 정적 분석 (TypeScript, ESLint)
```

프론트엔드에서는 컴포넌트 간 상호작용이 핵심이므로, 단일 함수를 격리 테스트하는 것보다 사용자 시나리오에 가까운 통합 테스트가 비용 대비 가장 높은 신뢰도를 제공합니다.

</details>

### Q3. TDD(Test-Driven Development)를 프론트엔드 UI 컴포넌트 개발에 도입할 때 현실적으로 겪게 되는 어려움이나 한계점은 무엇일까요?

* 의도: 방법론에 대한 맹신보다는, 잦은 UI 변경이 일어나는 프론트엔드 환경의 특수성과 트레이드오프를 인지하고 있는지 평가
* 키워드: `잦은 요구사항/디자인 변경` `높은 유지보수 비용` `시각적 요소(CSS/레이아웃) 테스트의 한계`

<details>
<summary>답변</summary>

**1. 잦은 요구사항/디자인 변경**:
프론트엔드 UI는 기획/디자인 피드백에 따라 자주 바뀝니다. TDD로 테스트를 먼저 작성해도, UI 변경이 발생하면 테스트도 함께 수정해야 하므로 **유지보수 비용이 이중으로 발생**합니다.

**2. 시각적 요소의 테스트 한계**:
CSS 레이아웃, 애니메이션, 반응형 디자인 등 시각적 요소는 코드 기반 테스트(JSDOM)로 검증하기 어렵습니다. "버튼이 화면 오른쪽에 있어야 한다"를 테스트로 먼저 작성하기는 비현실적입니다.

**3. 구현 전 설계의 어려움**:
백엔드의 API 입출력과 달리, UI 컴포넌트의 최종 형태는 개발 과정에서 점진적으로 결정되는 경우가 많습니다. 명확한 스펙 없이 테스트를 먼저 작성하면 의미 없는 테스트가 됩니다.

**실무적 접근**: UI 컴포넌트 자체보다는 **비즈니스 로직(Custom Hook, 유틸 함수)에 TDD를 적용**하고, UI 컴포넌트는 구현 후 행위 기반 통합 테스트를 작성하는 것이 현실적입니다.

</details>

### Q4. React Testing Library(RTL)를 사용할 때 `fireEvent` 대신 `userEvent`를 권장하는 이유는 무엇인가요? 또한, 비동기로 렌더링되는 UI를 테스트할 때 RTL의 어떤 유틸리티 함수들을 활용하시나요?

* 의도: 브라우저의 실제 이벤트를 얼마나 정교하게 시뮬레이션하는지, 비동기 렌더링 생명주기를 어떻게 다루는지 숙련도 평가
* 키워드: `실제 사용자 상호작용(키보드 딜레이, 포커스 등)` `userEvent` `findBy` `waitFor`

<details>
<summary>답변</summary>

**`fireEvent` vs `userEvent`**:

`fireEvent`는 단일 DOM 이벤트만 발생시킵니다. 반면 `userEvent`는 **실제 사용자 상호작용을 시뮬레이션**합니다:

- `userEvent.click(button)`: focus → pointerdown → mousedown → pointerup → mouseup → click 순서로 이벤트 발생
- `userEvent.type(input, 'hello')`: 한 글자씩 keydown → keypress → input → keyup 이벤트 발생 (키보드 딜레이 포함)
- 포커스 이동, 클립보드 동작 등 브라우저의 부수 효과도 재현

즉, `userEvent`가 실제 브라우저 동작에 더 가까우므로 **테스트 신뢰도가 높아집니다**.

**비동기 UI 테스트 유틸리티**:

- **`findBy*`**: Promise를 반환하며, 요소가 나타날 때까지 기다립니다 (내부적으로 `waitFor` + `getBy` 조합).
```javascript
const element = await screen.findByText('로딩 완료');
```
- **`waitFor`**: 콜백 내의 단언(assertion)이 성공할 때까지 반복 실행합니다.
```javascript
await waitFor(() => expect(screen.getByText('결과')).toBeInTheDocument());
```
- **`waitForElementToBeRemoved`**: 특정 요소가 DOM에서 사라질 때까지 기다립니다 (로딩 스피너 제거 확인 등).

</details>

### Q5. 테스트 코드 실행 시 콘솔에 종종 **"Warning: An update to Component inside a test was not wrapped in act(...)"** 라는 경고가 뜹니다. 이 에러가 발생하는 원인과 해결 방법은 무엇인가요?

* 의도: React의 비동기 상태 업데이트 렌더링 큐 메커니즘과 테스트 환경의 동기적 실행 사이의 타이밍 이슈 트러블슈팅 경험 검증
* 키워드: `비동기 상태 업데이트` `렌더링 완료 대기` `findBy 쿼리 사용` `waitFor 처리`

<details>
<summary>답변</summary>

**원인**: `act()` 경고는 테스트 코드가 **React의 상태 업데이트/렌더링이 완료되기 전에 단언(assertion)을 수행**할 때 발생합니다. React는 상태 변경 후 비동기적으로 리렌더링을 수행하는데, 테스트는 동기적으로 진행되므로 타이밍 불일치가 생깁니다.

예를 들어 API 호출 후 `setState`가 호출되는데, 테스트가 그 업데이트를 기다리지 않고 바로 결과를 확인하면 경고가 발생합니다.

**해결 방법**:

1. **`findBy*` 쿼리 사용** (가장 권장):
```javascript
// 비동기 상태 업데이트가 반영될 때까지 자동 대기
const result = await screen.findByText('데이터 로드 완료');
```

2. **`waitFor`로 감싸기**:
```javascript
await waitFor(() => {
  expect(screen.getByText('업데이트됨')).toBeInTheDocument();
});
```

3. **`userEvent`와 함께 `await` 사용**:
```javascript
await userEvent.click(submitButton); // 이벤트 후 상태 업데이트 대기
```

핵심은 **상태 변경을 유발하는 동작 후, 렌더링이 완료될 때까지 기다리는 비동기 쿼리를 사용**하는 것입니다. RTL의 `findBy*`와 `waitFor`는 내부적으로 `act()`를 포함하고 있어 이 문제를 자연스럽게 해결합니다.

</details>

### Q6. `setTimeout`이나 `setInterval` 같은 타이머 로직이 포함된 컴포넌트를 테스트할 때, 실제 10초, 20초를 기다리지 않고 테스트를 통과시키려면 어떤 기법을 사용해야 하나요?

* 의도: 테스트 실행 속도를 보장하기 위한 환경 제어(가짜 타이머) 기술 숙련도 확인
* 키워드: `Jest 가짜 타이머(jest.useFakeTimers)` `jest.advanceTimersByTime`

<details>
<summary>답변</summary>

**Jest의 가짜 타이머(Fake Timers)**를 사용합니다. 실제 시간이 흐르는 것이 아니라, 테스트 코드에서 시간을 프로그래밍적으로 제어합니다.

```javascript
beforeEach(() => {
  jest.useFakeTimers(); // 가짜 타이머 활성화
});

afterEach(() => {
  jest.useRealTimers(); // 실제 타이머 복원
});

test('10초 후 만료 메시지가 표시된다', () => {
  render(<Timer duration={10000} />);

  // 아직 만료 전
  expect(screen.queryByText('만료')).not.toBeInTheDocument();

  // 시간을 10초 앞으로 이동
  act(() => {
    jest.advanceTimersByTime(10000);
  });

  // 만료 메시지 확인
  expect(screen.getByText('만료')).toBeInTheDocument();
});
```

**주요 API**:
- `jest.useFakeTimers()`: `setTimeout`, `setInterval`, `Date` 등을 가짜로 교체
- `jest.advanceTimersByTime(ms)`: 지정한 밀리초만큼 시간을 앞으로 이동
- `jest.runAllTimers()`: 대기 중인 모든 타이머를 즉시 실행
- `jest.runOnlyPendingTimers()`: 현재 대기 중인 타이머만 실행 (재귀적 타이머에서 무한 루프 방지)

주의: 가짜 타이머 사용 시 `act()`로 감싸야 React의 상태 업데이트가 반영됩니다. Vitest에서도 `vi.useFakeTimers()` 등 동일한 패턴을 제공합니다.

</details>

### Q7. 프론트엔드 통합 테스트 시 백엔드 API 의존성을 끊어내기 위해 MSW(Mock Service Worker)를 도입하면 어떤 구조적 이점이 있나요? 기존의 `jest.mock('axios')` 같은 모듈 모킹 방식과 비교해 설명해 주세요.

* 의도: 네트워크 레벨의 모킹 원리(Service Worker)를 이해하고, 테스트 환경과 실제 개발 환경의 로직을 얼마나 통일성 있게 유지할 수 있는지 아키텍처 관점에서 평가
* 키워드: `네트워크 레벨 인터셉트` `Service Worker` `실제 fetch/axios 코드 수정 없음` `높은 신뢰도`

<details>
<summary>답변</summary>

**모듈 모킹 (`jest.mock('axios')`)의 한계**:
- 애플리케이션 코드의 **구현 세부사항에 결합**됩니다. HTTP 클라이언트를 axios에서 fetch로 교체하면 모든 모킹이 깨집니다.
- 실제 네트워크 요청이 발생하지 않으므로, 요청 헤더/URL 구성, 에러 응답 처리 등의 코드 경로가 테스트되지 않습니다.
- 모킹 설정이 모듈별로 흩어져 관리가 어렵습니다.

**MSW의 구조적 이점**:
- **네트워크 레벨에서 인터셉트**: Service Worker(브라우저) 또는 인터셉터(Node.js)가 실제 HTTP 요청을 가로챕니다. 애플리케이션 코드는 **수정 없이 그대로 실행**됩니다.
- **높은 신뢰도**: fetch든 axios든, 어떤 HTTP 클라이언트를 사용하든 동일하게 동작합니다. 요청 구성 → 네트워크 전송 → 응답 파싱까지의 전체 경로가 테스트됩니다.
- **핸들러 재사용**: 테스트, 개발 서버(Storybook), 로컬 개발 환경에서 동일한 핸들러를 공유할 수 있습니다.

```javascript
// 테스트와 개발 환경에서 동일한 핸들러 사용
export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([{ id: 1, name: 'Kim' }]);
  }),
];
```

</details>

### Q8. Context API나 외부 라이브러리(Zustand 등)로 관리되는 전역 상태를 사용하는 컴포넌트를 테스트할 때, 해당 전역 상태를 어떻게 주입(Mocking)하여 테스트 환경을 격리하시나요?

* 의도: 컴포넌트 트리의 의존성(Provider)을 이해하고, 테스트용 Wrapper 컴포넌트를 구성할 줄 아는지 확인
* 키워드: `Custom Render 함수` `테스트용 Provider 주입` `초기 상태(Initial State) 모킹`

<details>
<summary>답변</summary>

**Context API - Custom Render 함수 패턴**:

RTL의 `render`를 래핑하여 테스트에 필요한 Provider를 자동 주입하는 Custom Render를 만듭니다:

```javascript
// test-utils.js
function customRender(ui, { initialState, ...options } = {}) {
  function Wrapper({ children }) {
    return (
      <ThemeProvider>
        <AuthContext.Provider value={initialState?.auth}>
          {children}
        </AuthContext.Provider>
      </ThemeProvider>
    );
  }
  return render(ui, { wrapper: Wrapper, ...options });
}

export { customRender as render };
```

```javascript
// 테스트 파일
import { render } from './test-utils';

test('로그인 상태에서 프로필이 보인다', () => {
  render(<Profile />, {
    initialState: { auth: { user: { name: 'Kim' }, isLoggedIn: true } }
  });
  expect(screen.getByText('Kim')).toBeInTheDocument();
});
```

**Zustand - 스토어 초기화 패턴**:

Zustand는 각 테스트 전에 스토어를 초기 상태로 리셋하거나, 테스트용 초기 상태를 주입합니다:

```javascript
beforeEach(() => {
  useStore.setState({ count: 0, user: null }); // 매 테스트 전 초기화
});

test('특정 상태에서의 동작', () => {
  useStore.setState({ user: { name: 'Kim' } }); // 테스트용 상태 주입
  render(<Profile />);
});
```

</details>

### Q9. 테스트를 짤 때 '모든 외부 모듈과 하위 컴포넌트를 다 모킹(Mocking)하는 것'이 과연 좋은 테스트일까요? 모킹을 남용했을 때 발생하는 문제점은 무엇인가요?

* 의도: 모킹의 트레이드오프를 묻는 질문. 진짜 코드가 고장 났는데 테스트는 통과해버리는 '거짓 양성'의 위험성을 아는지 검증
* 키워드: `거짓 양성(False Positive)` `테스트 신뢰도 하락` `내부 구현 결합도 증가`

<details>
<summary>답변</summary>

**모킹 남용의 핵심 문제는 "거짓 양성(False Positive)"**입니다. 실제 코드에 버그가 있는데, 모킹된 환경에서는 테스트가 통과해버리는 상황입니다.

**구체적인 문제점**:

1. **테스트 신뢰도 하락**: 모킹된 모듈이 실제와 다르게 동작하면, "테스트는 통과했는데 프로덕션에서 터진다"는 최악의 상황이 발생합니다. API 응답 구조가 변경되었는데 모킹은 옛날 구조로 남아있는 경우가 대표적입니다.

2. **내부 구현 결합도 증가**: `jest.mock('./child-component')`처럼 하위 컴포넌트를 모킹하면, 파일 경로나 컴포넌트 분리 방식이 바뀔 때마다 테스트가 깨집니다. 리팩토링에 취약해집니다.

3. **유지보수 부담**: 모킹이 많을수록 테스트 설정 코드가 비대해지고, 실제 동작보다 모킹 설정을 이해하는 데 더 많은 시간이 듭니다.

**원칙**: 모킹은 **제어할 수 없는 외부 경계(네트워크 API, 타이머, 브라우저 API)**에만 최소한으로 적용하고, 내부 모듈이나 하위 컴포넌트는 가능한 한 실제 코드를 사용합니다. 이것이 통합 테스트의 가치가 높은 이유이기도 합니다.

</details>

### Q10. 프론트엔드 환경에서 TDD를 맹목적으로 도입하거나 스냅샷(Snapshot) 테스트에 지나치게 의존할 때 겪게 되는 현실적인 한계점은 무엇이며, 이를 실무에서 어떻게 보완해야 할까요?

* 의도: 프론트엔드 개발의 특수성(잦은 UI/UX 변경)을 이해하고, 유지보수 비용(ROI)을 고려하여 테스트 전략을 세울 수 있는지 평가
* 키워드: `잦은 요구사항 변경` `테스트 유지보수 비용` `거짓 음성(False Negative)` `스냅샷 업데이트 남용`

<details>
<summary>답변</summary>

**TDD 맹목적 도입의 한계**:
- UI는 기획/디자인 피드백에 따라 빈번하게 변경되므로, 구현 전에 작성한 테스트가 금방 무의미해집니다.
- 테스트 수정 비용이 기능 개발 비용에 육박하면 ROI가 떨어집니다.
- 시각적 요소(레이아웃, 애니메이션)는 코드 기반 테스트로 검증이 어렵습니다.

**스냅샷 테스트 과의존의 한계**:
- 스냅샷은 DOM 구조의 문자열 비교일 뿐, **의미 있는 변경과 무의미한 변경을 구분하지 못합니다**.
- 사소한 마크업 변경에도 실패하여 **거짓 음성(False Negative)**이 빈번합니다.
- 개발자들이 내용을 확인하지 않고 습관적으로 `--updateSnapshot`을 실행하면, 스냅샷 테스트의 의미가 사라집니다.

**실무 보완 전략**:
1. **TDD는 비즈니스 로직(Hook, 유틸)에 집중**: 순수 로직은 입출력이 명확하므로 TDD 적합
2. **스냅샷은 작고 안정적인 컴포넌트에만**: 아이콘, 디자인 토큰 등 변경이 드문 요소
3. **UI는 행위 기반 통합 테스트**: "사용자가 X하면 Y가 보인다" 패턴
4. **시각적 회귀는 Chromatic/Percy**: 픽셀 단위 비교로 CSS 깨짐 방지

</details>

### Q11. 스냅샷 테스트가 통과했다고 해서, 브라우저 화면에 UI가 정상적으로 렌더링되었다고 100% 확신할 수 없는 기술적인 이유는 무엇인가요?

* 의도: 스냅샷이 그저 'HTML/DOM 구조의 문자열 비교'일 뿐, 실제 CSS 렌더링 결과물이나 상호작용을 보장하지 못한다는 맹점을 아는지 확인
* 키워드: `DOM 구조 비교` `CSS 스타일링 미반영` `시각적 결함(Visual Bug)`

<details>
<summary>답변</summary>

스냅샷 테스트는 **컴포넌트의 렌더링 결과를 직렬화된 문자열(DOM 구조)**로 저장하고, 이전 스냅샷과 문자열 비교만 수행합니다. 다음을 검증하지 못합니다:

**1. CSS 스타일링 미반영**:
스냅샷은 DOM 구조만 비교하므로, `display: none`으로 숨겨져 있거나, `z-index`가 잘못되어 다른 요소에 가려진 것, 텍스트 색상이 배경과 같아 보이지 않는 것 등 **시각적 결함(Visual Bug)**을 감지하지 못합니다.

**2. 레이아웃 깨짐 미감지**:
테스트가 실행되는 JSDOM 환경은 실제 브라우저의 렌더링 엔진(레이아웃, 페인팅)이 없습니다. 요소의 크기, 위치, 오버플로우 등을 계산하지 않습니다.

**3. 상호작용 미검증**:
스냅샷은 정적인 한 시점의 DOM만 캡처합니다. 호버 효과, 스크롤 동작, 애니메이션 등의 상호작용 결과는 검증하지 않습니다.

**4. 크로스 브라우저 차이 미반영**:
JSDOM은 단일 환경이므로, Chrome/Safari/Firefox 간의 렌더링 차이를 감지할 수 없습니다.

이런 한계를 보완하기 위해 **시각적 회귀 테스트(Visual Regression Test)** 도구(Chromatic, Percy)가 필요합니다.

</details>

### Q12. TDD의 사이클(Red-Green-Refactor)을 프론트엔드 UI 컴포넌트 개발에 그대로 적용하기 어려운 이유는 무엇이라고 생각하시나요?

* 의도: 방법론에 얽매이지 않고 실용주의적인 관점을 가졌는지 묻는 질문. 비즈니스 로직(Hook 등)과 UI 렌더링의 테스트 접근 방식 분리 역량 검증
* 키워드: `시각적 요소의 모호성` `구현 전 설계의 어려움` `비즈니스 로직(Custom Hook) 위주의 TDD`

<details>
<summary>답변</summary>

**Red-Green-Refactor**는 "실패하는 테스트 작성 → 통과하는 최소 코드 작성 → 리팩토링"의 사이클입니다. UI 컴포넌트에 그대로 적용하기 어려운 이유는:

**1. 시각적 요소의 모호성**:
"버튼이 파란색이어야 한다", "카드가 2열로 배치되어야 한다" 같은 시각적 요구사항은 코드 테스트로 작성하기 어렵고, 작성하더라도 의미가 낮습니다 (CSS 클래스명 검증은 구현 세부사항 테스트).

**2. 구현 전 설계의 어려움**:
API 함수는 입출력이 명확하지만, UI 컴포넌트는 "어떤 HTML 구조로, 어떤 이벤트 흐름으로 구현할지"가 개발 과정에서 점진적으로 결정됩니다. 테스트를 먼저 작성하려면 구현 방식을 미리 확정해야 하는 모순이 생깁니다.

**3. 탐색적 개발 과정**:
UI 개발은 브라우저에서 실시간으로 확인하며 조정하는 탐색적 과정이 많습니다. 매번 테스트 → 구현 사이클을 거치면 이 흐름이 끊깁니다.

**실무적 분리**:
- **TDD 적합**: Custom Hook, 유틸 함수, 상태 관리 로직 (입출력이 명확)
- **구현 후 테스트 적합**: UI 컴포넌트 (행위 기반 통합 테스트)

</details>

### Q13. 기존에 RTL을 활용한 통합 테스트가 존재함에도 불구하고, Storybook과 Chromatic을 활용한 '시각적 회귀 테스트(Visual Regression Test)'를 파이프라인에 추가로 도입하는 이유는 무엇인가요?

* 의도: 컴포넌트 주도 개발(CDD)의 장점을 체감하고 있으며, 기능적 버그와 시각적 버그를 분리해서 방어할 줄 아는지 평가
* 키워드: `CSS/레이아웃 깨짐 방지` `픽셀 단위 비교` `독립적인 UI 개발` `디자이너와의 협업`

<details>
<summary>답변</summary>

**RTL 통합 테스트가 커버하지 못하는 영역**이 있기 때문입니다:

- RTL은 JSDOM 환경에서 실행되어 **CSS 렌더링, 레이아웃 계산, 시각적 결과를 검증하지 못합니다**.
- "기능은 정상인데 버튼이 화면 밖으로 밀려났다", "글로벌 CSS 변경으로 다른 페이지 여백이 깨졌다" 같은 시각적 버그를 놓칩니다.

**Storybook + Chromatic의 이점**:

1. **픽셀 단위 비교**: Chromatic은 실제 브라우저에서 컴포넌트를 렌더링하고 **스크린샷을 찍어 이전 버전과 픽셀 단위로 비교**합니다. CSS 깨짐, 레이아웃 밀림 등을 자동 감지합니다.

2. **독립적인 UI 개발**: Storybook에서 컴포넌트를 격리하여 다양한 상태(props 조합, 엣지 케이스)별로 시각적 결과를 확인할 수 있습니다.

3. **디자이너 협업**: Chromatic의 UI Review 기능으로 디자이너가 PR 단위로 시각적 변경 사항을 승인/거절할 수 있어, 코드 리뷰와 디자인 리뷰가 통합됩니다.

4. **CI/CD 파이프라인 통합**: PR마다 자동으로 시각적 변경 사항을 캡처하여, 의도치 않은 시각적 회귀를 머지 전에 차단합니다.

</details>

### Q14. 글로벌 CSS(예: reset.css)를 수정했더니, 전혀 상관없는 다른 페이지의 버튼 여백이 깨지는 사이드 이펙트가 발생했습니다. 기존의 단위/통합 테스트가 이를 잡아내기 어려운 이유는 무엇일까요?

* 의도: JSDOM 환경(RTL이 동작하는 가상 환경)과 실제 브라우저 렌더링 엔진의 차이를 이해하고 있는지 검증
* 키워드: `JSDOM의 한계` `레이아웃/페인팅 미지원` `실제 브라우저 렌더링`

<details>
<summary>답변</summary>

RTL 기반 테스트가 실행되는 **JSDOM**은 브라우저의 DOM API를 Node.js에서 에뮬레이션한 것이지, **실제 렌더링 엔진이 아닙니다**.

**JSDOM이 하지 않는 것**:
- **레이아웃 계산(Layout/Reflow)**: 요소의 실제 크기, 위치, 여백(margin/padding) 계산을 하지 않습니다. `getBoundingClientRect()`는 항상 0을 반환합니다.
- **CSS 스타일 적용**: 글로벌 CSS 파일의 변경이 다른 요소에 미치는 영향(캐스케이딩)을 계산하지 않습니다.
- **페인팅(Paint)**: 실제 픽셀을 그리지 않으므로, 시각적 결과물을 확인할 수 없습니다.

따라서 `reset.css`에서 `button { margin: 0; }`을 제거했을 때, JSDOM에서는 버튼의 여백 변화를 전혀 감지하지 못합니다. DOM 구조와 텍스트 내용은 변하지 않았으므로 기능 테스트는 모두 통과합니다.

**해결**: 이런 사이드 이펙트는 **실제 브라우저에서 렌더링하는 시각적 회귀 테스트(Chromatic, Percy)** 또는 **E2E 테스트(Cypress, Playwright의 스크린샷 비교)**로만 감지할 수 있습니다.

</details>

### Q15. Chromatic을 GitHub Actions 같은 CI/CD에 연동하여 자동화했을 때, 개발자뿐만 아니라 기획자나 디자이너와의 협업 관점에서 어떤 혁신적인 변화가 생기나요?

* 의도: 도구 도입이 단순한 코드 검증을 넘어 팀의 '커뮤니케이션 비용'을 어떻게 줄여주는지 거시적인 안목 평가
* 키워드: `UI 리뷰 자동화` `PR(Pull Request) 기반 변경 사항 시각화` `살아있는 문서(Living Documentation)`

<details>
<summary>답변</summary>

**1. PR 기반 시각적 변경 사항 자동 시각화**:
PR이 올라오면 Chromatic이 자동으로 변경된 컴포넌트의 Before/After 스크린샷을 생성합니다. 디자이너가 코드를 읽지 않고도 **"이 PR이 UI를 어떻게 바꾸는지"를 한눈에 확인**할 수 있습니다.

**2. UI 리뷰 프로세스 통합**:
코드 리뷰(개발자)와 디자인 리뷰(디자이너)가 하나의 PR에서 동시에 진행됩니다. 디자이너가 Chromatic에서 시각적 변경을 승인(Accept)하거나 수정 요청(Deny)할 수 있어, 별도의 디자인 QA 미팅이 줄어듭니다.

**3. 살아있는 문서(Living Documentation)**:
Storybook이 배포되면, 각 컴포넌트의 모든 상태(variant)가 항상 최신 상태로 문서화됩니다. 기획자가 "이 버튼의 비활성화 상태가 어떻게 생겼지?"를 Storybook에서 직접 확인할 수 있어, 슬랙으로 스크린샷을 주고받는 커뮤니케이션 비용이 대폭 감소합니다.

**4. 의도하지 않은 변경 자동 차단**:
글로벌 스타일 변경으로 인해 전혀 다른 페이지의 UI가 깨진 경우, Chromatic이 PR 단계에서 변경을 감지하여 머지를 차단합니다.

</details>

### Q16. Cypress를 활용해 E2E 테스트를 작성할 때, 백엔드 실제 서버와 연동하는 대신 네트워크 요청을 가로채서(Intercept) 테스트 더블(Test Double)로 처리하는 경우가 있습니다. 실제 서버를 쓰는 것과 모킹(Mocking)하는 것은 각각 어떤 상황에 적합한가요?

* 의도: E2E 테스트의 가장 큰 적인 'Flakiness(불안정성)'를 통제할 줄 알며, 테스트의 격리성과 완전성 사이에서 트레이드오프를 계산할 수 있는지 평가
* 키워드: `테스트 불안정성(Flakiness)` `외부 환경 격리` `cy.intercept` `실제 환경 검증`

<details>
<summary>답변</summary>

**네트워크 모킹 (`cy.intercept`)이 적합한 상황**:
- **일상적인 개발 과정의 E2E 테스트**: 빠른 피드백이 필요할 때
- **에러/엣지 케이스 테스트**: 500 에러, 타임아웃, 빈 응답 등 서버에서 재현하기 어려운 상황
- **외부 서비스 의존성 격리**: 결제 API, 소셜 로그인 등 제어할 수 없는 외부 서비스
- **CI/CD 속도 최적화**: 네트워크 지연 없이 빠르게 실행

```javascript
cy.intercept('GET', '/api/posts', { fixture: 'posts.json' }).as('getPosts');
cy.visit('/posts');
cy.wait('@getPosts');
```

**실제 서버 연동이 적합한 상황**:
- **릴리스 전 최종 검증(Smoke Test)**: 실제 환경과 동일한 조건에서 핵심 시나리오 확인
- **API 계약 검증**: 프론트엔드와 백엔드 간의 실제 요청/응답이 일치하는지 확인
- **데이터 무결성 확인**: DB에 실제로 데이터가 저장/수정되는지 검증

**Flakiness 관리**: 실제 서버를 사용하면 네트워크 지연, DB 상태, 서버 가용성 등으로 인해 테스트가 불안정해집니다. 따라서 **일상적 CI에서는 모킹, 릴리스 전에는 실제 서버**로 분리하는 전략이 실무적입니다.

</details>

### Q17. 단위/통합 테스트와 비교했을 때, E2E 테스트가 가지는 치명적인 단점들은 무엇이며, 실무 프로젝트에서 이 테스트들의 비율을 어떻게 가져가는 것이 이상적일까요?

* 의도: '테스트 피라미드' 또는 '테스트 트로피' 철학을 이해하고, 비용 대비 효과가 가장 좋은 테스트 전략을 설계할 수 있는지 검증
* 키워드: `느린 실행 속도` `높은 유지보수 비용` `병목 현상` `통합/단위 테스트 중심의 피라미드`

<details>
<summary>답변</summary>

**E2E 테스트의 치명적인 단점**:

1. **느린 실행 속도**: 실제 브라우저를 구동하고, 페이지 로드/네트워크 대기가 필요하므로 단위 테스트보다 수십~수백 배 느립니다.
2. **높은 Flakiness**: 네트워크 타이밍, 애니메이션 대기, 서버 상태 등 외부 요인으로 동일한 테스트가 때때로 실패합니다.
3. **높은 유지보수 비용**: UI 변경(셀렉터 변경 등)에 민감하고, 실패 시 원인 파악(프론트? 백엔드? 네트워크?)이 어렵습니다.
4. **CI 병목**: 실행 시간이 길어 배포 파이프라인의 병목이 됩니다.

**이상적인 테스트 비율 (테스트 트로피 기준)**:

| 계층 | 비율 | 대상 |
|------|------|------|
| **정적 분석** | 기본 | TypeScript, ESLint |
| **단위 테스트** | 20~30% | 복잡한 비즈니스 로직, 유틸, Hook |
| **통합 테스트** | 50~60% | 컴포넌트 상호작용, 사용자 시나리오 |
| **E2E 테스트** | 10~20% | 핵심 사용자 흐름 (회원가입, 결제 등) |

핵심은 **"가장 적은 비용으로 가장 높은 신뢰도"**를 주는 통합 테스트에 집중하고, E2E는 비즈니스적으로 절대 실패하면 안 되는 Critical Path에만 적용하는 것입니다.

</details>

### Q18. 로그인, 로그아웃, 특정 권한 체크처럼 여러 E2E 테스트 케이스에서 반복적으로 사용되는 로직을 그대로 복사해서 쓰면 유지보수가 매우 힘들어집니다. Cypress에서 이를 어떻게 우아하게 해결하나요?

* 의도: 테스트 코드 역시 '코드'이므로, 중복을 제거하고 추상화(Custom Commands)할 줄 아는 엔지니어링 역량 확인
* 키워드: `커스텀 커맨드(Custom Commands)` `로직 추상화` `DRY 원칙` `프로그래매틱 로그인(API 호출로 토큰 발급)`

<details>
<summary>답변</summary>

**Cypress Custom Commands**로 반복 로직을 추상화합니다:

```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  // 프로그래매틱 로그인: UI를 거치지 않고 API로 직접 인증
  cy.request('POST', '/api/auth/login', { email, password })
    .then((response) => {
      window.localStorage.setItem('token', response.body.token);
    });
});

Cypress.Commands.add('seedDatabase', (fixture) => {
  cy.request('POST', '/api/test/seed', { fixture });
});
```

```javascript
// 테스트 파일에서 간결하게 사용
describe('대시보드', () => {
  beforeEach(() => {
    cy.login('admin@test.com', 'password123');
    cy.visit('/dashboard');
  });

  it('관리자 메뉴가 보인다', () => {
    cy.findByText('사용자 관리').should('be.visible');
  });
});
```

**핵심 포인트**:

1. **프로그래매틱 로그인**: 매 테스트마다 로그인 폼을 입력하지 않고, **API 호출로 토큰을 직접 발급**받습니다. 테스트 속도가 크게 향상되고, 로그인 UI 변경에 영향받지 않습니다.

2. **DRY 원칙**: 공통 로직을 한 곳에서 관리하므로, 인증 방식이 변경되어도 Custom Command만 수정하면 됩니다.

3. **가독성 향상**: `cy.login()`처럼 의미가 명확한 추상화로 테스트의 의도가 잘 드러납니다.

</details>

### Q19. 프론트엔드 애플리케이션의 초기 로딩 속도를 개선하기 위해 번들 사이즈(Bundle Size)를 어떻게 최적화하시나요? Code Splitting과 Lazy Loading의 원리를 포함해 설명해 주세요.

* 의도: 번들러(Webpack 등)의 동작 원리를 이해하고 있으며, 불필요한 코드를 초기에 다운로드하지 않도록 제어하는 설계 능력이 있는지 평가
* 키워드: `Code Splitting(코드 분할)` `Dynamic Import(import())` `Lazy Loading` `청크(Chunk)` `TTI(Time To Interactive)`

<details>
<summary>답변</summary>

**Code Splitting(코드 분할)**:
하나의 거대한 번들을 여러 개의 작은 청크(Chunk)로 나누는 기법입니다. 번들러(Webpack, Vite 등)가 `import()` 동적 임포트 구문을 만나면, 해당 모듈을 별도의 청크로 분리합니다.

**Lazy Loading(지연 로딩)**:
분리된 청크를 **필요한 시점에만 다운로드**합니다. React에서는 `React.lazy()`와 `Suspense`를 조합합니다:

```javascript
const AdminPage = React.lazy(() => import('./pages/AdminPage'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/admin" element={<AdminPage />} />
      </Routes>
    </Suspense>
  );
}
```

사용자가 `/admin`에 접근하기 전까지 `AdminPage`의 JS 코드는 다운로드되지 않습니다.

**추가 최적화 전략**:
- **Tree Shaking**: 사용하지 않는 코드를 번들에서 제거 (ES Modules의 정적 분석 활용)
- **서드파티 라이브러리 분리**: `lodash` 전체 대신 `lodash/debounce`처럼 개별 함수만 import
- **텍스트 압축**: Gzip/Brotli로 전송 용량 감소

이를 통해 초기 번들 크기를 줄여 **TTI(Time To Interactive)**를 단축합니다.

</details>

### Q20. `webpack-bundle-analyzer` 같은 분석 도구를 통해 번들을 확인했을 때, 어떤 기준을 가지고 Code Splitting을 적용할 대상을 선정하시나요?

* 의도: 도구를 활용한 정량적 분석 경험과 최적화의 '우선순위'를 결정하는 실무 판단력 검증
* 키워드: `라우트(페이지) 기반 분할` `무거운 서드파티 라이브러리(lodash, moment 등) 분리` `중복 코드(Vendors) 분리`

<details>
<summary>답변</summary>

번들 분석 도구의 **트리맵(Treemap)**에서 다음 기준으로 분할 대상을 선정합니다:

**1. 라우트(페이지) 기반 분할 (최우선)**:
사용자가 처음 방문하지 않는 페이지(관리자 페이지, 설정 페이지 등)를 별도 청크로 분리합니다. 가장 효과가 크고 구현이 간단합니다.

**2. 무거운 서드파티 라이브러리 분리**:
번들 분석에서 큰 비중을 차지하는 라이브러리를 식별합니다:
- `moment.js` → `dayjs`로 대체 (70KB → 2KB)
- `lodash` 전체 import → 개별 함수 import
- 차트 라이브러리(Chart.js 등) → 해당 페이지에서만 동적 import

**3. 조건부 렌더링 컴포넌트**:
모달, 드롭다운, 리치 텍스트 에디터 등 **사용자 상호작용 후에만 표시되는 컴포넌트**를 Lazy Loading합니다.

**4. Vendors 청크 분리**:
자주 변경되지 않는 라이브러리(React, React DOM 등)를 별도 청크로 분리하여, 앱 코드 변경 시에도 **브라우저 캐시를 활용**합니다.

**판단 기준**: 분할의 ROI를 계산합니다. 2KB짜리 모듈을 분할하는 것은 HTTP 요청 오버헤드만 늘리므로, **일정 크기 이상**(일반적으로 30KB+)의 모듈을 대상으로 합니다.

</details>

### Q21. Lazy Loading을 적용하면 초기 로딩은 빨라지지만, 막상 해당 컴포넌트나 페이지로 이동할 때 네트워크 지연(네트워크 폭포 현상)이 발생할 수 있습니다. 이 트레이드오프를 어떻게 완화하시나요?

* 의도: 최적화가 불러올 수 있는 또 다른 UX 저하를 방어할 수 있는지 확인
* 키워드: `Suspense Fallback UI(스켈레톤)` `Preloading(사전 로딩)` `마우스 호버(Hover) 시점 로드`

<details>
<summary>답변</summary>

**1. Suspense Fallback UI (즉각적인 피드백)**:
청크 로드 중 빈 화면 대신 스켈레톤이나 스피너를 보여줘 **체감 대기 시간을 줄입니다**:
```jsx
<Suspense fallback={<PageSkeleton />}>
  <LazyPage />
</Suspense>
```

**2. Preloading (사전 로딩)**:
사용자가 **해당 페이지로 이동할 것이 예상되는 시점**에 미리 청크를 다운로드합니다:

```javascript
// 마우스 호버 시 사전 로딩
const AdminPage = React.lazy(() => import('./pages/AdminPage'));

function NavLink() {
  const preload = () => import('./pages/AdminPage'); // 청크 사전 다운로드

  return (
    <Link
      to="/admin"
      onMouseEnter={preload} // 호버 시점에 로드 시작
      onFocus={preload}      // 키보드 포커스 시에도
    >
      관리자
    </Link>
  );
}
```

**3. Next.js `<Link>` 자동 프리페칭**:
Next.js의 `<Link>` 컴포넌트는 뷰포트에 보이는 링크의 청크를 자동으로 프리페칭합니다.

**4. Route 기반 Prefetch**:
Idle 시간(브라우저가 한가할 때)에 다음에 방문할 가능성이 높은 라우트의 청크를 미리 로드합니다:
```html
<link rel="prefetch" href="/static/js/admin-page.chunk.js" />
```

</details>

### Q22. 브라우저의 렌더링 파이프라인 관점에서 Reflow(Layout)와 Repaint의 차이는 무엇이며, 60fps를 유지하는 부드러운 애니메이션을 구현하기 위해 CSS 속성을 어떻게 선택해야 하나요?

* 의도: 브라우저가 화면을 그리는 픽셀 파이프라인의 비용을 정확히 이해하고, 하드웨어 가속(GPU)을 활용할 줄 아는지 평가
* 키워드: `픽셀 파이프라인` `Reflow(기하학적 변화)` `Repaint(시각적 변화)` `Composite(합성)` `transform` `opacity` `GPU 가속`

<details>
<summary>답변</summary>

**브라우저 픽셀 파이프라인**:
```
JavaScript → Style → Layout(Reflow) → Paint(Repaint) → Composite
```

**Reflow (Layout)**: 요소의 **기하학적 속성**(크기, 위치)이 변경되면 발생합니다. 변경된 요소뿐 아니라 주변 요소들의 위치도 재계산해야 하므로 **가장 비용이 큽니다**.
- 트리거 속성: `width`, `height`, `margin`, `padding`, `top`, `left`, `font-size`

**Repaint (Paint)**: **시각적 속성**만 변경되면 Layout 없이 다시 그리기만 합니다. Reflow보다 비용이 낮습니다.
- 트리거 속성: `color`, `background-color`, `box-shadow`, `border-color`

**Composite (합성)**: **`transform`과 `opacity`만 변경**하면 Layout, Paint를 모두 건너뛰고 GPU에서 합성만 수행합니다. **가장 비용이 낮습니다**.

**60fps 애니메이션을 위한 원칙**:
```css
/* 나쁜 예: Reflow 발생 (Layout → Paint → Composite) */
.box { transition: width 0.3s, top 0.3s; }

/* 좋은 예: Composite만 발생 (GPU 가속) */
.box { transition: transform 0.3s, opacity 0.3s; }
```

60fps = 매 프레임 16.67ms 안에 처리해야 합니다. `transform`/`opacity`는 GPU 레이어에서 처리되어 메인 스레드를 블로킹하지 않으므로 이 시간 예산을 지킬 수 있습니다.

</details>

### Q23. 넓이나 높이(`width`, `height`)를 애니메이션으로 변경하면 화면이 심하게 버벅거리는데, 이를 `transform: scale()`로 변경하면 부드러워지는 기술적인 이유는 무엇인가요?

* 의도: Layout 단계를 건너뛰고 Composite 단계만 발생시키는 브라우저 최적화 원리 숙련도 확인
* 키워드: `Layout 단계 생략` `Render Tree 재계산 방지` `GPU 레이어 분리`

<details>
<summary>답변</summary>

**`width`/`height` 애니메이션이 버벅거리는 이유**:
매 프레임마다 **Layout(Reflow)**이 발생합니다. 요소의 크기가 바뀌면 자기 자신뿐 아니라, 주변 형제/부모 요소의 위치와 크기도 재계산(Render Tree 재계산)해야 합니다. 이 과정이 메인 스레드에서 수행되므로, 16.67ms(60fps) 내에 완료하지 못하면 프레임 드롭이 발생합니다.

**`transform: scale()`이 부드러운 이유**:
`transform`은 요소를 **별도의 GPU 레이어(Compositor Layer)로 분리(Promote)**합니다. 이 레이어의 변환은 GPU에서 처리되며:

1. **Layout 단계 생략**: 요소의 실제 크기(Layout Box)는 변하지 않고, GPU가 텍스처를 확대/축소할 뿐입니다. 주변 요소에 영향을 주지 않습니다.
2. **Paint 단계 생략**: 레이어의 내용물을 다시 그리지 않고, 이미 그려진 텍스처를 합성(Composite)만 합니다.
3. **메인 스레드 비점유**: GPU가 별도로 처리하므로, JS 실행이나 다른 렌더링 작업을 블로킹하지 않습니다.

단, `transform: scale()`은 시각적으로만 확대될 뿐 실제 레이아웃은 변하지 않으므로, 주변 요소와의 상호작용(클릭 영역 등)이 의도와 다를 수 있습니다.

</details>

### Q24. Chrome DevTools의 Performance 패널을 사용하여 성능 병목(Bottleneck)을 분석해 본 경험이 있나요? 플레임 차트(Flame Chart)에서 주로 어떤 부분을 눈여겨보시나요?

* 의도: 감이 아닌 '데이터'를 기반으로 성능 이슈를 트러블슈팅하는 실무 역량 검증
* 키워드: `Frame Drop(빨간 줄)` `Long Task(50ms 이상)` `Call Tree 분석` `병목 구간(Bottom-Up)`

<details>
<summary>답변</summary>

Performance 패널에서 주로 확인하는 항목들:

**1. Frame Drop (빨간 바)**:
상단의 프레임 바에서 **빨간색으로 표시된 프레임**은 16.67ms(60fps) 내에 렌더링이 완료되지 못한 프레임입니다. 이 지점을 먼저 찾아 해당 시간대를 확대합니다.

**2. Long Task (50ms 이상의 작업)**:
메인 스레드 타임라인에서 **회색 바 위에 빨간 삼각형**이 표시된 작업이 Long Task입니다. 이 작업이 실행되는 동안 사용자 입력이 처리되지 않아 버벅거림을 유발합니다.

**3. Flame Chart 분석**:
Long Task를 클릭하면 **콜 스택이 플레임 차트로 표시**됩니다:
- **넓은 막대**: 실행 시간이 긴 함수 → 최적화 대상
- **깊은 스택**: 호출 깊이가 깊은 함수 → 불필요한 중첩 호출 확인
- **노란색(Scripting)**: JS 실행 시간
- **보라색(Rendering)**: Layout/Style 재계산 시간
- **초록색(Painting)**: 페인팅 시간

**4. Bottom-Up 탭**:
가장 오래 걸린 함수를 **Self Time(자체 실행 시간) 기준으로 정렬**하여, 실제 병목이 되는 함수를 식별합니다. Total Time은 하위 호출을 포함하지만, Self Time이 높은 함수가 진짜 최적화 대상입니다.

</details>

### Q25. 무거운 이미지나 모달 컴포넌트를 렌더링할 때 발생하는 지연을 해결하기 위해, Lazy Loading과 Preloading(사전 로딩) 전략을 실무에서 어떻게 조합하여 사용하시나요?

* 의도: 리소스의 성격(당장 필요한가 vs 나중에 필요한가)에 따라 로딩 우선순위를 영리하게 설계할 수 있는지 평가
* 키워드: `우선순위` `레이지 로딩(Intersection Observer)` `프리로딩(rel="preload" 또는 JS 동기화)` `사용자 경험(UX)`

<details>
<summary>답변</summary>

리소스의 **필요 시점과 중요도**에 따라 전략을 분리합니다:

**1. 즉시 필요 + 중요 → Preloading (사전 로딩)**:
- LCP(Largest Contentful Paint) 이미지, Above the fold 히어로 이미지
- `<link rel="preload">` 또는 Next.js Image의 `priority` 속성으로 최우선 로드

**2. 나중에 필요 + 스크롤해야 보임 → Lazy Loading**:
- 스크롤 아래(Below the fold) 이미지, 갤러리 썸네일
- `loading="lazy"` 속성 또는 Intersection Observer로 뷰포트 진입 시 로드

**3. 사용자 액션 후 필요 → 예측적 Preloading + Lazy Loading 조합**:
- 모달, 리치 텍스트 에디터 등
- 기본적으로 Lazy Loading하되, **사용자 의도가 감지되는 시점**(호버, 포커스)에 미리 로드

```javascript
// 이미지: Intersection Observer로 Lazy Loading
<img loading="lazy" src="below-fold.jpg" />

// 모달 컴포넌트: 호버 시 Preload + 클릭 시 표시
const Modal = React.lazy(() => import('./HeavyModal'));
const preloadModal = () => import('./HeavyModal');

<button onMouseEnter={preloadModal} onClick={openModal}>
  모달 열기
</button>
```

핵심은 **"사용자가 필요로 하는 순서대로 리소스를 우선순위화"**하는 것입니다.

</details>

### Q26. 첫 화면(Above the fold)에 렌더링되는 가장 큰 핵심 이미지(LCP 요소)에 Lazy Loading을 적용하면 안 되는 이유는 무엇이며, 이 이미지는 어떻게 최적화해야 하나요?

* 의도: Core Web Vitals 중 하나인 LCP(Largest Contentful Paint)에 대한 지식과 이미지 최적화의 정석을 묻는 질문
* 키워드: `LCP 지연` `초도 렌더링 방해` `우선 로드(Priority)` `차세대 포맷(WebP/AVIF)` `적절한 해상도(srcset)`

<details>
<summary>답변</summary>

**Lazy Loading을 적용하면 안 되는 이유**:
LCP 이미지에 `loading="lazy"`를 적용하면, 브라우저가 **뷰포트에 이미지가 보이는지 확인한 후에야 다운로드를 시작**합니다. 이 확인 과정 자체가 지연을 유발하여 LCP 점수가 악화됩니다. 첫 화면에 바로 보이는 이미지는 **가능한 한 빨리 다운로드를 시작**해야 합니다.

**LCP 이미지 최적화 전략**:

1. **우선 로드 (Priority)**:
```html
<link rel="preload" as="image" href="hero.webp" />
<!-- 또는 Next.js -->
<Image src="/hero.jpg" priority />
```
`fetchpriority="high"` 속성으로 브라우저에 높은 우선순위를 명시할 수도 있습니다.

2. **차세대 포맷 (WebP/AVIF)**:
동일 품질에서 JPEG 대비 25~50% 작은 파일 크기. `<picture>` 태그로 폴백 제공:
```html
<picture>
  <source srcset="hero.avif" type="image/avif" />
  <source srcset="hero.webp" type="image/webp" />
  <img src="hero.jpg" alt="히어로 이미지" />
</picture>
```

3. **적절한 해상도 (srcset + sizes)**:
모바일에서 4K 이미지를 내려받지 않도록 디바이스별 최적 크기를 제공합니다.

4. **CLS 방지**: `width`/`height`를 명시하여 이미지 로드 전 공간을 확보합니다.

</details>

### Q27. 텍스트 압축(Gzip, Brotli)을 적용하면 네트워크 전송 용량은 획기적으로 줄어들지만, 클라이언트(브라우저) 측에서 압축을 해제하기 위해 CPU 연산 리소스가 소모됩니다. 이 비용은 무시해도 될 수준일까요?

* 의도: 네트워크 I/O 병목과 CPU 연산 병목 사이의 트레이드오프를 종합적으로 사고할 수 있는지 묻는 질문
* 키워드: `네트워크 전송 지연 vs 압축 해제 속도` `현대 브라우저의 효율성` `일반적으로 네트워크 비용 감소가 훨씬 유리함`

<details>
<summary>답변</summary>

**결론: 대부분의 경우 무시해도 될 수준이며, 압축의 이득이 압도적으로 큽니다.**

**네트워크 vs CPU 비용 비교**:
- **네트워크 전송**: 1MB 파일을 3G 환경에서 전송하면 수 초가 소요됩니다. Brotli 압축으로 70~80% 줄이면 전송 시간이 수 초 → 수백 ms로 단축됩니다.
- **압축 해제**: 현대 브라우저의 Brotli/Gzip 해제는 네이티브 C/C++ 코드로 구현되어 있어, 1MB 파일의 해제에 수 ms밖에 걸리지 않습니다.

**Brotli vs Gzip**:
- **Brotli**: Gzip보다 15~25% 더 높은 압축률. 텍스트 기반 에셋(JS, CSS, HTML)에 특히 효과적. 대부분의 현대 브라우저가 지원.
- **Gzip**: 레거시 호환성이 더 좋고, 서버 측 압축 속도가 빠름.

**주의할 경우**: 
- 이미 압축된 파일(JPEG, PNG, MP4 등)에 추가 압축을 적용하면 오히려 파일 크기가 커질 수 있습니다.
- 매우 작은 파일(1KB 미만)은 압축 오버헤드가 이득보다 클 수 있습니다.

서버(Nginx, CDN)에서 `Content-Encoding: br` 또는 `gzip`을 설정하면 자동으로 적용됩니다.

</details>

### Q28. 웹 폰트 적용 시 브라우저 렌더링 차이로 인해 발생하는 FOIT(Flash of Invisible Text)와 FOUT(Flash of Unstyled Text) 현상에 대해 설명하고, CSS의 `font-display` 속성을 활용해 이를 어떻게 제어하시나요?

* 의도: 브라우저별 폰트 렌더링 정책을 이해하고 있으며, 텍스트가 안 보이는 시간(FOIT)을 줄여 사용자 경험을 개선할 줄 아는지 평가
* 키워드: `FOIT(보이지 않는 텍스트)` `FOUT(기본 폰트 노출 후 전환)` `font-display: swap` `font-display: fallback`

<details>
<summary>답변</summary>

**FOIT (Flash of Invisible Text)**:
웹 폰트가 로드될 때까지 **텍스트가 아예 보이지 않는** 현상. Safari 등 일부 브라우저의 기본 동작으로, 최대 3초간 빈 화면이 될 수 있습니다. 사용자 경험에 치명적입니다.

**FOUT (Flash of Unstyled Text)**:
웹 폰트가 로드될 때까지 **시스템 기본 폰트(fallback)로 먼저 표시**한 후, 로드 완료 시 웹 폰트로 전환되는 현상. 텍스트가 "깜빡" 바뀌는 느낌이 있지만, 콘텐츠는 즉시 읽을 수 있습니다.

**`font-display` 속성으로 제어**:

```css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap; /* 가장 권장 */
}
```

| 값 | 동작 | 적합한 상황 |
|---|---|---|
| `swap` | 즉시 fallback 표시 → 로드 후 전환 (FOUT) | **대부분의 본문 텍스트** (콘텐츠 가독성 우선) |
| `fallback` | 100ms 대기 후 fallback → 3초 내 로드되면 전환 | 폰트 전환 깜빡임을 줄이고 싶을 때 |
| `optional` | 100ms 대기 → 로드 안 되면 fallback 유지 | 네트워크가 느린 환경, 폰트가 선택적일 때 |
| `block` | 3초간 텍스트 숨김 (FOIT) | 아이콘 폰트 등 폰트 없이는 의미 없는 경우 |

일반적으로 **`font-display: swap`**이 가장 권장됩니다. 콘텐츠를 즉시 보여주는 것이 UX에 유리합니다.

</details>

### Q29. 글로벌 서비스나 한글 폰트처럼 용량이 매우 큰(몇 MB 단위) 폰트 파일을 최적화하기 위해, 서브셋(Subset) 폰트와 `unicode-range` 속성을 어떻게 활용하시나요?

* 의도: 무거운 에셋 용량을 근본적으로 다이어트시키는 최적화 기법 숙련도 검증
* 키워드: `WOFF2 포맷` `불필요한 글자(특수기호 등) 제거(Subset)` `특정 언어만 로드(unicode-range)`

<details>
<summary>답변</summary>

**1. WOFF2 포맷 사용**:
WOFF2는 웹 폰트 전용 압축 포맷으로, TTF 대비 **약 30~50% 작은 파일 크기**를 제공합니다. 모든 현대 브라우저가 지원합니다.

**2. 서브셋(Subset) 폰트**:
사용하지 않는 글리프(특수문자, 한자 등)를 제거하여 파일 크기를 줄입니다. 한글 폰트는 11,172개의 완성형 글자를 포함하는데, 실제로 자주 사용되는 2,350자(KS X 1001)만 남기면 크기가 크게 줄어듭니다.

도구: `pyftsubset`(fonttools), Google Fonts의 자동 서브셋

**3. `unicode-range`로 필요한 범위만 로드**:
```css
/* 한글 서브셋 */
@font-face {
  font-family: 'NotoSansKR';
  src: url('/fonts/noto-kr.woff2') format('woff2');
  unicode-range: U+AC00-D7AF; /* 한글 완성형 범위 */
}

/* 라틴 서브셋 */
@font-face {
  font-family: 'NotoSansKR';
  src: url('/fonts/noto-latin.woff2') format('woff2');
  unicode-range: U+0020-007E; /* 기본 라틴 범위 */
}
```

브라우저는 페이지에서 **실제로 사용되는 문자의 unicode-range에 해당하는 파일만 다운로드**합니다. 영문만 있는 페이지에서는 한글 폰트 파일을 아예 요청하지 않습니다.

Google Fonts는 이 기법을 자동으로 적용하여 폰트를 수십 개의 작은 슬라이스로 나눠 제공합니다.

</details>

### Q30. 폰트나 작은 아이콘 이미지 리소스 등을 Base64 형태(Data URI)로 변환하여 CSS나 JS 파일 내부에 직접 임베딩했을 때, 초기 로딩 속도 관점에서 어떤 장단점이 존재하나요?

* 의도: HTTP 요청 횟수 감소의 이점과, 번들 사이즈 비대화라는 트레이드오프를 정확히 계산할 수 있는지 확인
* 키워드: `HTTP 왕복 비용 감소` `CSS/JS 파일 크기 증가` `브라우저 캐싱 효율 저하`

<details>
<summary>답변</summary>

**장점 - HTTP 왕복 비용 감소**:
- 별도의 HTTP 요청 없이 CSS/JS 파일과 함께 한 번에 다운로드됩니다.
- HTTP/1.1 환경에서 동시 연결 수 제한(6개)이 있을 때 효과적입니다.
- 매우 작은 아이콘(1KB 미만)은 HTTP 오버헤드가 파일 크기보다 클 수 있어, 임베딩이 유리합니다.

**단점 - 번들 사이즈 비대화 & 캐싱 효율 저하**:
1. **파일 크기 약 33% 증가**: Base64 인코딩은 바이너리를 텍스트로 변환하므로 원본 대비 약 1/3 크기가 증가합니다.
2. **브라우저 캐싱 효율 저하**: 별도 파일이면 이미지만 캐싱되지만, CSS에 임베딩되면 CSS가 변경될 때마다 이미지도 다시 다운로드됩니다.
3. **CSS 파싱 지연**: CSS 파일이 비대해지면 CSSOM 구성이 느려져 렌더링 시작이 지연됩니다.
4. **중복 다운로드**: 같은 이미지를 여러 CSS 파일에 임베딩하면 중복 전송됩니다.

**실무 기준**: HTTP/2 이상에서는 멀티플렉싱으로 요청 오버헤드가 줄어 Base64의 이점이 감소합니다. **1~2KB 이하의 아이콘에만 제한적으로 사용**하고, 그 이상은 별도 파일 + 캐싱이 유리합니다.

</details>

### Q31. 이미지를 지연 로딩(Lazy Loading)할 때 흔히 발생하는 문제인 CLS(Cumulative Layout Shift) 현상의 원인은 무엇이며, 이를 방지하기 위해 HTML이나 CSS 속성을 어떻게 설계해야 하나요?

* 의도: Core Web Vitals의 핵심 지표인 CLS를 정확히 인지하고, 반응형 웹 환경에서 이미지 공간을 미리 확보(Placeholder)하는 실무 기술 확인
* 키워드: `레이아웃 밀림 현상` `width/height 명시` `aspect-ratio(종횡비) 속성` `스켈레톤 UI`

<details>
<summary>답변</summary>

**CLS 발생 원인**:
이미지가 지연 로딩되면, 처음에는 이미지 영역의 높이가 0입니다. 이미지가 로드되면 갑자기 공간이 확보되면서 **아래에 있던 콘텐츠가 밀려 내려가는 "레이아웃 밀림(Layout Shift)"**이 발생합니다. 사용자가 읽고 있던 텍스트가 갑자기 이동하여 UX가 크게 저하됩니다.

**방지 전략**:

**1. `width`/`height` 명시**:
```html
<img src="photo.jpg" width="800" height="600" loading="lazy" alt="사진" />
```
브라우저가 이미지 로드 전에 종횡비를 계산하여 공간을 미리 확보합니다.

**2. `aspect-ratio` CSS 속성**:
```css
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
}
```
반응형 환경에서 컨테이너의 종횡비를 고정하여, 이미지 크기에 관계없이 레이아웃이 안정됩니다.

**3. 스켈레톤 UI / Placeholder**:
이미지 영역에 블러 처리된 저해상도 이미지(LQIP) 또는 스켈레톤 UI를 배치하여 공간을 확보하면서 시각적 피드백도 제공합니다.

**4. Next.js `<Image>` 컴포넌트**:
`width`/`height` 또는 `fill` 속성을 필수로 지정하게 하여, CLS를 구조적으로 방지합니다.

</details>

### Q32. 스크롤 이벤트 리스너 대신 `Intersection Observer` API를 활용하여 이미지 지연 로딩을 구현할 때, 브라우저 렌더링 성능(메인 스레드 점유율) 측면에서 얻을 수 있는 결정적인 이점은 무엇인가요?

* 의도: 동기적인 스크롤 이벤트의 비용(Reflow 유발 등)을 이해하고, 비동기적인 브라우저 최적화 API를 선택한 근거를 묻는 질문
* 키워드: `메인 스레드 블로킹 해소` `스크롤 이벤트 과부하(Debounce/Throttle) 제거` `브라우저 백그라운드 연산`

<details>
<summary>답변</summary>

**스크롤 이벤트 리스너의 문제**:
- `scroll` 이벤트는 스크롤 중 **초당 수십~수백 회** 발생합니다.
- 매 이벤트마다 `getBoundingClientRect()` 등을 호출하면 **강제 Reflow(Layout Thrashing)**가 발생합니다. 브라우저가 최신 레이아웃을 계산해야 하므로 메인 스레드가 블로킹됩니다.
- Throttle/Debounce로 완화할 수 있지만, 여전히 메인 스레드에서 콜백이 실행됩니다.

**Intersection Observer의 결정적 이점**:

1. **메인 스레드 비점유**: 요소의 가시성(Visibility) 감지를 **브라우저가 내부적으로(백그라운드에서) 최적화하여 처리**합니다. 렌더링 파이프라인과 통합되어 있어 별도의 레이아웃 계산이 불필요합니다.

2. **콜백 빈도 최소화**: 요소가 뷰포트에 **진입/이탈하는 시점에만** 콜백이 호출됩니다. 스크롤 중 불필요한 반복 실행이 없습니다.

3. **Throttle/Debounce 불필요**: 브라우저가 알아서 최적의 타이밍에 콜백을 호출하므로, 수동 제어가 필요 없습니다.

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.src = entry.target.dataset.src; // 실제 이미지 로드
      observer.unobserve(entry.target); // 한 번만 실행
    }
  });
}, { rootMargin: '200px' }); // 200px 전에 미리 로드 시작
```

</details>

### Q33. 브라우저의 HTTP 캐시(Cache-Control, ETag 등)를 활용하는 방법과, 프론트엔드 코드 내에서 메모리 캐싱을 구현하는 것은 어떤 차이가 있으며 각각 언제 사용해야 할까요?

* 의도: 네트워크 레벨의 최적화(HTTP 캐시)와 애플리케이션 레벨의 최적화 사이의 역할 분담을 인지하고 있는지 평가
* 키워드: `정적 에셋(이미지, JS) 브라우저 캐시 보관` `잦은 API 응답 데이터(애플리케이션 메모리 캐싱)`

<details>
<summary>답변</summary>

**HTTP 캐시 (브라우저/네트워크 레벨)**:
- **대상**: 정적 에셋(JS 번들, CSS, 이미지, 폰트)
- **동작**: 서버가 `Cache-Control` 헤더를 설정하면, 브라우저가 디스크/메모리에 응답을 저장하고 재요청 시 캐시에서 반환
- **주요 전략**:
  - `Cache-Control: max-age=31536000, immutable` + 파일명 해시(bundle.a1b2c3.js): 영구 캐싱, 내용 변경 시 파일명이 바뀌어 자동 무효화
  - `ETag` / `Last-Modified`: 조건부 요청으로 변경 여부만 확인 (304 Not Modified)
- **특징**: 네트워크 요청 자체를 줄여 가장 근본적인 최적화

**애플리케이션 메모리 캐싱 (코드 레벨)**:
- **대상**: API 응답 데이터, 계산 결과
- **동작**: SWR/React Query 같은 라이브러리가 API 응답을 메모리에 캐싱하고, stale-while-revalidate 전략으로 관리
- **특징**: 같은 데이터를 여러 컴포넌트에서 사용할 때 중복 요청 방지, 즉각적인 UI 업데이트

**사용 기준**:
| 구분 | HTTP 캐시 | 메모리 캐싱 |
|------|----------|-----------|
| 대상 | 정적 에셋 (잘 변하지 않음) | 동적 API 데이터 (자주 변함) |
| 지속성 | 디스크에 영구 저장 | 탭/세션 동안만 유지 |
| 제어 | 서버 헤더 설정 | 프론트엔드 코드로 제어 |

</details>

### Q34. 전역 상태 관리(예: Redux)를 사용할 때 `useSelector`로 값을 가져오는 과정에서 불필요한 컴포넌트 리렌더링이 자주 발생합니다. 이 원인은 무엇이며, Reselect 같은 라이브러리나 Memoization 기법을 통해 어떻게 해결하셨나요?

* 의도: 상태 객체의 참조(Reference)가 변경될 때 발생하는 렌더링 트리거의 원리를 이해하고, 파생된 데이터(Derived Data)를 효율적으로 계산할 수 있는지 평가
* 키워드: `참조 동일성(Reference Equality)` `객체 새로 생성 방지` `선택자(Selector) 메모이제이션` `Reselect`

<details>
<summary>답변</summary>

**원인 - 참조 동일성(Reference Equality) 문제**:
`useSelector`는 이전 반환값과 현재 반환값을 `===`(참조 비교)로 비교합니다. selector가 **매번 새로운 객체/배열을 생성**하면, 내용이 같아도 참조가 다르므로 불필요한 리렌더링이 발생합니다:

```javascript
// 나쁜 예: 매번 새 배열을 생성 → 매번 리렌더링
const activeUsers = useSelector(state =>
  state.users.filter(u => u.isActive) // filter는 항상 새 배열 반환
);
```

**해결 - Reselect로 메모이제이션된 Selector**:
```javascript
import { createSelector } from 'reselect';

const selectActiveUsers = createSelector(
  [(state) => state.users], // 입력 selector
  (users) => users.filter(u => u.isActive) // 결과 계산
);

// 컴포넌트에서 사용
const activeUsers = useSelector(selectActiveUsers);
```

`createSelector`는 **입력 값(state.users)이 변경되지 않으면 이전 결과를 그대로 반환**(동일 참조)합니다. `state.users`가 같은 참조인 한, `filter`가 재실행되지 않고 캐싱된 결과가 반환되어 리렌더링이 방지됩니다.

**추가 기법**:
- `useSelector`에 `shallowEqual`을 두 번째 인자로 전달하여 객체의 1단계 프로퍼티를 비교
- 상태 구조를 정규화(Normalize)하여 불필요한 중첩 참조 변경을 줄임

</details>

### Q35. 단순히 `React.memo`나 `useMemo`를 쓰는 것을 넘어, 특정 함수 내부의 로직 자체가 너무 무거워 메인 스레드를 병목시킬 때 코드 레벨이나 아키텍처 레벨에서 어떻게 개선할 수 있을까요?

* 의도: 시간 복잡도를 줄이는 알고리즘적 접근이나, 메인 스레드 블로킹을 우회하는 Web Worker 등의 고급 해결책을 떠올릴 수 있는지 검증
* 키워드: `시간 복잡도(O(n^2) -> O(n)) 개선` `배열 메서드 중첩 최적화` `Web Worker 도입`

<details>
<summary>답변</summary>

**1. 알고리즘/시간 복잡도 개선 (코드 레벨)**:
- `O(n^2)` → `O(n)`: 중첩 `filter + find` 대신 `Map`/`Set` 자료구조로 룩업 테이블 구성
- 불필요한 배열 순회 제거: `filter().map()` 체이닝 대신 `reduce`로 한 번에 처리
```javascript
// 개선 전: O(n * m) - 매번 users 배열을 순회
orders.map(order => ({
  ...order,
  user: users.find(u => u.id === order.userId) // O(m)
}));

// 개선 후: O(n + m) - Map으로 한 번만 색인
const userMap = new Map(users.map(u => [u.id, u])); // O(m)
orders.map(order => ({
  ...order,
  user: userMap.get(order.userId) // O(1)
}));
```

**2. Web Worker (아키텍처 레벨)**:
CPU 집약적인 작업(대량 데이터 정렬, 이미지 처리, 암호화 등)을 **별도 스레드(Web Worker)**에서 실행하여 메인 스레드를 블로킹하지 않습니다:
```javascript
// worker.js
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};

// 컴포넌트
const worker = new Worker('./worker.js');
worker.postMessage(largeData);
worker.onmessage = (e) => setResult(e.data);
```

**3. 시간 분할(Time Slicing)**:
작업을 작은 청크로 나누어 `requestIdleCallback`이나 `scheduler.yield()`로 프레임 사이사이에 실행하여 UI 응답성을 유지합니다.

</details>

### Q36. 불필요한 CSS 코드(Dead Code)나 사용하지 않는 모듈이 번들에 섞여 들어가 로딩을 느리게 만들 때, 이를 찾아내고 제거(Tree Shaking)하기 위해 어떤 과정을 거치셨나요?

* 의도: PurgeCSS 같은 도구 활용 경험이나, 모듈 임포트 방식(Named vs Default)에 따른 트리 쉐이킹(Tree Shaking) 동작 원리 이해도 확인
* 키워드: `Tree Shaking` `사이드 이펙트(Side Effects) 속성` `미사용 CSS 제거` `Named Import 활용`

<details>
<summary>답변</summary>

**JS Tree Shaking**:
번들러가 **ES Modules의 정적 분석**을 통해 사용되지 않는 export를 번들에서 제거합니다.

**Tree Shaking이 잘 동작하려면**:
1. **Named Import 사용**: `import { debounce } from 'lodash-es'`는 Tree Shaking 가능하지만, `import _ from 'lodash'`는 전체 번들이 포함됩니다.
2. **ES Modules 형식**: CommonJS(`require`)는 정적 분석이 불가능하므로 Tree Shaking이 안 됩니다. `-es` 버전의 라이브러리를 사용합니다.
3. **`sideEffects` 설정**: `package.json`에 `"sideEffects": false`를 명시하면, 번들러가 미사용 모듈을 안전하게 제거합니다. CSS 파일 등 사이드 이펙트가 있는 파일은 `"sideEffects": ["*.css"]`로 예외 처리합니다.

**CSS Dead Code 제거**:
- **PurgeCSS / Tailwind CSS의 내장 Purge**: HTML/JSX 파일을 스캔하여 실제 사용되는 클래스만 남기고 나머지를 제거합니다.
- **Chrome DevTools Coverage 탭**: 실제 사용되지 않는 CSS/JS의 비율을 시각적으로 확인합니다.

**분석 도구 활용**:
- `webpack-bundle-analyzer`: 번들 구성을 트리맵으로 시각화하여 불필요한 모듈 식별
- `import-cost` (VS Code 확장): import 시 실시간으로 모듈 크기 표시

</details>

### Q37. 시맨틱(Semantic) 태그를 사용하는 것이 웹 접근성 측면에서 왜 중요한가요? `<div>`에 `onClick` 이벤트를 달아 버튼처럼 렌더링하는 것과 `<button>` 태그를 사용하는 것은 스크린 리더 환경에서 어떤 차이를 만드나요?

* 의도: HTML 본연의 역할(Role, Name, Value)을 이해하고, 스크린 리더가 요소를 해석하는 기본 원리를 알고 있는지 평가
* 키워드: `시맨틱 마크업` `Role(역할)` `키보드 포커싱` `WAI-ARIA`

<details>
<summary>답변</summary>

**시맨틱 태그가 중요한 이유**:
브라우저는 HTML을 파싱하여 **접근성 트리(Accessibility Tree)**를 생성하고, 스크린 리더는 이 트리를 읽어 사용자에게 정보를 전달합니다. 시맨틱 태그는 **역할(Role), 이름(Name), 상태(State)**를 자동으로 접근성 트리에 노출합니다.

**`<div onClick>` vs `<button>`의 차이**:

| 항목 | `<div onClick>` | `<button>` |
|------|----------------|-----------|
| **Role** | 스크린 리더가 "텍스트"로 읽음 | "버튼"이라고 읽어줌 |
| **키보드 포커스** | Tab으로 포커스 불가 | 자동으로 Tab 포커스 가능 |
| **키보드 활성화** | Enter/Space 반응 없음 | Enter/Space로 클릭 동작 |
| **접근성 트리** | role 없음 | `role="button"` 자동 |

`<div>`를 버튼처럼 쓰려면 직접 추가해야 하는 코드:
```html
<div role="button" tabindex="0"
     onClick={handler}
     onKeyDown={(e) => (e.key === 'Enter' || e.key === ' ') && handler(e)}>
  클릭
</div>
```

**`<button>`을 쓰면 이 모든 것이 자동**입니다. 시맨틱 태그를 사용하면 코드가 간결해지고, 접근성이 기본으로 보장되며, WAI-ARIA 속성을 직접 관리할 필요가 줄어듭니다.

</details>

### Q38. `<img>` 태그의 `alt` 속성은 항상 작성해야 하나요? 장식용 이미지이거나 텍스트를 단순히 시각적으로 꾸미기 위한 이미지일 경우 `alt` 속성을 어떻게 처리하는 것이 접근성에 좋나요?

* 의도: 무의미한 정보(예: "장식용 선 이미지")를 스크린 리더가 불필요하게 읽지 않도록 제어하는 실무 요령 확인
* 키워드: `빈 alt="" 속성 제공` `aria-hidden="true"` `스크린 리더 무시`

<details>
<summary>답변</summary>

**`alt` 속성은 항상 존재해야 하지만, 값은 이미지의 역할에 따라 달라집니다.**

**1. 의미 있는 이미지 → 설명적 alt 텍스트**:
```html
<img src="chart.png" alt="2024년 매출 추이 그래프: 1분기 대비 4분기 30% 상승" />
```
이미지가 전달하는 정보를 텍스트로 설명합니다.

**2. 장식용 이미지 → 빈 alt=""**:
```html
<img src="decorative-line.png" alt="" />
```
`alt=""`(빈 문자열)을 지정하면 스크린 리더가 이 이미지를 **완전히 건너뜁니다**. `alt` 속성 자체를 생략하면 스크린 리더가 파일명("decorative-line.png")을 읽어버리므로, 반드시 빈 값을 명시해야 합니다.

**3. CSS 배경 이미지로 대체**:
순수 장식 이미지는 HTML `<img>` 대신 CSS `background-image`로 처리하면 접근성 트리에 아예 포함되지 않습니다.

**4. 아이콘 + 텍스트 조합**:
텍스트와 함께 사용되는 아이콘은 중복 정보이므로:
```html
<button>
  <img src="search-icon.svg" alt="" /> <!-- 장식용: 빈 alt -->
  검색
</button>
```
또는 `aria-hidden="true"`를 사용하여 접근성 트리에서 제외합니다.

</details>

### Q39. 커스텀 체크박스나 토글 스위치를 만들 때, 시각적으로는 숨겨져 있지만 스크린 리더는 읽을 수 있게(`sr-only` 등) 텍스트를 제공하는 CSS 기법에 대해 알고 계시나요? `display: none`을 쓰면 안 되는 이유는 무엇인가요?

* 의도: 접근성 트리(Accessibility Tree)에서 요소가 제외되는 조건을 정확히 아는지 묻는 핵심 질문
* 키워드: `display: none / visibility: hidden(접근성 트리에서 제외됨)` `sr-only (클리핑, position: absolute)`

<details>
<summary>답변</summary>

**`display: none`을 쓰면 안 되는 이유**:
`display: none`과 `visibility: hidden`은 요소를 **접근성 트리(Accessibility Tree)에서도 제거**합니다. 스크린 리더가 해당 요소를 완전히 무시하므로, 숨긴 텍스트를 읽어줄 수 없습니다.

**sr-only(Screen Reader Only) 기법**:
시각적으로는 보이지 않지만, 접근성 트리에는 남아있어 스크린 리더가 읽을 수 있는 CSS 패턴입니다:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
```

**동작 원리**: 요소를 1x1px로 축소하고 `clip-path`로 잘라내어 **화면에서는 보이지 않지만 DOM과 접근성 트리에는 존재**합니다.

**사용 예시**:
```html
<label>
  <input type="checkbox" class="sr-only" /> <!-- 네이티브 체크박스: 숨기되 접근성 유지 -->
  <span class="custom-checkbox"></span>      <!-- 커스텀 시각적 체크박스 -->
  이용약관에 동의합니다
</label>
```

Tailwind CSS의 `sr-only` 클래스, Bootstrap의 `visually-hidden` 클래스가 이 패턴을 제공합니다.

</details>

### Q40. 모달(Modal) 창이나 팝업을 띄웠을 때 발생하는 '키보드 트랩(Keyboard Trap)' 문제를 방지하고, 포커스 순서(Focus Order)를 논리적으로 유지하기 위해 어떤 기술적인 처리를 해주어야 하나요?

* 의도: 커스텀 UI를 만들 때 DOM 구조의 시각적 순서와 논리적(Tab) 순서를 일치시키고, 포커스를 가두는(Focus Trap) 고급 제어 역량 검증
* 키워드: `Focus Trap` `tabindex` `포커스 이동 제한` `이전 포커스 기억`

<details>
<summary>답변</summary>

모달이 열리면 키보드 사용자는 **모달 내부에서만 Tab 이동**이 가능해야 하고, 모달을 닫으면 **이전 위치로 포커스가 돌아가야** 합니다.

**Focus Trap 구현 핵심**:

1. **모달 열릴 때 포커스 이동**: 모달이 열리면 모달 내부의 첫 번째 포커스 가능 요소로 포커스를 이동합니다.
```javascript
useEffect(() => {
  if (isOpen) {
    previousFocusRef.current = document.activeElement; // 이전 포커스 기억
    modalRef.current.querySelector('button, [tabindex]')?.focus();
  }
}, [isOpen]);
```

2. **Tab 키 가두기**: 모달 내의 마지막 요소에서 Tab을 누르면 첫 번째 요소로, 첫 번째 요소에서 Shift+Tab을 누르면 마지막 요소로 순환합니다.

3. **ESC 키로 닫기**: 키보드 사용자가 모달을 닫을 수 있도록 ESC 키 핸들러를 추가합니다.

4. **모달 닫힐 때 포커스 복원**: 모달을 닫으면 이전에 기억해둔 요소로 포커스를 되돌립니다.
```javascript
const handleClose = () => {
  setIsOpen(false);
  previousFocusRef.current?.focus(); // 이전 포커스 복원
};
```

5. **배경 콘텐츠 비활성화**: 모달 바깥의 콘텐츠에 `aria-hidden="true"`와 `inert` 속성을 적용하여, 스크린 리더와 키보드가 배경 콘텐츠에 접근하지 못하게 합니다.

실무에서는 `focus-trap-react`, Radix UI, Headless UI 같은 라이브러리가 이 패턴을 제공합니다.

</details>

### Q41. `tabindex` 속성에 `0`을 주는 것과 `-1`을 주는 것은 각각 어떤 상황에서 사용하며, 양수(예: `tabindex="1"`)를 사용하는 것을 지양해야 하는 이유는 무엇인가요?

* 의도: 키보드 포커스 제어의 기본 속성인 `tabindex`의 부작용을 알고 있는지 평가
* 키워드: `tabindex="0"(포커스 가능하게 만듦)` `-1(스크립트로만 포커스 이동)` `양수(문서 전체의 논리적 흐름 파괴)`

<details>
<summary>답변</summary>

**`tabindex="0"`**:
- 원래 포커스를 받지 않는 요소(`<div>`, `<span>` 등)를 **Tab 키로 포커스 가능하게** 만듭니다.
- DOM 순서에 따라 자연스럽게 Tab 순서에 포함됩니다.
- 사용: 커스텀 버튼, 인터랙티브 카드 등 비 네이티브 요소에 포커스가 필요할 때

**`tabindex="-1"`**:
- Tab 키로는 포커스되지 않지만, **JavaScript(`element.focus()`)로 프로그래밍적으로만 포커스 가능**합니다.
- 사용: 모달 열림 시 모달 컨테이너로 포커스 이동, 에러 발생 시 에러 메시지로 포커스 이동, Skip Navigation 대상 영역

```html
<div id="main-content" tabindex="-1">
  <!-- Tab으로 직접 오지는 않지만, Skip Nav 링크 클릭 시 포커스 가능 -->
</div>
```

**양수 `tabindex`를 지양해야 하는 이유**:
`tabindex="1"` 이상의 양수를 사용하면, 해당 요소가 **문서 전체에서 가장 먼저 Tab 포커스**를 받습니다. 여러 요소에 양수를 부여하면 DOM의 자연스러운 순서가 완전히 깨지고, 유지보수가 극도로 어려워집니다. 특히 여러 개발자가 각자 다른 값을 부여하면 예측 불가능한 Tab 순서가 됩니다. **항상 0 또는 -1만 사용**하고, Tab 순서는 DOM 순서로 제어하는 것이 원칙입니다.

</details>

### Q42. 복잡한 대시보드나 GNB(Global Navigation Bar)가 상단에 길게 있을 때, 키보드 사용자가 메인 콘텐츠로 바로 건너뛸 수 있도록 돕는 '블록 우회(Skip Navigation)' 기법을 구현해 본 적이 있나요?

* 의도: 접근성 가이드라인 2.4.1(블록 우회)을 실제 UI 설계에 반영해 본 경험 확인
* 키워드: `스킵 내비게이션(Skip to content)` `화면 최상단 앵커 링크`

<details>
<summary>답변</summary>

**Skip Navigation**은 키보드 사용자가 매 페이지마다 네비게이션 링크를 일일이 Tab해야 하는 번거로움을 해결합니다. 페이지 최상단에 **앵커 링크**를 배치하여, Tab 한 번으로 메인 콘텐츠로 바로 이동할 수 있게 합니다.

```html
<body>
  <!-- 평소에는 화면 밖에 숨겨져 있다가, 포커스 시 나타남 -->
  <a href="#main-content" class="skip-nav">
    본문으로 건너뛰기
  </a>

  <nav><!-- 수십 개의 네비게이션 링크 --></nav>

  <main id="main-content" tabindex="-1">
    <!-- 메인 콘텐츠 -->
  </main>
</body>
```

```css
.skip-nav {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: 9999;
  padding: 1rem;
  background: #000;
  color: #fff;
}

.skip-nav:focus {
  top: 0; /* 포커스 받으면 화면에 나타남 */
}
```

**동작 흐름**:
1. 사용자가 페이지에서 Tab을 누르면 첫 번째로 "본문으로 건너뛰기" 링크에 포커스됩니다.
2. Enter를 누르면 `#main-content`로 이동하여, 네비게이션을 건너뜁니다.
3. 건너뛰지 않으려면 Tab을 한 번 더 눌러 네비게이션으로 진입합니다.

`main` 요소에 `tabindex="-1"`을 설정하여, 앵커 이동 후 해당 영역이 포커스를 받을 수 있게 합니다. WCAG 2.4.1 "블록 우회" 기준을 충족하는 필수 기법입니다.

</details>

### Q43. 사용자가 회원가입 폼을 제출했는데 에러가 발생하여 빨간 글씨로 경고 메시지가 떴습니다. 화면을 볼 수 없는 사용자는 이 에러가 났다는 사실을 어떻게 인지할 수 있도록 코드를 작성해야 하나요?

* 의도: `aria-invalid`나 `aria-describedby`를 활용한 폼 검증(Form Validation) 접근성 처리 능력 평가
* 키워드: `aria-invalid` `aria-describedby` `포커스 이동` `명확한 에러 원인 제공`

<details>
<summary>답변</summary>

빨간 글씨만으로는 스크린 리더 사용자가 에러를 인지할 수 없습니다. 다음과 같이 접근성을 보장합니다:

```html
<label for="email">이메일</label>
<input
  id="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<span id="email-error" role="alert">
  유효한 이메일 주소를 입력해 주세요.
</span>
```

**핵심 속성들**:

1. **`aria-invalid="true"`**: 스크린 리더가 이 입력 필드에 "유효하지 않음(invalid)" 상태를 알려줍니다.

2. **`aria-describedby="email-error"`**: 입력 필드와 에러 메시지를 연결합니다. 스크린 리더가 입력 필드에 포커스하면 "이메일, 유효하지 않음, 유효한 이메일 주소를 입력해 주세요"라고 읽어줍니다.

3. **`role="alert"`**: 에러 메시지가 동적으로 나타날 때, 스크린 리더가 **현재 작업을 중단하고 즉시 이 메시지를 읽어줍니다** (내부적으로 `aria-live="assertive"` 동작).

4. **포커스 이동**: 폼 제출 실패 시 **첫 번째 에러 필드로 포커스를 이동**하여, 사용자가 즉시 수정할 수 있게 합니다:
```javascript
const firstErrorField = document.querySelector('[aria-invalid="true"]');
firstErrorField?.focus();
```

색상만으로 에러를 표현하지 않고, 아이콘이나 텍스트로도 에러 상태를 전달하는 것이 색각 이상 사용자를 위해 중요합니다.

</details>

### Q44. '남은 시간 10초' 알림이나 화면 우측 하단의 '토스트(Toast) 메시지'처럼, 포커스를 옮기지 않고도 화면의 동적인 변화를 스크린 리더가 즉시 읽어주게 하려면 어떤 WAI-ARIA 속성을 사용해야 할까요?

* 의도: SPA 환경에서 가장 많이 쓰이는 동적 영역 알림 속성(`aria-live`) 숙련도 검증
* 키워드: `aria-live="polite"(작업 완료 후 읽음)` `aria-live="assertive"(즉시 읽음)` `role="alert"`

<details>
<summary>답변</summary>

**`aria-live` 속성**은 해당 영역의 내용이 동적으로 변경될 때, 스크린 리더가 **포커스 이동 없이 자동으로 변경 사항을 읽어주도록** 지시합니다.

**`aria-live="polite"` (정중한 알림)**:
- 스크린 리더가 **현재 읽고 있는 내용을 마친 후** 변경 사항을 읽습니다.
- 적합: 토스트 메시지, 검색 결과 개수 업데이트, 채팅 메시지 도착

```html
<div aria-live="polite">
  <!-- JS로 내용이 변경되면 스크린 리더가 읽어줌 -->
  저장되었습니다.
</div>
```

**`aria-live="assertive"` (즉각적 알림)**:
- 스크린 리더가 **현재 작업을 중단하고 즉시** 변경 사항을 읽습니다.
- 적합: 에러 메시지, 세션 만료 경고, 긴급 알림

```html
<div aria-live="assertive">
  세션이 10초 후 만료됩니다.
</div>
```

**`role="alert"` (단축 속성)**:
`role="alert"`은 `aria-live="assertive"` + `aria-atomic="true"`의 단축형입니다:
```html
<div role="alert">오류가 발생했습니다.</div>
```

**주의**: `aria-live` 영역은 **페이지 로드 시 이미 DOM에 존재**해야 합니다. 동적으로 `aria-live` 요소 자체를 추가하면 일부 스크린 리더가 감지하지 못합니다. 빈 컨테이너를 미리 두고, 내용만 변경하는 방식이 안정적입니다.

</details>

### Q45. 화려한 인터랙션(예: 3D 전환, 빠른 깜빡임)이 들어간 페이지를 구현할 때, 시각적 피로도나 발작(가이드라인 2.3)을 예방하기 위해 CSS나 JS에서 어떤 설정을 감지하고 애니메이션을 분기 처리하시나요?

* 의도: 사용자의 디바이스 환경 설정(접근성 옵션)을 존중하는 미디어 쿼리 처리 경험 확인
* 키워드: `@media (prefers-reduced-motion: reduce)` `애니메이션 정지/제거`

<details>
<summary>답변</summary>

운영체제의 **"동작 줄이기(Reduce Motion)" 접근성 설정**을 CSS 미디어 쿼리로 감지하여 애니메이션을 비활성화합니다.

**CSS에서 감지 및 분기**:
```css
/* 기본: 애니메이션 적용 */
.hero-banner {
  animation: slideIn 0.5s ease-out;
  transition: transform 0.3s;
}

/* 사용자가 "동작 줄이기"를 켠 경우: 애니메이션 제거 */
@media (prefers-reduced-motion: reduce) {
  .hero-banner {
    animation: none;
    transition: none;
  }

  /* 또는 전체적으로 비활성화 */
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**JavaScript에서 감지**:
```javascript
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

if (prefersReducedMotion.matches) {
  // 3D 전환, 패럴랙스 등 복잡한 애니메이션 비활성화
}

// 실시간 변경 감지
prefersReducedMotion.addEventListener('change', (e) => {
  setReducedMotion(e.matches);
});
```

**React Hook 활용**:
```javascript
function usePrefersReducedMotion() {
  const [reduced, setReduced] = useState(
    window.matchMedia('(prefers-reduced-motion: reduce)').matches
  );
  useEffect(() => {
    const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
    const handler = (e) => setReduced(e.matches);
    mq.addEventListener('change', handler);
    return () => mq.removeEventListener('change', handler);
  }, []);
  return reduced;
}
```

WCAG 2.3 "발작 및 신체 반응"과 2.2.2 "일시 정지, 중지, 숨기기" 기준을 충족합니다. 초당 3회 이상 깜빡이는 콘텐츠는 발작을 유발할 수 있으므로 절대 지양해야 합니다.

</details>
