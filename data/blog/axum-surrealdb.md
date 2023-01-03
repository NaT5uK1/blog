---
title: 构建具备jwt鉴权的全栈应用
date: '2022-08-31'
tags: ['Rust', 'Surrealdb', 'axum', 'react', 'redux']
draft: false
summary:
images: []
layout: PostSimple
canonicalUrl:
authors: ['default']
---

## 项目创建

```shell
pnpm create vite

✔ Project name: … jwt-app
✔ Select a framework: › react
✔ Select a variant: › react

cd jwt-app
pnpm i @reduxjs/toolkit react-redux react-router-dom
pnpm i -D vite-plugin-windicss windicss

cargo new server
wget https://www.toptal.com/developers/gitignore/api/macos,rust -O ./server/.gitignore

git init
git add .
git commit -m "init"

tree . -L 2

.
├── index.html
├── node_modules
├── package.json
├── pnpm-lock.yaml
├── public
│   └── vite.svg
├── server
│   ├── Cargo.toml
│   └── src
├── src
│   ├── App.css
│   ├── App.jsx
│   ├── assets
│   ├── index.css
│   └── main.jsx
└── vite.config.js
```

### 客户端

- 配置 WindiCSS

`vite.config.js`

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import WindiCSS from 'vite-plugin-windicss'

export default defineConfig({
  plugins: [react(), WindiCSS()],
})
```

`src/main.jsx`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'
import 'virtual:windi.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

## Surrealdb 初探

### 安装

```shell
brew install surrealdb/tap/surreal
```
