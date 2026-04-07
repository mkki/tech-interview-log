### Q1. 자바스크립트의 한계점을 타입스크립트가 어떻게 해결하는지, '컴파일 타임'과 '런타임'의 관점에서 설명해 주세요.

* 의도: 동적 vs 정적 타입, TS 도입 목적 이해
* 키워드: 컴파일 타임(사전 에러 검출), 런타임 에러 감소, 타입 소거(Type Erasure)

<details>
<summary>답변</summary>

JavaScript는 동적 타입 언어로 변수의 타입이 런타임에 결정되므로, 타입 관련 버그가 실제 코드 실행 시점에서야 발견됩니다. TypeScript는 정적 타입 시스템을 도입하여 **컴파일 타임에 타입 에러를 사전 검출**합니다. 예를 들어 숫자 변수에 문자열 메서드를 호출하는 실수를 코드 작성 시점에 잡아줍니다.

TS 코드는 JS로 트랜스파일될 때 모든 타입 정보가 제거(Type Erasure)되므로, **런타임 오버헤드가 전혀 없습니다**. 결과적으로 컴파일 타임에 안전성을 확보하면서 런타임 성능에는 영향을 주지 않는 구조입니다.

</details>

### Q2. 타입스크립트로 작성한 인터페이스(`interface`)나 타입(`type`) 구문들은 런타임(브라우저) 환경에서 어떻게 동작하나요?

* 의도: TS → JS 변환 과정 이해
* 키워드: 트랜스파일, 타입 소거, 런타임 오버헤드 없음

<details>
<summary>답변</summary>

**동작하지 않습니다.** interface, type, 제네릭 등 TS 고유 구문은 트랜스파일 과정에서 완전히 소거(Type Erasure)됩니다. 브라우저는 순수 JavaScript만 실행하므로, 타입 구문은 런타임에 존재하지 않습니다.

따라서 런타임 오버헤드가 없고, 타입 정보를 런타임에서 활용할 수도 없습니다. 런타임에 타입 검증이 필요하다면 typeof, instanceof 같은 JS 자체 연산자 또는 Zod 같은 런타임 검증 라이브러리를 별도로 사용해야 합니다.

</details>

### Q3. `tsconfig.json`의 `strict: true` 옵션이 구체적으로 어떤 에러들을 사전에 막아주나요?

* 의도: 실무 설정 경험 + 타입 안정성 이해
* 키워드: noImplicitAny, strictNullChecks, 엄격한 타입 검사

<details>
<summary>답변</summary>

`strict: true`는 다음 옵션들을 한꺼번에 활성화합니다:

- **noImplicitAny**: 타입을 추론할 수 없을 때 암시적 any를 금지 → 타입 누락 방지
- **strictNullChecks**: null/undefined를 모든 타입에 자동 포함하지 않음 → `Cannot read property of null` 런타임 에러 사전 차단
- **strictFunctionTypes**: 함수 매개변수 타입을 반변적(contravariant)으로 엄격 검사
- **strictBindCallApply**: bind, call, apply 호출 시 인자 타입 검사
- **strictPropertyInitialization**: 클래스 프로퍼티가 생성자에서 초기화되는지 검사
- **noImplicitThis**: this의 타입이 암시적 any가 되는 것을 금지
- **alwaysStrict**: 모든 파일에 `"use strict"` 자동 적용
- **useUnknownInCatchVariables**: catch 절 변수를 any 대신 unknown으로 처리

실무에서는 프로젝트 초기부터 `strict: true`로 시작하는 것이 권장됩니다.

</details>

### Q4. 외부 API 응답이나 알 수 없는 값을 다룰 때 `any`와 `unknown` 중 어떤 것을 사용해야 하며, 그 이유는 무엇인가요?

* 의도: 타입 안정성에 대한 태도 평가
* 키워드: unknown 우선, 타입 안전성, 조작 제한

<details>
<summary>답변</summary>

**unknown을 사용해야 합니다.**

- `any`: 모든 연산을 허용하여 타입 검사를 완전히 우회합니다. 프로퍼티 접근, 메서드 호출이 제한 없이 가능하므로 런타임 에러 위험이 높습니다.
- `unknown`: 타입이 확인되기 전까지 **어떤 조작도 허용하지 않습니다**. 반드시 타입 가드(typeof, instanceof 등)를 통해 타입을 좁힌 후에야 값을 사용할 수 있습니다.

unknown은 "타입 안전한 any"로, 개발자에게 검증을 강제하여 런타임 안전성을 보장합니다.

</details>

### Q5. `unknown` 타입 변수의 프로퍼티 접근/메서드 호출 시 필요한 사전 작업은 무엇인가요?

* 의도: 타입 좁히기 이해
* 키워드: 타입 가드, typeof, instanceof, narrowing

<details>
<summary>답변</summary>

**타입 좁히기(Narrowing)**가 필요합니다. unknown 변수는 타입이 확인되기 전까지 프로퍼티 접근이나 메서드 호출이 불가능합니다.

좁히기 방법:
- **typeof**: 기본 타입 확인 (`typeof value === 'string'`)
- **instanceof**: 클래스 인스턴스 확인 (`value instanceof Date`)
- **in 연산자**: 프로퍼티 존재 확인 (`'name' in value`)
- **사용자 정의 타입 가드**: `value is Type` 반환 타입으로 커스텀 검증 함수 작성

