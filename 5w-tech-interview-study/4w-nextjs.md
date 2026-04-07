### Q1. React의 기본 렌더링 방식인 CSR(Client-Side Rendering)이 가지는 한계점(특히 SEO와 초기 로딩 속도 측면)을 설명하고, Next.js의 사전 렌더링(Pre-rendering)이 이를 어떻게 해결하는지 설명해 주세요.

* 의도: React와 Next.js의 근본적인 차이를 이해하고, 비즈니스 요구사항(검색 노출, 빠른 첫 화면)에 맞춰 프레임워크를 도입할 수 있는 근거를 아는지 평가
* 키워드: `빈 HTML(<div id="root">)` `자바스크립트 번들 다운로드` `크롤러 봇` `TTV(Time To View) 개선`

<details>
<summary>답변</summary>

React의 CSR은 서버가 `<div id="root"></div>`만 담긴 빈 HTML을 보내고, 브라우저가 자바스크립트 번들을 모두 다운로드하고 실행한 뒤에야 화면을 그립니다. 이로 인해 두 가지 한계가 있습니다:

1. **SEO 문제**: 검색 엔진 크롤러 봇(특히 Google 외의 봇)은 자바스크립트를 실행하지 않거나 제한적으로 실행하므로, 빈 HTML만 인덱싱하여 검색 노출이 어렵습니다.
2. **초기 로딩 속도(TTV) 저하**: 사용자는 JS 번들 다운로드 → 파싱 → 실행이 완료될 때까지 빈 화면(또는 로딩 스피너)을 봐야 합니다.

Next.js의 **사전 렌더링(Pre-rendering)**은 서버에서 React 컴포넌트를 미리 HTML로 렌더링하여 완성된 HTML을 보냅니다. 크롤러는 완성된 콘텐츠가 담긴 HTML을 바로 읽을 수 있어 SEO가 개선되고, 사용자도 JS 실행 전에 화면을 볼 수 있어 TTV(Time To View)가 단축됩니다.

</details>

### Q2. 사전 렌더링을 하면 사용자가 화면을 더 빨리 볼 수는 있지만(TTV 단축), 화면의 버튼을 클릭해도 아무런 반응이 없는 '데드존(Dead Zone)' 시간이 발생할 수 있습니다. 이 현상의 원인은 무엇인가요?

* 의도: 화면이 보이는 시점(TTV)과 상호작용 가능한 시점(TTI) 사이의 간극(Gap)을 인지하고 있는지 확인
* 키워드: `TTI(Time To Interact)` `자바스크립트 실행 지연` `Hydration 대기 시간`

<details>
<summary>답변</summary>

사전 렌더링된 HTML은 정적인 뼈대일 뿐, 이벤트 핸들러(onClick 등)가 아직 연결되지 않은 상태입니다. 화면이 보이는 시점(TTV)과 실제 상호작용이 가능한 시점(TTI) 사이에 간극이 존재합니다.

이 데드존이 발생하는 원인은 **Hydration 대기 시간** 때문입니다:

1. 브라우저가 서버에서 받은 HTML을 즉시 화면에 그립니다 (TTV).
2. 이후 자바스크립트 번들을 다운로드하고 파싱합니다.
3. React가 Hydration 과정을 통해 정적 HTML에 이벤트 리스너를 연결합니다 (TTI).

2~3번 과정이 완료되기 전까지 사용자는 화면은 보이지만 버튼을 클릭해도 아무 반응이 없는 데드존을 경험합니다. 번들 크기가 클수록, 기기 성능이 낮을수록 이 간극은 더 길어집니다.

</details>

### Q3. 과거 JSP나 PHP 같은 전통적인 MPA(Multi-Page Application)의 서버 렌더링 방식과 Next.js의 사전 렌더링은 어떤 결정적인 차이가 있나요?

* 의도: Next.js가 단순한 서버 렌더링 도구가 아니라, 초기 로딩 이후에는 SPA처럼 부드럽게 동작한다는 하이브리드(Hybrid) 특성 이해도 검증
* 키워드: `첫 페이지만 서버 렌더링` `이후 클라이언트 라우팅(CSR)` `<Link> 컴포넌트 프리페칭`

<details>
<summary>답변</summary>

전통적인 MPA(JSP, PHP)는 **매 페이지 이동마다** 서버에 요청을 보내고, 서버가 완전한 HTML을 새로 생성하여 전체 페이지를 리로드합니다. 이로 인해 페이지 전환 시 화면이 깜빡이고 느립니다.

Next.js는 **하이브리드(Hybrid) 방식**을 사용합니다:

1. **첫 번째 페이지 요청**: 서버에서 사전 렌더링된 완성 HTML을 받습니다 (MPA와 동일).
2. **이후 페이지 이동**: Hydration이 완료된 후에는 `<Link>` 컴포넌트를 통해 **클라이언트 사이드 라우팅(CSR)**으로 동작합니다. 전체 페이지를 새로 받지 않고, 필요한 데이터(JSON)와 JS 청크만 받아 변경 부분만 교체합니다.

추가로 `<Link>` 컴포넌트는 뷰포트에 보이는 링크의 JS 번들을 **프리페칭(Prefetching)**하여, 사용자가 클릭하기 전에 미리 리소스를 다운로드합니다. 결국 "첫 로딩은 MPA처럼 빠르고, 이후 탐색은 SPA처럼 부드러운" 두 장점을 모두 가집니다.

</details>

### Q4. Next.js에서 제공하는 렌더링 전략인 SSR(Server-Side Rendering), SSG(Static Site Generation), ISR(Incremental Static Regeneration)의 동작 방식과 차이점을 설명해 주세요. (프로젝트 특성에 따른 도입 기준 포함)

* 의도: 데이터의 갱신 주기와 개인화 여부에 따라 최적의 렌더링 전략을 아키텍처 관점에서 결정할 수 있는지 평가
* 키워드: `빌드 타임(Build time)` `요청 타임(Request time)` `캐싱` `데이터 갱신` `서버 부하`

<details>
<summary>답변</summary>

**SSG (Static Site Generation)**:
- **빌드 타임**에 HTML을 미리 생성하여 CDN에 캐싱합니다.
- 응답 속도가 가장 빠르고 서버 부하가 없습니다.
- 적합: 마케팅 페이지, 블로그, 문서 등 데이터가 자주 변하지 않는 페이지.

**SSR (Server-Side Rendering)**:
- **매 요청 타임**마다 서버가 HTML을 생성합니다.
- 항상 최신 데이터를 보여줄 수 있지만, 요청마다 서버 연산이 필요합니다.
- 적합: 사용자별 개인화 페이지, 실시간 데이터가 필요한 대시보드, 검색 결과 페이지.

**ISR (Incremental Static Regeneration)**:
- SSG처럼 빌드 시 정적 생성하되, 설정한 `revalidate` 시간이 지나면 **백그라운드에서 페이지를 재생성**합니다.
- SSG의 빠른 응답 속도와 SSR의 데이터 최신성을 절충합니다.
- 적합: 상품 상세 페이지, 뉴스 기사 등 주기적으로 데이터가 갱신되는 페이지.

도입 기준은 **데이터 갱신 주기**와 **개인화 여부**로 판단합니다: 변경이 거의 없으면 SSG, 주기적 갱신이면 ISR, 요청마다 다른 데이터면 SSR을 선택합니다.

</details>

### Q5. 블로그 게시글처럼 데이터가 가끔 변경되는 페이지에 SSG를 적용했습니다. 이후 글이 수정되었을 때, 전체 사이트를 다시 빌드하지 않고 변경된 데이터만 반영하려면 어떻게 해야 하나요?

* 의도: SSG의 낡은 데이터(Stale data) 문제를 해결하는 ISR의 핵심 메커니즘 숙련도 확인
* 키워드: `ISR` `revalidate 시간 설정` `온디맨드 재검증(On-demand Revalidation)`

<details>
<summary>답변</summary>

**ISR(Incremental Static Regeneration)**을 사용합니다. 두 가지 방식이 있습니다:

**1. 시간 기반 재검증 (Time-based Revalidation)**:
```typescript
// App Router
fetch(url, { next: { revalidate: 60 } }); // 60초마다 재검증
```
설정한 시간이 경과한 후 다음 요청이 들어오면, 우선 캐싱된 이전(stale) 페이지를 즉시 반환하고, 백그라운드에서 새 페이지를 재생성합니다. 재생성이 완료되면 그 다음 요청부터 새 페이지가 제공됩니다(stale-while-revalidate 전략).

**2. 온디맨드 재검증 (On-demand Revalidation)**:
```typescript
// API Route 또는 Server Action 내부에서
revalidatePath('/blog/post-1'); // 특정 경로 재검증
revalidateTag('blog-posts');    // 특정 태그 재검증
```
CMS의 Webhook 등에서 글이 수정될 때 즉시 호출하면, 시간을 기다리지 않고 해당 페이지만 즉시 재생성할 수 있습니다. 전체 사이트 빌드 없이 변경된 페이지만 갱신하는 것이 핵심입니다.

