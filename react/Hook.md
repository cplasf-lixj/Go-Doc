# Hook

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。例如，`useState` 是允许你在 React 函数组件中添加 state 的 Hook。

## 1. useState

```js
// 通过在函数组件里调用useState来给组件添加一些内部 state。
// useState() 方法里面唯一的参数就是初始值，可以是数字、字符串、对象等
// useState 会返回一对值：当前状态和一个让你更新它的函数
import React, { useState } from 'react';
function Example() {
  // 声明一个叫 "count" 的 state 变量  
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
// 方括号是JavaScript的 数组解构。
```

## 2. useEffect

*Effect Hook* 可以让你在函数组件中执行副作用操作

### 2.1 无需清楚的effect

**在 React 更新 DOM 之后运行一些额外的代码。**比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。

```js
// 将 useEffect 放在组件内部让我们可以在 effect 中直接访问 count state 变量（或其他 props）。
// 默认情况下，useEffect在第一次渲染之后和每次更新之后都会执行。
import React, { useState, useEffect } from 'react';
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {    document.title = `You clicked ${count} times`;  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### 2.2 需要清楚的effect

如**订阅外部数据源**，清除工作是非常重要的，可以防止引起内存泄露。

```js
// Effect如果返回一个函数，则React将会在执行清楚操作时调用它：
// 在effect中返回一个函数是effect可选的清除机制。
// React 会在组件卸载的时候执行清除操作。
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {    
    function handleStatusChange(status) {      
      setIsOnline(status.isOnline);    
    }    
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);    
    // Specify how to clean up after this effect:    
    return function cleanup() {      
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);    
    };  
  });
  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

### 2.3 通过跳过Effect进行性能优化

在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。如果某些特定值在两次重渲染之间没有发生变化，可以通知 React **跳过**对 effect 的调用，只要传递数组作为 `useEffect` 的第二个可选参数即可：

```js
// 1. [count] 作为第二个参数
// 2. 如果 count 的值是 5，而且组件重渲染的时候count还是等于 5，React将对前一次渲染的 [5] 和后一次渲染的 [5] 进行比较。因为数组中的所有元素都是相等的(5 === 5)，React会跳过这个effect，这就实现了性能的优化。
// 3. 当渲染时，如果 count 的值更新成了 6，React 将会把前一次渲染时的数组 [5] 和这次渲染的数组 [6] 中的元素进行对比。这次因为 5 !== 6，React就会再次调用 effect。
// 4. 如果数组中有多个元素，即使只有一个元素发生变化，React 也会执行 effect。
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```

## 3. Hook规则

### 3.1 只在最顶层使用Hook

**不要在循环，条件或嵌套函数中调用 Hook，** 确保总是在 React 函数的最顶层以及任何 return 之前调用他们。

### 3.2 只在React函数中使用Hook

**不要在普通的 JavaScript 函数中调用 Hook。**

- ✅ 在 React 的函数组件中调用 Hook
- ✅ 在自定义 Hook 中调用其他 Hook

## 4. 自定义Hook

**自定义 Hook 是一个函数，其名称以 “`use`” 开头，函数内部可以调用其他的 Hook。**

**自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。**

```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {  
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

````js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
````

**自定义 Hook 必须以 “`use`” 开头**，否则，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了Hook 的规则。

## 5. Hook API索引

[Hook API](https://zh-hans.reactjs.org/docs/hooks-reference.html)