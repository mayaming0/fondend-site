# ⚛️ React 知识整理

> **项目学习路径总结** — 基于 React 18 + Redux Toolkit + React Router v6

---

## 📦 一、项目基建

| 工具                               | 用途                         |
| ---------------------------------- | ---------------------------- |
| `create-react-app`                 | 脚手架                       |
| `antd` (v5)                        | UI 组件库（Layout、Menu 等） |
| `react-router-dom` (v6)            | 路由管理                     |
| `@reduxjs/toolkit` + `react-redux` | 状态管理                     |
| `use-immer`                        | 简化不可变数据操作           |
| `uuid`                             | 唯一 ID 生成                 |

---

## 🧠 二、核心 Hooks

### 1. `useState` — 状态管理基石

```js
const [state, setState] = useState(initialValue);
```

**关键要点：**

- **直接替换**：`setState(newValue)` — 直接用新值替换
- **函数式更新**：`setState(prev => prev + 1)` — 基于旧值计算新值
- **批量更新**：React 18 自动批处理，多次 `setState` 合并为一次渲染
- **状态保留规则**：UI 树相同位置 + 相同组件类型 → state 保留；用 `key` 强制重置

**更新对象 / 数组（不可变模式）：**

| 操作 | 对象                         | 数组                                              |
| ---- | ---------------------------- | ------------------------------------------------- |
| 添加 | `{...obj, newKey: val}`      | `[...arr, newItem]` 或 `[newItem, ...arr]`        |
| 修改 | `{...obj, key: newVal}`      | `arr.map(i => i.id === id ? {...i, changed} : i)` |
| 删除 | `const {key, ...rest} = obj` | `arr.filter(i => i.id !== id)`                    |
| 插入 | —                            | `[...arr.slice(0, i), item, ...arr.slice(i)]`     |

---

### 2. `useReducer` — 复杂状态逻辑集中管理

```js
const [state, dispatch] = useReducer(reducer, initialState);
// 带初始化函数（惰性求值）：
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

**模式：**

```
dispatch({ type: 'added', payload }) → reducer(state, action) → newState
```

**Redux vs useReducer：** `useReducer` 是局部状态方案，适合组件树内共享复杂逻辑；Redux 是全局状态方案，适合跨多层的全局数据流。

> 💡 **进阶：** `useImmerReducer` 允许在 reducer 中直接「修改」draft，由 Immer 自动生成不可变 state。

---

### 3. `useContext` — 跨层数据传递

**三步法：**

```js
// 1. 创建
const MyContext = createContext(defaultValue)
// 2. 提供
<MyContext.Provider value={data}>  // value 变化 → 所有消费者重渲染
// 3. 消费
const data = useContext(MyContext)
```

**最佳实践 — Context + Reducer 组合：**

```js
// 拆分两个 Context：数据 + dispatch，避免不必要重渲染
const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

function TasksProvider({ children }) {
	const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
	return (
		<TasksContext.Provider value={tasks}>
			<TasksDispatchContext.Provider value={dispatch}>
				{children}
			</TasksDispatchContext.Provider>
		</TasksContext.Provider>
	);
}
// 封装自定义 Hook
const useTasks = () => useContext(TasksContext);
const useTasksDispatch = () => useContext(TasksDispatchContext);
```

> ⚠️ **注意：** `Context.Provider` 的 `value` 变化时，**所有**消费该 Context 的组件都会重渲染。拆分 Context 或用 `useMemo` 包裹 value 可优化。

---

### 4. `useRef` & `forwardRef` & `useImperativeHandle`

```js
// 存储不触发渲染的值 / 操作 DOM
const ref = useRef(initialValue)        // ref.current 读写
<input ref={ref} />                      // DOM 绑定

// 自定义组件暴露 DOM
const MyInput = forwardRef((props, ref) => (
  <input {...props} ref={ref} />
))