</details>

### Q6. 최신 데이터를 보여주기 위해 SSR을 과도하게 사용하면 TTFB(Time to First Byte)가 느려져 오히려 사용자 경험을 해칠 수 있습니다. 이를 완화하기 위한 전략이 있다면 무엇일까요?

* 의도: SSR의 치명적인 단점(서버 병목)을 인지하고, CDN 캐싱이나 스트리밍(Streaming) 같은 대안을 고려해 본 적 있는지 검증
* 키워드: `서버 응답 대기` `CDN` `Suspense를 활용한 스트리밍 렌더링`

<details>
<summary>답변</summary>

SSR은 서버가 모든 데이터를 가져와 HTML을 완성할 때까지 응답을 보내지 않으므로, 무거운 API 호출이 있으면 TTFB가 길어집니다. 이를 완화하는 전략은 다음과 같습니다:

**1. 스트리밍 렌더링 (Streaming with Suspense)**:
전체 페이지를 한꺼번에 보내는 대신, 준비된 부분부터 HTML 청크(Chunk) 단위로 보냅니다. 느린 데이터 페칭 부분은 `<Suspense>`로 감싸서 fallback UI(스켈레톤 등)를 먼저 보내고, 데이터가 준비되면 해당 부분만 스트리밍으로 교체합니다.

**2. CDN 캐싱 + Cache-Control 헤더**:
SSR 응답에 `Cache-Control: s-maxage=60, stale-while-revalidate` 같은 헤더를 설정하여 CDN 엣지에서 캐싱합니다. 모든 요청이 Origin 서버까지 도달하지 않으므로 서버 부하와 TTFB를 줄일 수 있습니다.

**3. 하이브리드 전략**:
페이지 전체를 SSR하지 않고, 정적 셸은 SSG/ISR로 제공하고, 개인화되거나 실시간성이 필요한 부분만 클라이언트에서 패칭하는 방식으로 서버 연산을 최소화합니다.

</details>

### Q7. Next.js에서 서버가 만들어낸 정적 HTML에 React의 상호작용(이벤트 리스너 등)을 연결하는 과정인 '수분화(Hydration)'에 대해 설명해 주세요

* 의도: 브라우저 환경에서 Next.js가 React 앱으로 생명력을 얻는 과정(내부 메커니즘)에 대한 깊이 있는 이해도 측정
* 키워드: `정적 HTML 뼈대` `자바스크립트 매칭` `이벤트 바인딩` `React Tree 생성`

<details>
<summary>답변</summary>

Hydration은 서버에서 생성한 정적 HTML에 React의 상호작용 기능을 부여하는 과정입니다. 구체적인 단계는 다음과 같습니다:

1. **서버**: React 컴포넌트를 실행하여 정적 HTML 문자열을 생성하고 브라우저에 전송합니다. 이 HTML은 이벤트 핸들러가 없는 뼈대입니다.
2. **브라우저**: HTML을 즉시 화면에 렌더링합니다 (사용자가 화면을 볼 수 있음).
3. **JS 번들 로드**: React와 컴포넌트 코드가 담긴 자바스크립트 번들을 다운로드하고 파싱합니다.
4. **Hydration 실행**: React가 `hydrateRoot()`를 호출하여 **메모리 내에 Virtual DOM(React Tree)을 생성**하고, 이를 기존 서버 HTML DOM과 매칭합니다. 새로 DOM을 만드는 것이 아니라, 기존 DOM 노드를 그대로 재사용하면서 이벤트 리스너를 바인딩합니다.
5. **완료**: 정적이던 HTML이 완전한 React 앱으로 동작하게 됩니다 (상태 관리, 이벤트 처리, 라우팅 등).

핵심은 DOM을 새로 생성하지 않고 **기존 DOM을 "입양(adopt)"하여 이벤트를 연결**하는 것입니다.

</details>

### Q8. 개발하다가 **"Text content does not match server-rendered HTML" (Hydration Mismatch)** 에러를 겪어본 적이 있나요? 이 에러는 주로 어떤 상황에서 발생하며, 어떻게 해결하나요?

* 의도: 서버와 클라이언트의 렌더링 결과물이 다를 때 발생하는 Next.js의 고질적인 에러 트러블슈팅 경험 검증
* 키워드: `서버/클라이언트 시간(Date) 차이` `window 객체 참조 에러` `useEffect 마운트 후 렌더링` `suppressHydrationWarning`

<details>
<summary>답변</summary>

Hydration Mismatch는 서버에서 렌더링한 HTML과 클라이언트에서 Hydration 시 생성한 Virtual DOM이 일치하지 않을 때 발생합니다.

**주요 발생 상황**:
- **시간/날짜 차이**: `new Date()`를 직접 렌더링하면 서버와 브라우저의 시간대나 실행 시점이 달라 불일치 발생
- **window/localStorage 참조**: 서버에는 `window` 객체가 없으므로, 조건부 렌더링 시 서버/클라이언트 결과가 다름
- **랜덤 값 사용**: `Math.random()` 등이 서버/클라이언트에서 다른 값 생성
- **잘못된 HTML 중첩**: `<p>` 안에 `<div>` 등 브라우저의 자동 보정으로 구조가 달라짐

**해결 방법**:
1. **`useEffect` + 상태로 분리**: 클라이언트 전용 값은 `useEffect` 내에서 상태를 업데이트하여 마운트 후 렌더링
```javascript
const [time, setTime] = useState('');
useEffect(() => { setTime(new Date().toLocaleString()); }, []);
```
2. **`suppressHydrationWarning`**: 의도적으로 불일치가 허용되는 요소에 속성 추가 (시간 표시 등)
3. **`dynamic(() => import(...), { ssr: false })`**: 해당 컴포넌트를 아예 서버 렌더링에서 제외

</details>

### Q9. Hydration 과정 자체가 거대한 자바스크립트 번들을 다운로드하고 실행해야 하므로 무거운 작업이 될 수 있습니다. 최근 React와 Next.js 생태계에서는 이 불필요한 Hydration 비용을 줄이기 위해 어떤 아키텍처 패러다임을 도입했나요?

* 의도: 클라이언트 번들 사이즈를 줄이기 위한 최근의 패러다임(리액트 서버 컴포넌트) 트렌드 파악 여부 평가
* 키워드: `리액트 서버 컴포넌트(RSC, React Server Components)` `Zero Bundle Size`

<details>
<summary>답변</summary>

**리액트 서버 컴포넌트(RSC, React Server Components)**가 도입되었습니다.

기존 방식에서는 모든 컴포넌트의 JS 코드가 클라이언트로 전송되어 Hydration 대상이 되었습니다. RSC는 이를 근본적으로 바꿉니다:

- **서버 컴포넌트**: 서버에서만 실행되고, **렌더링 결과(HTML/RSC Payload)만** 클라이언트로 전송됩니다. 컴포넌트의 자바스크립트 코드 자체는 클라이언트 번들에 포함되지 않습니다. 이것이 **"Zero Bundle Size"**의 의미입니다.
- **클라이언트 컴포넌트**: `'use client'`를 선언한 컴포넌트만 JS 번들에 포함되어 Hydration 대상이 됩니다.

이로써 상호작용이 없는 컴포넌트(데이터 표시, 레이아웃 등)는 Hydration이 불필요해지므로 클라이언트 번들 크기가 크게 줄어들고, Hydration 비용이 상호작용이 필요한 컴포넌트만으로 한정됩니다. 또한 서버 컴포넌트에서 사용하는 무거운 라이브러리(마크다운 파서, 날짜 라이브러리 등)도 번들에서 제외됩니다.

</details>

### Q10. Next.js 13 이후 도입된 App Router는 기존 Pages Router와 비교했을 때 아키텍처 관점에서 어떤 혁신적인 변화가 있었나요? 신규 프로젝트를 시작한다면 둘 중 어떤 것을 선택할 것이며, 그 이유는 무엇인가요?

* 의도: 프레임워크의 패러다임 변화(React Server Components 기본 적용, 라우팅 시스템의 변화)를 명확히 이해하고, 기술 도입의 타당한 근거를 제시할 수 있는지 평가
* 키워드: `디렉토리 기반 라우팅` `중첩 레이아웃(Nested Layouts)` `React Server Components(RSC) 기본화` `스트리밍(Streaming)`

<details>
<summary>답변</summary>

**App Router의 주요 변화**:

