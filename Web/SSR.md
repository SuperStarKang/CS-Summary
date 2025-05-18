## Server Side Rendering: 서버 사이드 랜더링

- 서버가 사용자의 요청을 받았을 때, React 컴포넌트를 실행해서 HTML을 만든 후,
- 그 완성된 HTML을 브라우저에 응답으로 보내는 방식

### 브라우저에서의 동작

- 브라우저는 이미 랜더링된 HTML을 받아오기 때문에 즉시 페이지 내용이 보임
- 이후 React가 다시 활성화되어 SPA처럼 작동하게 됨

## SSR vs CSR 비교

| 구분           | CSR (Client Side Rendering) | SSR (Server Side Rendering) |
| ------------ | --------------------------- | --------------------------- |
| 페이지 생성 위치    | 브라우저에서 JS 실행 후 렌더링          | 서버에서 렌더링 후 HTML 전송          |
| 최초 로딩 속도     | 느릴 수 있음 (JS 로딩, 실행 필요)      | 빠름 (HTML 바로 도착)             |
| SEO (검색 최적화) | 낮음                          | **높음 (HTML에 콘텐츠 포함)**       |
| 예시 프레임워크     | React + Vite, CRA           | **Next.js, Nuxt.js** 등      |
## Next.js에서 SSR을 제공하는 방식

- Next.js는 페이지 단위로 SSR을 제어할 수 있음
- 예시
	```tsx
	// pages/uniforms.tsx
	import { GetServerSideProps } from 'next'

	export default function UniformPage({ team }: { team: string }) {
	  return <h1>{team} Uniform Page</h1>
	}

	export const getServerSideProps: GetServerSideProps = async (context) => {
	  const team = context.query.team || 'Real Madrid'
	  return {
	    props: {
	      team,
	    },
	  }
	}
```
#### 동작 흐름

1. 사용자가 `/uniforms?team=Barcelona`로 접속
2. Next.js 서버가 `getServerSideProps()` 실행 → team 값 추출
3. 그 결과로 HTML 렌더링 → 브라우저에 바로 HTML 전달

즉, React 컴포넌트가 서버에서 실행되어 HTML을 생성함.