// 限制暴露的功能
useImperativeHandle(ref, () => ({
  focus() { inputRef.current.focus() },
  // 只暴露 focus，隐藏其他 DOM 能力
}), [])
```

**循环列表中多个 ref：**

```js
const itemsRef = useRef(new Map())
// 回调 ref
<div ref={(node) => itemsRef.current.set(id, node)} />
```

**`flushSync` — 强制同步更新 DOM：**

```js
flushSync(() => setTodos([...todos, newTodo]));
// 此时 DOM 已更新，可安全操作
listRef.current.scrollIntoView();
```

> `useRef` 与 `useState` 的核心区别：ref 变化**不触发重渲染**，适合存储「不需要展示」的值（定时器 ID、DOM 引用等）。

---

### 5. `useEffect` — 处理副作用

```js
useEffect(() => {
	// 副作用逻辑
	return () => cleanup(); // 清理函数（卸载或重新执行前调用）
}, [dependencies]);
```

| 依赖模式     | 执行时机          |
| ------------ | ----------------- |
| 无第二个参数 | 每次渲染后执行    |
| `[]`         | 仅挂载时执行      |
| `[a, b]`     | a 或 b 变化时执行 |

**⚠️ 不滥用 Effect — React 哲学：**

| 场景                  | 推荐做法               | 不推荐                     |
| --------------------- | ---------------------- | -------------------------- |
| 缓存计算结果          | `useMemo`              | `useEffect` + `setState`   |
| 用户交互后重置 state  | 事件处理函数中直接操作 | 监听 state 变化的 Effect   |
| 数据转换（过滤/排序） | 渲染时直接计算         | Effect + setState 存储结果 |

**`useEffectEvent`（实验性）：**
在 Effect 中安全读取最新 props/state，无需加入依赖数组。

```js
const onVisit = useEffectEvent((visitedCount) => {
	/* 读取最新 props */
});
useEffect(() => {
	onVisit(count);
}, [count]);
```

---

### 6. `useMemo` & `useCallback` — 性能优化

```js
// 缓存计算结果（引用相等性）
const sortedList = useMemo(() => arr.sort(), [arr])

// 缓存函数引用
const handleClick = useCallback(() => fn(id), [id])

// 搭配 memo 跳过子组件重渲染
const Child = memo(({ onClick, data }) => { ... })
```

| Hook          | 缓存目标            | 搭配   |
| ------------- | ------------------- | ------ |
| `useMemo`     | 计算结果 / 对象引用 | `memo` |
| `useCallback` | 函数引用            | `memo` |

> **不要过早优化！** 只在确实有性能问题时使用。`useMemo` 和 `useCallback` 本身有开销。

---

### 7. `useTransition` — 降低非紧急更新优先级

```js
const [isPending, startTransition] = useTransition();

startTransition(() => {
	setTab(nextTab); // 标记为低优先级更新
});
```

**核心价值：** 让用户操作（输入、点击）不被昂贵的渲染阻塞。`isPending` 可在过渡期间展示加载反馈。

> `startTransition` 包裹的更新会被中断，以响应更高优先级的交互（如输入）。

---

## 🔗 三、React Router v6

使用 `createBrowserRouter` + 嵌套路由：

```js
const router = createBrowserRouter([
	{
		path: "/",
		element: <App />,
		children: [
			{ path: "stateLearn", element: <StateLearn /> },
			{ path: "usingReducer", element: <UsingReducer /> },
			// ...
		],
	},
]);
```

**常用 API：** `useNavigate()` 编程式导航、`<Outlet />` 嵌套路由出口、`<Link>` / `<NavLink>` 声明式导航。

---

## 🗃️ 四、Redux Toolkit

### 配置三步法

```js
// 1. 创建 slice
const counterSlice = createSlice({
	name: "counter",
	initialState: { value: 0, status: "idle" },
	reducers: {
		increment: (state) => {
			state.value += 1;
		}, // Immer 可变写法
		incrementByAmount: (state, action) => {
			state.value += action.payload;
		},
	},
});
export const { increment, incrementByAmount } = counterSlice.actions;

// 2. 配置 store
const store = configureStore({
	reducer: { counter: counterReducer },
});

// 3. 组件中使用
const count = useSelector((state) => state.counter.value);
const dispatch = useDispatch();
dispatch(increment());
```

### 异步处理

```js
export const incrementAsync = createAsyncThunk(
	"counter/fetchCount",
	async (amount) => {
		const res = await fetch(`/api/count?amount=${amount}`);
		return res.data;
	},
);