1. **RSC 기본 적용**: 모든 컴포넌트가 기본적으로 서버 컴포넌트입니다. 클라이언트 컴포넌트가 필요할 때만 `'use client'`를 명시합니다.
2. **중첩 레이아웃(Nested Layouts)**: `layout.tsx` 파일을 통해 라우트 세그먼트별로 레이아웃을 정의하고, 페이지 이동 시 변경된 세그먼트만 리렌더링됩니다.
3. **스트리밍(Streaming)**: `loading.tsx`와 `<Suspense>`를 활용한 점진적 렌더링이 기본 지원됩니다.
4. **컴포넌트 레벨 데이터 페칭**: `getServerSideProps` 같은 페이지 레벨 함수 대신, 각 서버 컴포넌트에서 직접 `async/await`로 데이터를 가져옵니다.
5. **Server Actions**: `'use server'` 지시어로 폼 처리를 위한 별도 API Route 없이 서버 함수를 직접 호출합니다.

**신규 프로젝트라면 App Router**를 선택합니다. RSC를 통한 번들 최적화, 중첩 레이아웃의 성능 이점, 스트리밍 지원 등이 현대 웹 앱에 적합하고, Next.js 팀이 공식적으로 권장하는 방향이기 때문입니다.

</details>

### Q11. App Router에서는 `layout.tsx`를 통해 중첩 레이아웃을 아주 쉽게 구현할 수 있습니다. 기존 Pages Router의 `_app.tsx`나 `_document.tsx`를 사용하던 방식과 비교했을 때 렌더링 성능이나 상태 유지 측면에서 어떤 점이 좋아졌나요?

* 의도: 페이지 이동 시 전체가 리렌더링되지 않고 변경된 부분만 렌더링되는 부분 렌더링(Partial Rendering) 원리 이해도 확인
* 키워드: `부분 렌더링(Partial Rendering)` `레이아웃 상태 유지(State Preservation)` `불필요한 리렌더링 방지`

<details>
<summary>답변</summary>

**Pages Router의 한계**:
- `_app.tsx`는 모든 페이지를 감싸는 단일 래퍼로, 페이지가 전환될 때마다 컴포넌트 전체가 리렌더링됩니다.
- 특정 라우트 그룹에만 적용되는 레이아웃을 구현하려면 수동으로 조건 분기가 필요했습니다.

**App Router의 개선점**:

1. **부분 렌더링(Partial Rendering)**: 페이지 이동 시 공유하는 `layout.tsx`는 리렌더링되지 않고, 변경된 하위 세그먼트(`page.tsx`)만 새로 렌더링됩니다. 예를 들어 `/dashboard/settings`에서 `/dashboard/analytics`로 이동하면, `dashboard/layout.tsx`는 그대로 유지되고 하위 page만 교체됩니다.

2. **상태 유지(State Preservation)**: 레이아웃이 리렌더링되지 않으므로, 레이아웃 내의 상태(사이드바 열림/닫힘, 스크롤 위치 등)가 페이지 전환 간에도 유지됩니다.

3. **불필요한 리렌더링 방지**: 서버 컴포넌트인 레이아웃은 클라이언트 번들에 포함되지 않으므로, 하위 페이지 전환 시 레이아웃의 JS 실행이 불필요합니다.

</details>

### Q12. App Router가 강력하지만, 기존 Pages Router에서 App Router로 마이그레이션할 때 개발자들이 가장 많이 겪는 어려움(또는 진입 장벽)은 무엇이라고 생각하시나요?

* 의도: 신기술의 장점만 맹신하지 않고, 에코시스템 호환성이나 학습 곡선 등 현실적인 트레이드오프를 인지하고 있는지 검증
* 키워드: `서드파티 라이브러리 호환성('use client' 강제)` `멘탈 모델의 변화(서버/클라이언트 분리)` `캐싱 정책의 복잡도`

<details>
<summary>답변</summary>

**1. 멘탈 모델의 근본적 변화**:
기존에는 모든 컴포넌트가 클라이언트에서 실행되었지만, App Router에서는 "이 컴포넌트가 서버에서 실행되는가, 클라이언트에서 실행되는가"를 항상 의식해야 합니다. `useState`, `useEffect` 등을 사용하려면 `'use client'`를 선언해야 하고, 서버/클라이언트 경계에서의 데이터 직렬화 제약을 이해해야 합니다.

**2. 서드파티 라이브러리 호환성**:
많은 기존 라이브러리가 클라이언트 전용(`useState`, `useContext` 사용)이므로, `'use client'`로 래핑하는 어댑터 컴포넌트를 만들어야 하는 경우가 많습니다. 초기에는 호환되지 않는 라이브러리도 있었습니다.

**3. 캐싱 정책의 복잡도**:
4단계 캐싱 시스템(Request Memoization, Data Cache, Full Route Cache, Router Cache)이 존재하며, 각각의 생명주기와 무효화 방법이 다릅니다. 의도치 않게 오래된 데이터가 보이는 문제를 디버깅하기 어렵습니다.

**4. 점진적 마이그레이션의 복잡성**:
Pages Router와 App Router를 동시에 사용할 수 있지만, 두 시스템 간의 라우팅 충돌이나 상태 공유 문제가 발생할 수 있습니다.

</details>

### Q13. 리액트 서버 컴포넌트(RSC)와 클라이언트 컴포넌트의 가장 결정적인 차이는 무엇인가요? 특히 서버 컴포넌트가 'Zero Bundle Size(번들 사이즈 제로)'를 달성할 수 있는 원리를 설명해 주세요.

* 의도: RSC의 도입 목적이 '클라이언트로 보내는 자바스크립트 양을 줄여 성능을 극대화하는 것'임을 알고 있는지, 런타임 환경의 차이를 이해하는지 평가
* 키워드: `서버에서만 실행(Node.js 환경)` `클라이언트 번들 제외` `데이터베이스 직접 접근` `브라우저 API 사용 불가`

<details>
<summary>답변</summary>

**결정적인 차이는 실행 환경과 번들 포함 여부**입니다.

**서버 컴포넌트**:
- **서버(Node.js 환경)에서만 실행**됩니다. 실행 결과인 렌더링 출력(RSC Payload)만 클라이언트로 전송되고, 컴포넌트 코드 자체는 클라이언트 JS 번들에 포함되지 않습니다.
- 이것이 "Zero Bundle Size"의 원리입니다: 서버 컴포넌트가 import하는 라이브러리(마크다운 파서, 하이라이터 등)도 함께 번들에서 제외됩니다.
- 데이터베이스, 파일 시스템에 직접 접근 가능합니다.
- 제약: `useState`, `useEffect` 등 상태/생명주기 훅, `onClick` 등 이벤트 핸들러, 브라우저 API(`window`, `localStorage`) 사용 불가.

**클라이언트 컴포넌트** (`'use client'`):
- 서버에서 사전 렌더링(SSR)된 후, **브라우저에서도 실행**되어 Hydration됩니다.
- JS 코드가 클라이언트 번들에 포함됩니다.
- 상태, 이벤트, 브라우저 API 모두 사용 가능합니다.

</details>

### Q14. 서버 컴포넌트에서 클라이언트 컴포넌트로 데이터(Props)를 넘겨줄 때 "Props must be serializable(직렬화 가능해야 합니다)"라는 에러를 자주 마주하게 됩니다. 이 에러가 발생하는 이유는 무엇이며, 어떻게 해결해야 하나요?

* 의도: 서버와 브라우저 사이의 네트워크 경계(Network Boundary)를 이해하고, 직렬화(Serialization) 불가능한 데이터(함수, Date 객체 등) 처리 경험 확인
* 키워드: `네트워크 경계(Network Boundary)` `JSON 직렬화 불가(Function, Map, Set 등)` `데이터 가공 후 전달`

<details>
<summary>답변</summary>

서버 컴포넌트와 클라이언트 컴포넌트 사이에는 **네트워크 경계(Network Boundary)**가 존재합니다. 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 Props는 네트워크를 통해 직렬화(Serialization)되어 전송되므로, **JSON으로 변환 가능한 데이터만 전달**할 수 있습니다.

**직렬화 불가능한 데이터**:
- 함수(Function), 클래스 인스턴스
- `Date` 객체, `Map`, `Set`
- 순환 참조가 있는 객체

**해결 방법**:
1. **서버에서 가공하여 원시값으로 변환**:
```typescript
// 서버 컴포넌트
const date = await getDate();
// Date 객체 대신 문자열로 변환하여 전달
<ClientComponent dateStr={date.toISOString()} />
```

2. **함수는 클라이언트 컴포넌트 내부에서 정의**:
이벤트 핸들러 등의 함수는 서버에서 전달하지 않고, 클라이언트 컴포넌트 내부에서 직접 정의합니다.

3. **Server Actions 활용**:
서버 로직을 `'use server'` 함수로 만들면, 참조(ID)가 전달되어 클라이언트에서 호출 가능합니다.

</details>

### Q15. 클라이언트 컴포넌트(`'use client'`) 내부에서 서버 컴포넌트를 직접 `import`해서 렌더링할 수 있나요? 불가능하다면, 클라이언트 컴포넌트 안에 서버 컴포넌트를 위치시키기 위해 어떤 패턴을 사용해야 하나요?

