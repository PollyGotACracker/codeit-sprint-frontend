# Animation

- [https://cubic-bezier.com/](https://cubic-bezier.com/)
- [https://reactbits.dev/](https://reactbits.dev/)

## transition과의 차이점

| 차이        | transition           | keyframes             |
| ----------- | -------------------- | --------------------- |
| 시점별 조작 | 시작과 끝만 가능     | 중간 과정도 조작 가능 |
| 반복        | 불가능               | 가능                  |
| 자동실행    | 불가능(trigger 필요) | 가능                  |

## `animation-fill-mode`

- `none`: 애니메이션 시작 전후로 기존 스타일 적용(기본값)
- `forwards`: 애니메이션이 끝나면 마지막 keyframe 스타일 유지(시작 전은 기존 스타일 적용)
- `backwards`: 애니메이션 시작 전 첫 keyframe 스타일 적용(끝난 후 기존 스타일 적용)
- `both`: 시작되기 전 `backwards`, 끝난 후 `forwards` 적용

## 애니메이션 라이브러리

1. motion(구 Framer motion)

- CSS 로만 구현하기 어려운 기능 제공

```tsx
// client components
import { motion } from "motion/react";
```

```tsx
// server components
import * as motion from "motion/react-client";
```

- 최적화

```tsx
// /lib/feature.ts
// domAnimation(기본 기능), domMax(추가적인 복잡한 기능)
// motion 대신 react-m 패키지로부터 m 컴포넌트를 사용(간소화)

import { domAnimation } from "motion/react";
export default domAnimation;
```

```tsx
// /providers/LazyMotionProvider.tsx
"use client";

import { LazyMotion } from "motion/react";

// 동적 import/비동기 로딩을 통해 페이지 로드 직전에 필요한 기능만 로드하도록 설정(chunking)
const loadFeatures = () => import("@/lib/feature").then((res) => res.default);

export default function LazyMotionProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  // LazyMotion으로 LazyLoading, strict 옵션으로 m 컴포넌트만 사용하도록 강제
  return (
    <LazyMotion features={loadFeatures} strict>
      {children}
    </LazyMotion>
  );
}
```

```tsx
// /app/page.tsx
import * as m from "motion/react-m";
```

2. gasp
   - 복잡한 타임라인 기반 애니메이션 구현
   - motion 보다 더 오래되고 많이 사용되는 라이브러리

```shell
npm install gsap @gsap/react
```

3. three.js
   - JS로 작성된 WebGL 기반의 3D 그래픽 라이브러리
   - c.f. react-three-fiber, react-three-drei

```shell
npm install three
npm i --save-dev @types/three
```

4. remotion:react, typescript 코드로 비디오 생성
5. react-countup: 숫자 카운트 애니메이션
6. 캐러셀 라이브러리: React Slick, Embla Carousel, Swiper
7. react-intersection-observer
8. [Lenis](https://lenis.darkroom.engineering/)
9. [React Bits](https://reactbits.dev/)