좁히기를 통과한 블록 안에서 TS 컴파일러가 자동으로 타입을 추론하여 안전한 접근이 가능해집니다.

</details>

### Q6. 함수 반환 타입 `void`와 `never`의 차이는 무엇인가요?

* 의도: TS 특수 타입 이해
* 키워드: void(정상 종료), never(도달 불가), throw, 무한 루프

<details>
<summary>답변</summary>

- **void**: 함수가 **정상적으로 종료**되지만 의미 있는 값을 반환하지 않음. 실제로는 undefined를 반환합니다.
- **never**: 함수의 **끝에 도달할 수 없음**. 항상 예외를 던지거나(`throw`), 무한 루프를 실행하는 함수에 해당합니다.

```typescript
const logMessage = (msg: string): void => {
  console.log(msg);
};

const throwError = (msg: string): never => {
  throw new Error(msg);
};
```

never는 모든 타입의 서브타입이므로 Exhaustiveness Check에도 활용됩니다.

</details>

### Q7. 유니온(`|`)과 인터섹션(`&`) 타입의 차이를 객체 호환성 관점에서 설명해주세요.

* 의도: 구조적 타이핑 이해
* 키워드: 합집합, 교집합, 프로퍼티 기반 타입

<details>
<summary>답변</summary>

- **유니온(A | B)**: A 또는 B 중 하나. **값의 범위는 합집합**이지만, 타입 좁히기 전에는 **공통 프로퍼티만** 안전하게 접근 가능합니다.
- **인터섹션(A & B)**: A이면서 동시에 B. **값의 범위는 교집합**이지만, **양쪽의 모든 프로퍼티**에 접근 가능합니다.

```typescript
type A = { name: string };
type B = { age: number };

type Union = A | B;        // 공통 프로퍼티가 없으므로 타입 좁히기 전에는 어떤 프로퍼티도 접근 불가
type Intersection = A & B; // name과 age 모두 접근 가능
```

직관과 반대로, 객체 관점에서 유니온은 "접근 가능한 프로퍼티가 줄어들고", 인터섹션은 "접근 가능한 프로퍼티가 늘어납니다".

</details>

### Q8. 객체 A(name)와 B(name, age)가 있을 때 B를 A에 할당할 수 있나요?

* 의도: 서브타입 개념 이해
* 키워드: 구조적 타이핑, 포함 관계, 업캐스팅

<details>
<summary>답변</summary>

**가능합니다.** TypeScript는 구조적 타이핑(Structural Typing)을 사용하므로, B는 A가 요구하는 `name` 프로퍼티를 이미 포함하고 있어 A의 서브타입으로 인정됩니다. 추가 프로퍼티(`age`)가 있어도 할당 가능하며, 이를 업캐스팅이라 합니다.

```typescript
type A = { name: string };
type B = { name: string; age: number };

const b: B = { name: '홍길동', age: 30 };
const a: A = b; // OK — B는 A의 서브타입
```

**단, 객체 리터럴을 직접 할당할 때는 잉여 프로퍼티 검사(Excess Property Check)가 적용되어** `const a: A = { name: '홍길동', age: 30 }`은 에러가 발생합니다. 이는 오타 방지를 위한 추가 안전장치입니다.

</details>

### Q9. 타입 선언(`: Type`)과 타입 단언(`as Type`)의 차이와 단언을 피해야 하는 이유는?

* 의도: 타입 안전성 태도 평가
* 키워드: 컴파일 검사 vs 무시, 개발자 책임, 런타임 에러 위험

<details>
<summary>답변</summary>

- **타입 선언(`: Type`)**: 컴파일러가 할당되는 값의 타입 호환성을 **검사**합니다. 불일치 시 에러를 발생시킵니다.
- **타입 단언(`as Type`)**: 개발자가 "이 값은 이 타입이다"라고 **컴파일러에게 강제**합니다. 타입 검사를 우회합니다.

```typescript
const user: User = { name: '홍길동' };         // 선언 — 누락된 프로퍼티가 있으면 에러
const user = { name: '홍길동' } as User;       // 단언 — 누락되어도 에러 없음
```

단언은 컴파일러의 안전장치를 무시하므로, 실제 런타임 값과 단언한 타입이 불일치하면 런타임 에러로 이어집니다. 가능하면 선언을 사용하고, 단언은 DOM 접근(`document.getElementById as HTMLInputElement`) 등 컴파일러가 추론할 수 없는 제한적 상황에서만 사용해야 합니다.

</details>

### Q10. 서로소 유니온 타입을 활용하면 어떤 장점이 있나요?

* 의도: 상태 모델링 능력 평가
* 키워드: Discriminated Union, 리터럴 타입, 상태 분리

<details>
<summary>답변</summary>

서로소 유니온(Discriminated Union)은 **공통 리터럴 프로퍼티(판별자)**를 통해 각 케이스를 명확히 구분합니다.

```typescript
type Success = { status: 'success'; data: string };
type Error = { status: 'error'; message: string };
type Result = Success | Error;

const handleResult = (result: Result) => {
  if (result.status === 'success') {
    console.log(result.data);    // data에 안전 접근
  } else {
    console.log(result.message); // message에 안전 접근
  }
};
```