* 의도: RSC의 가장 중요한 제약 사항과 이를 우회하는 `children` prop 패턴(컴포지션) 숙련도 검증
* 키워드: `직접 import 불가(클라이언트 컴포넌트로 변환됨)` `children 속성(Composition 패턴)` `슬롯(Slots)`

<details>
<summary>답변</summary>

**직접 import하면 불가능합니다.** 클라이언트 컴포넌트가 서버 컴포넌트를 `import`하면, 해당 서버 컴포넌트도 자동으로 클라이언트 컴포넌트로 변환되어 서버 컴포넌트의 이점(Zero Bundle Size 등)을 잃게 됩니다.

**해결: children prop을 활용한 컴포지션(Composition) 패턴**:

```tsx
// ServerComponent.tsx (서버 컴포넌트)
export default async function ServerComponent() {
  const data = await fetchData();
  return <div>{data.title}</div>;
}

// ClientWrapper.tsx ('use client')
'use client';
export default function ClientWrapper({ children }) {
  const [isOpen, setIsOpen] = useState(false);
  return <div onClick={() => setIsOpen(!isOpen)}>{children}</div>;
}

// page.tsx (서버 컴포넌트)
export default function Page() {
  return (
    <ClientWrapper>
      <ServerComponent /> {/* children으로 주입 */}
    </ClientWrapper>
  );
}
```

`page.tsx`(서버 컴포넌트)에서 `ServerComponent`를 `children`으로 `ClientWrapper`에 전달하면, `ServerComponent`는 서버에서 렌더링된 결과(RSC Payload)가 전달되므로 서버 컴포넌트로서의 이점을 유지합니다. 이 패턴을 "슬롯(Slots) 패턴"이라고도 합니다.

</details>

### Q16. App Router 환경에서 개발할 때, 컴포넌트를 서버 컴포넌트와 클라이언트 컴포넌트로 나누는 본인만의 명확한 기준(Client Boundary 패턴 등)이 있다면 설명해 주세요.

* 의도: '최대한 서버 컴포넌트를 유지하고, 상호작용이 필요한 부분만 클라이언트 컴포넌트로 분리(Push state down)'하는 실무적인 아키텍처 설계 능력 파악
* 키워드: `Client Boundary` `상태(State)와 생명주기(Lifecycle)` `브라우저 API` `트리 끝단(Leaf node) 배치`

<details>
<summary>답변</summary>

핵심 원칙은 **"기본은 서버 컴포넌트, 필요한 곳만 클라이언트 컴포넌트"**입니다.

**클라이언트 컴포넌트가 필요한 경우**:
- `useState`, `useReducer` 등 **상태(State)** 사용
- `useEffect` 등 **생명주기(Lifecycle)** 훅 사용
- `onClick`, `onChange` 등 **이벤트 핸들러** 필요
- `window`, `localStorage` 등 **브라우저 API** 접근
- Context API를 통한 상태 소비(`useContext`)

**설계 전략 - "Push Client Boundary Down"**:

클라이언트 경계(`'use client'`)를 컴포넌트 트리의 **가능한 한 끝단(Leaf node)에 배치**합니다.

```
Page (서버) → ArticleContent (서버) → 데이터 표시만
           → Sidebar (서버)        → LikeButton (클라이언트 - onClick 필요)
                                   → ShareButton (클라이언트 - 상태 필요)
```

큰 컴포넌트 전체를 `'use client'`로 만들지 않고, 상호작용이 필요한 작은 부분만 분리하여 클라이언트 컴포넌트로 만듭니다. 이렇게 하면 서버 컴포넌트의 Zero Bundle Size 이점을 최대한 유지할 수 있습니다.

</details>

### Q17. Redux나 Context API 같은 전역 상태 제공자(Provider)는 `useState` 등을 사용하기 때문에 필연적으로 클라이언트 컴포넌트(`'use client'`)가 되어야 합니다. 이를 최상위 `layout.tsx`에 감싸면 그 하위의 모든 컴포넌트가 클라이언트 컴포넌트가 되는 것 아닌가요?

* 의도: 가장 많이 오해하는 RSC의 트리 구조와 Context Provider 설계 패턴 인지 확인
* 키워드: `children으로 주입` `하위 컴포넌트는 여전히 서버 컴포넌트로 동작 가능` `트리 오염 방지`

<details>
<summary>답변</summary>

**아닙니다.** 이것은 가장 흔한 RSC 오해 중 하나입니다.

Provider 자체는 `'use client'`여야 하지만, **`children`으로 전달되는 하위 컴포넌트는 여전히 서버 컴포넌트로 동작**할 수 있습니다.

```tsx
// providers.tsx
'use client';
export function Providers({ children }) {
  return <ThemeContext.Provider value={...}>{children}</ThemeContext.Provider>;
}

// layout.tsx (서버 컴포넌트)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

핵심은 `children`이 **서버 컴포넌트(layout.tsx)에서 이미 렌더링된 결과**로 전달된다는 것입니다. `Providers` 컴포넌트가 `children`을 직접 `import`하는 것이 아니라, 부모인 서버 컴포넌트가 렌더링한 결과를 "구멍(Slot)"에 끼워 넣는 것이므로, 하위 컴포넌트는 서버 컴포넌트의 이점을 그대로 유지합니다.

단, `useContext`로 해당 Context를 **소비(consume)**하는 컴포넌트는 클라이언트 컴포넌트여야 합니다.

</details>

### Q18. 상단에 `'use client'` 지시어를 선언한 클라이언트 컴포넌트는 서버에서 전혀 렌더링(SSR)되지 않고 오직 브라우저에서만 렌더링(CSR)되나요?

* 의도: RSC의 '클라이언트 컴포넌트'와 기존 React의 CSR을 혼동하지 않고, Next.js의 사전 렌더링(Pre-rendering) 개념과 연결 지을 수 있는지 확인
* 키워드: `오해 바로잡기` `클라이언트 컴포넌트도 서버에서 사전 HTML로 렌더링(SSR)됨` `이후 Hydration 진행`

<details>
<summary>답변</summary>

**아닙니다.** 이것은 매우 흔한 오해입니다.

`'use client'`는 "이 컴포넌트가 브라우저에서만 실행된다"는 의미가 **아닙니다**. 정확한 의미는 "이 컴포넌트는 클라이언트 번들에 포함되어 Hydration 대상이 된다"입니다.

**실제 동작**:
1. **서버**: 클라이언트 컴포넌트도 서버에서 **HTML로 사전 렌더링(SSR)**됩니다. 초기 상태를 기반으로 정적 HTML이 생성됩니다.
2. **브라우저**: 사전 렌더링된 HTML이 즉시 표시된 후, JS 번들이 로드되면 Hydration을 통해 이벤트 리스너가 연결되고 상호작용이 가능해집니다.

즉, 서버 컴포넌트와 클라이언트 컴포넌트의 차이는 "어디서 렌더링되는가"가 아니라:
- **서버 컴포넌트**: 서버에서만 실행, JS 번들 미포함, Hydration 없음
- **클라이언트 컴포넌트**: 서버에서 SSR + 브라우저에서 Hydration, JS 번들 포함

</details>

### Q19. App Router에서는 기존 Pages Router에서 쓰던 `getServerSideProps`나 `getStaticProps` 함수들이 사라지고, 확장된 웹 표준 `fetch` API를 사용하게 되었습니다. 이러한 변화가 가져온 아키텍처 및 렌더링 측면의 이점은 무엇인가요?

* 의도: 프레임워크 전용 API에 의존하던 과거의 방식(페이지 레벨 페칭)에서 벗어나, 컴포넌트 레벨로 데이터를 페칭하는 패러다임의 변화(RSC와의 시너지)를 이해하고 있는지 평가
* 키워드: `컴포넌트 레벨 페칭` `Props Drilling 감소` `React Server Components(RSC)` `fetch 옵션(cache, next: { revalidate })`

<details>
<summary>답변</summary>

**1. 컴포넌트 레벨 데이터 페칭**:
Pages Router에서는 `getServerSideProps`/`getStaticProps`로 **페이지 최상위에서만** 데이터를 가져올 수 있었고, 하위 컴포넌트에 Props Drilling으로 전달해야 했습니다. App Router에서는 각 서버 컴포넌트가 직접 `async/await`로 데이터를 가져옵니다:

```tsx
// 데이터가 필요한 컴포넌트에서 직접 fetch
async function UserProfile({ userId }) {
  const user = await fetch(`/api/users/${userId}`);
  return <div>{user.name}</div>;
}
```

**2. Props Drilling 감소**:
각 컴포넌트가 필요한 데이터를 스스로 가져오므로, 부모에서 자식으로 데이터를 여러 단계 전달할 필요가 없습니다.

**3. 세밀한 캐싱 제어**:
`fetch` 옵션으로 컴포넌트별 캐싱 전략을 다르게 설정할 수 있습니다:
```typescript
fetch(url, { cache: 'force-cache' });         // SSG처럼 동작
fetch(url, { cache: 'no-store' });            // SSR처럼 동작
fetch(url, { next: { revalidate: 3600 } });   // ISR처럼 동작
```

**4. 웹 표준 준수**:
프레임워크 전용 API 대신 웹 표준 `fetch`를 확장하므로, 학습 비용이 줄고 코드 이식성이 높아집니다.

</details>

### Q20. 예전에는 최상위 페이지에서만 데이터를 불러와 자식 컴포넌트로 내려줘야 했습니다. App Router에서는 깊은 곳에 있는 자식 컴포넌트 여러 개가 각각 동일한 API로 `fetch`를 호출해도 성능상 문제가 발생하지 않는 이유는 무엇인가요?

* 의도: Next.js 캐싱 시스템의 첫 번째 관문인 '리퀘스트 메모이제이션(Request Memoization)'의 중복 호출 방지 원리를 아는지 확인
* 키워드: `중복 제거(Deduplication)` `단일 렌더링 패스` `네트워크 요청 최적화`

<details>
<summary>답변</summary>

**Request Memoization(리퀘스트 메모이제이션)** 덕분입니다.

React가 서버에서 컴포넌트 트리를 렌더링하는 **단일 렌더링 패스(Single Render Pass)** 동안, 동일한 URL과 옵션으로 호출된 `fetch` 요청을 자동으로 **중복 제거(Deduplication)**합니다.

```
ComponentA → fetch('/api/user')  ← 실제 네트워크 요청 발생
ComponentB → fetch('/api/user')  ← 캐싱된 결과 반환 (네트워크 요청 없음)
ComponentC → fetch('/api/user')  ← 캐싱된 결과 반환 (네트워크 요청 없음)
```

**동작 원리**:
1. 첫 번째 `fetch` 호출 시 실제 네트워크 요청이 발생하고, 결과가 **인메모리에 캐싱**됩니다.
2. 같은 렌더링 패스 내에서 동일한 요청이 들어오면 캐싱된 결과를 즉시 반환합니다.
3. 렌더링 패스가 끝나면 이 메모이제이션은 자동으로 초기화됩니다.

따라서 개발자는 "이미 상위에서 fetch했으니까 Props로 내려줘야지"라고 고민할 필요 없이, 데이터가 필요한 컴포넌트에서 자유롭게 fetch하면 됩니다. 프레임워크가 중복을 알아서 처리합니다.

</details>

### Q21. 외부 API가 `fetch`를 사용하지 않는 라이브러리(예: Prisma 같은 DB ORM이나 외부 SDK)일 경우, Next.js의 강력한 캐싱 기능(`revalidate` 등)을 어떻게 적용할 수 있나요?

* 의도: `fetch`가 아닌 데이터 소스에도 Next.js의 데이터 캐시(Data Cache)를 적용할 수 있는 실무적인 대안 숙련도 평가
* 키워드: `React cache 함수` `Next.js unstable_cache`

<details>
<summary>답변</summary>

`fetch`를 사용하지 않는 데이터 소스에는 두 가지 방법으로 캐싱을 적용할 수 있습니다.

**1. React `cache` 함수 (Request Memoization)**:
단일 렌더링 패스 내에서 동일한 함수 호출을 중복 제거합니다. 렌더링이 끝나면 캐시가 초기화되므로, 요청 간 캐싱은 아닙니다.

```typescript
import { cache } from 'react';

