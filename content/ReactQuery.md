# TanStack Query 学习笔记

## 概述

TanStack Query 是一个强大的数据同步库，用于 React 应用的状态管理和数据获取。
_注：本文档基于 TanStack Query 最新版本整理，具体使用请参考 https://tanstack.com/query/latest_

```bash
"@tanstack/react-query": "^5.91.0",
"@tanstack/react-query-devtools": "^5.91.3",
```

## 安装

### 基础安装

```bash
npm i @tanstack/react-query
```

### ESLint 插件（可选）

```bash
npm i -D @tanstack/eslint-plugin-query
```

### 开发工具（可选）

```bash
npm i @tanstack/react-query-devtools
```

## 基础配置

### 设置 React Query

```javascript
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
	return (
		<QueryClientProvider client={queryClient}>
			{/* 你的应用组件 */}
		</QueryClientProvider>
	);
}
```

### 开发工具集成

```javascript
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

function App() {
	return (
		<QueryClientProvider client={queryClient}>
			{/* 你的应用其他部分 */}
			<ReactQueryDevtools initialIsOpen={false} />
		</QueryClientProvider>
	);
}
```

## useQuery Hook

### 基本用法

`useQuery` 是获取数据的主要钩子函数。

**参数：**

- `key`：唯一标识符，用于缓存和查询
- `queryFn`：用于获取数据的函数
- `options`：配置选项

### 代码示例

```javascript
import { useQuery } from '@tanstack/react-query'

// 通用的数据获取函数
function fetcher<T>(url: string): Promise<T> {
  return fetch(url).then((res) => res.json())
}

// 使用示例
function Page() {
  const query = useQuery(['todos'], fetcher)
  const { status, fetchStatus, isLoading, isFetching, data, error } = query;
  // 首次加载
  if (isLoading && isFetching) return <span>Loading...</span>
  // 有缓存的后台刷新
  if (status === 'success' && fetchStatus === 'fetching') return <span>Refreshing...</span>
  // 失败
  if (status === 'error' && fetchStatus === 'idle') return <span>Error: {error.message}</span>
  // 网络暂停
  if (fetchStatus === 'paused') return <span>Network paused. Click to resume.</span>

  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

```javascript
// { id: 1, name: "张三", email: "zhang@example.com", avatar: "...", ... }
const { data: userInfo } = useQuery({
	queryKey: ["user", 1],
	queryFn: fetchUser,
	select: (user) => ({
		// 重组数据
		name: user.name,
		email: user.email,
		avatar: user.avatar,
	}),
});
```

### 返回对象属性

| 属性名        | 类型       | 描述                                                |
| ------------- | ---------- | --------------------------------------------------- |
| `data`        | `T`        | 查询返回的数据                                      |
| `error`       | `Error`    | 查询过程中的错误对象                                |
| `isLoading`   | `boolean`  | 是否正在加载（第一次加载）                          |
| `isFetching`  | `boolean`  | 是否正在获取数据（包括后台刷新）                    |
| `isError`     | `boolean`  | 是否有错误                                          |
| `refetch`     | `function` | 手动重新获取数据的函数                              |
| `status`      | `string`   | 描述数据本身的状态（'pending'、'error'、'success'） |
| `fetchStatus` | `string`   | 描述获取过程的状态（'idle'、'fetching'、'paused'）  |

### 查询键（Query Keys）

查询键用于唯一标识查询，并用于缓存管理。

```javascript
useQuery(["todos"], fetcher); // 旧版本（v4 及之前）

useQuery({
	// 新版本（v5 及之后）
	queryKey: ["todo", 5, { preview: true }],
	queryFn: fetchTodo,
});
```

---

### 查询函数参数（QueryFunctionContext）

- queryKey：获取查询参数
- client：查询客户端实例
- signal：请求取消（必须使用，避免内存泄漏）
- pageParam：无限查询的页码
- meta：传递额外配置

### queryOptions | TypeScript辅助工具

在多个地方共享查询键和查询函数的同时，又能确保它们保持相互关联，最好的方法就是使用查询选项辅助工具

```typescript
import { queryOptions } from "@tanstack/react-query";

