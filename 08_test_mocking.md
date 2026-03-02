# Test - Mocking (Jest)

- 가짜 객체를 사용하여 테스트를 진행하는 기법
- 핵심적인 기능에만 집중하고 불필요한 부분은 단순화(e.g. 운전면허 기능시험)
- 실제 API 호출로 인해 발생하는 문제 해결
  - 테스트 실행 속도 지연
  - 비용 발생
  - 데이터 변경 가능성
  - 네트워크나 서버 상태에 따른 테스트 실패 가능성
  - 에러 상황이나 특수한 응답 관련 테스트의 어려움
- 실제로 재현하기 어려운 환경에서 동작하는 기능을 테스트 가능(재난, 화재 등)

## 에러

> Matcher error: received value must be a mock or spy function

- `toHaveBeenCalled...` 같은 호출 추적 matcher 를 사용하려면,  
  `jest.fn()` 이나 `jest.spyOn()` 을 사용하여 함수를 모킹해야 한다.

## 모킹 API

### `jest.fn()`

- 빈 모킹 함수로 만들고 내부 구조를 다시 구현할 때
- `jest.spyOn()` 과 달리 원래 기능으로 복구하기 어려움

```js
const original = obj.greet;
obj.greet = jest.fn();
// 테스트 코드 ...
// 테스트가 끝난 후 복구하여 다음 테스트에 영향을 주지 않도록 함
obj.greet = original; // 직접 오버라이딩 필요
```

#### `.mockResolvedValue()`

- 가짜 함수인 `jest.fn()` 이 Promise 를 반환하도록 설정하는 메서드
- 비동기 함수 결과를 모방하여 반환값을 정의

```js
global.fetch = jest.fn().mockResolvedValue({
  ok: true,
  json: jest.fn().mockResolvedValue({
    id: 1,
    name: "Leanne Graham",
    address: {
      street: "Kulas Light",
      suite: "Apt. 556",
      city: "Gwenborough",
    },
  }),
});
```

### `.toHaveBeenCalled()`

- 특정 함수의 파라미터로 받는 콜백 함수가 존재할 때 이 함수가 실행되는지 확인

```js
// fetch 모킹 생략 ...
const url = "https://api.example.com/user/1";
const callback = jest.fn();
await fetchData(url, callback);
expect(callback).toHaveBeenCalled();
```

### `.toHaveBeenCalledWith()`

- 특정 함수의 파라미터로 받는 콜백 함수가 실행되면서 특정 인자를 받고 실행되는지 확인

```js
// fetch 모킹 생략 ...
const url = "https://api.example.com/user/1";
const callback = jest.fn();
await fetchData(url, callback);
// 전달하는 데이터는 fetchData 내에서 callback 에 전달하는 데이터 형식과 동일하여야 한다.
expect(callback).toHaveBeenCalledWith({
  userId: 1,
  formattedName: "Leanne Graham",
  address: "Kulas Light Apt. 556 Gwenborough",
});
```

### `.toHaveBeenCalledTimes(n)`

- 함수가 의도한 대로 n회 실행되는지 확인
- 만약 함수가 각 테스트 케이스(`test`, `it`)가 아닌 상위에서 모킹될 경우 호출 횟수가 누적되므로 반드시 초기화해야 한다.

### `jest.mock()`

- axios 등 외부 라이브러리를 모킹해야 할 때
- 자동으로 호이스팅되며 내부 메소드가 `jest.fn()` 으로 교체됨됨

```js
const axios = require("axios");

jest.mock("axios");

describe("apiClient.js 테스트", () => {
  beforeEach(() => {
    axios.get.mockResolvedValue({
      data: {
        id: 1,
        name: "Leanne Graham",
        address: {
          street: "Kulas Light",
          suite: "Apt. 556",
          city: "Gwenborough",
        },
      },
    });
  });

  // 테스트 케이스 작성 ...
});
```

- 직접 구현한 모듈을 모킹할 때

```js
const logger = require("./logger");
jest.mock("./logger");

test("결제 금액 0 이하일 때 결제 실패 시 로그 확인", () => {
  processPayment(0);
  // logger.error는 로그 파일을 남기지 않음
  expect(logger.error).toHaveBeenCalledWith(
    "결제 처리 중 오류 발생: 결제 금액은 0보다 커야 합니다.",
  );
});
```

### `jest.spyOn()`

- 원래 구현은 유지하면서 함수 호출 여부만 추적하고 싶을 때(기능을 유지한 채로 모킹)
- 로그 표시, DB 저장, 이메일 전송 등 부수 효과가 있는 함수를 테스트할 때 `mockImplementation` 과 함께 사용
- `jest.fn()` 과 달리 `mockRestore` 을 사용하여 쉽게 복구 가능
- 객체와 메소드명을 받아서 모킹

```js
const spy = jest.spyOn(obj, "greet");
// 테스트 코드 ...
// 테스트가 끝난 후 복구하여 다음 테스트에 영향을 주지 않도록 함
spy.mockRestore();
```

```js
test("올바른 결제 정보로 결제 성공 시 로그 확인", () => {
  jest.spyOn(console, "log").mockImplementation(() => {});

  processPayment(10000, "KRW", "card");
  // mockImplementation 으로 인해 터미널에 로그는 표시되지 않으나 실행 여부는 확인할 수 있음
  expect(console.log).toHaveBeenCalledWith(
    "10000원이 card 방식으로 성공적으로 결제 완료됐습니다.",
  );
});
```

#### `.mockImplementation()`

