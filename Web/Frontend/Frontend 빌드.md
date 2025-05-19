## 1. TypeScript → JavaScript 트랜스파일

**주체:** `tsc` 또는 번들러 내장 트랜스파일러 (`vite`, `webpack`, `babel`, `esbuild` 등)
**과정**:
- `.ts` / `.tsx` 파일들을 **타입 검사 + ES6+ JS로 변환**
- JSX는 `React.createElement` 또는 JSX Runtime으로 바뀜
- 타입 정보는 **컴파일 시점에서만 사용되고**, **런타임에는 제거됨**
```tsx
// App.tsx (입력)
const App: React.FC = () => <h1>Hello</h1>;

// 트랜스파일 결과 (출력)
const App = () => React.createElement("h1", null, "Hello");
```
## 2. 모듈 [[번들링]] 및 최적화

**주체:** `webpack`, `vite`, `rollup` 등
**과정**:
1. **모듈을 분석**해서, 각 JS 파일들의 의존 관계를 파악합니다 (`import`, `export`)
2. 모든 JS 파일들을 하나 또는 몇 개의 파일로 **번들링(bundle)** 합니다.
3. **Tree Shaking**을 통해 사용되지 않는 코드 제거
4. JS/CSS/이미지 등은 **압축(minify)** 및 **해시 붙이기 (캐싱용)**
```js
dist/
├── index.html
├── assets/
│   ├── main.9e78a9.js     ← 모든 JS를 하나로 묶은 파일
│   └── style.43be7b.css   ← CSS 번들
```
## 3. `index.html` 템플릿 구성

**과정**:
- `public/index.html`을 기반으로 번들된 자산을 `<script>`, `<link>` 태그로 삽입합니다.
- `vite`, `webpack` 플러그인이 이걸 자동으로 처리합니다.
- 실제 출력되는 `index.html`은 다음처럼 됩니다:
	```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>React App</title>
    <link href="/assets/style.43be7b.css" rel="stylesheet">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/assets/main.9e78a9.js"></script>
  </body>
</html>

```
- 사용자가 접속하면 이 HTML 하나만 불러오고, 이후 경로 이동은 모두 JS 로직으로 처리됨 (SPA 특징)
## 빌드 과정 흐름도
```scss
[ .tsx / .ts / .jsx / .js ]
       │
       ▼ (tsc / babel)
[ 트랜스파일된 JS ]
       │
       ▼ (vite / webpack)
[ 번들링, 최적화 ]
       │
       ▼
[ index.html + main.js + style.css (정적 리소스) ]
       │
       ▼
[ nginx가 서빙 → SPA로 동작 ]

```