장점:
- switch/if에서 판별자로 분기 시 **자동 타입 좁히기**
- 각 상태에 필요한 프로퍼티만 정확히 정의 → **잘못된 상태 조합 불가**
- 새 케이스 추가 시 Exhaustiveness Check로 **미처리 누락 방지**

</details>

### Q11. 선택적 프로퍼티 vs 서로소 유니온, 타입 안정성 차이는?

* 의도: 잘못된 상태 방지 설계 능력
* 키워드: invalid state 방지, 명확한 상태 분리

<details>
<summary>답변</summary>

**선택적 프로퍼티**는 유효하지 않은 상태 조합을 허용합니다:

```typescript
// 잘못된 설계 — status가 'error'인데 data가 존재하는 상태가 가능
type Result = {
  status: 'success' | 'error';
  data?: string;
  message?: string;
};
```

**서로소 유니온**은 각 상태에 필요한 프로퍼티만 정확히 정의합니다:

```typescript
// 올바른 설계 — 불가능한 상태가 타입 레벨에서 차단됨
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };
```

서로소 유니온은 "Make impossible states impossible" 원칙을 구현하여, 런타임에서 발생할 수 있는 잘못된 상태를 컴파일 타임에 원천 차단합니다.

</details>

### Q12. switch + never를 활용한 Exhaustiveness Check 경험이 있나요?

* 의도: 유지보수 안전성 평가
* 키워드: exhaustive check, default never

<details>
<summary>답변</summary>

switch문의 default에서 값을 `never` 타입에 할당하면, 모든 케이스를 처리하지 않았을 때 **컴파일 에러**가 발생합니다.

```typescript
type Shape = 'circle' | 'square' | 'triangle';

const getArea = (shape: Shape) => {
  switch (shape) {
    case 'circle': return '원';
    case 'square': return '사각형';
    // 'triangle' 처리 누락
    default: {
      const _exhaustive: never = shape;
      // Error: Type 'triangle' is not assignable to type 'never'
      throw new Error(`Unhandled shape: ${_exhaustive}`);
    }
  }
};
```

유니온에 새 멤버를 추가했을 때, 관련 switch문에서 처리를 빠뜨린 곳을 컴파일 타임에 즉시 발견할 수 있어 **유지보수 안전성**이 크게 향상됩니다.

</details>

### Q13. 사용자 정의 타입 가드란 무엇이며 언제 사용하나요?

* 의도: 런타임 데이터 검증 능력
* 키워드: is 키워드, type predicate, 커스텀 가드

<details>
<summary>답변</summary>

`value is Type` 형태의 반환 타입(type predicate)을 가진 함수로, **런타임 검증 로직을 캡슐화하여 TS 컴파일러에 타입 좁히기 정보를 전달**합니다.

```typescript
interface User {
  name: string;
  email: string;
}

const isUser = (value: unknown): value is User => {
  return (
    typeof value === 'object' &&
    value !== null &&
    'name' in value &&
    'email' in value
  );
};

const handleData = (data: unknown) => {
  if (isUser(data)) {
    console.log(data.name); // User로 좁혀져 안전 접근
  }
};
```

typeof/instanceof로 판별할 수 없는 **복잡한 커스텀 타입**(API 응답 객체, 유니온 멤버)을 검증할 때 사용합니다.

</details>

### Q14. boolean 반환 함수는 타입 가드로 동작하나요?

* 의도: 타입 가드 동작 원리 이해
* 키워드: 타입 좁히기 불가, 컨텍스트 전달 실패

<details>
<summary>답변</summary>

**기본적으로 동작하지 않습니다.** 반환 타입을 `boolean`으로 명시하면 타입 좁히기가 수행되지 않습니다.

```typescript
// 반환 타입을 boolean으로 명시 — 타입 좁히기 안 됨
const isString = (value: unknown): boolean => typeof value === 'string';

const example = (value: unknown) => {
  if (isString(value)) {
    value.toUpperCase(); // Error: 여전히 unknown
  }
};
```

함수 내부의 검증 컨텍스트가 호출부로 **전파되지 않기 때문**입니다. `is` 키워드를 명시해야 컴파일러가 해당 블록에서 타입을 좁힐 수 있습니다:

```typescript
// type predicate — 타입 좁히기 됨
const isString = (value: unknown): value is string => typeof value === 'string';
```

**TS 5.5+ 보충:** TypeScript 5.5부터 **Inferred Type Predicates** 기능이 추가되어, 반환 타입을 명시하지 않으면 컴파일러가 자동으로 타입 가드를 추론합니다. 단, 명시적 반환 타입이 없고 매개변수가 변이되지 않는 등의 조건을 만족해야 합니다.

</details>

### Q15. 외부 API JSON을 런타임에서 검증하려면 어떻게 하나요?

* 의도: 실무 데이터 검증 능력
* 키워드: in 연산자, hasOwnProperty, unknown narrowing

<details>
<summary>답변</summary>

TS 타입은 런타임에 소거되므로 **별도의 런타임 검증**이 필요합니다.

**수동 검증 방법:**
- `typeof`: 기본 타입 확인
- `in` 연산자: 프로퍼티 존재 여부 확인
- 사용자 정의 타입 가드: 위 조건을 조합하여 검증 함수 작성

```typescript
const isApiResponse = (data: unknown): data is ApiResponse => {
  return (
    typeof data === 'object' &&
    data !== null &&
    'status' in data &&
    'payload' in data
  );
};
```

