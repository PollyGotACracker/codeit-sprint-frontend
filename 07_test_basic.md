# Test - basic (jest)

## 테스트 코드의 목적

- 작성한 지 오래된 코드나 다른 사람이 작성한 코드를 수정하면서 실수가 발생할 수 있음
- 에러 가능성이 있는 코드를 배포했을 때 사용자 이탈이 발생할 수 있음

## Jest

- Meta 에서 개발한 Javascript 테스트 프레임워크
- 현재 프론트엔드 테스트에서 가장 사용률이 높은 도구
- 테스트 실행, 검증, 모킹 등 여러 기능과 다양한 상황에 맞는 검증 함수(matcher) 제공
- 설정이 간단하며 병렬 실행으로 테스트 속도가 빠름
- UI 컴포넌트의 변경 사항을 쉽게 트래킹(스냅샷 테스팅)

| 특징          | Jest      | Mocha                   | Vitest    |
| ------------- | --------- | ----------------------- | --------- |
| 설정 난이도   | 쉬움      | 중간                    | 쉬움      |
| 내장 기능     | 매우 많음 | 적음 (추가 패키지 필요) | 많음      |
| React 지원    | 기본 지원 | 추가 설정 필요          | 좋음      |
| 실행 속도     | 빠름      | 중간                    | 매우 빠름 |
| 커뮤니티 크기 | 매우 큼   | 큼                      | 성장 중   |

### 준비

1. 프로젝트 초기화 및 설치

```shell
# 프로젝트 초기화
npm init
# jest 설치
npm i -D jest
npm i -D @types/jest
```

2. package.json 스크립트 작성
   - `jest --verbose`: 통과한 테스트 설명도 확인해야 할 경우
   - `jest --watchAll`: 파일을 저장하는 경우(변경사항 발생) 즉시 테스트 실행

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watchAll"
  }
}
```

3. 필요한 경우 jest.config.js 작성

- [setupFiles](https://jestjs.io/docs/configuration#setupfiles-array): 모든 테스트 코드 실행 전 1회 실행
- [setupFilesAfterEnv](https://jestjs.io/docs/configuration#setupfilesafterenv-array): 모든 테스트 파일 실행 전 각각 반복하여 실행

### AAA 패턴

- Arrange(준비), Act(실행), Assert(검증)의 세 단계로 구성
- 테스트 코드 가독성 향상, 테스트 목적과 흐름 파악이 쉬워짐
- Given, When, Then 패턴과 동일한 의미

```js
describe("formatDate", () => {
  test("날짜를 YYYY-MM-DD 형식으로 변환한다", () => {
    const date = new Date("2024-01-01"); // Arrange
    const result = formatDate(date); // Act
    expect(result).toBe("2024-01-01"); // Assert
  });

  test("유효하지 않은 날짜면 빈 문자열을 반환한다", () => {
    const date = null; // Arrange
    const result = formatDate(date); // Act
    expect(result).toBe(""); // Assert
  });
});
```

### 기본 구조

#### `describe`

- 관련 테스트를 그룹화
- 테스트 실행 시 `describe` 설명을 포함하여 로그 표시
- 토글 및 내부 들여쓰기가 적용되어 직관성 향상

#### `test` / `it`

- 테스트 케이스 하나를 정의하는 함수
- 기능은 동일, 영어 작성 시 `it`이 문장처럼 읽혀 자연스러움
  - `it('should ... ')`
- 한국어 환경에서는 `test` 사용 권장

#### `expect`

- 테스트 대상의 실제 값을 받아 matcher로 기댓값과 비교하는 함수

### Matcher 종류

- `toBe`:
  - 엄격한 동등 비교(`===`)
  - 원시 타입의 값 비교
- `toEqual`:
  - 깊은 동등 비교
  - 객체를 비교해야 할 때
- `toStrictEqual`:
  - `toEqual` 보다 더 엄격하게 비교하여 `undefined` 인 속성과 배열 빈 슬롯 비교 가능
- `toContain`:
  - 문자열에서 핵심 문자만 포함되는지 판별
  - 배열 내 항목 포함 여부 판별

- `toBeTruthy`, `toBeFalsy`: truthy, falsy 값 판별
- `toBeNull`, `toBeUndefined`, `toBeDefined`: 값이 `null` 인지, `undefined` 인지, 정의되어 있는지 판단

#### 숫자 Matcher

- `toBeGreaterThan`, `toBeGreaterThanOrEqual`
- `toBeLessThan`, `toBeLessThanOrEqual`
- `toBeCloseTo`: 부동 소수점(floating point) 비교

#### 문자열 Matcher

- `toMatch`: 정규식을 사용한 판별
- `toContain`: 문자열 포함 여부

#### 배열과 이터러블 Matcher

- `toContain`: 배열 내 항목 포함 판별
- `toHaveLength`: 배열 길이 확인

#### 부정 Matcher

- `.not`: 다른 matcher 앞에 쓰여 부정 판별별

#### 예외 Matcher

- `toThrow`: 예외 발생 테스트
  > **함수 직접 호출이 아닌 함수 레퍼런스를 전달해야 함**

```js
// X
expect(throwError()).toThrow();
// O: 함수를 래핑
expect(() => throwError()).toThrow(); ✅
```

#### 객체 Matcher

- `toHaveProperty`: 객체에 특정 속성이 존재하는지 확인

```js
expect(user).toHaveProperty("name");
// 중첩된 속성도 확인 가능
expect(user).toHaveProperty("address.city");
// 특정 속성과 값이 있는지 확인 가능
expect(user).toHaveProperty("address.country", "대한민국");
```

- `objectContaining`: 객체 내 여러 속성이 존재하는지 한번에 확인

```js
expect(user).toEqual(
  expect.objectContaining({
    name: "Leanne Graham",
    age: 30,
  }),
);

// 중첩된 경우
expect(user).toEqual(
  expect.objectContaining({
    address: expect.objectContaining({
      city: "Busan",
    }),
  }),
);
```

- `expect.arrayContaining`: `toContain` 과 달리 여러 개의 배열 요소를 한번에 확인

```js
expect(fruits).toEqual(expect.arrayContaining(["banana", "apple"]));
```

##### jest-extended: matcher 확장 라이브러리

- 복잡한 객체를 다뤄야 할 때 유용하게 사용할 수 있음

1. 패키지 설치

```shell
npm install --save-dev jest-extended
```

2. testSetup.js 파일 작성

```js
// testSetup.js

const matchers = require("jest-extended");
expect.extend(matchers);
```

3. package.json

```json
// package.json
{
  "jest": {
    "setupFilesAfterEnv": ["./testSetup.js"]
  }
}
```
