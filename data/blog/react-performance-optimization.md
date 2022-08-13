---
title: React性能优化
date: '2022-08-10'
tags: ['React', 'Frontend']
draft: false
summary: React常见的性能优化手段
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/oracle-cloud-vm-configuration-guide
authors: ['default']
---

## 序言

​ 我们知道，React 在触发组件更新时，会采取双 fiber 树的方式进行从 root 节点开始的整树 diff。容易想到，假如重新渲染整棵树势必会造成性能浪费，所以 React 在组件更新阶段内置了性能优化策略，即在某些情况下只作比较而跳过部分节点的渲染阶段。此文就来探究一下性能优化策略。

## 性能优化策略

​ 在 React 中，有三种可变状态`props` `state` `context`，当一个组件不存在这三种可变状态**_或_**存在的可变状态的值不会变化时，则该组件是可以满足性能优化条件的，可以跳过该组件的渲染阶段以达到性能优化的目的。

## 命中性能优化的方式

### 拆分可变状态

#### 示例 1

```javascript
import { useState } from 'react'

export default function App() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <h1>{count}</h1>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        click
      </button>
      <ExpensiveComponent />
    </div>
  )
}

const ExpensiveComponent = () => {
  const now = performance.now()
  while (performance.now() - now < 100) {}
  console.log('Expensive Component rendered')
  return <h1>Expensive Component</h1>
}
```

​ 对于上面的`ExpensiveComponent`组件，不存在`props` `state` `context`这三种可变状态，但每当点击按钮时都会触发`ExpensiveComponent`组件的重新渲染。这是因为`App`组件包含`state`， 每当`state`变化，`App`组件及其子孙节点都会重新渲染。

​ 显然这并不是我们需要的，根据性能优化策略，`ExpensiveComponent`满足性能优化条件，我们希望跳过该组件的渲染阶段。具体做法是，将`state`相关逻辑抽离为独立组件`Counter`

```javascript
import { useState } from 'react'

export default function App() {
  return (
    <div>
      <Counter />
      <ExpensiveComponent />
    </div>
  )
}

const ExpensiveComponent = () => {
  const now = performance.now()
  while (performance.now() - now < 100) {}
  console.log('Expensive Component rendered')
  return <h1>Expensive Component</h1>
}

const Counter = () => {
  const [count, setCount] = useState(0)
  return (
    <>
      <h1>{count}</h1>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        click
      </button>
    </>
  )
}
```

​ 当`Counter`中的`state`变化时，则会以`Counter`为根节点重新渲染整棵子树，如此一来，避免了`ExpensiveComponent`的重复渲染，达到性能优化的目的。

#### 示例 2

```javascript
import { useState } from 'react'

export default function App() {
  const [count, setCount] = useState(0)
  return (
    <div title={count}>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        click
      </button>
      <ExpensiveComponent />
    </div>
  )
}

const ExpensiveComponent = () => {
  const now = performance.now()
  while (performance.now() - now < 100) {}
  console.log('Expensive Component rendered')
  return <h1>Expensive Component</h1>
}
```

​ 当可变状态组件与不可变状态组件产生父子关系时，可以用一个包裹组件来抽离可变状态：

```javascript
import { useState } from 'react'

export default function App() {
  return (
    <CounterWrapper>
      <ExpensiveComponent />
    </CounterWrapper>
  )
}

const ExpensiveComponent = () => {
  const now = performance.now()
  while (performance.now() - now < 100) {}
  console.log('Expensive Component rendered')
  return <h1>Expensive Component</h1>
}

const CounterWrapper = ({ children }) => {
  const [count, setCount] = useState(0)
  return (
    <div title={count}>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        click
      </button>
      {children}
    </div>
  )
}
```

​ 此时`App`组件不包含可变状态，当`CounterWrapper`中的`state`变化时，触发`CounterWrapper`的重新渲染，由于`ExpensiveComponent`是以`props.children`的形式传入且自身是不可变的，也可以命中性能优化策略跳过渲染。

> 结论：当父组件满足性能优化条件，子孙组件**_可能_**命中性能优化策略

### 性能优化 api

​ 既然可以通过拆分可变状态的方式达到性能优化的目的，那性能优化 api 为什么会被提出呢？

#### 示例 3

```javascript
import { useState, useContext, createContext } from 'react'

const numContext = createContext()
const setNumContext = createContext()

export default function App() {
  const [num, setNum] = useState(0)
  return (
    <numContext.Provider value={num}>
      <setNumContext.Provider value={setNum}>
        <Middle />
      </setNumContext.Provider>
    </numContext.Provider>
  )
}

const Middle = () => {
  return (
    <>
      <Show />
      <Button />
      {/* _jsx(Button,{}) */}
    </>
  )
}

const Show = () => {
  const num = useContext(numContext)
  return <h1>{num}</h1>
}

const Button = () => {
  const setNum = useContext(setNumContext)
  console.log('Button rendered')
  return (
    <button
      onClick={() => {
        setNum(() => Math.round(Math.random() * 100))
      }}
    >
      random
    </button>
  )
}
```

​ 如上述代码所示，`Middle`组件是不可变的，它的子组件`Button`也是不可变的，虽然拥有 context 可变状态，但是`setNumContext`本身是一个函数，函数本身是不可变的。按照之前的理论，`Button`组件会命中性能优化策略而跳过渲染，然而事实是，每当点击按钮`Button`组件都被重新渲染了。大大的问号当头一棒，原因是，在将 jsx 解析为 js 代码时，碰到没有`props`的节点会向`_jsx`函数传入`{}`，然后在新旧节点比较时 React 默认采取的是`全等比较`，那么新的节点和旧的`props`值都为`{}`，但是由于它们的地址不同，就会造成`newProps !== oldProps`，所以`Button`节点被认为发生了更新，触发了重新渲染。

> 这是一个具有传染性的规则，当从组件树某一个节点开始不能命中性能优化策略时，即使它的子孙组件可以命中性能优化策略，也会由于`props`的全等比较而重新渲染。如此，React.memo()应运而生。

##### React.memo()

​ `memo()`本质上是在 props 比较时，把默认的`全等比较`替换为了`浅比较（shallowEqual）`，`{}`的浅比较一定是全等的。所以将`memo()`用在希望命中性能优化策略组件的父组件上，即可保证从父组件开始新旧 props 的值是全等的。

```javascript
import { useState, useContext, createContext, memo } from 'react'

const numContext = createContext()
const setNumContext = createContext()

export default function App() {
  const [num, setNum] = useState(0)
  return (
    <numContext.Provider value={num}>
      <setNumContext.Provider value={setNum}>
        <Middle />
      </setNumContext.Provider>
    </numContext.Provider>
  )
}

const Middle = memo(() => {
  return (
    <>
      <Show />
      <Button />
      {/* _jsx(Button,{}) */}
    </>
  )
})

const Show = () => {
  const num = useContext(numContext)
  return <h1>{num}</h1>
}

const Button = () => {
  const setNum = useContext(setNumContext)
  console.log('Button rendered')
  return (
    <button
      onClick={() => {
        setNum(() => Math.round(Math.random() * 100))
      }}
    >
      random
    </button>
  )
}
```

##### PureComponent

​ 实际上在`memo()`未出现之前的 class 组件时期，使用 class 组件继承`PureComponent`类组件，即是对 props 和 state 进行了一层浅比较。

##### useMemo

​ 缓存一个变量，当依赖项不发生改变时，该变量始终读取缓存值。如果不使用`useMemo`，即使依赖的值不发生改变，每次渲染都会重新创建一个新变量。

##### useCallback

​ 缓存一个函数。其他同`useMemo`
