---
layout: react
title: React 优化，memo、useCallback、useMemo
date: 2020-03-14 04:46:05
tags: React
---


近来 Hooks 推出，慢慢的函数式组件使用场景变多，但要注意在函数式组件中重新渲染和计算会造成多余性能开销。

所以本文将会在后面介绍 React 如何优化，以及所使用的方法

<!-- more -->

### 前言

我们在写 React 项目时，都是由组件（Component）搭积木一样组合构成。但是在复杂的项目中，如果没有经过优化，就可能在性能上不尽人意。

近来 Hooks 推出，慢慢的函数式组件使用场景变多，但要注意在函数式组件中重新渲染和计算会造成多余性能开销。

所以本文将会在后面介绍 React 如何优化，以及所使用的方法：

- **memo**
- **useMemo**
- **useCallback**

### React.memo

子组件通常依赖父组件的**状态（State）**或者**事件（Event）**，使用 props 传入子组件中。如果父组件传递的数据变化，会导致子组件重新渲染。虽然可能子组件没有改变，这就浪费的性能。

这时候就轮到 React.memo 出场了，它是以HOC（高阶组件 Higher Order Component）的方式使用。

```jsx
const MyComponent = React.memo(function MyComponent(props){
  /* render using props */
});
```

那我们来实际举个例子：

```jsx

const SubComponent = (props) => {
  const { name } = props;
  console.log(name);
  return <div>name: {name}</div>;
}

const MemoSubComponent = React.memo(SubComponent);

const ParentComponent = () => {
  const [count, setCount] = React.useState(0);
  
  return (
      <div>
          <div>{count}</div>
          <SubComponent name={'SubComponent'} />
          <MemoSubComponent name={'MemoSubComponent'} />
          <button onClick={() => setCount(count + 1)}>Click me</button>
      </div>
  );
}

```
[例子链接](https://codesandbox.io/s/throbbing-water-60qt9)

以上例子代码运行后，点击 Click me 按钮，在 Console 控制台中，可以看到优化过的 MemoSubComponent 组件不会重复打印相同的 name，而没有优化的子组件 SubComponent 在父组件每一次 render 时也跟着重新渲染了。

但是要注意的是，React.memo 采用的是 **浅比较**，所以如果你的 props 里面有 Object 时，由于父组件每次渲染，props 内存将会重新分配新地址，所以 React.memo 防止重复渲染就会失败。

解决这个问题可以在在 React.memo 第二个参数中传递一个对比函数，让我们自己来处理需不需要更新。用法类似于 Class 组件的 shouldComponentUpdate。

```jsx
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
React.memo(MyComponent, areEqual);
```

还有一种办法我们可以看看 React.useCallback。

### React.useCallback

上面说到：
> 传递的 props 中有 Object 时，由于父组件每次渲染，props 内存将会重新分配新地址 

useCallback 方法返回的结果将会缓存，如果依赖的数据没有变化，则缓存可用，否则重新计算。

```jsx
const memoNumber = React.useCallback(() => a + b, [a, b]);
```
按照纯函数逻辑，以上代码中如果我们 a 或者 b 没有变化，那么结果肯定是一致的，缓存即有效。

> React 在官方文件中有提到， every value referenced inside the callback should also appear in the dependencies array，因此在使用 useCallback 时必须注意 依赖数组 中是否都包含了在 useCallback 中使用的变量，否则可能导致 useCallback 失效。


### React.useMemo

还有一种情况与父组件中无关，但是在子组件重新渲染时，如果某个计算太复杂，这样重复渲染会做重复的复杂计算，即使它计算出来的结果前后是一致的。

例如：

```jsx
const calc = (a, b) => {
    // 假设这里做了复杂的计算，暂时用次幂模拟
    return a ** b;
}
const MyComponent = (props) => {
    const {a, b} = props;
    const c = calc(a, b);
    return <div>c: {c}</div>;
}
```

如果 calc 计算耗时 1000ms，那么每次渲染都要等待这么久，怎么优化呢？
> a, b 值不变的情况下，得出的 c 定是相同的。

故，我们可以用 useMemo 把值给缓存起来，避免重复计算相同的结果。


```jsx
const calc = (a, b) => {
    // 假设这里做了复杂的计算，暂时用次幂模拟
    return a ** b;
}
const MyComponent = (props) => {
    const {a, b} = props;
    // 缓存
    const c = React.useMemo(() => calc(a, b), [a, b]);
    return <div>c: {c}</div>;
}
```

> 正因为需要知道 a, b 值是否改变，才能判断出缓存的 c 值是否可用，所以同样的，需要在 useMemo 第二个参数，依赖数组中把函数内用到的变量写上。
> 也不要什么东西都放在 useMemo 中处理，要按情况来使用，要不然又变成反优化了噢！