function groupOptions(id: number) {
	return queryOptions({
		queryKey: ["groups", id],
		queryFn: () => fetchGroups(id),
		staleTime: 5 * 1000,
	});
}
useQuery(groupOptions(1));
useQueries({
	queries: [groupOptions(1), groupOptions(2)],
});
```

## useQueries Hook | 并行查询

`useQueries` 是用于同时管理多个查询的钩子函数。

```javascript
import { useQueries } from "@tanstack/react-query";

const results = useQueries({
	queries: [
		{ queryKey: ["user", 1], queryFn: fetchUser },
		{ queryKey: ["posts"], queryFn: fetchPosts },
		{ queryKey: ["todos"], queryFn: fetchTodos },
	],
});
// 或
const results = useQueries({
	queries: users.map((user) => {
		return {
			queryKey: ["user", user.id],
			queryFn: () => fetchUserById(user.id),
		};
	}),
});

console.log(results);
/*
[
  { data: user, isLoading: false, isError: false, ... },  // 第一个查询
  { data: posts, isLoading: true, isError: false, ... },   // 第二个查询
  { data: null, isLoading: false, isError: true, ... },    // 第三个查询
]
*/
```

## useIsFetching | 检查是否有查询正在进行

```javascript
import { useIsFetching } from "@tanstack/react-query";

function GlobalLoadingIndicator() {
	const isFetching = useIsFetching();

	return isFetching ? (
		<div>Queries are fetching in the background...</div>
	) : null;
}
```

## useMutation Hook

处理**数据修改操作**的 Hook，用于 POST、PUT、DELETE 等会改变服务器状态的操作。典型的例子：提交表单、删除用户、更新设置

### 基本用法

```javascript
const mutation = useMutation({
	// 1. 核心函数：执行修改操作
	mutationFn: async (newData) => {
		const response = await api.createData(newData);
		return response.data;
	},

	// 2. 可选：乐观更新配置
	onMutate: async (variables) => {
		// 在请求发送前执行
		// 用途：提前更新UI，待请求成功后再更新数据，场景：提交评论时，先显示评论内容，接口成功再更新数据，接口失败则回滚
	},

	// 3. 可选：成功/失败回调
	// 如果组件在回调完成之前卸载，则不会触发
	// data: 接口返回的数据
	// variables: 传递给mutationFn的参数
	// onMutateResult: onMutate的返回值
	// context: 上下文对象
	onSuccess: (data, variables, onMutateResult, context) => {
		// 请求成功时执行
	},

	onError: (error, variables, onMutateResult, context) => {
		// 请求失败时执行
	},

	onSettled: (data, error, variables, onMutateResult, context) => {
		// 无论成功失败都会执行
		// onSuccess/onError 完成后才执行，相当于onfinish
	},

	// 4. 重试配置
	retry: 3, // 失败时重试次数
	retryDelay: 1000, // 重试延迟
});
```

```javascript
const { mutate, isPending, isSuccess, isError, error } = useCreateArticle();
mutate(
	{ title, content },
	{
		onSuccess: () => {
			// ...
		},
	},
);
```

```js
// 1. 创建 QueryClient 实例 - React Query 的核心管理器
const queryClient = new QueryClient();

// 2. 为 "addTodo" 变更设置全局默认配置
// 这会在应用级别为所有使用 ['addTodo'] 作为 mutationKey 的变更提供默认行为
queryClient.setMutationDefaults(["addTodo"], {
	// mutationFn: 实际执行数据变更的异步函数
	mutationFn: addTodo,

	// onMutate: 在变更执行前立即调用，用于乐观更新
	// 乐观更新：假设变更会成功，先更新UI，如果失败再回滚
	onMutate: async (variables, context) => {
		// 取消当前正在进行的 todos 查询，避免数据冲突
		await context.client.cancelQueries({ queryKey: ["todos"] });

		// 创建一个临时的乐观待办事项对象
		// uuid() 生成唯一ID，确保临时对象不与其他数据冲突
		const optimisticTodo = { id: uuid(), title: variables.title };

		// 将乐观待办事项添加到 todos 列表中
		// setQueryData 直接更新缓存，UI会立即响应
		context.client.setQueryData(["todos"], (old) => [
			...old,
			optimisticTodo,
		]);

		// 返回乐观更新的上下文信息
		// 这个返回值会传递给 onSuccess 和 onError 的 onMutateResult 参数
		return { optimisticTodo };
	},

	// onSuccess: 变更成功完成时调用
	onSuccess: (result, variables, onMutateResult, context) => {
		// 用服务器返回的实际数据替换临时的乐观待办事项
		// onMutateResult.optimisticTodo.id 是之前在 onMutate 中生成的临时ID
		context.client.setQueryData(["todos"], (old) =>
			old.map((todo) =>
				// 找到临时待办事项，用服务器返回的真实数据替换它
				todo.id === onMutateResult.optimisticTodo.id ? result : todo,
			),
		);
	},

	// onError: 变更失败时调用
	onError: (error, variables, onMutateResult, context) => {
		// 从 todos 列表中移除乐观待办事项（回滚操作）
		// 过滤掉之前添加的临时待办事项
		context.client.setQueryData(["todos"], (old) =>
			old.filter((todo) => todo.id !== onMutateResult.optimisticTodo.id),
		);
	},

	// retry: 失败时自动重试次数
	retry: 3,
});