export const getUser = cache(async (id: string) => {
  const user = await prisma.user.findUnique({ where: { id } });
  return user;
});
```

**2. Next.js `unstable_cache` (Data Cache)**:
`fetch`의 데이터 캐시처럼 **요청 간에도 캐싱이 유지**되며, `revalidate` 옵션과 태그 기반 무효화를 지원합니다.

```typescript
import { unstable_cache } from 'next/cache';

const getCachedUser = unstable_cache(
  async (id: string) => prisma.user.findUnique({ where: { id } }),
  ['user-cache'],                    // 캐시 키
  { revalidate: 3600, tags: ['users'] } // 1시간마다 재검증, 태그 설정
);
```

이후 `revalidateTag('users')`를 호출하면 해당 캐시를 무효화할 수 있습니다. 두 함수의 차이는 `cache`는 렌더링 패스 내 중복 제거, `unstable_cache`는 요청 간 영구 캐싱이라는 점입니다.

</details>

### Q22. Next.js App Router의 성능을 극대화하는 4단계 캐싱 시스템(Request Memoization, Data Cache, Full Route Cache, Router Cache) 중 2가지 이상을 골라 그 동작 원리와 생명주기(Lifecycle)를 설명해 주세요.

* 의도: Next.js의 가장 복잡하고 핵심적인 아키텍처를 정확히 이해하고, 의도치 않은 캐싱(Stale Data) 문제를 디버깅할 수 있는 깊이 있는 지식 검증
* 키워드: `렌더링 패스(생명주기)` `서버 사이드 캐시` `클라이언트 사이드 캐시` `인메모리 vs 영구 저장`

<details>
<summary>답변</summary>

**1. Request Memoization (서버 사이드, 인메모리)**:
- **위치**: 서버, React 렌더링 중
- **동작**: 단일 렌더링 패스 내에서 동일한 `fetch(URL, options)` 호출을 중복 제거합니다.
- **생명주기**: 렌더링 패스가 시작되면 생성되고, **렌더링이 끝나면 즉시 소멸**합니다. 요청 간에는 공유되지 않습니다.
- **용도**: 컴포넌트 트리의 여러 곳에서 같은 데이터를 자유롭게 fetch할 수 있게 합니다.

**2. Data Cache (서버 사이드, 영구 저장)**:
- **위치**: 서버, 영구적 저장소
- **동작**: `fetch` 응답을 **요청 간에도 유지**되는 캐시에 저장합니다. `cache: 'force-cache'`(기본)로 영구 캐싱, `next: { revalidate: N }`으로 시간 기반 재검증이 가능합니다.
- **생명주기**: 명시적으로 무효화(`revalidatePath`, `revalidateTag`)하거나 revalidate 시간이 경과할 때까지 **배포 간에도 유지**됩니다.
- **용도**: 외부 API 응답을 캐싱하여 불필요한 네트워크 요청을 줄입니다.

**3. Full Route Cache (서버 사이드, 영구 저장)**:
- **동작**: 정적 라우트의 완성된 HTML과 RSC Payload를 빌드 타임에 캐싱합니다. Data Cache가 무효화되면 함께 무효화됩니다.

**4. Router Cache (클라이언트 사이드, 인메모리)**:
- **동작**: 방문한 라우트 세그먼트의 RSC Payload를 브라우저 메모리에 저장합니다. 뒤로 가기/앞으로 가기 시 서버 요청 없이 즉시 복원됩니다.
- **생명주기**: 세션(탭) 동안 유지되며, 새로고침 시 초기화됩니다.

</details>

### Q23. 실무에서 "게시글을 수정했는데 화면에 바로 반영이 안 돼요!"라는 버그 리포트를 받았습니다. 4단계 캐시 중 어느 부분을 의심해야 하며, 이를 해결하기 위해 어떤 캐시 무효화(Invalidation) 전략을 사용하시겠습니까?

* 의도: 데이터를 업데이트한 후(Mutation) 사용자에게 최신 상태를 보여주기 위한 실무적인 캐시 무효화 함수 사용 역량 평가
* 키워드: `Data Cache 무효화` `revalidatePath` `revalidateTag` `온디맨드 재검증`

<details>
<summary>답변</summary>

가장 먼저 **Data Cache**를 의심해야 합니다. `fetch`로 가져온 게시글 데이터가 서버에 캐싱되어 있어, DB가 업데이트되어도 캐싱된 오래된 데이터를 계속 반환하는 것이 가장 흔한 원인입니다.

**해결 전략 - 온디맨드 재검증(On-demand Revalidation)**:

게시글 수정 Server Action 내부에서 캐시를 무효화합니다:

```typescript
'use server';
import { revalidatePath, revalidateTag } from 'next/cache';

