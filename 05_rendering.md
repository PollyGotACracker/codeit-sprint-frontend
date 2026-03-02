# Rendering

- [web.dev: 렌더링 성능](https://web.dev/articles/rendering-performance?hl=ko)

## Critical Rendering Path

- [React.js의 렌더링 방식 살펴보기 - 이정환](https://www.youtube.com/watch?v=N7qlk_GQRJU&t=299s)

### DOM(Document Object Model): 문서 객체 모델

- HTML을 브라우저가 해석하기 쉬운 방식으로 변환한 객체 트리(배치, 모양 등)
- CSSOM 또한 css 스타일 코드를 브라우저가 해석하기 쉽게 변환한 것

### 렌더링 과정

1. HTML, CSS 파싱: DOM, CSSOM tree 생성
2. Render Tree 생성: DOM과 CSSOM 결합, blueprint 역할
3. 레이아웃(Layout): Render Tree를 기반으로 실제 웹 페이지에 요소들의 배치를 결정
4. 페인팅(Painting): Layout의 결과를 보고 요소들을 실제 화면에 그려냄

### Reflow, Repaint

- JS가 DOM을 수정하면 업데이트가 발생하는데, 이때 Critical Rendering Path가 다시 실행됨
- Layout과 Painting 단계는 많은 비용(연산)이 필요한 단계이다.
- Reflow: Layout 재계산
- Repaint: Painting 재계산
- DOM 수정은 최소화되어야 한다.
  - 반복문 내에서 모든 `li` 태그 내부 텍스트를 변경하지 않고,  
    `li` 변경이 반영된 직렬화한 내용물을 `ul` 태그에 한번 삽입함으로써 계산 속도 최적화

### React virtual DOM

- DOM 수정을 최소화하여 대부분의 상황에서 충분히 빠른 업데이트를 구현
- Render Phase => Commit Phase
- Nextjs는 SSR/SSG 등 방식으로 서버에서 HTML 문자열을 바로 전달, 브라우저에 도착하면 JS 로드=>hydration=>virtual DOM 처리

1. Render Phase: 컴포넌트를 계산하고 업데이트 사항을 파악하는 단계

- React 컴포넌트가 렌더링해야 하는 UI를 Virtual DOM 이라는 객체 값으로 변환하는 과정
  - 컴포넌트가 호출될 때 DOM 반환값은 React Element 라는 객체이다.
  - React Element는 UI가 렌더링하고자 하는 모든 정보를 포함한다.

  ```js
  {
  type: "div",
  key: null,
  ref: null,
  props: {
    id: "main",
    children: {
      type: "p",
      key: null,
      ref: null,
      props: {
        children: "Hello",
      }
      // ...
    }
  }
  }
  ```

  - virtual DOM은 실제 DOM이 아닌, React Element 객체 값의 집합이다.

2. Commit Phase: 변경사항을 실제 DOM에 반영하는 단계

- Virtual DOM을 Actual DOM에 반영
- 이후 Critical Rendering Path 과정을 거침

### 업데이트 발생

- 업데이트 발생 시 Render Phase 를 처음부터 다시 실행하여 새로운 Virtual DOM을 생성
- Diffing: 새롭게 반환된 React Element 로부터 변경사항이 반영된 Next VDOM 과 이전 렌더링에 사용된 Prev VDOM을 비교
- Reconciliation(재조정): 동시 발생한 업데이트 부분만을 모아 Actual DOM에 한 번만 반영

## requestAnimationFrame

- `setTimeout` / `setInterval` 은 지연 가능성이 있으며, 브라우저의 렌더링 사이클과 동기화되지 않아 프레임 드랍이 발생할 수 있음
- `requestAnimationFrame`을 사용하여 브라우저 렌더링 사이클과 동기화된 타이밍에 애니메이션을 실행함
  - 프레임 드랍 없이 부드러운 애니메이션 구현 가능
  - 탭이 비활성화되면 자동으로 실행을 멈춰 불필요한 리소스 낭비 방지

```ts
let animationId: number;

function animate() {
  // 애니메이션 로직 작성

  // 다음 프레임 요청
  animationId = requestAnimationFrame(animate);
}

// 애니메이션 시작
animationId = requestAnimationFrame(animate);

// React useEffect cleanup 에 반드시 작성
// 애니메이션 중지
// cancelAnimationFrame(animationId);
```

## `will-change` 속성

브라우저가 특정 요소를 **미리 GPU 레이어로 분리**해 애니메이션 준비를 완료하도록 힌트를 줌

> `will-change` 가 없으면 브라우저는 애니메이션 실행 직전에 레이어를 생성하므로, 첫 시작 시 버벅임이 발생할 수 있음

### 개발자도구에서 확인 방법

**More Tools → Layers** 패널

| 상태               | 동작                                            |
| ------------------ | ----------------------------------------------- |
| `will-change` 있음 | 애니메이션 실행 전부터 레이어가 분리되어 있음   |
| `will-change` 없음 | 애니메이션 실행 후 레이어 생성 → 종료 후 사라짐 |

### 주의사항

`will-change`를 남용하면\*GPU 메모리 낭비 및 성능 저하 발생

### 효과적인 사용 상황

- 반복적이고 지속적인 애니메이션
- 이미지, gradient 등 paint 비용이 큰 요소의 애니메이션

### JS로 동적 제어하는 방법 (권장)

```js
element.addEventListener("mouseenter", () => {
  element.style.willChange = "transform";
});

element.addEventListener("animationend", () => {
  element.style.willChange = "auto"; // 애니메이션 종료 후 제거
});
```

> 애니메이션 실행 직전에 부여하고, 끝난 후 제거하여 GPU 메모리 낭비를 방지