// 3. 在组件中使用变更
// 注意：这里只需要指定 mutationKey，其他配置会自动继承 setMutationDefaults 中的设置
const mutation = useMutation({ mutationKey: ["addTodo"] });

// 4. 触发变更执行
// mutate 方法会调用 addTodo 函数，并传递 { title: 'title' } 作为参数
mutation.mutate({ title: "title" });

// 5. 离线恢复机制 - 脱水（序列化）
// 如果变更因为设备离线而被暂停，可以在应用退出时序列化状态
// 这通常用于 PWA 或移动应用的离线功能
const state = dehydrate(queryClient);
// state 现在包含了所有查询和变更的当前状态，可以存储到本地存储

// 6. 离线恢复机制 - 水合（反序列化）
// 当应用重新启动时，从存储中恢复状态
hydrate(queryClient, state);
// 这会恢复之前所有的查询和变更状态

// 7. 恢复被暂停的变更
// 当应用重新上线时，恢复之前因离线而暂停的变更
queryClient.resumePausedMutations();
// 这会导致之前暂停的变更继续执行
```

## prefetchQuery Hook | prefetchInfiniteQuery Hook

预取数据 ，旨在通过提前加载数据来优化用户体验

### 基本使用

```javascript
// 普通查询
await queryClient.prefetchQuery({
	queryKey: ["todos"],
	queryFn: fetchTodos,
	staleTime: 5000, // 仅为预取设置
});

// 无限查询 (预取前 3 页)
await queryClient.prefetchInfiniteQuery({
	queryKey: ["projects"],
	queryFn: fetchProjects,
	initialPageParam: 0,
	getNextPageParam: (lastPage) => lastPage.nextCursor,
	pages: 3, // 预取页数
});
```

### 使用场景

**A. 事件驱动预取 (Event Handlers)**

在用户交互（如悬停、聚焦）时触发，适合预测用户下一步操作。

- **最佳实践**：必须设置 `staleTime`，避免频繁重复请求。
- **示例**：按钮 `onMouseEnter`时预取详情数据。

**B. 组件内预取 (Components)**

用于解决“请求瀑布”问题（如父组件加载完才触发子组件加载）。

**C. 依赖预取与代码分割 (Dependent Queries)**

解决“先加载组件代码，再加载数据”的串行问题。

- **策略**：在父级查询函数 (`queryFn`) 中，根据返回结果动态预取子组件数据。
- **效果**：实现**组件代码加载**与**数据加载**的并行化。
- **权衡 (Trade-off)**：预取逻辑会提升父级 Bundle 体积，需根据“命中率”决定是否值得。

```javascript
// 假设我们有一个 Feed页面，其条目可能包含标准项和一种特殊的、包含复杂图表的项。由于图表组件（GraphFeedItem）及其数据获取逻辑较重，我们对其使用了 React.lazy进行代码分割，实现按需加载。
// 先加载 Feed 数据，同时预取图表数据，避免“请求瀑布”问题。
// Feed.tsx
// 1. 图表组件被懒加载
const GraphFeedItem = React.lazy(() => import("./GraphFeedItem"));