**라이브러리 활용:** Zod, io-ts 같은 스키마 검증 라이브러리를 사용하면, 선언적 스키마 정의 한 번으로 **런타임 검증과 TS 타입 추론을 동시에** 수행할 수 있어 실무에서 권장됩니다.

</details>

### Q16. `type` vs `interface` 차이와 사용 기준은?

* 의도: 설계 판단 능력
* 키워드: 선언 병합, extends vs &, 유니온 가능 여부

<details>
<summary>답변</summary>

| 특성 | interface | type |
|---|---|---|
| 선언 병합 | 가능 (같은 이름 여러 번 선언 시 자동 합침) | 불가 |
| 확장 방식 | `extends` | `&` (인터섹션) |
| 유니온/튜플 | 불가 | 가능 |
| 대상 | 객체 형태만 | 모든 타입 표현 가능 |

**사용 기준:**
- **객체 형태의 공개 API/계약** 정의: `interface` — 선언 병합으로 확장성 확보
- **유니온, 튜플, 리터럴, 복합 타입**: `type` — interface로 표현 불가능
- **프로젝트 내 일관성** 유지가 가장 중요하므로, 팀 컨벤션을 따르는 것이 우선

</details>

### Q17. 선언 병합(Declaration Merging)이란 무엇이며 왜 유용한가요?

* 의도: 타입 확장 경험
* 키워드: 타입 보강, 글로벌 확장, OCP

<details>
<summary>답변</summary>

같은 이름의 interface를 **여러 번 선언하면 자동으로 하나로 합쳐지는** 기능입니다.

```typescript
interface Window {
  myCustomProperty: string;
}
// 기존 Window 타입에 myCustomProperty가 추가됨
```

**활용 사례:**
- **타입 보강(Module Augmentation)**: 외부 라이브러리 타입에 프로퍼티 추가 (예: Express의 Request에 user 필드 추가)
- **글로벌 타입 확장**: Window, ProcessEnv 등에 커스텀 프로퍼티 추가

기존 코드를 수정하지 않고 확장하므로 **OCP(개방-폐쇄 원칙)**를 따릅니다. type은 선언 병합이 불가능하므로, 이런 확장이 필요할 때 interface를 사용합니다.

</details>

### Q18. 튜플/유니온 타입 정의 시 type vs interface 선택 기준은?

* 의도: 타입 도구 선택 능력
* 키워드: type만 가능, 대수 타입

<details>
<summary>답변</summary>

**type을 사용해야 합니다.** 튜플과 유니온은 interface로 표현할 수 없습니다.

```typescript
// 튜플 — type만 가능
type Coordinate = [number, number];

// 유니온 — type만 가능
type Result = Success | Error;

// 리터럴 유니온 — type만 가능
type Direction = 'up' | 'down' | 'left' | 'right';
```

이러한 **대수적 타입(Algebraic Types)**은 type의 고유 영역입니다. interface는 객체 형태(Object Shape)만 표현할 수 있으므로, 비객체 타입이나 여러 타입의 합성이 필요할 때는 type이 유일한 선택입니다.

</details>

### Q19. 클래스의 접근 제어자(public/private/protected) 장점과 활용 사례는?

* 의도: OOP 이해
* 키워드: 캡슐화, 정보 은닉, 로직 분리

<details>
<summary>답변</summary>

- **public**: 어디서든 접근 가능 (기본값)
- **private**: 해당 클래스 내부에서만 접근 가능
- **protected**: 클래스 내부 + 자식 클래스에서 접근 가능

```typescript
class UserService {
  public name: string;
  private apiKey: string;
  protected baseUrl: string;

  constructor(name: string, apiKey: string) {
    this.name = name;
    this.apiKey = apiKey;
    this.baseUrl = '/api';
  }

  private buildHeaders() {
    return { Authorization: this.apiKey };
  }

  public fetchUser() {
    return fetch(this.baseUrl, { headers: this.buildHeaders() });
  }
}
```

**장점:** 캡슐화를 통해 내부 구현(apiKey, buildHeaders)을 숨기고, 외부에는 공개 API(fetchUser)만 노출합니다. 의도치 않은 외부 접근을 방지하고 변경 영향 범위를 제한합니다.

</details>

### Q20. JS `#private` vs TS `private` 차이는?

* 의도: 런타임 vs 컴파일 차이 이해
* 키워드: 런타임 은닉, 컴파일 체크, 타입 소거

<details>
<summary>답변</summary>

- **JS `#private`**: ECMAScript 표준으로, **런타임에서 실제로 은닉**됩니다. 외부에서 `obj.#field` 구문으로 접근하면 SyntaxError가 발생하고, 대괄호 표기법(`obj['#field']`)은 별개의 프로퍼티를 참조하므로 실제 값에 도달할 수 없습니다.
- **TS `private`**: 컴파일 타임에만 접근을 제한합니다. 타입 소거 후 생성된 JS에서는 **일반 프로퍼티로 노출되어 런타임에서 접근 가능**합니다.

```typescript
class Example {
  #jsPrivate = 'JS 런타임 은닉';    // 런타임에서도 접근 불가
  private tsPrivate = 'TS 컴파일 타임 제한'; // JS로 변환 후 접근 가능
}
```

보안이 중요한 데이터는 `#private`을, 일반적인 캡슐화 목적이면 TS `private`을 사용합니다.