- 함수(메소드)의 내부 구현을 재작성
- `jest.fn()` 과 유사하게 동작

```js
jest.spyOn(console, "log").mockImplementation(() => {});
```

### `mockReturnValue`

- Promise 를 반환하지 않는 가짜 함수에서 반환값을 지정

```js
const mockFn = jest.fn().mockReturnValue(123);
```

### `mockReturnValueOnce`

- 함수가 호출될 때마다 다른 값을 반환하는 시나리오 테스트

```js
test("mockReturnValueOnce 테스트", () => {
  const mockFn = jest
    .fn()
    .mockReturnValueOnce(1) // 첫 번째 호출에만 1 반환
    .mockReturnValueOnce(2) // 이후 호출은 2 반환
    .mockReturnValueOnce(3); // 이후 호출은 3 반환

  expect(mockFn()).toBe(1);
  expect(mockFn()).toBe(2);
  expect(mockFn()).toBe(3);
});
```

### `mockResolvedValueOnce`

- 한 번의 호출에 대해 Promise 반환값 모킹
- 비동기 함수를 여러 번 호출해서 각각 다른 반환값을 반환하는 시나리오 테스트

```js
test("mockResolvedValueOnce 테스트", async () => {
  const mockFn = jest
    .fn()
    .mockResolvedValueOnce({ id: 1, name: "첫 번째 응답" })
    .mockResolvedValueOnce({ id: 2, name: "두 번째 응답" })
    .mockResolvedValueOnce({ id: 3, name: "세 번째 응답" });

  // 각 호출마다 다른 결과값 반환
  const result1 = await mockFn();
  const result2 = await mockFn();
  const result3 = await mockFn();

  expect(result1).toEqual({ id: 1, name: "첫 번째 응답" });
  expect(result2).toEqual({ id: 2, name: "두 번째 응답" });
  expect(result3).toEqual({ id: 3, name: "세 번째 응답" });
});
```

### `mockRejectedValue`

- 비동기 함수가 실패했을 때의 상황을 테스트

```js
test("mockRejectedValue 테스트", async () => {
  // 테스트 코드가 변경되어 catch 문이 실행되지 않는 실수 방지(이 경우 PASS 처리 됨)
  // `expect.assertions`: 테스트 케이스 내에서 expect 문이 최소 n회 실행되어야 함을 명시
  expect.assertions(1);

  // new Error("API 호출 실패") 는 catch 문의 (error) 에 들어감
  const errorMessage = "API 호출 실패";
  const mockFn = jest.fn().mockRejectedValue(new Error(errorMessage));

  try {
    await mockFn();
  } catch (error) {
    expect(error.message).toBe(errorMessage);
  }
});
```

## Promise 테스트

> Promise를 통해 테스트 시 `return` 문 반드시 작성(정상 작동하나 문서에서 권장)

```js
return fetchData(url).then((result) => {
  expect(result).toEqual({
    userId: 1,
    formattedName: "김철수",
    address: "테스트 거리 테스트 호수 서울",
  });
});

return expect(fetchData(url)).resolves.toEqual({
  userId: 1,
  formattedName: "김철수",
  address: "테스트 거리 테스트 호수 서울",
});

return fetchData(url).catch((error) => {
  expect(error.message).toBe(errorMessage);
});

return expect(fetchData(url)).rejects.toThrow(errorMessage);
```

## Callback 테스트(`done`)

- 레거시 코드 또는 특정 라이브러리 작업 시 필요
- 정의한 콜백 함수가 언제 끝나는지를 명시

```js
test("API 호출 콜백 테스트", (done) => {
  // API 호출 및 콜백 실행 ...

  // 성공 케이스 테스트
  fetchUserData(1, () => {
    // expect 를 사용한 API 호출, 콜백 실행 테스트 ...
    done(); // 테스트 완료 알림
  });
});
```

### 타이머 모킹

- `setTimeout`, `setInterval` 등 실제 타이머 시간을 기다리지 않고 즉시 코드를 실행
- `jest.useFakeTimers()`: Javascript 의 내장 타이머 함수를 모킹
- `jest.runAllTimers()`: 대기 중인 모든 타이머를 즉시 실행

```js
function timerFunction(callback) {
  setTimeout(() => {
    callback("타이머 완료");
  }, 3000);
}

test("타이머 모킹", () => {
  jest.useFakeTimers(); // 가짜 타이머 사용
  const callback = jest.fn();

  timerFunction(callback);

  // 콜백이 아직 호출되지 않았는지 확인
  expect(callback).not.toHaveBeenCalled();

  // 모든 타이머를 즉시 실행
  jest.runAllTimers();

  // 콜백이 호출되었는지 확인
  expect(callback).toHaveBeenCalled();
  expect(callback).toHaveBeenCalledWith("타이머 완료");
});
```

## 생명주기 훅

- `beforeAll`: 그룹 내 모든 테스트 실행 전 1회
- `afterAll`: 그룹 내 모든 테스트 실행 후 1회
- `beforeEach`: 각 테스트 실행 전마다
- `afterEach`: 각 테스트 실행 후마다

```js
describe("apiClient.js 테스트", () => {
  const callback = jest.fn();

  beforeEach(() => {
    // global.fetch 모킹도 매 테스트마다 작성하는 대신 beforeEach 에서 작성할 수 있다.

    // jest.clearAllMocks(); // 모든 mock의 호출 기록 초기화
    // .mockClear(): callback 함수의 호출 기록 초기화
    callback.mockClear();
  });

  // 테스트 케이스 작성 ...
});
```
