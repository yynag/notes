# React

## React基础

本节采用 **UI通用方法 + React如何实现** 的方式来记录笔记，有利于统一整个前端的体系。

### 基本单元

React 基本单元是组件，而组件是一段可以 使用标签进行扩展 的 JavaScript 函数。

```jsx
// React 组件是常规的 JavaScript 函数。
// 组件可以复用和嵌套。
// 组件的名称必须以大写字母开头，否则无法运行。
// ⚠️ 不要在组件里面定义组件，要在顶层中定义。
function Profile() {
  // 使用标签，这种语法被称为 JSX。
  // 必须用双括号括起来。
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  ) 
}

// 定义可以导出的主函数
export default function Gallery() {
// 在里面使用
  return (
    <section>
      <h1>了不起的科学家</h1>
      <Profile />
      <Profile />
    </section>
  );
}
```

> 组件：https://react.docschina.org/learn/your-first-component 
> 组件导入和导出：https://react.docschina.org/learn/importing-and-exporting-components 
> JSX 的使用和注意事项：https://react.docschina.org/learn/importing-and-exporting-components 
> JSX 使用 JS 对象：https://react.docschina.org/learn/javascript-in-jsx-with-curly-braces 

---

[渲染列表](https://react.docschina.org/learn/rendering-lists) 需要使用 Key 用于唯一标识 Item。


### 数据响应

如果通过父子 Props 来传递数据，可以根据传递的参数来呈现不同的UI。

> Props：https://react.docschina.org/learn/passing-props-to-a-component 

对于通过 Props 来响应数据的组件，会直接触发整个组件的重建。

条件渲染：可以根据具体逻辑来返回不同的View树，如果不想返回可以直接返回null。

> 条件渲染：https://react.docschina.org/learn/conditional-rendering 


### 可变和不可变/有状态与无状态

不可变/无状态：

React 的不可变通过 纯函数 来实现，即 同样的输入产生同样的结果，没有任何副作用。好处：

1. 不依赖外部环境（无副作用），可以在任何环境运行，比如 服务器。

2. 对未变化的组件进行缓存，下次需要的时候就可以跳过渲染。

3. 纯函数可以随时终止，不用担心外部数据的一致性，避免浪费。

React 提供了 “严格模式”，在严格模式下开发时，它将会调用每个组件函数两次。通过重复调用组件函数，严格模式有助于找到违反这些规则的组件。严格模式在生产环境下不生效，因此它不会降低应用程序的速度。如需引入严格模式，你可以用 <React.StrictMode> 包裹根组件。一些框架会默认这样做。

> 保持组件纯粹 https://react.docschina.org/learn/keeping-components-pure 

副作用（非纯函数）在一个组件中一般就是 事件处理函数，它会改变外界的东西，但它并不是在渲染期执行，所以可以不是纯函数，这个不影响。

如果实在没办法用 事件处理 的方式解决 副作用，比如连接聊天室外部服务，那就用 useEffect 方法 来声明启动和结束，这是最后的手段。

> 哪些地方 可能 引发副作用 https://react.docschina.org/learn/keeping-components-pure#where-you-can-cause-side-effects 

---

可变/有状态：

useState 保存了组件不会重置的数据，那怎么保证每次拿出来的数据是正确的？方法就是保证调用顺序，内部维护一个增长的 Index，每个 State 次序是固定的，就按固定的拿就可以了。 

useState 的原理就是，当 setState 的时候，改变值，然后触发 DOM 更新。

> State：组件的记忆 https://react.docschina.org/learn/state-a-components-memory  

使用 setState 改变组件会触发重渲染，React 会比较前后哪些发生变化，然后去渲染，这就要注意：如果组件变更层次较高，会触发大范围的重新渲染。

> 渲染和提交 https://react.docschina.org/learn/render-and-commit  

在一个渲染周期内，调用多次 setState 效果只有一次，因为流程就是，运行完程序，标记为脏，而不是我调用一次 setState 就渲染一次，调用一次就渲染一次。

> state 如同一张快照 https://react.docschina.org/learn/state-as-a-snapshot 

如果你想在下次渲染之前多次更新同一个 state，你可以像 setNumber(n => n + 1) 这样传入一个根据队列中的前一个 state 计算下一个 state 的 函数，而不是像 setNumber(number + 1) 这样传入 下一个 state 值。这是一种告诉 React “用 state 值做某事”而不是仅仅替换它的方法。

setState(x) 实际上会像 setState(n => x) 一样运行，只是没有使用 n！

> 把一系列 state 更新加入队列 https://react.docschina.org/learn/queueing-a-series-of-state-updates 

应该 把所有存放在 state 中的 JavaScript 对象都视为只读的，并且使用 setXXX 去更新对象，因为 1、更新比较策略优化，2、不会触发调试改变。

```jsx
// 可以使用对象展开语法，这样就很方便。
// 请注意 ... 展开语法本质是是“浅拷贝”——它只会复制一层。
setPerson({
  ...person, // 复制上一个 person 中的所有字段
  firstName: e.target.value // 但是覆盖 firstName 字段 
});

// 更新嵌套对象
setPerson({
  ...person, // 复制其它字段的数据 
  artwork: { // 替换 artwork 字段 
    ...person.artwork, // 复制之前 person.artwork 中的数据
    city: 'New Delhi' // 但是将 city 的值替换为 New Delhi！
  }
});
```

使用 第三方库 Immer 编写简洁的更新逻辑，可以让对象扁平化修改。

> 更新 state 中的对象 https://react.docschina.org/learn/updating-objects-in-state 

数组自己看，CURD 不能直接修改，也可以使用 Immer 这个库。

> 更新 state 中的数组 https://react.docschina.org/learn/updating-arrays-in-state 

可以使用 Key 来重新渲染某个组件。

> 方法二：使用 key 来重置 state https://react.docschina.org/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key 

### 事件处理

内置的组件都支持浏览器事件处理函数，你也可以自己定制。

> https://react.docschina.org/reference/react-dom/components/common#common-props 
> 命名事件处理函数 prop  https://react.docschina.org/learn/responding-to-events#naming-event-handler-props 

要处理点击事件，请使用 <button onClick={handleClick}> 而不是 <div onClick={handleClick}>。

事件处理函数 可以用参数传递。

事件传播：在 React 中所有事件都会传播，除了 onScroll，它仅适用于你附加到的 JSX 标签。

阻止传播：e.stopPropagation()

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      // 你也可以在调用父元素 onClick 函数之前，向这个处理函数添加更多代码。此模式是事件传播的另一种 替代方案 。它让子组件处理事件，同时也让父组件指定一些额外的行为。与事件传播不同，它并非自动。但使用这种模式的好处是你可以清楚地追踪因某个事件的触发而执行的整条代码链。
      onClick();
    }}>
      {children}
    </button>
  );
}
```

阻止默认行为：e.preventDefault();

```jsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('提交表单！');
    }}>
      <input />
      <button>发送</button>
    </form>
  );
}
```

> 响应事件 https://react.docschina.org/learn/responding-to-events. 


### 动画和过渡




### 通信：父子/兄弟/任意

父子：使用 Props 来传递数据，组件不能改变 Props，只能通知父组件去改变。

> 参见：https://react.docschina.org/learn/passing-props-to-a-component 

```jsx
function Avatar({ person, size }) { // 在这里 person 和 size 是可访问的 }
```

---

兄弟：通过 Props 通知父改变数据，然后传递到兄弟节点之间。


> 在组件间共享状态 https://react.docschina.org/learn/sharing-state-between-components 


一个是 State，然后 Reducer，还有 Context，最后是 Context + Reducer，就不再需要任何状态管理了。

> 状态管理 https://react.docschina.org/learn/managing-state  

State：

Reducer：管理多个数据状态的变化，适合多个数据放在同一个业务逻辑里统一处理的情况。也可以使用 Immer 简化 reducers 。

> 迁移状态逻辑至 Reducer 中 https://react.docschina.org/learn/extracting-state-logic-into-a-reducer 

Context：

> 使用 Context 深层传递参数 https://react.docschina.org/learn/passing-data-deeply-with-context 

Reducer + Context：Reducer 可以整合组件的状态更新逻辑。Context 可以将信息深入传递给其他组件。你可以组合使用它们来共同管理一个复杂页面的状态。

> 使用 Reducer 和 Context 拓展你的应用 https://react.docschina.org/learn/scaling-up-with-reducer-and-context 


> 状态管理 https://react.docschina.org/learn/managing-state 


### 操作 DOM

有些数据的 存储和修改 不想让 React 捕捉到，所以用 ref。原理：在 useState的基础上，不用 setXXX 函数，通过内部修改，内部读取。

ref 的最佳实践：https://react.docschina.org/learn/referencing-values-with-refs#best-practices-for-refs 

> 使用 ref 引用值 https://react.docschina.org/learn/referencing-values-with-refs 

用 ref 获胜 DOM 节点，做一些操作，比如 获得焦点，滚动到那个位置。

何时使用：https://react.docschina.org/learn/manipulating-the-dom-with-refs#when-react-attaches-the-refs 

最佳实践：https://react.docschina.org/learn/manipulating-the-dom-with-refs#best-practices-for-dom-manipulation-with-refs 

使用 flushSync 强制同步渲染，然后其他操作。https://react.docschina.org/learn/manipulating-the-dom-with-refs#flushing-state-updates-synchronously-with-flush-sync 

> 使用 ref 操作 DOM https://react.docschina.org/learn/manipulating-the-dom-with-refs 

## 特性

### Hook

以“use”开头的函数都被称为 Hook。

Hook 是特殊的函数，只在 React 渲染时有效，目的是能让你 “hook” 到不同的 React 特性中去。

Hooks ——以 use 开头的函数只能在组件或自定义 Hook 的最顶层调用。不能在条件语句、循环语句或其他嵌套函数内调用 Hook。

### 受控组件/非受控组件

当组件中的重要信息是由 props 而不是其自身状态驱动时，就可以认为该组件是“受控组件”。这就允许父组件完全指定其行为。

非受控组件通常很简单，因为它们不需要太多配置。但是当你想把它们组合在一起使用时，就不那么灵活了。受控组件具有最大的灵活性，但它们需要父组件使用 props 对其进行配置。

> 受控组件和非受控组件 https://react.docschina.org/learn/sharing-state-between-components#controlled-and-uncontrolled-components 


==================================================

## Effect

一个组件包含三个东西：渲染逻辑代码（包括数据和UI）、时间处理程序 和 副作用Effect。

Effect 的作用和使用场景 是负责与外部系统进行同步，比如连接服务，发送日志，视频播放器 等。

所以如果你有某种需求，并不是与外部系统进行同步，可能不应该使用 Effect。 

Effect 包含 启动代码 + 清理代码 + 依赖，其中 启动代码 + 清理代码 用来保证原子性，不会产生任何中间态的副作用，让函数尽量回归纯粹性，而 依赖 是保证只有依赖变化才会导致代码运行。

useEffect 执行时机：

```jsx
useEffect(() => {
  // 这里的代码会在每次渲染后执行
});

useEffect(() => {
  // 这里的代码只会在组件挂载后执行
}, []);

useEffect(() => {
  //这里的代码只会在每次渲染后，并且 a 或 b 的值与上次渲染不一致时执行
}, [a, b]);
```

开发环境下默认严格模式，会对 Effect 会执行两次进行检查，所以有的时候要构造情况来清理，比如网络请求的时候，可以使用带有缓存的方法。详细参见：[Effect 中有哪些好的数据获取替代方案？](https://react.docschina.org/learn/synchronizing-with-effects#what-are-good-alternatives-to-data-fetching-in-effects)。

如果是日志记录，重复多次的确没办法清理，这个时候注意就行。

> [使用 Effect 同步](https://react.docschina.org/learn/synchronizing-with-effects)

---

不需要使用 Effect 的情况：

- 不必使用 Effect 来转换渲染所需的数据。
	
	Effect 的行为有点类似 Mounted 生命周期函数，有时候喜欢在这个阶段去更改UI状态，除非你要获取外部浏览器的什么参数。如果想计算数据，可以直接计算。
	```jsx
	  const [firstName, setFirstName] = useState('Taylor');
  	  const [lastName, setLastName] = useState('Swift');
 	   // ✅ 非常好：在渲染期间进行计算
  	  const fullName = firstName + ' ' + lastName;
	```

- 不必使用 Effect 来处理用户事件。

- 缓存昂贵的计算。可以使用 useMemo Hook 缓存。

- 当 props 变化时重置所有 state。这种情况就是，只能通过触发组件重新创建，如果带有副作用的组件，可以把组件进一步拆分，可以使用 Key 来重新创建组件。

- 当 prop 变化时调整部分 state。直接在代码里计算，如果带有 Effect 还不能创建的话，就拆分组件。

- 在事件处理函数中共享逻辑。当你不确定某些代码应该放在 Effect 中还是事件处理函数中时，先自问 为什么 要执行这些代码。Effect 只用来执行那些显示给用户时组件 需要执行 的代码。【 不是行为代码 】

- 发送 POST 请求。基于行为的方法不应该在 Effect 中执行，但是有些情况 组件显示给用户的时候需要执行。

- 链式计算。

- 初始化应用。还是那个原则，是否是行为，如果不是就用 Effect，然后做好清理方法。

- 等等等。。。。【 当搞清楚原则后，就可以灵活运用了。 】

> [你可能不需要 Effect](https://react.docschina.org/learn/you-might-not-need-an-effect)

---

生命周期：

组件可以挂载、更新或卸载。Effect 只能做两件事：开始同步某些东西，然后停止同步它。

> [响应式 Effect 的生命周期](https://react.docschina.org/learn/lifecycle-of-reactive-effects)





