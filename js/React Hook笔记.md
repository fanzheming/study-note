# React Hook笔记

## 介绍

1. **Hook 是什么？** 

	Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。例如，`useState` 是允许你在 React 函数组件中添加 state 的 Hook

2.  **在哪里使用Hook ?**

	```js
	// 函数组件中
	const Example = props => {
	  // 你可以在这使用 Hook
	  return <div/>
	}
	```

	>  Hook 在 类组件内部是不起作用的

3. `React 16.8` 的新增特性

4. 类组件的缺陷：

  |        **问题**        |           **解决方案**           |                  **缺点**                  | hook                                          |
  | :--------------------: | :------------------------------: | :----------------------------------------: | --------------------------------------------- |
  | 生命周期臃肿、逻辑耦合 |                无                |                                            | effect hook模拟生命周期，细分逻辑减小逻辑臃肿 |
  |      逻辑难以复用      | HOC高阶组件、renderProps渲染属性 |                组件嵌套地狱                | 自定义hook                                    |
  |      this指向问题      |         匿名函数 ()=>{}          | 每次都创建新的函数，子组件重复不必要的渲染 | 函数组件不存在this                            |
  |                        |               bind               |         增加和业务逻辑无关的代码量         |                                               |

## 基础hook

### `useState`

1. 作用：

	存储函数组件内部状态。一般来说，在函数调用后变量就会”消失”，而 state 中的变量会被 React 保留

2. 用法：

	```js
	import React, { useState } from 'react';
	
	const Example = props =>  {
	  // 声明一个叫 “count” 的 state 变量
	  // count为变量名  setCount为更新该变量的函数
	  const [count, setCount] = useState(0);
	  return (
	  	<p>You clicked {count} times</p>
		<button onClick={()=>{setCount(count+1)}>click</button>
	  )
	}
	```

	>如果我们想要在 state 中存储两个不同的变量，只需调用 `useState()` 两次即可