async function updatePost(id: string, data: FormData) {
  await db.post.update({ where: { id }, data: { ... } });

  // 방법 1: 경로 기반 무효화 - 해당 경로의 Full Route Cache와 Data Cache 무효화
  revalidatePath(`/posts/${id}`);

  // 방법 2: 태그 기반 무효화 - 더 세밀한 제어
  revalidateTag(`post-${id}`);
}
```

태그 기반 무효화를 사용하려면 fetch 시 태그를 지정해야 합니다:
```typescript
fetch(`/api/posts/${id}`, { next: { tags: [`post-${id}`] } });
```

**무효화 연쇄 효과**: Data Cache가 무효화되면 → Full Route Cache도 무효화 → 다음 요청 시 서버가 새로운 데이터로 페이지를 재생성합니다. Router Cache(클라이언트)는 `router.refresh()`를 호출하거나, Server Action 실행 후 자동으로 갱신됩니다.

</details>

### Q24. Next.js의 'Request Memoization'과 React의 `useMemo` 훅은 이름이 비슷합니다. 어떤 환경에서 무엇을 캐싱하는지 그 차이를 명확히 구분하여 설명해 주실 수 있나요?

* 의도: 이름만 비슷한 두 개념을 서버 환경(네트워크 요청)과 클라이언트 환경(CPU 연산)의 맥락에서 분리해서 이해하고 있는지 확인
* 키워드: `서버 측 네트워크 요청 캐싱(Request Memoization)` `클라이언트 측 렌더링 연산 비용 캐싱(useMemo)`

<details>
<summary>답변</summary>

둘 다 "이전 결과를 기억하여 재사용한다"는 메모이제이션 개념이지만, 실행 환경과 캐싱 대상이 완전히 다릅니다.

**Request Memoization**:
- **환경**: 서버 (렌더링 패스 중)
- **캐싱 대상**: `fetch` **네트워크 요청의 응답**
- **동작**: 같은 렌더링 패스 내에서 동일한 URL+옵션의 fetch를 중복 제거
- **생명주기**: 하나의 서버 렌더링 패스 동안만 유지
- **목적**: 여러 서버 컴포넌트가 같은 API를 호출해도 실제 네트워크 요청은 1회만 발생

**`useMemo`**:
- **환경**: 클라이언트 (브라우저)
- **캐싱 대상**: **CPU 연산(계산)의 결과값**
- **동작**: 의존성 배열 값이 변경되지 않으면 이전 계산 결과를 재사용
- **생명주기**: 컴포넌트가 마운트되어 있는 동안 유지
- **목적**: 비용이 큰 계산(정렬, 필터링 등)의 불필요한 재실행 방지

정리하면, Request Memoization은 **서버에서 네트워크 I/O 비용**을 줄이고, `useMemo`는 **클라이언트에서 CPU 연산 비용**을 줄입니다.

</details>

### Q25. 특정 API 호출이 5초 이상 걸리는 무거운 서버 컴포넌트가 있을 때, 이로 인해 전체 페이지의 화면 렌더링이 지연되는 병목(Bottleneck) 현상을 '스트리밍(Streaming)'과 `Suspense`를 활용하여 어떻게 해결할 수 있나요?

* 의도: 전통적인 SSR의 고질적인 단점(All or Nothing - 데이터를 다 가져올 때까지 빈 화면)을 극복하는 스트리밍 렌더링의 원리 이해 및 UX 개선 역량 평가
* 키워드: `청크(Chunk)` `점진적 렌더링(Progressive Rendering)` `TTFB(Time To First Byte) 단축` `Fallback UI`

<details>
<summary>답변</summary>

전통적인 SSR은 **All or Nothing** 방식으로, 모든 데이터 페칭이 완료될 때까지 전체 페이지 HTML 응답을 보내지 않습니다. 5초짜리 API가 있으면 전체 TTFB가 5초 이상이 됩니다.

**스트리밍(Streaming) + Suspense**로 해결합니다:

```tsx
export default function Page() {
  return (
    <div>
      <Header />           {/* 즉시 렌더링 */}
      <MainContent />       {/* 즉시 렌더링 */}
      <Suspense fallback={<SkeletonLoader />}>
        <SlowDataComponent /> {/* 5초 걸리는 컴포넌트 */}
      </Suspense>
    </div>
  );
}
```

**동작 원리**:
1. 서버는 `<Suspense>` 바깥의 `Header`, `MainContent`를 **즉시 HTML 청크(Chunk)로 스트리밍**합니다. 동시에 `SlowDataComponent` 자리에는 `<SkeletonLoader/>`가 Fallback UI로 전송됩니다.
2. 사용자는 TTFB 직후부터 페이지의 대부분을 볼 수 있습니다 (**점진적 렌더링**).
3. `SlowDataComponent`의 데이터 페칭이 완료되면, 서버가 해당 부분의 HTML을 추가 청크로 스트리밍합니다.
4. 브라우저가 인라인 스크립트를 통해 Fallback UI를 실제 컨텐츠로 교체합니다.

결과적으로 5초짜리 API가 전체 페이지의 TTFB를 지연시키지 않습니다.

</details>

### Q26. Next.js의 `loading.tsx` 파일을 사용하는 것과 특정 컴포넌트를 직접 `<Suspense>`로 감싸는 것은 렌더링 차단 범위(Scope)와 UI/UX 관점에서 어떤 차이가 있나요?

* 의도: 프레임워크가 제공하는 파일 기반 라우팅과 React 고유 기능의 차이를 알고, 더 정교한 UI(스켈레톤 등)를 설계할 수 있는지 검증
* 키워드: `페이지 전체 블로킹(loading.tsx)` `부분적 독립 렌더링(<Suspense>)` `컴포넌트 분할`

<details>
<summary>답변</summary>

**`loading.tsx`**:
- Next.js가 자동으로 해당 라우트 세그먼트의 `page.tsx` 전체를 `<Suspense>`로 감싸는 것과 동일합니다.
- **페이지 전체**가 로딩 상태로 대체됩니다.
- 페이지 이동 시 즉시 로딩 UI를 보여줘 사용자 피드백이 빠릅니다.
- 적합: 페이지 전체가 데이터에 의존하는 경우.

```
layout.tsx
├── loading.tsx  → page.tsx 전체를 Suspense로 감쌈
└── page.tsx
```

**직접 `<Suspense>` 사용**:
- 페이지 내 **특정 컴포넌트만** 독립적으로 로딩 상태를 관리합니다.
- 나머지 부분은 즉시 렌더링되고, 느린 부분만 Fallback UI로 표시됩니다.
- 적합: 페이지의 일부만 느린 데이터에 의존하는 경우.

```tsx
export default function Page() {
  return (
    <>
      <StaticHeader />                        {/* 즉시 표시 */}
      <Suspense fallback={<FeedSkeleton />}>
        <SlowFeed />                          {/* 독립 로딩 */}
      </Suspense>
      <Suspense fallback={<AdSkeleton />}>
        <SlowAdBanner />                      {/* 독립 로딩 */}
      </Suspense>
    </>
  );
}
```

**정리**: `loading.tsx`는 페이지 단위의 간편한 로딩 처리, `<Suspense>`는 컴포넌트 단위의 정교한 로딩 처리입니다. 실무에서는 둘을 조합하여 사용합니다.

</details>

### Q27. 데이터를 페칭하는 중에 서버 에러가 발생했습니다. 이 에러가 전체 페이지를 하얗게(White Screen of Death) 망가뜨리지 않고, 에러가 난 그 컴포넌트 부분만 에러 UI로 대체하려면 스트리밍 환경에서 어떤 처리를 해야 하나요?

* 의도: 예외 상황(Error Handling)에 대한 방어적 설계, 즉 에러 바운더리(Error Boundary) 개념을 App Router에 맞게 알고 있는지 확인
* 키워드: `error.tsx` `Error Boundary` `컴포넌트 격리` `회복성(Resilience)`

<details>
<summary>답변</summary>

App Router에서는 **`error.tsx`** 파일을 통해 에러를 격리합니다.

**`error.tsx`의 동작 원리**:
- Next.js가 자동으로 해당 라우트 세그먼트를 **React Error Boundary**로 감쌉니다.
- 에러가 발생하면 해당 세그먼트만 에러 UI로 대체되고, 상위 레이아웃과 다른 세그먼트는 정상 동작을 유지합니다.

```tsx
// app/dashboard/error.tsx
'use client'; // Error Boundary는 반드시 클라이언트 컴포넌트

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>문제가 발생했습니다</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>다시 시도</button>
    </div>
  );
}
```

**스트리밍 환경에서의 격리**:
`<Suspense>`와 `error.tsx`(또는 직접 Error Boundary)를 조합하면, 각 스트리밍 청크가 독립적으로 에러를 처리합니다:

```tsx
<Suspense fallback={<Skeleton />}>
  <ErrorBoundary fallback={<ErrorUI />}>
    <RiskyComponent />  {/* 이 부분만 에러 UI로 대체 */}
  </ErrorBoundary>
</Suspense>
<SafeComponent />       {/* 정상 동작 유지 */}
```

`reset()` 함수를 호출하면 해당 세그먼트의 리렌더링을 시도하여 **회복성(Resilience)**을 제공합니다.

</details>

### Q28. API Route를 따로 만들지 않고 폼(Form) 데이터를 직접 처리하는 Server Actions의 동작 원리는 무엇이며, 이를 도입했을 때 기존 클라이언트 사이드 데이터 페칭 방식 대비 어떤 이점이 있나요?

* 의도: Next.js 최신 폼 처리 패러다임(RPC 메커니즘)을 이해하고, 자바스크립트 없이도 폼이 동작하는 '점진적 향상'의 개념을 알고 있는지 평가
* 키워드: `"use server"` `RPC(Remote Procedure Call)` `보일러플레이트 감소` `점진적 향상(Progressive Enhancement)`

<details>
<summary>답변</summary>

**Server Actions**는 `'use server'` 지시어로 정의된 서버 측 함수를, 클라이언트에서 마치 로컬 함수처럼 호출할 수 있게 하는 **RPC(Remote Procedure Call) 메커니즘**입니다.

**동작 원리**:
```tsx
// actions.ts
'use server';
export async function createPost(formData: FormData) {
  const title = formData.get('title');
  await db.post.create({ data: { title } });
  revalidatePath('/posts');
}