</details>

### Q21. React에서 비즈니스 로직을 클래스로 분리했을 때 장점은?

* 의도: 설계 능력 평가
* 키워드: SoC, 테스트 용이성, 결합도 감소

<details>
<summary>답변</summary>

**관심사의 분리(Separation of Concerns)**로 UI 로직과 비즈니스 로직을 독립시킵니다.

장점:
- **테스트 용이성**: 비즈니스 로직 클래스는 React에 의존하지 않으므로, DOM이나 렌더링 없이 순수 단위 테스트가 가능합니다.
- **결합도 감소**: 로직 변경이 컴포넌트에 직접 영향을 주지 않고, 반대로 UI 변경이 로직에 영향을 주지 않습니다.
- **재사용성**: 같은 비즈니스 로직을 여러 컴포넌트에서 공유할 수 있습니다.

컴포넌트는 UI 렌더링에 집중하고, 복잡한 계산/검증/변환 로직은 별도 클래스에 위임하는 구조가 유지보수에 유리합니다.

</details>

### Q22. 클래스가 인터페이스를 implements 하는 이유는?

* 의도: 설계 원칙 이해
* 키워드: 계약, 다형성, 의존성 역전

<details>
<summary>답변</summary>

인터페이스는 클래스가 반드시 구현해야 할 **계약(Contract)**을 정의합니다.

```typescript
interface DataFetcher {
  fetch(url: string): Promise<unknown>;
}

class AxiosFetcher implements DataFetcher {
  async fetch(url: string) { /* axios 구현 */ }
}

class FetchApiFetcher implements DataFetcher {
  async fetch(url: string) { /* fetch API 구현 */ }
}
```

장점:
- **다형성**: 같은 인터페이스를 구현한 다른 클래스로 교체 가능
- **의존성 역전 원칙(DIP)**: 구체 클래스가 아닌 인터페이스(추상화)에 의존하여 결합도를 낮춤
- **구현 강제**: 필수 메서드를 누락하면 컴파일 에러 → 안전한 계약 이행

</details>

### Q23. API 클래스를 인터페이스에 의존하면 어떤 장점이 있나요?

* 의도: 테스트/유지보수 설계
* 키워드: Mocking, 결합도 감소

<details>
<summary>답변</summary>

- **Mocking 용이**: 테스트 시 실제 API 호출 없이 Mock 구현체로 교체 가능합니다. 인터페이스만 만족하면 어떤 객체든 주입할 수 있습니다.
- **결합도 감소**: 실제 HTTP 라이브러리(fetch → axios 등) 변경 시, 인터페이스만 구현하면 되므로 사용하는 코드는 변경할 필요 없습니다.
- **병렬 개발**: 인터페이스만 합의하면 API 구현과 UI 개발을 동시에 진행할 수 있습니다.

```typescript
// 테스트에서 Mock으로 교체
class MockFetcher implements DataFetcher {
  async fetch(url: string) {
    return { data: 'mock' };
  }
}
```

</details>

### Q24. 여러 인터페이스 구현이 가능한가요? 다중 상속 한계 보완은?

* 의도: 설계 유연성 이해
* 키워드: 다중 구현, ISP, 역할 조합

<details>
<summary>답변</summary>

**가능합니다.** TypeScript에서 클래스는 여러 인터페이스를 implements할 수 있습니다. 클래스 다중 상속(extends)은 불가하지만, 여러 인터페이스 구현으로 이를 보완합니다.

```typescript
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

class UserService implements Serializable, Loggable {
  serialize() { return JSON.stringify(this); }
  log() { console.log('UserService'); }
}
```

**ISP(인터페이스 분리 원칙)**에 따라 역할별 작은 인터페이스를 설계하고, 필요한 역할만 조합하여 구현하는 것이 유연한 설계입니다.

</details>

### Q25. 제네릭을 사용하는 이유와 `extends` 제약의 목적은?

* 의도: 제네릭 본질 이해
* 키워드: 재사용성, 타입 제약

<details>
<summary>답변</summary>

**제네릭은 타입을 매개변수화**하여 다양한 타입에서 재사용 가능한 코드를 작성합니다. any처럼 타입을 포기하지 않으면서 유연성을 확보합니다.

**extends 제약**은 제네릭이 특정 구조를 가져야 한다는 조건을 부여하여, 안전한 프로퍼티 접근을 보장합니다.

```typescript
// extends 없이 — T의 프로퍼티 접근 불가
const getId = <T>(item: T) => item.id; // Error

// extends로 제약 — id 프로퍼티 접근 보장
const getId = <T extends { id: number }>(item: T) => item.id; // OK
```

제네릭은 "어떤 타입이든"이 아니라 "조건을 만족하는 어떤 타입이든" 처리할 수 있게 합니다.

</details>

### Q26. `<T extends { id: number }>`에서 id 없는 객체 전달 시?

* 의도: 제네릭 제약 이해
* 키워드: 컴파일 에러, 구조적 타이핑

<details>
<summary>답변</summary>

**컴파일 에러가 발생합니다.** `T extends { id: number }` 제약에 의해 전달되는 객체는 최소한 `{ id: number }`를 포함해야 합니다.

```typescript
const getById = <T extends { id: number }>(item: T): number => item.id;

getById({ id: 1, name: '홍길동' }); // OK — id: number 포함
getById({ name: '홍길동' });         // Error — id 프로퍼티 없음
getById({ id: '문자열' });            // Error — id가 number가 아님
```

