# NextJS

- [Feature-Driven Architecture with Next.js](https://dev.to/rufatalv/feature-driven-architecture-with-nextjs-a-better-way-to-structure-your-application-1lph): Next.js 기반 기능 중심 아키텍처 구조 가이드
- [캐싱 공식 문서 (EN)](https://nextjs.org/docs/app/guides/caching)
- [캐싱 공식 문서 (KO)](https://nextjs-ko.org/docs/app/building-your-application/caching)

## Request Memoization

- 동일 페이지 & 동일 URL의 GET 요청을 렌더링 중 자동으로 중복 제거
- RSC(React Server Component) 에서 native `fetch` 를 사용할 때 자동 동작
- 예를 들어 한 페이지에서 여러 컴포넌트가 같은 API를 각각 fetch할 때,  
  원래라면 동일한 요청이 3번 나가야 하지만 Next.js가 자동으로 중복을 제거해서 실제로는 1번만 요청

```ts
// Header 컴포넌트
const user = await fetch("/api/user");

// Sidebar 컴포넌트
const user = await fetch("/api/user");

// Profile 컴포넌트
const user = await fetch("/api/user");
```

- axios 등 라이브러리를 사용할 경우 별도 설정 필요
- `react`의 `cache()`로 직접 메모이제이션 가능

```ts
import { cache } from "react";
import axios from "axios";

const getUser = cache(async () => {
  return axios.get("/api/user");
});
```

### `cacheTag`: 태그 정의

- 캐시할 데이터에 태그를 정의

```ts
// fetch 방식
fetch("https://api.example.com/posts", { next: { tags: ["posts"] } });

// use cache 방식 (Next.js 15+)
async function getPosts() {
  "use cache";
  cacheTag("posts");
  return await db.query("SELECT * FROM posts");
}
```

### `revalidateTag`: 캐시를 stale 로 표시

- `profile="max"` 옵션을 쓰면 해당 태그를 stale로 표시
- 다음 요청 시 기존 캐시를 먼저 내려주면서 백그라운드에서 새 데이터를 fetch  
  (stale-while-revalidate)
- [참고](https://nextjs.org/docs/app/api-reference/functions/revalidateTag)

```ts
"use server";
import { revalidateTag } from "next/cache";

export async function updatePost() {
  await db.updatePost();
  revalidateTag("posts", "max"); // stale로 표시
}
```

### `updateTag`

- Server Action에서 즉시 캐시를 만료시키는 용도
- 다음 요청 시 캐시를 사용하지 않고 새로 fetch 후 응답하므로 느릴 수 있음
- 최신 데이터를 즉시 읽어야 하는 read-your-own-writes 시나리오에 적합
- [참고](https://nextjs.org/docs/app/getting-started/caching-and-revalidating)

```ts
"use server";
import { updateTag } from "next/cache";

export async function createPost() {
  await db.createPost();
  updateTag("posts"); // 즉시 만료, 다음 요청에서 새로 fetch
}
```

## Pages

- ErrorBoundary => error.tsx
- Suspense => loading.tsx
