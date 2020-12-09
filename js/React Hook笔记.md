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

	>  Hook 在 `class组件内部`是不起作用的

3. **`React 16.8` 的新增特性**

4. **class组件的缺陷**：

|               **问题**               |           **解决方案**           |                  **缺点**                  | hook                                            |
| :----------------------------------: | :------------------------------: | :----------------------------------------: | ----------------------------------------------- |
| 生命周期臃肿、不相关的逻辑组合在一起 |                无                |                                            | effect hook模拟生命周期，把相关的逻辑组合在一起 |
|             逻辑难以复用             | HOC高阶组件、renderProps渲染属性 |                组件嵌套地狱                | 自定义hook                                      |
|             this指向问题             |         匿名函数 ()=>{}          | 每次都创建新的函数，子组件重复不必要的渲染 | 函数组件不存在this                              |
|                                      |               bind               |         增加和业务逻辑无关的代码量         |                                                 |

5. **Hook使用规则**：

	- 不要在普通的 JavaScript 函数中调用 Hook
	- 在 React 的函数组件中调用 Hook
	- 在自定义 Hook 中调用其他 Hook 
	- ==不要在条件判断或者循环语句中使用Hook==，因为Hook能正确赋值都依赖于<u>每次渲染时稳定的执行顺序（Hook调用的顺序）</u>

	

## 基础hook

### `useState`

1. 作用：

	存储函数组件内部状态。一般来说，在函数调用后变量就会”消失”，而 state 中的变量会被 React 保留

2. 用法：

	```js
	import React, { useState } from 'react'
	
	const Example = props =>  {
	  // 声明一个叫 “count” 的 state 变量
	  // count为变量名  setCount为更新该变量的函数
	    
	  //  写法1：
	  const [count, setCount] = useState(0)
	  //  写法2： 如果初始 state 需要通过复杂计算获得，那么可以传入一个函数，该函数只在初始渲染时被调用
	  const [count, setCount] = useState(()=>{
	      return someExpensiveComputation(props)
	  });
	    
	  return (
	      <p>You clicked {count} times</p>
		// 写法1：
		<button onClick={()=>{setCount(count+1)}>click</button>
	    // 写法2：  函数式更新，不依赖当前的state
		<button onClick={()=>{setCount(c => c+1)}>click</button>
	  )
	}
	```

> 1. 如果我们想要在 state 中存储多个不同的变量，只需调用 `useState()` 多次即可
> 2. `useState()`的变更函数如果传入当前的 state（state不变），则不会触发更新（React 使用 Object.is来比较 state）

### `useEffect`

1. 作用：
	使用 `useEffect` 完成副作用操作，赋值给 `useEffect` 的函数会在==组件渲染到屏幕之后执行==；
	模拟生命周期，使得`相关的逻辑`（副作用）组合在一起（在class组件中几个副作用都写在相同的生命周期函数中，逻辑混在一起）

2. 用法:

  ```js
  // 类似componentDidMount（仅在组件挂载时执行）
  useEffect(() => {
      // ...不需要清除的副作用
  }, [])
  
  // 类似componentDidMount、componentDidUpdate（仅在组件挂载和x或y任意其一变动时执行）
  useEffect(() => {
      // ...不需要清除的副作用
  }, [x,y]) // x,y为依赖
  
  // 类似componentDidMount、componentDidUpdate（仅在组件挂载和更新时执行）
  useEffect(() => {
      // ...不需要清除的副作用
  })
  
  // 类似componentDidMount、componentDidUpdate、componentWillUnmount（仅在组件挂载、更新和卸载时执行）
  useEffect(() => {
      // ...需要清除的副作用
      return () => {
          // ...清除副作用
      }
  })
  ```

  示例:

  ```js
  import React, { useState, useEffect } from 'react'
  
  const Example = props =>  {
    const [count, setCount] = useState(0)
    const [isOnline, setIsOnline] = useState(null)
    
    // 例1：  一些操作不需要清除副作用   指定依赖可以在组件更新时不重复执行对应副作用的代码
    // 组件挂载和更新时都会执行
    useEffect(() => {
      // 不需要清除的副作用
      document.title = `You clicked ${count} times`;
    };
      
    // ↓  对上面副作用的性能优化（只在组件挂载及依赖的count变量发生改变时执行）
    useEffect(() => {
        // 不需要清除的副作用
      document.title = `You clicked ${count} times`;
    }, [count]))
   
    
    // 例2：  一些操作需要清除副作用，在return中返回清除副作用的函数
    useEffect(() => {
      function handleStatusChange(status) {
        setIsOnline(status.isOnline);
      }
  	// 订阅，这是一个需要清除的副作用
      ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
      return () => {
          // 在卸载时进行取消订阅，消除副作用
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
      }
    })  
    // ...
  }
  ```