구조적 타이핑에 의해 `{ id: number }` 이외의 추가 프로퍼티가 있어도 통과하지만, id가 없거나 타입이 다르면 제약 조건을 충족하지 못해 컴파일 에러입니다.

</details>

### Q27. 제네릭을 피하고 단순 타입을 써야 하는 경우는?

* 의도: 오버엔지니어링 판단
* 키워드: 가독성, 단순성, YAGNI

<details>
<summary>답변</summary>

다음 경우에는 제네릭 대신 단순 타입이 적절합니다:

1. **타입 매개변수가 한 곳에서만 사용되는 경우**: `<T>(x: T): void`에서 T가 반환 타입이나 다른 매개변수에 사용되지 않으면, `(x: unknown): void`와 실질적으로 같은 효과이므로 제네릭이 불필요합니다.
2. **구체적 타입이 명확한 경우**: 항상 User를 처리하는 함수에 `<T>`를 넣는 것은 오버엔지니어링입니다.
3. **가독성이 크게 저하되는 경우**: 중첩 제네릭이 과도하면 이해하기 어렵습니다.

**YAGNI(You Aren't Gonna Need It)** 원칙에 따라 실제로 여러 타입에서 재사용할 필요가 생길 때 제네릭으로 리팩토링하는 것이 낫습니다. 추상화를 미리 예측하여 적용하지 않습니다.

</details>

### Q28. Promise + 제네릭으로 API 응답 타입 설계 방법은?

* 의도: 실무 API 설계 능력
* 키워드: Promise<T>, 공통 응답 구조

<details>
<summary>답변</summary>

공통 응답 구조를 제네릭으로 정의하고, `Promise<T>`로 비동기 반환 타입을 명시합니다.

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
}

const fetchUser = async (id: number): Promise<ApiResponse<User>> => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// 사용 시 자동 타입 추론
const result = await fetchUser(1);
console.log(result.data.name); // string으로 추론됨
```

공통 응답 래퍼(`ApiResponse<T>`)를 한 번 정의하면 모든 API에서 재사용할 수 있어 DRY 원칙을 지킵니다.

</details>

### Q29. 중첩 제네릭(Pagination 등) 사용 경험은?

* 의도: 복잡한 타입 설계
* 키워드: PaginationResponse<T>, nested generic

<details>
<summary>답변</summary>

제네릭을 중첩하여 페이지네이션 + 공통 응답 구조를 조합할 수 있습니다.

```typescript
interface PaginatedData<T> {
  items: T[];
  totalCount: number;
  page: number;
  pageSize: number;
  hasNext: boolean;
}

interface ApiResponse<T> {
  data: T;
  status: number;
}

// 중첩 제네릭 조합
type UserListResponse = ApiResponse<PaginatedData<User>>;

const fetchUsers = async (page: number): Promise<UserListResponse> => {
  const response = await fetch(`/api/users?page=${page}`);
  return response.json();
};

// result.data.items[0].name — 최종 프로퍼티까지 타입 추론
```

각 레이어(응답 래퍼, 페이지네이션, 엔티티)를 독립적 제네릭으로 정의하면 조합이 자유로워집니다.

</details>

### Q30. API 에러 타입은 어떻게 처리하나요?

* 의도: 에러 핸들링 능력
* 키워드: unknown, 커스텀 에러, AxiosError

<details>
<summary>답변</summary>

catch 블록의 에러는 `unknown`으로 처리하고, **타입 가드로 분기**합니다.

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string
  ) {
    super(message);
  }
}

const fetchData = async () => {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new ApiError(response.status, '요청 실패');
    }
    return response.json();
  } catch (error: unknown) {
    if (error instanceof ApiError) {
      console.error(`API 에러 ${error.statusCode}: ${error.message}`);
    } else if (error instanceof Error) {
      console.error(`일반 에러: ${error.message}`);
    } else {
      console.error('알 수 없는 에러');
    }
  }
};
```

Axios 사용 시 `isAxiosError()` 타입 가드를 활용하며, 커스텀 에러 클래스를 정의하여 에러 종류별 처리를 구조화합니다.

</details>

### Q31. map/filter가 제네릭인 이유는?

* 의도: 타입 추론 이해
* 키워드: 동적 반환 타입, 콜백 추론

<details>
<summary>답변</summary>

입력 배열의 요소 타입에 따라 **콜백 매개변수 타입과 반환 타입이 동적으로 결정**되어야 하기 때문입니다.

```typescript
const numbers = [1, 2, 3];                   // number[]
const strings = numbers.map(n => String(n));  // string[] — 반환 타입 자동 추론

const users: User[] = [/* ... */];
const names = users.map(u => u.name);         // string[] — User의 name 프로퍼티 타입 추론
```

제네릭이 없다면 `map(callback: (item: any) => any): any[]` 형태가 되어 타입 안전성이 사라지거나, 모든 타입 조합에 대한 오버로드가 필요합니다. 제네릭 덕분에 **호출 시점에 타입이 자동 추론**됩니다.

</details>

### Q32. map을 직접 구현하면 제네릭을 어떻게 설계하나요?

* 의도: 제네릭 함수 구현 능력
* 키워드: <T, U>, 콜백 반환 타입

