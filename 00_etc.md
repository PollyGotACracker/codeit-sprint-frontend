# ETC

## 웹 접근성

- HTML 시맨틱 마크업
- `alt` 속성: 대체 텍스트. 스크린 리더, 이미지 표시가 되지 않는 상황 등에서 이미지 의미 전달
- `<inputDOM>.focus()`;
- ARIA 속성:
  - HTML 기본 시맨틱이 부족할 때
  - 동적 콘텐츠에서의 정보 제공: React 컴포넌트, AJAX 로드 UI
  - JS로 토글 시 값도 반드시 상태를 업데이트해야 함
  - [role, state, property 정리](https://www.digitala11y.com/wai-aria-1-1-cheat-sheet/)
  - [치트 시트](https://aria.markuplint.dev/)
    > e.g. `role-"dialog"`: 모달창임을 알림
    > `aria-modal="true"`: 현재 이 창이 활성화되어 있으므로 배경이 되는 콘텐츠를 무시하라고 알림
    > `arai-labelledby="<자식 요소의 id>"`: 모달 제목을 알림
- metadata: SEO 및 접근성 향상. 콘텐츠 파악 가능
  - static metatdata
  - generated metadata: 동적 데이터 기반 생성 [참고](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function)

## Atomic Design

- 화학의 원자 개념에서 영감을 받은 UI 컴포넌트 설계 방법론
- UI를 작은 단위부터 쌓아 올리는 Bottom-Up 방식

### 5단계 구조

```
Atoms (원자)
  └─ 더 이상 분리할 수 없는 최소 단위
  └─ 예: Button, Input, Label, Icon

Molecules (분자)
  └─ Atom 2개 이상의 조합
  └─ 예: SearchBar (Input + Button), FormField (Label + Input)

Organisms (유기체)
  └─ Molecule/Atom의 복합 조합, 독립적인 UI 섹션
  └─ 예: Header (Logo + Nav + SearchBar), ProductCard

Templates (템플릿)
  └─ Organism을 배치한 페이지 레이아웃 (실제 데이터 없음)
  └─ 예: BlogLayout, DashboardLayout

Pages (페이지)
  └─ Template에 실제 데이터를 주입한 최종 결과물
  └─ 예: HomePage, UserProfilePage
```

### 장점

- 재사용성 극대화
- 디자인 시스템과 궁합 좋음
- 작은 단위부터 테스트 용이
- UI 일관성 유지
- 단계 축소 및 커스텀 그룹화로 유연한 적용 가능

### 단점

- 컴포넌트 분류 기준이 주관적
- 과도한 추상화로 복잡도 증가 가능
- Molecule / Organism 경계 모호 (기능이 같아도 크기로 폴더가 나뉨)
- Props Drilling 심화 (organisms → templates → pages)
- 팀마다 해석 차이로 일관성 유지 어려움

- 단계를 축소시키거나 다른 방식으로 그룹화하는 방법도 가능하므로, 상황과 요구사항에 맞게 적용할 것

## Feature-Sliced Design (FSD)

- 기능(Feature) 단위로 코드를 분리하는 아키텍처 방법론
- 대규모 프로젝트에서 팀 간 충돌을 줄이고 모듈 독립성을 높이기 위해 설계됨

### 레이어 구조 (위 → 아래로 의존)

```
app/         → 앱 초기화, 라우팅, 글로벌 프로바이더
pages/       → 각 페이지 컴포넌트
widgets/     → 여러 feature를 조합한 독립적 UI 블록
features/    → 사용자 인터랙션, 비즈니스 로직 단위
entities/    → 도메인 모델 (User, Product 등)
shared/      → 재사용 가능한 UI, 유틸, API 설정 등
```

> **규칙:** 상위 레이어는 하위 레이어를 참조할 수 있지만, **하위 → 상위 참조는 금지**

### 슬라이스(Slice) & 세그먼트(Segment)

각 레이어는 **슬라이스**(도메인별 폴더)로 나뉘고, 슬라이스 내부는 **세그먼트**로 구성됨

```
features/
  auth/
    ui/        → 컴포넌트
    model/     → 상태, 비즈니스 로직
    api/       → API 호출
    lib/       → 헬퍼 함수
    index.ts   → Public API (외부 노출만 여기서)
```

### 장점

- 기능별 독립성 높음
- 대규모 팀에 적합
- 코드 위치 예측 가능
- 리팩토링 범위 명확

### 단점

- 초기 러닝커브 있음
- 소규모 프로젝트엔 과함
- 레이어 경계 판단이 모호할 때 있음

## Compound Component Pattern (합성 컴포넌트 패턴)

- 많은 props 를 정의하기보다 `children` 으로 내부 요소 구성을 위임
- 세부 요소의 스타일링은 작은 컴포넌트로 분리
  - `Modal.Content = ModalContent` 형태로 정적 프로퍼티 할당 후 export
  - 함수 객체에 프로퍼티를 추가하는 것과 동일한 원리
- UI 라이브러리의 컴포넌트 구조를 참고하여 작성

```tsx
// 각 세부 요소 컴포넌트
const ModalHeader = ({ children }) => <div className="header">{children}</div>;
const ModalFooter = ({ children }) => <div className="footer">{children}</div>;

// 메인 컴포넌트에 정적 프로퍼티로 할당
const Modal = ({ children }) => <div className="modal">{children}</div>;
Modal.Header = ModalHeader;
Modal.Footer = ModalFooter;

export default Modal;
```

```tsx
// 사용
<Modal>
  <Modal.Header>제목</Modal.Header>
  <p>내용</p>
  <Modal.Footer>닫기</Modal.Footer>
</Modal>
```

## 기타

- 게으른 초기화 (Lazy Initialization): 콜백을 통해 상태 초기값을 지연 계산하는 패턴

- PostCSS: JS 플러그인으로 CSS를 변환·처리하는 트랜스파일 도구
- Panda CSS: CSS-in-JS 방식의 타입 안전 스타일링 라이브러리

- BFF 패턴 (Backend for Frontend): 프론트엔드 전용 백엔드 서버를 두는 아키텍처 패턴
- ADR (Architecture Decision Record): 아키텍처 의사결정 과정과 근거를 문서화하는 기록
- A/B 테스트: 두 가지 버전을 비교해 더 높은 반응을 보이는 콘텐츠를 선택하는 방법

### AI 용어

- Prompt Engineering: AI 모델의 출력을 최적화하기 위해 입력 프롬프트를 설계하는 기법
- Harness Engineering: 테스트 환경 및 실험 인프라를 설계·관리하는 기법
- MCP (Model Context Protocol): AI 모델과 외부 도구·데이터를 연결하는 표준 프로토콜
