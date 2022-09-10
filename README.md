# React+TypeScriptを1つのHTMLファイルのみ(プリコンパイルなし)で実行するサンプル

## はじめに

React公式ページ
[既存のウェブサイトに React を追加する](https://ja.reactjs.org/docs/add-react-to-a-website.html#optional-try-react-with-jsx)
で、`jsx`をhtmlファイルに直接記載して実行するサンプルがありました。

[@babel/standalone](https://babeljs.io/docs/en/babel-standalone)を読み込み、トランスパイルと実行を行うことで実現しています。

```html
  <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
  <div id="root"></div>
  <script type="text/babel">

    function MyApp() {
      return <h1>Hello, world!</h1>;
    }

    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<MyApp />);
  </script>
```
上記サンプルを元に、htmlだけでTypeScriptとReactを実行するサンプルプログラムを作成します。
* Reactをimportで利用する(ES Module)・・・ブラウザが`ES Module`を読み込めるようになった
* TypeScript(tsx)化・・・Babel7でTypeScriptのトランスパイルをサポート


### 手順

1. [@babel/standalone](https://babeljs.io/docs/en/babel-standalone)を使い、簡単なReactアプリケーションを作成(javascript)
1. Reactを&lt;script&gt;タグから、import経由で読み込むように変更
1. TypeScript化(babelのプリセットをtypescriptに変更)
1. React部分を別ファイル(.tsx)に切り出す(おまけ、tsxファイルを読み込み実行できることを確認)


## 1. [@babel/standalone](https://babeljs.io/docs/en/babel-standalone)を使い、簡単なReactアプリケーションを作成(javascript)

[既存のウェブサイトに React を追加する](https://ja.reactjs.org/docs/add-react-to-a-website.html#optional-try-react-with-jsx) を少し変更して、クリックするとカウントアップする処理を作成

* `jsx`はブラウザが処理できないため、`type="text/babel"`に記載します(ブラウザが認識しないContent-Typeを指定する)。
b
abelのランタイムトランスパイラは、上記からスクリプトを読み込み、トランスパイルと実行を行います。

js_standalone_react.html
```html
<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <title>React js-standalone</title>
  <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script type="text/babel">
    const Counter = () => {
      const [count, setCount] = React.useState(0);
      const handleClick = () => setCount((n) => n + 1);
      return (
        <>
          <div>Count: {count}</div>
          <button onClick={handleClick}>Increment</button>
        </>
      );
    };
    ReactDOM.render(<Counter />, document.getElementById('app'));
  </script>
</head>
<body>
  <div id="app"></div>
</body>
</html>


```

クリックするごとにCountが1ずつ増えることが確認できます。

![image](https://user-images.githubusercontent.com/42880820/189474043-2bfbf5fe-be57-475b-a779-583e8719feb5.png)


### 起動方法

エクスプローラーから直接htmlを開いても動作します。

webサーバ経由で開く場合は`http-server`などを利用してください。
```bash
$ npx http-server
Starting up http-server, serving ./
～～以下省略～～
```



## 2. Reactを&lt;script&gt;タグから、import経由で読み込むように変更

&lt;script&gt;タグで読み込むreactはnode module形式で提供されるため、ブラウザからimportすることができません(webpackなどのバンドラが必要)

そのため、ES Module版の[React](https://cdn.skypack.dev/react)を利用します。

npmパッケージをES Moudulesとして読み込めるように変換してくれる[https://cdn.skypack.dev](https://cdn.skypack.dev)を利用します。

js_esm_standalone_react.html
```html
<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <title>React jsx-esm-standalone</title>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <!-- @babel/standaloneでESModuleを利用するために、data-type="module"を追加 -->
  <script type="text/babel" data-type="module">
    // CDN経由でReactを読み込む(ESModuleであればブラウザから直接利用可能)
    import React, { useState } from "https://cdn.skypack.dev/react";
    import ReactDOM from "https://cdn.skypack.dev/react-dom";

    const Counter = () => {
      const [count, setCount] = useState(0);
      const handleClick = () => setCount((n) => n + 1);
      return (
        <>
          <div>Count: {count}</div>
          <button onClick={handleClick}>Increment</button>
        </>
      );
    };
    ReactDOM.render(<Counter />, document.getElementById('app'));
  </script>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

## 3. TypeScript化(babelのプリセットをtypescriptに変更)

JavaScriptをTypeScript化します。
(TypeScriptがブラウザで動作することを確認するためのサンプル)

ts_standalone_react.html
```html
<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <title>React Elements</title>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script>
    // typescript用のpresetを登録する
    Babel.registerPreset('tsx', {
      presets: [
        [Babel.availablePresets['typescript'], // preset-typescriptを指定
          {allExtensions: true, isTSX: true}   // allExtensionsは、isTSX利用時に必要なためセット
        ]],
      },
    );
  </script>
  <script type="text/babel" data-type="module" data-presets="tsx,react">
    import React, { useState } from "https://cdn.skypack.dev/react@17";
    import ReactDOM from "https://cdn.skypack.dev/react-dom@17";

    const Counter: React.Component = () => {
      const [count, setCount] = React.useState<number>(0);
      const handleClick = () => setCount(n => n + 1);
      return (
        <div>
          <div>Count: {count}</div>
          <button onClick={handleClick}>Increment</button>
        </div>
      );
    }
    ReactDOM.render(<Counter />, document.getElementById('app'));
  </script>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

## 4. React部分を別ファイル(.tsx)に切り出す(おまけ、tsxファイルを読み込み実行できることを確認)

* &lt;script type="text/babel"&gt;を`Counter.tsx`に抜き出し、srcでファイル名を指定します


index.html
```html
<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <title>React Elements</title>
  <script  src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script>
    Babel.registerPreset('tsx', {
      presets: [
        [Babel.availablePresets['typescript'], {allExtensions: true, isTSX: true}]],
      },
    );
  </script>
  <script type="text/babel" data-type="module" data-presets="tsx,react" src="Counter.tsx">
  </script>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

Counter.tsx
```typescript
import React, { useState } from "https://cdn.skypack.dev/react@17";
import ReactDOM from "https://cdn.skypack.dev/react-dom@17";

const Counter: React.FC = () => {
  const [count, setCount] = React.useState<number>(0);
  const handleClick = React.useCallback(
    () => setCount((n) => n + 1),
    [setCount]
  );
  return (
    <div>
      <div>Count: {count}</div>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
};
ReactDOM.render(<Counter />, document.getElementById("app"));
```