<details>
<summary>답변</summary>

입력 요소 타입 T와 출력 요소 타입 U, 두 개의 타입 매개변수로 설계합니다.

```typescript
const myMap = <T, U>(
  arr: T[],
  callback: (item: T, index: number) => U
): U[] => {
  const result: U[] = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i], i));
  }
  return result;
};

// 사용 — T=number, U=string으로 자동 추론
const result = myMap([1, 2, 3], (n) => String(n)); // string[]
```

T는 입력 배열 요소 타입, U는 콜백 반환 타입으로, **입출력 타입이 독립적으로 추론**됩니다. 콜백의 반환 타입에서 U가 결정되므로 명시적 타입 인자 없이도 추론이 가능합니다.

</details>

### Q33. TSX에서 제네릭 화살표 함수 에러 해결 방법은?

* 의도: 실무 트러블슈팅 경험
* 키워드: <T,>, extends unknown, function 사용

<details>
<summary>답변</summary>

TSX 파일에서 `<T>`가 **JSX 태그 열기로 파싱**되어 구문 에러가 발생합니다.

```tsx
// Error — <T>가 JSX 태그로 해석됨
const identity = <T>(value: T): T => value;
```

**해결 방법 3가지:**

1. **트레일링 콤마**: JSX로 파싱되지 않도록 힌트 제공
```tsx
const identity = <T,>(value: T): T => value;
```

2. **extends 제약 추가**: 명확한 제네릭 문법으로 인식
```tsx
const identity = <T extends unknown>(value: T): T => value;
```

3. **function 키워드**: 화살표 함수 대신 사용
```tsx
function identity<T>(value: T): T { return value; }
```

실무에서는 1번(트레일링 콤마) 방식이 가장 간결하여 널리 사용됩니다.

</details>

### Q34. 유틸리티 타입(Pick, Omit 등) 활용 경험/원리는?

* 의도: DRY 적용 능력
* 키워드: mapped type, keyof

<details>
<summary>답변</summary>

유틸리티 타입은 내부적으로 **Mapped Type과 keyof**를 기반으로 동작합니다.

주요 유틸리티 타입:
- **Pick<T, K>**: T에서 K 키만 추출
- **Omit<T, K>**: T에서 K 키를 제외
- **Partial<T>**: 모든 프로퍼티를 선택적으로
- **Required<T>**: 모든 프로퍼티를 필수로
- **Readonly<T>**: 모든 프로퍼티를 읽기 전용으로

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserProfile = Pick<User, 'id' | 'name' | 'email'>;
type CreateUserInput = Omit<User, 'id'>;
type UpdateUserInput = Partial<Omit<User, 'id'>>;
```

기존 타입에서 파생 타입을 만들어 **중복 정의를 방지(DRY)**하고, 원본 타입 변경 시 파생 타입도 자동 반영됩니다.

</details>

### Q35. Readonly<T>를 직접 구현한다면?

* 의도: 맵드 타입 이해
* 키워드: [P in keyof T], readonly

<details>
<summary>답변</summary>

Mapped Type으로 T의 모든 키를 순회하며 `readonly` 수정자를 추가합니다.

```typescript
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

동작 원리:
- `keyof T`: T의 모든 키를 유니온으로 추출
- `P in keyof T`: 각 키를 P로 순회
- `T[P]`: 각 키의 원래 값 타입을 유지
- `readonly`: 각 프로퍼티에 읽기 전용 수정자 추가

```typescript
interface User { name: string; age: number; }
type ReadonlyUser = MyReadonly<User>;
// { readonly name: string; readonly age: number; }
```

</details>

### Q36. Record vs 인덱스 시그니처 차이는?

* 의도: 타입 안정성 이해
* 키워드: 제한된 키, literal union

<details>
<summary>답변</summary>

- **Record<K, V>**: K가 리터럴 유니온이면 **정해진 키만 허용**. 타입 안전성 높음.
- **인덱스 시그니처 `[key: string]: V`**: **모든 문자열 키 허용**. 유연하지만 안전성 낮음.

```typescript
// Record — 'active' | 'inactive' 키만 허용
type Status = 'active' | 'inactive';
type StatusLabels = Record<Status, string>;

const labels: StatusLabels = {
  active: '활성',
  inactive: '비활성',
  // pending: '대기' → Error: 존재하지 않는 키
};

// 인덱스 시그니처 — 아무 문자열 키나 허용
type FlexibleMap = { [key: string]: string };
const map: FlexibleMap = { anything: 'ok', other: 'fine' };
```

키 집합이 명확하면 Record, 동적인 키가 필요하면 인덱스 시그니처를 사용합니다. Record는 **키 누락도 컴파일 에러**로 잡아줍니다.

</details>

### Q37. 조건부 타입에서 infer 역할은?

* 의도: 고급 타입 추론 이해
* 키워드: 타입 추출, infer

<details>
<summary>답변</summary>

`infer`는 조건부 타입의 extends 절에서 **타입을 추론하여 변수처럼 캡처**합니다.

```typescript
// Promise 내부 타입 추출
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnpackPromise<Promise<string>>; // string
type B = UnpackPromise<number>;          // number

// 배열 요소 타입 추출
type ElementType<T> = T extends (infer E)[] ? E : T;

type C = ElementType<string[]>; // string
type D = ElementType<number>;   // number
```