// page.tsx
export default function Page() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">작성</button>
    </form>
  );
}
```

빌드 시 Next.js가 서버 함수에 고유 엔드포인트(ID)를 부여하고, 클라이언트에서 폼 제출 시 해당 엔드포인트로 POST 요청을 보냅니다.

**이점**:
1. **보일러플레이트 감소**: API Route 생성, fetch 호출, 로딩/에러 상태 관리 등의 코드가 불필요합니다.
2. **점진적 향상(Progressive Enhancement)**: HTML `<form>`의 `action` 속성을 사용하므로, **자바스크립트가 로드되기 전에도 폼이 동작**합니다. JS가 로드되면 클라이언트에서 가로채어 SPA처럼 동작합니다.
3. **자동 캐시 무효화**: Server Action 내에서 `revalidatePath`/`revalidateTag`를 호출하면 관련 캐시가 즉시 갱신됩니다.

</details>

### Q29. 서버 액션은 엔드포인트가 숨겨져 있을 뿐 결국 외부에서 호출 가능한 API와 같습니다. 악의적인 사용자가 클라이언트를 거치지 않고 서버 액션을 직접 호출할 때를 대비해 어떤 보안 처리(권한 검증 등)를 해주어야 하나요?

* 의도: 서버 액션의 은닉성으로 인해 흔히 놓치기 쉬운 보안 취약점(인가 누락)을 방어할 수 있는 실무 지식 검증
* 키워드: `인가(Authorization)` `입력값 검증(Zod 활용)` `세션 체크` `클로저 변수 오염 주의`

<details>
<summary>답변</summary>

Server Action은 외부에서 POST 요청으로 직접 호출 가능한 **공개 엔드포인트**와 동일하므로, 반드시 서버 액션 내부에서 보안 처리를 해야 합니다.

**1. 인가(Authorization) - 세션 체크**:
```typescript
'use server';
export async function deletePost(postId: string) {
  const session = await getServerSession();
  if (!session) throw new Error('Unauthorized');

  const post = await db.post.findUnique({ where: { id: postId } });
  if (post.authorId !== session.user.id) throw new Error('Forbidden');

  await db.post.delete({ where: { id: postId } });
}
```

**2. 입력값 검증 (Zod 등 활용)**:
클라이언트에서 보내는 데이터를 절대 신뢰하지 않고, 서버에서 반드시 재검증합니다.
```typescript
const schema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1),
});

export async function updatePost(formData: FormData) {
  const parsed = schema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) throw new Error('Invalid input');
}
```

**3. 클로저 변수 오염 주의**:
서버 컴포넌트에서 Server Action에 서버 데이터를 클로저로 전달할 때, 해당 값이 클라이언트에 노출될 수 있습니다. 민감한 값은 클로저 대신 서버에서 직접 조회해야 합니다.

</details>

### Q30. 폼 제출이 완료되어 데이터베이스가 업데이트된 후, 사용자가 보는 화면(예: 모의 면접 리뷰 리스트)을 최신 상태로 갱신하기 위해 서버 액션 내부에서 어떤 Next.js 내장 함수들을 호출하시나요?

* 의도: Mutation 이후의 상태 동기화(Cache Invalidation) 흐름을 정확히 꿰뚫고 있는지 평가
* 키워드: `revalidatePath` `revalidateTag` `라우트 캐시 무효화`

<details>
<summary>답변</summary>

Server Action 내부에서 데이터를 변경한 후, 다음 함수들로 캐시를 무효화하여 화면을 최신 상태로 갱신합니다:

**1. `revalidatePath(path)`**:
특정 경로의 캐시를 무효화합니다. 해당 경로의 Data Cache와 Full Route Cache가 모두 갱신됩니다.
```typescript
'use server';
export async function submitReview(formData: FormData) {
  await db.review.create({ data: { ... } });

  revalidatePath('/reviews');           // 리스트 페이지 갱신
  revalidatePath('/reviews/[id]', 'page'); // 특정 페이지만 갱신
}
```

**2. `revalidateTag(tag)`**:
특정 태그가 붙은 모든 fetch 캐시를 무효화합니다. 여러 경로에 걸친 데이터를 한 번에 갱신할 수 있어 더 세밀한 제어가 가능합니다.
```typescript
export async function submitReview(formData: FormData) {
  await db.review.create({ data: { ... } });

  revalidateTag('reviews'); // 'reviews' 태그가 붙은 모든 캐시 무효화
}
```

**3. `redirect(path)`**:
캐시 무효화 후 사용자를 다른 경로로 리디렉션합니다.
```typescript
import { redirect } from 'next/navigation';
// ...
revalidatePath('/reviews');
redirect('/reviews'); // 최신 데이터가 반영된 리스트로 이동
```

Server Action 실행 후 Next.js가 자동으로 Router Cache(클라이언트)도 갱신하므로, 별도의 클라이언트 처리 없이 최신 상태가 반영됩니다.

</details>

### Q31. 복잡한 대시보드나 모달 창을 구현할 때, 패럴렐 라우트(Parallel Routes)와 인터셉팅 라우트(Intercepting Routes)를 조합하여 사용하면 기존의 상태(State) 기반 모달 대비 어떤 UX적/구조적 이점이 있나요?

* 의도: URL 기반의 상태 관리라는 고급 아키텍처 패턴을 이해하고, 공유 가능하고 뒤로 가기가 지원되는 완벽한 모달을 구현해 본 경험이 있는지 확인
* 키워드: `URL 공유(Deep Linking)` `브라우저 뒤로 가기 지원` `독립적 로딩/에러 처리` `@folder` `(..)folder`

<details>
<summary>답변</summary>

**기존 상태 기반 모달의 한계**:
- `useState`로 열림/닫힘을 관리하므로 URL에 상태가 반영되지 않습니다.
- URL을 공유하면 모달이 열리지 않고, 새로고침 시 모달이 사라집니다.
- 브라우저 뒤로 가기로 모달을 닫을 수 없습니다.

**패럴렐 라우트 + 인터셉팅 라우트의 이점**:

1. **URL 공유(Deep Linking)**: 모달이 URL에 연동되므로 `/photos/123`을 공유하면 해당 사진을 바로 볼 수 있습니다.

2. **브라우저 뒤로 가기 지원**: 모달이 라우트이므로, 뒤로 가기 버튼으로 자연스럽게 모달을 닫을 수 있습니다.

3. **독립적 로딩/에러 처리**: 각 슬롯(`@slot`)이 독립적으로 `loading.tsx`와 `error.tsx`를 가질 수 있습니다.

**구조 예시**:
```
app/
├── @modal/
│   ├── (..)photos/[id]/   ← 인터셉팅: 리스트에서 클릭 시 모달로 열림
│   │   └── page.tsx
│   └── default.tsx
├── photos/[id]/
│   └── page.tsx           ← 직접 접속 시 전체 페이지로 열림
└── layout.tsx             ← { children, modal } 두 슬롯 렌더링
```

4. **대시보드 활용**: 패럴렐 라우트만으로도 하나의 레이아웃에서 여러 독립적인 뷰(차트, 테이블, 알림 등)를 동시에 렌더링하고, 각각 독립적으로 로딩/에러 처리가 가능합니다.

</details>

### Q32. 인터셉팅 라우트를 사용하여 리스트 페이지 위에 모달을 띄웠습니다. 이 상태에서 사용자가 브라우저의 '새로고침(F5)' 버튼을 누르거나 해당 URL을 다른 사람에게 공유해서 직접 접속하면 화면이 어떻게 변하며, 그 이유는 무엇인가요?

* 의도: Next.js 라우팅의 핵심인 Soft Navigation(클라이언트 라우팅)과 Hard Navigation(서버 초기 로딩)의 차이점 이해도 검증
* 키워드: `Soft Navigation(모달 오버레이)` `Hard Navigation(전체 페이지 전환)` `컨텍스트 분리`

<details>
<summary>답변</summary>

**Soft Navigation (클라이언트 라우팅으로 접근한 경우)**:
리스트 페이지에서 항목을 클릭하면 클라이언트 사이드 라우팅이 발생합니다. 이때 인터셉팅 라우트(`(..)photos/[id]`)가 요청을 가로채서, 리스트 페이지 위에 **모달 오버레이**로 콘텐츠를 표시합니다. 리스트 페이지는 뒤에 그대로 유지됩니다.

**Hard Navigation (새로고침 또는 URL 직접 접속)**:
새로고침(F5)하거나 URL을 직접 입력하면 **서버에서 처음부터 페이지를 로드(Hard Navigation)**합니다. 이 경우 인터셉팅이 발생하지 않고, 원본 라우트(`photos/[id]/page.tsx`)가 **전체 페이지(Full Page)**로 렌더링됩니다.

**이유**:
인터셉팅 라우트는 **클라이언트 라우팅(Soft Navigation) 시에만** 동작합니다. 클라이언트 Router가 존재해야 "이 요청을 가로채서 모달로 보여줄지" 판단할 수 있기 때문입니다. Hard Navigation은 서버가 URL에 매칭되는 실제 라우트를 그대로 렌더링합니다.

이것이 인터셉팅 라우트의 장점입니다: 앱 내 탐색 시엔 모달로 빠르게 미리보기, URL 공유/새로고침 시엔 전체 페이지로 완전한 콘텐츠를 제공합니다.

</details>

### Q33. 패럴렐 라우트(`@slot`)를 사용할 때 특정 슬롯이 현재 URL과 일치하지 않는 상태가 되면 404 에러가 발생할 수 있습니다. 이를 방지하기 위해 사용하는 `default.tsx` 파일의 역할은 무엇인가요?

* 의도: 다중 뷰 라우팅 시 발생할 수 있는 Unmatched Routes 에러 핸들링 역량 평가
* 키워드: `Unmatched slots` `Fallback UI` `이전 상태 유지`

<details>
<summary>답변</summary>

패럴렐 라우트에서 여러 슬롯(`@slot`)이 동시에 렌더링될 때, URL이 변경되면 일부 슬롯은 현재 URL에 매칭되는 `page.tsx`가 없을 수 있습니다. 이를 **Unmatched Slots**라고 합니다.

**Soft Navigation 시**: Next.js가 이전에 렌더링된 슬롯의 상태를 그대로 유지합니다. 문제 없음.

**Hard Navigation 시 (새로고침, 직접 접속)**: 이전 상태가 없으므로 매칭되지 않는 슬롯에 대해 **404 에러**가 발생합니다.

**`default.tsx`의 역할**:
`default.tsx`는 슬롯이 현재 URL과 매칭되지 않을 때 보여줄 **Fallback UI**를 정의합니다.

```
app/
├── @modal/
│   ├── photos/[id]/page.tsx   ← /photos/123일 때 매칭
│   └── default.tsx            ← 매칭 안 될 때 (보통 null 반환)
├── @sidebar/
│   ├── page.tsx
│   └── default.tsx
└── layout.tsx
```

```tsx
// @modal/default.tsx
export default function Default() {
  return null; // 모달이 없는 상태에서는 아무것도 렌더링하지 않음
}
```

`default.tsx`를 제공하면 Hard Navigation 시에도 매칭되지 않는 슬롯이 404 대신 Fallback UI를 표시하여 안정적으로 동작합니다.

</details>

### Q34. 웹 성능 지표 중 하나인 LCP(Largest Contentful Paint)를 개선하기 위해 `next/image` 컴포넌트가 내부적으로 수행하는 최적화 작업들은 무엇인가요? 일반 `<img>` 태그를 사용할 때와 비교하여 설명해 주세요.

* 의도: 단순 프레임워크 사용법을 넘어 Core Web Vitals(웹 성능 지표)에 대한 이해가 있는지, 그리고 프론트엔드 최적화의 기본인 이미지 처리 원리를 아는지 검증
* 키워드: `WebP/AVIF 자동 변환` `Lazy Loading` `사이즈 최적화(Srcset)` `CLS(Cumulative Layout Shift) 방지`

<details>
<summary>답변</summary>

`next/image`는 일반 `<img>` 태그 대비 다음과 같은 자동 최적화를 수행합니다:

**1. 포맷 자동 변환 (WebP/AVIF)**:
브라우저의 `Accept` 헤더를 확인하여 WebP, AVIF 등 최신 경량 포맷으로 자동 변환합니다. 같은 품질에서 파일 크기가 크게 줄어 LCP가 개선됩니다.

**2. 반응형 사이즈 최적화 (Srcset)**:
디바이스 크기에 맞는 적절한 크기의 이미지를 자동으로 생성하고 `srcset`을 설정합니다. 모바일에서 불필요하게 큰 이미지를 다운로드하지 않습니다.

**3. Lazy Loading (지연 로딩)**:
기본적으로 뷰포트 밖의 이미지는 지연 로딩합니다. 화면에 보이는 이미지부터 로드하여 초기 로딩 속도를 개선합니다. LCP 대상 이미지에는 `priority` 속성을 설정하여 즉시 로드합니다.

**4. CLS(Cumulative Layout Shift) 방지**:
`width`와 `height`를 필수로 지정하게 하여, 이미지 로드 전에 공간을 미리 확보합니다. 이미지가 늦게 로드되어 레이아웃이 밀리는 현상을 방지합니다.

**5. 서버 사이드 최적화**:
이미지 최적화가 서버(또는 CDN)에서 수행되어, 원본 이미지를 클라이언트에 보내고 브라우저에서 처리하는 부하를 줄입니다.

</details>

### Q35. 유저 프로필 사진처럼 외부 서버(S3 등)에서 불러와서 너비와 높이를 미리 알 수 없는 이미지를 `next/image`로 렌더링하려면 어떤 속성을 사용해야 하며, 부모 컴포넌트에는 어떤 CSS 처리가 필요한가요?

* 의도: 런타임에 동적으로 크기가 결정되는 이미지를 CLS 없이 안전하게 렌더링하는 실무 테크닉 확인
* 키워드: `fill 속성` `부모 요소 position: relative` `object-fit: cover`

<details>
<summary>답변</summary>

`width`와 `height`를 미리 알 수 없는 외부 이미지에는 **`fill` 속성**을 사용합니다.

```tsx
<div style={{ position: 'relative', width: '200px', height: '200px' }}>
  <Image
    src="https://s3.amazonaws.com/bucket/profile.jpg"
    alt="프로필 사진"
    fill
    style={{ objectFit: 'cover' }}
    sizes="200px"
  />
