---
id: code-splitting
title: Code-Splitting
permalink: docs/code-splitting.html
---

## Bundling {#bundling}

大部分 React 應用程式會使用像是 [Webpack](https://webpack.js.org/)、[Rollup](https://rollupjs.org/) 或 [Browserify](http://browserify.org/) 的工具來 bundle 它們的檔案。Bundle 是將 import 的檔案合併為單一的檔案過程：「Bundle」。網頁可以引入 bundle，以一次載入整個應用程式。

Bundle 是將被 import 的檔案合併成一個單一的檔案：「bundle」。這個 bundle 檔案可以被引入到網頁內來載入整個應用程式。

#### 範例 {#example}

**應用程式：**

```js
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```

```js
// math.js
export function add(a, b) {
  return a + b;
}
```

**Bundle：**

```js
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

> 注意：
>
> 你的 bundle 後的最終結果看起來會與此不同。

如果你使用 [Create React App](https://create-react-app.dev/)、[Next.js](https://nextjs.org/)、[Gatsby](https://www.gatsbyjs.org/)，或者是類似的工具，會有一個內建的 Webpack 設定來 bundle 你的應用程式。

如果沒有的話，你需要自己設定 bundle。例如，拜訪 Webpack 文件的 [Installation](https://webpack.js.org/guides/installation/) 和 [Getting Started](https://webpack.js.org/guides/getting-started/) 指南。

## Code-Splitting {#code-splitting}

Bundle 非常棒，但隨著你的應用程式成長，你的 bundle 也將會隨著增長。特別是你引入了大量的第三方函式庫。
你需要隨時留意 bundle 後的程式碼，這樣你就不會得意外的讓 bundle 檔案變得太大，以至於你的應用程式需要很長的時間才能被載入。

為了避免 bundle 的結果過大，最好的解決問題的方式是開始「split」你的 bundle。
Code-Splitting 是透過由像是 [Webpack](https://webpack.js.org/guides/code-splitting/)、
[Rollup](https://rollupjs.org/guide/en/#code-splitting) 和
Browserify（經由 [factor-bundle](https://github.com/browserify/factor-bundle)）的 bundler 所支援的功能，
它會建立多個 bundle，可以在 runtime 時動態的被載入。

Code-splitting 可以幫助你「延遲載入」目前使用者所需要的東西，這可以大幅提供你的應用程式效能。雖然你還沒有減少應用程式的程式碼總數量，但你可以避免載入使用者目前使用不到的程式碼，來減少初始載入應用程式的時間。

## `import()` {#import}

將 code-splitting 引入到你的應用程式最好的方式是透過動態 `import()` 語法。

**加入前：**

```js
import { add } from './math';

console.log(add(16, 26));
```

**加入後：**

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

當 Webpack 遇到這種語法時，它將自動的在你的應用程式啟動 code-splitting。如果你使用 Create React App 的話，它已經幫你設定好了，你可以立即的[使用它](https://create-react-app.dev/docs/code-splitting/)。在 [Next.js](https://nextjs.org/docs/advanced-features/dynamic-import) 也內建支援這個功能。

如果你是自行設定 Webpack，你可以閱讀 Webpack 的 [code-splitting 指南](https://webpack.js.org/guides/code-splitting/)。你的 Webpack 設定看起來應該[像這樣](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

當使用 [Babel](https://babeljs.io/) 時，你將需要確保 Babel 可以解析動態的 import 語法而不是去轉換它。你可能會需要 [@babel/plugin-syntax-dynamic-import](https://classic.yarnpkg.com/en/package/@babel/plugin-syntax-dynamic-import)。

## `React.lazy` {#reactlazy}

`React.lazy` 讓你 render 一個動態 import 的 component 作為正常的 component。

**加入前：**

```js
import OtherComponent from './OtherComponent';
```

**加入後：**

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

當首次 render 這個 component 時，將會自動的載入包含 `OtherComponent` 的 bundle。

`React.lazy` 接受一個必須呼叫一個動態 `import()` 的 function。它必須回傳一個 `Promise`，resolve 一個包含 React component 的 `default` export 的 module。

lazy component 應在 `Suspense` component 內 render，這使我們可以在等待 lazy component 載入時，顯示一些 fallback 內容（像是一個載入的符號）。

```js
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

`fallback` prop 接受在等待 component 載入時要 render 的任何 React element。你可以將 `Suspense` component 放在 lazy component 上方的任何位置。你甚至可以包覆多個 lazy component 到 `Suspense` component 內。

```js
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```

### 避免 Fallbacks {#avoiding-fallbacks}
任何 component 都可能因 render 而暫停，即使是已經向使用者顯示的 component。為了讓螢幕內容始終保持一致，如果一個已經顯示的 component suspend，React 必須將 tree 隱藏到最近的 `<Suspense>` 邊界。但是，從使用者的角度來看，這可能會讓人迷惑。

思考這個 tab switcher：

```js
import React, { Suspense } from 'react';
import Tabs from './Tabs';
import Glimmer from './Glimmer';

const Comments = React.lazy(() => import('./Comments'));
const Photos = React.lazy(() => import('./Photos'));

function MyComponent() {
  const [tab, setTab] = React.useState('photos');

  function handleTabSelect(tab) {
    setTab(tab);
  };

  return (
    <div>
      <Tabs onTabSelect={handleTabSelect} />
      <Suspense fallback={<Glimmer />}>
        {tab === 'photos' ? <Photos /> : <Comments />}
      </Suspense>
    </div>
  );
}

```

在這個範例中，如果 tab 從 `'photos'` 改變為 `'comments'`，但是 `Comments` 暫停，使用者會看到一個 glimmer。這很合理因為使用者不想要看到 `Photos`，`Comments` component 還沒準備 render 任何東西，React 需要保持使用者體驗一致，所以它別無選擇，只能顯示上面的 `Glimmer`。

但是，有時候這樣的使用者體驗並不理想。在特定場景下，有時候在新的 UI 準備好之前，顯示「舊」的 UI 會更好。你可以使用新的 [`startTransition`](/docs/react-api.html#starttransition) API 來讓 React 執行此操作：

```js
function handleTabSelect(tab) {
  startTransition(() => {
    setTab(tab);
  });
}
```

這裡，你告訴 React 設定 tab 成 `'comments'` 不是一個緊急更新，但它是一個 [transition](/docs/react-api.html#transitions) 所以可能需要一些時間。React 將會保持舊的 UI 交互，並且在準備好時切換成顯示 `<Comments />`。更多資訊請參考 [Transitions](/docs/react-api.html#transitions)。

### 錯誤邊界 {#error-boundaries}

如果其他的 module 載入失敗（例如，因為網路失敗），它將會觸發一個錯誤。你可以透過[錯誤邊界](/docs/error-boundaries.html)處理這些錯誤來呈現一個好的使用者體驗和管理恢復。一旦你建立了你的錯誤邊界，你可以在任何的 lazy component 上方使用它，當網路發生錯誤時可以顯示一個錯誤狀態。

```js
import React, { Suspense } from 'react';
import MyErrorBoundary from './MyErrorBoundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

## 基於 Route 的 Code-Splitting {#route-based-code-splitting}

在你的應用程式決定採用 code-splitting 可能有點棘手。你想要確保選擇的位置可以適當的 split bundle，但不會破壞使用者的體驗。

Route 是一個開始的好地方。Web 上大多數的人都習慣花一些時間來等待頁面的過渡。你也傾向於重新 render 一次整個頁面，所以你的使用者不能同時與頁面上的其他 element 做互動。

這裡是如何在你的應用程式使用像是 [React Router](https://reactrouter.com/) 的函式庫與 `React.lazy` 來設定基於 route 的 code-splitting。

```js
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  </Router>
);
```

## Named Exports {#named-exports}

`React.lazy` 目前只支援 default exports。如果你想 import 的 module 使用 named export，你可以建立一個中介 module 來重新 export 它做為預設。這可以確保 tree shaking 不會 pull 無用的 component。

```js
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```

```js
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```

```js
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```