infer는 "이 위치의 타입이 뭔지 추론해서 U에 담아라"라는 의미로, 복잡한 타입에서 **내부 타입을 동적으로 추출**할 때 핵심적으로 사용됩니다.

</details>

### Q38. ReturnType의 infer 동작 원리는?

* 의도: 내부 구현 이해
* 키워드: (...args) => infer R

<details>
<summary>답변</summary>

```typescript
type ReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : any;
```

동작 과정:
1. `T extends (...args: any) => any`: T를 함수 타입으로 제약
2. `T extends (...args: any) => infer R`: T가 함수 타입이면 **반환 타입 위치**를 R로 추론
3. `? R : any`: 추론 성공 시 R 반환, 실패 시 any

```typescript
const greet = (name: string): string => `Hello, ${name}`;
type GreetReturn = ReturnType<typeof greet>; // string

const fetchUser = async (): Promise<User> => { /* ... */ };
type FetchReturn = ReturnType<typeof fetchUser>; // Promise<User>
```

같은 패턴으로 Parameters<T>는 매개변수 위치에 infer를 적용합니다.

</details>

### Q39. 분산 조건부 타입이란?

* 의도: 유니온 + 조건부 이해
* 키워드: 유니온 분산, Exclude 원리

<details>
<summary>답변</summary>

유니온 타입이 **제네릭 조건부 타입의 타입 매개변수로 전달**되면, 각 멤버에 대해 조건이 **개별 적용된 후 결과가 유니온으로 합쳐**집니다.

```typescript
type Exclude<T, U> = T extends U ? never : T;

// Exclude<'a' | 'b' | 'c', 'a'>의 분산 과정:
// = ('a' extends 'a' ? never : 'a')   → never
//   | ('b' extends 'a' ? never : 'b') → 'b'
//   | ('c' extends 'a' ? never : 'c') → 'c'
// = never | 'b' | 'c'
// = 'b' | 'c'
```

분산은 **제네릭 타입 매개변수가 naked(래핑되지 않은) 상태**일 때만 발생합니다. `[T] extends [U]`처럼 배열로 감싸면 분산을 방지할 수 있습니다.

</details>

### Q40. keyof + 인덱스드 액세스 활용 방법은?

* 의도: 동적 타입 추출 능력
* 키워드: keyof, T[K]

<details>
<summary>답변</summary>

`keyof T`로 객체 키를 유니온으로 추출하고, `T[K]`(인덱스드 액세스 타입)로 해당 키의 값 타입에 접근합니다.

```typescript
interface User {
  id: number;
  name: string;
  isActive: boolean;
}

type UserKeys = keyof User; // 'id' | 'name' | 'isActive'
type NameType = User['name']; // string

// 타입 안전한 프로퍼티 접근 함수
const getValue = <T, K extends keyof T>(obj: T, key: K): T[K] => {
  return obj[key];
};

const user: User = { id: 1, name: '홍길동', isActive: true };
const name = getValue(user, 'name');      // string으로 추론
const active = getValue(user, 'isActive'); // boolean으로 추론
// getValue(user, 'invalid');              // Error: 존재하지 않는 키
```

키와 값 타입의 관계를 정확하게 유지하여 타입 안전한 동적 접근이 가능합니다.

</details>

### Q41. 객체 키를 유니온 타입으로 추출하는 방법은?

* 의도: value → type 변환 능력
* 키워드: typeof, keyof, as const

<details>
<summary>답변</summary>

`as const` + `typeof` + `keyof`를 조합합니다.

```typescript
const STATUS = {
  ACTIVE: 'active',
  INACTIVE: 'inactive',
  PENDING: 'pending',
} as const;

// 키 추출
type StatusKeys = keyof typeof STATUS;
// 'ACTIVE' | 'INACTIVE' | 'PENDING'

// 값 추출
type StatusValues = (typeof STATUS)[keyof typeof STATUS];
// 'active' | 'inactive' | 'pending'
```

핵심 단계:
1. `as const`: 객체의 값을 **리터럴 타입으로 고정** (readonly + literal narrowing)
2. `typeof STATUS`: 런타임 값을 **타입으로 변환**
3. `keyof`: 타입에서 **키를 유니온으로 추출**

as const 없이는 값이 `string`으로 넓혀져서 `'active'` 같은 리터럴 타입을 얻을 수 없습니다.

</details>

### Q42. 템플릿 리터럴 타입으로 키 리매핑 가능한가요?

* 의도: 최신 TS 기능 이해
* 키워드: Key Remapping, as, Capitalize

<details>
<summary>답변</summary>

**가능합니다.** TS 4.1+에서 Mapped Type의 `as` 절로 키를 리매핑할 수 있습니다.

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  name: string;
  age: number;
}

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }
```

**내장 문자열 유틸리티 타입:**
- `Capitalize<S>`: 첫 글자 대문자 (`'hello'` → `'Hello'`)
- `Uncapitalize<S>`: 첫 글자 소문자
- `Uppercase<S>`: 전체 대문자
- `Lowercase<S>`: 전체 소문자

```typescript
// 이벤트 핸들러 타입 자동 생성
type EventHandlers<T> = {
  [K in keyof T as `on${Capitalize<string & K>}Change`]: (value: T[K]) => void;
};
// { onNameChange: (value: string) => void; onAgeChange: (value: number) => void }
```

</details>