> effect hook中使用到的函数建议定义在该hook内部	[官网说明](https://react.docschina.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)
>
> 如果我的 effect 的依赖频繁变化，我该怎么办？	[官网说明](https://react.docschina.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)

### `useContext`

1. 作用：
	接收一个 context 对象（`React.createContext` 的返回值，含有Provider和Consumer）并返回该 context 的value，==并且该value是上层组件中距离当前组件最近的value==；（示例中会有体现）
	相当于 class 组件中的 `static contextType = MyContext` 或者 `<MyContext.Consumer>`

	|             **写法**             |                          **优缺点**                          |
	| :------------------------------: | :----------------------------------------------------------: |
	|      `<MyContext.Consumer>`      | 如果有多个context的话，会有嵌套地狱;且第一个子节点必须传个回调函数，增加工作量 |
	| `static contextType = MyContext` |        只能在class组件中使用，并且只能设置一个上下文         |
	|           `useContent`           |                  支持多个context,且写法简单                  |

	

2. 用法：

	```js
	const MyContext = React.createContext(initialValue)
	const value = useContext(MyContext)
	```

	示例：

	```js
	const themes = {
	  light: {
	    background: "#eeeeee"
	  },
	  dark: {
	    background: "#222222"
	  }
	}
	
	// 创建一个上下文，默认值是light
	const ThemeContext = React.createContext(themes.light)
	
	function App() {
	  return (
	     // 提供一个上下文值是dark
	    <ThemeContext.Provider value={themes.dark}>
	      <Toolbar />
	    </ThemeContext.Provider>
	  )
	}
	
	function Toolbar(props) {
	  return (
	      // 提供一个上下文值是light  （此时该上下文离ThemedButton组件最近）
	    <ThemeContext.Provider value={themes.light}>
	      <ThemedButton />
	    </ThemeContext.Provider>
	  )
	}
	
	function ThemedButton() {
	  // theme是light，因为最靠近此组件的上下文提供的值是light
	  const theme = useContext(ThemeContext)
	  return (
	    <button style={{ background: theme.background}}>
	      I am styled by theme context!
	    </button>
	  )
	}
	```

## 额外hook

### `useReducer`

1. 作用：
	类似 `redux` 中的功能，相较于 `useState`，它更适合一些逻辑较复杂且包含多个子值，或者下一个 `state` 依赖于之前的 `state` 等等的特定场景

2. 用法：

	```js
	
	const initialState = {
	   count: 0
	}
	
	const reducer = (state, action) => {
	  switch (action.type) {
	    case 'increment':
	      return {count: state.count + 1}
	    case 'decrement':
	      return {count: state.count - 1}
	    default:
	      throw new Error()
	  }
	}
	
	function App() {
	  const [state, dispatch] = useReducer(reducer, initialState)
	  return (
	    <>
	      点击次数: {state.count}
	      <button onClick={() => dispatch({type: 'increment'})}>+</button>
	      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
	    </>
	  )
	}
	```

	> `useReducer`可以配合`useContext`实现简易版redux（但不可替代redux，redux不仅仅可用在react中，还提供了中间件功能），实现原理为 context传递dispatch给下层所有子组件，子组件可调用dispatch改变状态。可以看下面的例子👇
	>
	> ```js
	>     const Ctx = React.createContext(null);
	> 
	>     const initialState = {
	>       count: 0
	>     }
	> 
	>     const reducer = (state, action) => {
	>       switch (action.type) {
	>         case 'increment':
	>           return { count: state.count + 1 }
	>         case 'decrement':
	>           return { count: state.count - 1 }
	>         default:
	>           throw new Error()
	>       }
	>     }
	> 
	>     const App = () => {
	>       const [state, dispatch] = React.useReducer(reducer, initialState)
	> 
	>       return (
	>         <Ctx.Provider value={{state,dispatch}}>
	>           Father count: {state.count}
	>           <button onClick={()=>dispatch({type:'increment'})}>+</button>
	>           <button onClick={()=>dispatch({type:'decrement'})}>-</button>
	>           <Son />
	>         </Ctx.Provider>
	>       )
	>     }
	> 
	>     const Son = () => {
	>       const {state,dispatch} = React.useContext(Ctx)
	>       return (
	>         <div>
	>           Son count: {state.count}
	>           <button onClick={()=>dispatch({type:'increment'})}>+</button>
	>           <button onClick={()=>dispatch({type:'decrement'})}>-</button>
	>         </div>
	>       )
	>     }
	> ```
	>
	> [官网说明](https://react.docschina.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)

### `useMemo`

1. 作用：
	把`回调函数`和`依赖项数组`作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 ==memoized 值==（缓存了这个值）

2. 目的： 性能优化，避免在每次渲染时都进行高开销的计算

3. 用法：

	```js
	const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])
	```

	> 需要注意的是，==`useMemo` 会在渲染的时候执行==，而不是渲染之后执行，这一点和 `useEffect` 有区别，所以 `useMemo` 不建议有副作用相关的逻辑

### `useCallback`

1. 作用：
	把`回调函数`及`依赖项数组`作为参数传入 `useCallback`，它将==返回该回调函数的 memoized 版本==（缓存了这个函数），该回调函数仅在某个依赖项改变时才会更新

2. 目的：性能优化

3. 用法：

	```js
	const memoizedCallback = useCallback(
	  () => {
	    doSomething(a, b);
	  },
	  [a, b],
	)
	```

	>==`useMemo`返回一个被缓存的值，而`useCallback`返回一个被缓存的函数==
	>
	>`useCallback`是`useMemo`的语法糖，useCallback(fn, deps)  相当于 useMemo(() => fn, deps)

### `useRef`

1. 作用：
	`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变

2. 用法：

	```js
	const refContainer = useRef(initialValue)
	```

3. 用例
	（1）访问子组件

	```js
	function TextInputWithFocusButton() {
	  const inputEl = useRef(null); // 访问子组件时初始值设为null
	  const onButtonClick = () => {
	    // `current` 指向已挂载到 DOM 上的input元素
	    inputEl.current.focus();
	  };
	  return (
	    <>
	      <input ref={inputEl} type="text" />
	      <button onClick={onButtonClick}>Focus the input</button>
	    </>
	  );
	}
	```

	（2）缓存值

	`useRef` 会在每次渲染时返回同一个 ref 对象，在其current属性上赋值可以达到缓存效果

	```js
	function App () {
	  const [ count, setCount ] = useState(0)
	  const timer = useRef(null)
	  let timer2 
	  
	  useEffect(() => {
	    let id = setInterval(() => {
	      setCount(count => count + 1)
	    }, 500)
	
	    timer.current = id
	    timer2 = id
	    return () => {
	      clearInterval(timer.current)
	    }
	  }, [])
	    
	  const onClickRef = useCallback(() => {
	    clearInterval(timer.current)
	  }, [])
	
	  const onClick = useCallback(() => {
	    clearInterval(timer2)
	  }, [])
	
	  return (
	    <div>
	      点击次数: { count }
	      <button onClick={onClick}>普通</button>
	      <button onClick={onClickRef}>useRef</button>
	    </div>
	    )
	}
	```

### `useImperativeHandle`

1. 作用：
	子组件可以选择性的暴露给父组件一些方法，`useImperativeHandle`应当与 `forwardRef` 一起使用

2. 用法：

	```js
	const FancyInput = (props, ref) => {
	  const inputRef = useRef();
	  useImperativeHandle(ref, () => ({
	    focus: () => {
	      inputRef.current.focus();
	    }
	  }));
	  return <input ref={inputRef} ... />;
	}
	FancyInput = forwardRef(FancyInput);
	
	
	const App = () => {
	    const fancyRef = useRef(null)
	    
	    const onClickHandle = () => {
	        fancyRef.current.focus()
	    }
	    return (
			<FancyInput ref={inputRef} /> 
			<button onClick={onClickHandle}>点我</button>
		)
	}
	```

### `useLayoutEffect`

1. 作用：
	在处理DOM的时候,当你的useEffect里面的操作需要处理DOM,并且会改变页面的样式,就需要用这个,否则可能会出现出现闪屏问题

2. useEffect和useLayoutEffect的区别：
	`useEffect`: 
	      基本上90%的情况下,都应该用这个。这个是在render结束后，callback函数执行，但是==不会阻塞浏览器的绘制==，算是某种异步的方式吧。但是class组件的componentDidMount 和componentDidUpdate是同步的,在render结束后就运行。useEffect在大部分场景下都比class的方式性能更好

	`useLayoutEffect`:

	​     当你的useEffect里面的操作需要处理DOM，改变页面的样式，就需要用这个，否则可能会出现出现闪屏问题,。callback函数会在DOM更新完成后立即执行,但是==会在浏览器进行任何绘制之前运行完成,阻塞了浏览器的绘制==

	[参考简书](https://www.jianshu.com/p/412c874c5add)

## 自定义hook

1. 作用：

	可以将组件逻辑提取到可重用的函数中

	自定义 Hook 是一个函数，其名称以 “`use`” 开头，函数内部可以调用其他的 Hook

	我们可以自由的决定它的参数是什么，以及它应该返回什么

2. 例子：
	实现一个useReducer

	```js
	function useReducer(reducer, initialState) {
	  const [state, setState] = useState(initialState);
	
	  function dispatch(action) {
	    const nextState = reducer(state, action);
	    setState(nextState);
	  }
	
	  return [state, dispatch];
	}
	```

	











