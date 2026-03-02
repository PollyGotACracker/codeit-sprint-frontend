# Javascript

- [visualizejs.com](https://visualizejs.com/javascript)

## `globalThis`

### 기존의 환경별 전역 객체

- 브라우저: `window`
- Node.js: `global`
- Worker(Web/Service): `self`
- 여러 환경에서 동작해야 하는 라이브러리를 만들 때 분기가 필요했다.

### `globalThis` 의 등장

- `globalThis` 는 ECMAScript 표준이며 모든 실행 환경에서 사용 가능하다.
- [참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/globalThis)

## Web Worker 와 Service Worker

- 둘 다 Worker 개념을 기반으로 한다.
- 추가 스레드에서 실행되므로 메인 스레드를 block 하지 않고 실행된다.
- Window 나 Document 객체에 접근할 수 없어 DOM과 상호작용이 불가능하다.

### Worker: 백그라운드 스레드

- JavaScript는 기본적으로 싱글 스레드이다.
- 브라우저에서 JS 코드는 메인 스레드 하나에서 돌아가는데, 이 메인 스레드는 UI 렌더링도 담당한다.
- 그래서 JS에서 무거운 작업을 돌리면 화면이 멈춘다(버벅임, 클릭 안 먹힘 등).
- Worker는 이 문제를 해결하기 위해 브라우저가 제공하는 별도의 백그라운드 스레드이다.
- 메인 스레드랑 분리되어 있으므로 UI 렌더링이 끊기지 않는다.
- 대신 메인 스레드와 완전히 격리되어 있기 때문에 DOM(Document, Window)에는 접근할 수 없고, `postMessage`로 메인 스레드와 데이터를 주고받는다.

### Web Worker: 연산 보조

- UI block 을 피하기 위해 무거운 연산이 필요한 작업을 보조 스레드에서 실행할 때 사용된다.
- 수명이 자신이 속한 탭과 밀접하게 결합되어 있다.

### Service Worker: 네트워크 프록시

- 브라우저와 서버 사이에서 네트워크 요청을 가로챌 수 있다(fetch event).
- 이를 통해 응답을 캐시하고, 캐시된 응답을 반환할 수 있어 오프라인 지원이 가능하다.
- 수명이 탭과 독립적이므로 탭이 닫혀 있어도 푸시 알림 수신(push event)이나 백그라운드 동기화가 가능하다.
