---
title: 使用hooks和mobx构建你的react项目
date: 2019-11-06 18:36:55
tags:
  - react
  - react hooks
  - mobx
  - 前端
categories:
  - [技术, 前端, react]
---

使用 React hooks 和 mobx 开发 React 项目也有一段时间了，对我来说，这是代码最少最简洁的技术组合和最舒适的开发方式。之前使用 class 组件，和 redux 管理状态的方式，深感其代码之繁琐。这篇博客通过使用 hooks 和 mobx 构建一个 todo APP，带大家来体验一下船新的开发方式

## 什么是 React hooks

> “Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。” -- 官网

Hook 发布也有一定时间了，相信大家对它都不会很陌生。如果不是很清楚的话，可以阅读[官网](https://zh-hans.reactjs.org/docs/hooks-intro.html)文档，写的很清晰明了, 也可以参考一下我朋友写的一篇 blog [react-hooks](https://songqiuxiao.github.io/2019/10/25/react-hooks/#more), 在此不过多介绍。

## 什么是 mobx

> 一个比 Redux 简单的多的状态管理工具，无需配置繁琐的 reducer 和 action，功能强大，编码时间减半，快乐加倍。灵感来自 excel 表格中的反应式编程原理，在使用过程中你会对此有深刻的感受。

### 三大概念

| 概念              | 对应的 Excel 特性         | 对应的 api  |
| ----------------- | ------------------------- | ----------- |
| State(状态)       | Excel 表格中的数据        | @observable |
| Derivations(衍生) | Excel 中的公式            | @computed   |
| Actions(动作)     | 在 Excel 中输入一个新的值 | @action     |

在此同样不展开深入 mobx 原理特性，以及和 redux 孰优孰劣的对比，请参考[官网](https://cn.mobx.js.org/)

## 写一个 Todo APP

### 1. create-react-app 并精简项目结构, 跑 eject

> create-react-app hooks-mobx-demo && cd hooks-mobx-demo

> yarn eject

### 2. 安装依赖, 配置 babel

> yarn add mobx mobx-react uuid

> yarn add --dev @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators @babel/helper-create-regexp-features-plugin

在 package.json 中配置 babel，这样你可以使用修饰器 decorator

```js
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
    [
      "@babel/plugin-proposal-decorators",
      {
        "legacy": true
      }
    ],
    [
      "@babel/plugin-proposal-class-properties",
      {
        "loose": true
      }
    ]
  ]
}
```

### 3. 编写 store

```js
import { observable, action, computed } from 'mobx';  // mobx的几个核心api
import { createContext } from 'react'; 
import uuid from 'uuid/v4'; // 用来生成随机数作为ID

class Store {
  // 可观察的状态，这里是用来存放todos的数组
  @observable todos = [
    {
      id: uuid(),
      desc: 'Play 🏀',
      isComplete: false
    },
    {
      id: uuid(),
      desc: 'Go to 🏥',
      isComplete: true
    }
  ];

  // 筛选出已完成的todos
  @computed get completedTodos() {
    return this.todos.filter(todo => todo.isComplete === true);
  }

  // 筛选出未完成的todos
  @computed get notCompletedTodos() {
    return this.todos.filter(todo => todo.isComplete === false);
  }

  // 计算todo的数量
  @computed get countTodos() {
    return this.todos.length;
  }

  // 添加todo的action
  @action addTodo = todo => (this.todos = [...this.todos, todo]);

  // toggle todo action
  @action toggleTodo = id =>
    (this.todos = this.todos.map(todo =>
      todo.id === id ? (todo = { ...todo, isComplete: !todo.isComplete }) : todo
    ));
}

const StoreContext = createContext(new Store());

export default StoreContext;
```

### 4. 编写 todo list

```JS
import React, { useContext, Fragment } from 'react';
import { observer } from 'mobx-react';  // 使得组件可以相应store的状态更改
import StoreContext from './store';

const TodoList = () => {
  // 用hook引入store
  const store = useContext(StoreContext);
  // 引入store中的变量和函数
  const { completedTodos, notCompletedTodos, toggleTodo, countTodos } = store;

  return (
    <Fragment>
      <div>Todos Completed</div>
      <ul>
        {completedTodos.map(item => (
          <li key={item.id} style={{ color: 'green' }} onClick={() => toggleTodo(item.id)}>
            {item.desc}
          </li>
        ))}
      </ul>
      <div>Todos Not Completed</div>
      <ul>
        {notCompletedTodos.map(item => (
          <li key={item.id} style={{ color: 'red' }} onClick={() => toggleTodo(item.id)}>
            {item.desc}
          </li>
        ))}
      </ul>
      <hr />
      <div>There are {countTodos} todos in total</div>
    </Fragment>
  );
};

// 用observer包装组件
export default observer(TodoList);
```

### 5. 编写 todo input

```JS
import React, { useState, useContext, Fragment } from 'react';
import { observer } from 'mobx-react';
import StoreContext from './store';
import uuid from 'uuid/v4';

const TodoInput = () => {
  const [input, setInput] = useState('');
  const store = useContext(StoreContext);
  const { addTodo } = store;
  const handleAddTodo = () => {
    addTodo({
      id: uuid(),
      desc: input,
      isComplete: false
    });
    setInput('');
  };
  return (
    <Fragment>
      <input placeholder="Add a todo here" value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={handleAddTodo}>Add</button>
    </Fragment>
  );
};

export default observer(TodoInput);
```

### 6. 把 TodoInput, TodoList import 到 App 组件中

```JS
import React from 'react';
import TodoInput from './TodoInput.jsx';
import TodoList from './TodoList.jsx';
function App() {
  return (
    <div>
      <TodoInput />
      <TodoList />
    </div>
  );
}

export default App;
```

### 7. 项目结构和效果

![项目结构](./img/hooks-mobx-demo.png)
![效果](./img/todo-app.png)

你也可以在我的[Github](https://github.com/stekovinbranturry/react-todo-app/tree/master/react-hooks-with-mobx) 中找到这个 demo 的代码

### 结语

通过这个例子，我们对使用 hook 和 mobx 编写 React 有了一个初步的认识。我们可以感受到 hook 和 mobx 代码之简洁，写起来非常直接了当。大家尝试一下吧~