function Feed() {
	const { data, isPending } = useQuery({
		queryKey: ["feed"],
		queryFn: async (...args) => {
			// 1. 先获取 Feed 列表
			const feed = await getFeed(...args);

			// 2. 关键步骤：遍历列表，对图表项进行条件预取
			for (const feedItem of feed) {
				if (feedItem.type === "GRAPH") {
					// 立即（不 await）发起图表数据的预取请求
					queryClient.prefetchQuery({
						queryKey: ["graph", feedItem.id],
						queryFn: getGraphDataById, // 注意：这里导入了获取函数
					});
				}
			}
			// 3. 返回 Feed 数据
			return feed;
		},
	});

	if (isPending) return "Loading feed...";

	return (
		<>
			{data.map((feedItem) => {
				// 3. 只有遇到类型为 'GRAPH' 的项时，才会开始加载 GraphFeedItem 的 JS 代码包
				if (feedItem.type === "GRAPH") {
					return (
						<GraphFeedItem key={feedItem.id} feedItem={feedItem} />
					);
				}
				return (
					<StandardFeedItem key={feedItem.id} feedItem={feedItem} />
				);
			})}
		</>
	);
}
```

```javascript
// GraphFeedItem.tsx
// 4. 组件代码加载完成后，渲染时才触发其内部的数据查询
function GraphFeedItem({ feedItem }) {
	const { data, isPending } = useQuery({
		queryKey: ["graph", feedItem.id],
		queryFn: getGraphDataById,
	});
	// ... 渲染图表
}
```

## 配置项（Query Options）

### 基本配置

- `queryKey`：查询键

- `queryFn`：查询函数

- `enabled`：是否启用查询

- `staleTime`：数据过时时间

- `gcTime`：缓存时间 这个机制在查询仍被使用的情况下不会起任何作用。只有当查询不再被使用时，该机制才会开始发挥作用。时间一到，相关数据就会被“清除”，从而防止缓存规模不断增大

- `retry`：重试次数 (Number | Boolean)

- `retryDelay`：重试延迟时间

- `refetchOnWindowFocus`: 窗口聚焦时重新获取数据 default: true

- `placeholderData` : 占位数据 分页查询时很有用 placeholderData: keepPreviousData

    ```javascript
    import { keepPreviousData, useQuery } from "@tanstack/react-query";
    useQuery({
    	queryKey: ["projects", page],
    	queryFn: () => fetchProjects(page),
    	// 在请求新数据的过程中，上一次成功获取的数据仍然存在，即便查询键已经发生了变化。当新数据到达时，之前的数据就会被替换
    	placeholderData: keepPreviousData,
    });
    ```

## 查询无效化 与 过滤器

### 一、过滤器（Filters）核心概念

过滤器是用于**批量匹配**查询（Query）或变更（Mutation）的条件对象，作为筛选器传递给缓存操作方法。

**QueryFilters 主要属性**：

- `queryKey` - 查询键（支持前缀匹配）
- `exact` - 精确匹配（默认 false）
- `type` - 匹配类型：`'active' | 'inactive' | 'all'`
- `predicate` - 自定义过滤函数

**MutationFilters 主要属性**：

- `mutationKey` - 变更键
- `exact` - 精确匹配
- `fetching` - 匹配进行中的变更
- `predicate` - 自定义过滤函数

### 二、查询无效化三方法对比

| 方法                    | 核心作用                     | 数据保留              | 网络请求        | 适用场景                         |
| :---------------------- | :--------------------------- | :-------------------- | :-------------- | :------------------------------- |
| **`invalidateQueries`** | 标记为过时，触发后台重新获取 | ✅ 保留（标记 stale） | ✅ 触发重新获取 | 数据更新后同步、定时刷新         |
| **`removeQueries`**     | 从缓存彻底删除               | ❌ 完全删除           | ❌ 不触发       | 清理敏感数据、释放内存、重置状态 |
| **`cancelQueries`**     | 中止进行中的网络请求         | ✅ 保留现有数据       | ❌ 中止当前请求 | 防止竞态、优化性能、搜索防抖     |

### 三、核心区别总结

1. **`invalidateQueries`**：最常用，实现无缝数据更新，UI 无闪烁
2. **`removeQueries`**：最彻底，立即释放内存，但会导致 UI 显示 Loading
3. **`cancelQueries`**：最特殊，仅中止请求，不改变缓存状态
