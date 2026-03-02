# Observer

## IntersectionObserver

- [참고 1](https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API)
- [참고 2](https://developer.mozilla.org/ko/docs/Web/API/IntersectionObserver)

```ts
const callback = (entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      console.log({ entry });
      // 단 한번만 실행해야 할 경우 콜백 내에서 옵저버를 중단할 수 있다.
      // observer.unobserve(entry.target);
    }
  });
};
const options = {
  root: document.querySelector("#parent"),
  rootMargin: "0px",
  threshold: 1.0,
};
const observer = new IntersectionObserver(callback, options);

const element = document.querySelector("#target");
if (element) {
  observer.observe(element);
}

// React useEffect cleanup 에 반드시 작성
// 1. 특정 요소 하나의 옵저버 중단
// if (element) {
//  observer.unobserve(element);
// }

// 2. 감시 중인 모든 요소에 대해 옵저버 중단
// observer.disconnect();
```

### `entries`

- 감지하고 있는 모든 요소
- `IntersectionObserver.takeRecords()` 에서 반환하는 값
- 여러 요소에 동일 observer 적용 가능

```js
document.querySelectorAll(".target").forEach((el) => {
  observer.observe(el);
});
```

### options

- `root`
  - 교차 감지의 뷰포트 역할을 할 조상 요소
  - `null` 이면 브라우저 뷰포트가 기준
  - 반드시 감시 대상(target)의 조상 요소여야 함

- `rootMargin`
  - root 요소의 마진을 가상으로 확장/축소(offset)
  - CSS margin처럼 작성: `"10px 20px 30px 40px"` (top right bottom left)
  - 양수: 감지 범위 확장 (실제 교차 전에 미리 감지)
  - 음수: 감지 범위 축소 (더 깊이 들어와야 감지)
  - 기본값: `"0px 0px 0px 0px"`

- `threshold`
  - 콜백이 실행되는 대상의 가시성 백분율 (0.0 ~ 1.0)
  - `1.0`: target이 100% 완전히 root 안에 들어왔을 때 콜백 실행
  - `0`: 1px이라도 걸치면 실행
  - `0.5`: 50% 들어왔을 때 실행
  - 배열도 가능: `[0, 0.25, 0.5, 0.75, 1.0]` 각 지점마다 콜백 실행
  - 기본값: `0.0`
