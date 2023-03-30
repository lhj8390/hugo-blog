+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "React Webpack 개념 및 설정"
author = "lhj8390"
description = "React 초기 설정인 Webpack 기본 개념 및 초기 설정에 대해 설명한다."
date = "2022-06-05"
tags = ["frontend", "webpack", "react.js"]
categories = ["web"]
subcategories = ["react"]
+++

## 웹팩(Webpack)이란?

---

> **모듈 번들링**
>
> 한 프로그램으로 작동하는 하나의 파일을 여러 파일로 분할하고 구성할 수 있는 자바스크립트 모듈 \
> → 다수의 js 파일을 하나의 js 파일로 만들어주는 것 <br/>
>
> <br/>

### Webpack 사용 이유

1. **SPA 사용 확대** : 하나의 html 페이지에 여러개의 자바스크립트 파일들이 포함되는 경우가 많아졌다.
2. **관리의 이점** : 연관 되어 있는 자바스크립트 종송석 있는 파일들을 하나의 파일로 묶어줘서 관리하기 편하다.
3. **시간 단축** : 컴파일 시 여러 모듈들의 파일을 읽어오는데 시간이 오래 걸리는데, 그 부분을 해결하기 위해 여러 파일을 하나의 파일로 번들링 해준다.
4. **웹페이지 성능 최적화**
   <br/><br/>

## Babel이란?

---

> **최신 ES6버전을 구 버전인 ES5로 변환해주는 역할.**
>
> 최신 버전의 자바스크립트 문법은 브라우저가 이해하지 못한다.\
> → babel이 브라우저가 이해할 수 있는 문법으로 변환
>
> ES6, ES7 등의 최신 문법을 사용해서 코딩을 할 수 있기 때문에 **생산성이 향상**
>
> <br/><br/>

## Webpack 기본 명령어

---

### Webpack 설치

```
$ npm install webpack webpack-cli html-webpack-plugin webpack-dev-server --save-dev
```

- **html-webpack-plugin** : 웹팩으로 빌드한 결과물을 HTML 파일로 생성해주는 플러그인
- **webpack-dev-server** : 웹팩으로 빌드 후 빌드한 파일을 서버로 띄어주는 역할
  <br/><br/>

## Webpack 설정 파일 생성

---

### Webpack 기본 설정

```jsx
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  mode: 'development',
  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'index_bundle.js',
  },
};
```

**entry** : entry를 통해 Reack 파일이 시작하는 entry 포인트를 지정해주었다. \
**output** : \_\_dirname은 현재 디렉터리로, /dist 폴더에 번들링한 index_bundle.js 이름의 파일을 넣어주겠다고 명시.
<br /><br />

### babel 설정

```jsx
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_module/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
};
```

webpack 설정 내부에서 babel loader가 실행할 규칙을 지정하였다.

**test** : .js, .jsx로 끝나는 파일은 babel loader가 컴파일해준다고 명시 \
**exclude** : /node_module/은 babel 컴파일에서 제외
<br /><br />

### devServer 설정

```jsx
module.exports = {
  devServer: {
    host: 'localhost',
    port: 3000,
    historyApiFallback: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
      },
    },
  },
};
```

webpack-dev-server가 사용할 host와 port를 지정하였다. \
proxy를 통해 /api로 접근하면 http://localhost:8000 서버에서 해당 요청을 받아주도록 설정!
<br /><br />

<aside>
<strong>CORS 에러?</strong>

  개발 편의상 로컬에 webpack-dev-server를 띄어놓는 경우가 많은데,
  보통 다른 도메인에서 요청을 보낼 경우 브라우저 보안으로 인해 CORS 에러가 발생한다.
  하지만 프록시 설정을 할 경우 같은 도메인에서 온 요청으로 인식하여 CORS 에러가 나지 않는다.

</aside>
<br /><br />

## Webpack 실행

---

```javascript
"scripts": {
    "build": "webpack --config ./webpack.config.js",
    "start": "webpack-dev-server --config ./webpack.config.js"
  },

```

**npm start** : webpack.config.js 설정 파일을 이용하여 webpack-dev-server를 구동한다. \
**npm build** : webpack.config.js 설정 파일을 이용하여 webpack 빌드를 수행한다.
<br /><br />

### 전체 webpack.config.js 코드

```jsx
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  mode: 'development',
  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'index_bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_module/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
  devServer: {
    host: 'localhost',
    port: 3000,
    historyApiFallback: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
      },
    },
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
};
```