</div>
```

**핵심 포인트**:

1. **`fill` 속성**: `width`/`height` 대신 사용하며, 이미지가 부모 요소의 크기에 맞춰 채워집니다 (`position: absolute` + `width: 100%` + `height: 100%`와 유사).

2. **부모 요소 `position: relative`**: `fill` 이미지는 absolute로 배치되므로, 부모 요소에 반드시 `position: relative`(또는 `absolute`, `fixed`)를 설정해야 정확한 위치에 렌더링됩니다.

3. **`object-fit: cover`**: 이미지의 비율을 유지하면서 영역을 채웁니다. `contain`을 사용하면 비율을 유지하되 영역 안에 맞춥니다.

4. **`sizes` 속성**: 반응형 이미지 최적화를 위해 실제 표시될 크기를 브라우저에 알려줍니다. 이 정보를 바탕으로 적절한 크기의 이미지를 요청합니다.

5. **`next.config.js`에 외부 도메인 등록**: 외부 이미지를 사용하려면 `images.remotePatterns`에 해당 도메인을 허용해야 합니다.

</details>

### Q36. 프로젝트의 SEO(검색 엔진 최적화)를 위해 각 페이지마다 동적으로 메타 데이터나 Open Graph(OG) 이미지를 생성하려면 Next.js의 어떤 기능을 활용해야 하나요?

* 의도: 동적 라우팅 환경에서의 SEO 메타 태그 주입 및 `next/og` 활용 능력 파악
* 키워드: `generateMetadata` `동적 OG 이미지(ImageResponse)` `크롤러 대응`

<details>
<summary>답변</summary>

**1. `generateMetadata` 함수 (동적 메타 데이터)**:
각 페이지의 `page.tsx`(또는 `layout.tsx`)에서 `generateMetadata` 함수를 export하여 동적으로 메타 데이터를 생성합니다.

```tsx
// app/posts/[id]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.id);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [`/api/og?title=${encodeURIComponent(post.title)}`],
    },
  };
}
```

Next.js가 이 함수의 반환값을 `<head>` 태그의 메타 태그로 자동 변환합니다. 데이터 페칭도 가능하므로 DB에서 가져온 데이터를 기반으로 동적 메타 정보를 생성할 수 있습니다.

**2. 동적 OG 이미지 (`ImageResponse`)**:
`next/og`의 `ImageResponse`를 활용하여 JSX로 OG 이미지를 동적 생성합니다.

```tsx
// app/api/og/route.tsx
import { ImageResponse } from 'next/og';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title');

  return new ImageResponse(
    <div style={{ fontSize: 48, background: 'white', width: '100%', height: '100%' }}>
      {title}
    </div>,
    { width: 1200, height: 630 }
  );
}
```

이미지를 별도로 디자인 도구에서 만들 필요 없이, 코드로 동적 생성하여 각 게시글마다 고유한 OG 이미지를 제공할 수 있습니다. 생성된 이미지는 정적으로 캐싱되어 성능에도 부담이 없습니다.

</details>
