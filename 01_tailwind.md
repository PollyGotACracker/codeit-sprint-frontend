# Tailwind

- [CSS 변환](https://divmagic.com/ko/tools/css-to-tailwind)
- [Theme variables](https://tailwindcss.com/docs/theme#theme-variable-namespaces)
- [속성 치트시트](https://www.creative-tim.com/twcomponents/cheatsheet)
- nextjs, vercel 호환
- 빌드 시 실제 사용되는 클래스만 포함(Just In Time, Tree Shaking)
- 런타임 오버헤드가 매우 적음

## 주의점

- `bg-${color}-500` 방식 불가능: JS가 아닌 단순 텍스트로 인식
- tailwind 스타일을 prop 으로 받아서 사용할 경우, 기본값으로 적용된 동일 스타일에 일부 override 되지 않음

## 프로젝트 설정

- 속성 드롭다운 표시
  - Tailwind CSS IntelliSence 확장프로그램 설치
- 속성 자동 정렬
  1. prettier, prettier-plugin-tailwindcss devDependencies 로 설치
  2. .prettierrc 또는 package.json 에 플러그인 설정
  3. IDE 설정 확인
     - Default Formatter: prettier, Format On Save: true
  4. IDE 재시작

```shell
npm install -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

```json
// package.json
{
  "prettier": {
    "plugins": ["prettier-plugin-tailwindcss"]
  }
}
```

## 지시어(v4)

### `@theme`

```css
/* global.css */
@import "tailwindcss";

@theme {
  /* 커스텀 변수 */
  --color-mycolor-500: #c71eff;
  /* 기존 변수 오버라이딩 */
  --breakpoint-md: 800px;
  /* 기존 변수 초기화 및 새로운 변수 정의 */
  --breakpoint-*: initial;
  --breakpoint-tablet: 800px;
}

@utility card {
  background-color: var(--color-mycolor-500);
}
```

### `@utility`

- hover: 도 적용 가능, class="hover:my-class"
- .my-class 사용 시 hover:my-class 불가

```css
@import "tailwindcss";

@utility my-class {
  background-color: black;
  @apply p-5;
}
```

### `@layer`

- class 에서 작성하는 스타일이 복잡해질 때 사용
- 스타일 적용 우선순위 구조화: components > base
  - base: 기본 태그 스타일에 주로 사용
  - components: 특정 컴포넌트 등 스타일에 주로 사용

```css
@import "tailwindcss";

/* 
  components 스타일이 base 스타일을 overriding
  .h1 처럼 클래스 사용 가능 
*/
@layer components {
  h1 {
    @apply text-2xl;
  }
}

@layer base {
  h1 {
    @apply text-3xl;
  }
}
```

### `@varient`

- tailwind 선택자: `dark:...`, `hover:...` ...
- 중첩 작성 가능

```css
@import "tailwindcss";

.my-element {
  background: black;
  @variant dark {
    background: white;
    @variant hover {
      background: red;
    }
  }
}

/*
.my-element { background: black; }

@media (prefers-color-scheme: dark) {
  .my-element { background: white; }
}

@media (prefers-color-scheme: dark) {
  .my-element:hover { background: red; }
}
*/
```

### `@apply`

- 클래스 스타일 또는 `@utility` 스타일 등 사용 가능

```css
@import "tailwindcss";

@utility my-class {
  background-color: black;
  @apply p-5;
}
```

## 조건부 클래스

- `clsx`(clsx)
  - 조건부 스타일 구조화, 조건에서 반환되는 falsy 값 제거
- `twMerge`(tailwind-merge)
  - 스타일 overriding 문제 해결
- `cva`(class-variance-authority) 또는 `tv`(tailwind-variants)
  - 기본값 및 스타일을 객체 형태로 정의
- 위 API들을 하나로 묶어 유틸 함수(`cn` 등)로 정의하여 사용

```tsx
export default function Card({
  children,
  variant = "default",
  className,
}: CardProps) {
  const styles = twMerge(clsx(variants({ variant, className })));
  return <div className={styles}>{children}</div>;
}

const variants = cva("props 외 기본 스타일 속성...", {
  variants: {
    variant: {
      default: "...",
      outlined: "...",
      elevated: "...",
    },
  },
  // 기본값 적용
  defaultVariants: {
    variant: "default",
  },
});
```

## 클래스, 선택자

- [참고](https://tailwindcss.com/docs/hover-focus-and-other-states)

- 익숙하지 않은 선택자만 작성하였음
  - `has-[선택자]`(부모): `has()`, 자식에게 해당 선택자가 있을 경우 부모에게 스타일 적용
  - `focus-within:`(부모):`:focus-within`, 자식 input에 focus 되는 경우 부모에게 스타일 적용
  - `group`(부모), `group-[상태 선택자]:`(자식): 부모가 해당 상태일 경우 자식에게 스타일 적용
  - `in-range`: input에서 값이 `min`, `max` 사이일 때

### 기타 웹 접근성

- `motion-reduce:hidden`
- `aria-checked:`

## 폰트

### @font-face

```css
@import "tailwindcss";

@font-face {
  font-family: "Pacifico";
  font-style: normal;
  font-weight: 100 900;
  font-display: swap;
  src: url("../fonts/Pacifico.woff2") format("woff2");
}

/* tailwind 커스텀 변수 정의 */
@theme {
  /* font-family 이름 */
  --font-pacifico: "Pacifico";
}
```

### @import

```css
/* @import url() 은 최상단 작성 */
@import url("https://fonts.googleapis.com/css2?family=Pacifico&display=swap");
@import "tailwindcss";

/* tailwind 커스텀 변수 정의 */
@theme {
  /* url에서 family 이름 */
  --font-pacifico: "Pacifico";
}
```

### next/font/google 또는 next/font/local

```css
@import "tailwindcss";

/* 이미 있는 변수를 사용할 경우 반드시 inline 으로 작성 */
/* `variable` 에서 사용한 변수명 사용 */
@theme inline {
  --font-pacifico: var(--font-pacifico);
}
```

## 참고

### CSS in JS

- emotion, styled-components
- js변수나 props를 사용한 쉬운 동적 스타일링
- 컴포넌트 단위 클래스명 스코핑
- js 번들 크기 증가로 인한 초기 로딩 시간 증가
- js 동적 스타일링 및 스타일 실시간 처리로 인하여 실행 시점에서 연산 부하

### UI 라이브러리

#### 스타일 X: 기능만 제공, 스타일 자유 구현

1. Headless UI: Tailwind CSS 팀에서 개발하여 호환성 높음
2. Radix UI: 고유 디자인 시스템과 함께 사용. 많은 컴포넌트 제공

#### 스타일 O: 번들 크기 상대적 높음, 커스터마이징 어려움

1. Material UI: 컴포넌트 수 많음, 사용 빈도 높음. Tailwind CSS 와 호환성 낮음
2. HeroUI(NextUI): Tailwind CSS, React Aria 기반. 완전한 컴포넌트 제공. 상대적으로 작은 커뮤니티
3. shadcn/ui: Tailwind CSS 기반. 직접 수정 가능한 컴포넌트 제공. 필요 컴포넌트만 다운로드