// extraReducers 处理异步生命周期
extraReducers: (builder) => {
	builder
		.addCase(incrementAsync.pending, (state) => {
			state.status = "loading";
		})
		.addCase(incrementAsync.fulfilled, (state, action) => {
			state.status = "idle";
			state.value += action.payload;
		})
		.addCase(incrementAsync.rejected, (state) => {
			state.status = "failed";
		});
};
```

### 手写 Thunk

```js
export const incrementIfOdd = (amount) => (dispatch, getState) => {
	const value = selectors(getState());
	if (value % 2 === 1) dispatch(incrementByAmount(amount));
};
```

---

## 🧩 五、Immer — 不可变数据简化方案

| 原生（不可变）        | Immer（可变写法）         |
| --------------------- | ------------------------- |
| `{...obj, x: newVal}` | `draft.x = newVal`        |
| `arr.map(...)`        | `draft[index].prop = val` |
| `arr.filter(...)`     | `draft.splice(idx, 1)`    |
| `[...arr, item]`      | `draft.push(item)`        |

**两种使用方式：**

- **`useImmer`**：替代 `useState`，`const [state, update] = useImmer(init)`，直接修改 `draft`
- **`useImmerReducer`**：替代 `useReducer`，reducer 中直接操作 `draft`，Immer 自动生成不可变 state

---

## 🏛️ 六、浅谈 React 设计模式

### 状态提升

多个子组件共享状态时，将状态提升到最近的共同父组件中，通过 props 下传。

### 组合 vs 继承

React 推荐**组合**而非继承。`children` prop 和自定义 slot 模式实现灵活复用。

### 受控 vs 非受控

- **受控组件**：表单值由 React state 驱动（`value={state} onChange={setState}`）
- **非受控组件**：表单值由 DOM 自身维护（`ref` 读取）

### 高阶组件（HOC）

虽不如 Hook 流行，但仍有场景（权限校验、日志注入）：

```js
function withAuth(WrappedComponent) {
	return function AuthWrapper(props) {
		const { user } = useAuth();
		return user ? <WrappedComponent {...props} /> : <Login />;
	};
}
```

### Render Props

```js
<DataProvider render={(data) => <Child data={data} />} />
```

---

## 🚀 七、补充知识点（进阶）

### `Suspense` + `lazy` — 代码分割

```js
const LazyPage = lazy(() => import('./Page'))
<Suspense fallback={<Loading />}>
  <LazyPage />
</Suspense>
```

### `ErrorBoundary` — 错误边界

class 组件实现 `getDerivedStateFromError` 和 `componentDidCatch`，捕获子组件渲染错误。

### `Portal` — 传送门

```js
createPortal(<Modal />, document.body); // 渲染到父组件 DOM 层级之外
```

### 事件机制

React 事件是**合成事件**（SyntheticEvent），事件委托到 root 节点，具有跨浏览器一致性。

### Fiber 架构

React 16+ 引入的协调引擎，支持：

- **增量渲染**：将渲染工作拆分为可中断的小单元
- **优先级调度**：高优先级更新（用户输入）优先于低优先级（数据加载）
- **并发模式**：React 18 并发特性（`useTransition`、`useDeferredValue`、`Suspense`）

### React 18 并发新特性

| API                | 用途                                                       |
| ------------------ | ---------------------------------------------------------- |
| `useTransition`    | 标记非紧急更新                                             |
| `useDeferredValue` | 延迟更新某个值，类似防抖但更智能                           |
| `Suspense` 增强    | 服务端渲染支持                                             |
| 自动批处理         | 所有 `setState` 自动批处理，无需 `unstable_batchedUpdates` |

---

## 📐 八、React 渲染机制

### 渲染触发条件

1. **首次渲染**：根组件初始化
2. **状态更新**：`setState` / `dispatch` 触发
3. **Context 变化**：Provider 的 value 变化
4. **父组件重渲染**：默认情况下子组件跟随重渲染（除非 `memo`）

### React 运行流程

```
触发更新 → 调度优先级 → 协调（Reconciliation）→ 提交（Commit）→ DOM 更新
                              ↓
                        Fiber 树 diff
```

### `key` 的重要性

- 帮助 React 识别元素身份，优化 diff 性能
- 用 `key` 强制重置组件状态（不同 key → 卸载旧组件 → 挂载新组件）
- ❌ 不使用数组索引作为 key（除非列表静态不变）

---

## 🔍 九、Class 组件（了解即可）

```js
class MyComponent extends Component {
	constructor(props) {
		super(props);
		this.state = { count: 0 };
	}
	render() {
		return <p>{this.state.count}</p>;
	}
}
```

|          | 函数组件                  | Class 组件                                                          |
| -------- | ------------------------- | ------------------------------------------------------------------- |
| 状态     | `useState` / `useReducer` | `this.state` + `this.setState`                                      |
| 生命周期 | `useEffect`               | `componentDidMount` / `componentDidUpdate` / `componentWillUnmount` |
| this     | 无                        | 需绑定或箭头函数                                                    |
| 代码量   | 少                        | 多                                                                  |
| 推荐度   | ✅ 现代 React 首选        | ⚠️ 仅维护旧项目                                                     |

---

> 📝 **持续更新中** — 本文件由项目代码学习总结生成，随学习进度补充完善。
