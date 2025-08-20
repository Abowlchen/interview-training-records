# React核心原理学习计划 - 第一阶段

## 📚 学习目标
通过3周时间，深入理解React的核心原理，包括Virtual DOM、Diff算法、Fiber架构和生命周期机制。

---

## 🗓️ 第一周：Virtual DOM与Diff算法

### 📖 理论学习

#### 1. Virtual DOM原理

**核心概念：**
- Virtual DOM是React在内存中维护的虚拟DOM树
- 本质是JavaScript对象，描述真实DOM结构
- 通过对比新旧Virtual DOM树，计算出最小变更集

**Virtual DOM结构示例：**
```javascript
// 真实DOM
<div className="container">
  <h1>Hello</h1>
  <p>World</p>
</div>

// 对应的Virtual DOM
const vdom = {
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: {
          children: 'Hello'
        }
      },
      {
        type: 'p',
        props: {
          children: 'World'
        }
      }
    ]
  }
}
```

**Virtual DOM优势：**
1. **性能优化**：批量更新，减少DOM操作
2. **跨平台**：抽象层，可渲染到不同平台
3. **可预测性**：函数式编程，状态可预测

#### 2. Diff算法深度解析

**React Diff算法三大策略：**

##### 策略一：Tree Diff（树级别对比）
- 只对比同层级节点，不跨层级比较
- 时间复杂度从O(n³)降到O(n)

```javascript
// 示例：跨层级移动会被识别为删除+创建
// 旧树
<div>
  <p>
    <span>text</span>
  </p>
</div>

// 新树
<div>
  <span>text</span>
</div>

// React会：删除p节点，创建新的span节点
```

##### 策略二：Component Diff（组件级别对比）
- 同类型组件：继续Tree Diff
- 不同类型组件：直接替换整个组件子树

```javascript
// 示例：组件类型变化
// 旧组件
function ComponentA() {
  return <div>A</div>
}

// 新组件
function ComponentB() {
  return <div>B</div>
}

// React会：卸载ComponentA，挂载ComponentB
```

##### 策略三：Element Diff（元素级别对比）
- 使用key属性优化列表渲染
- 三种操作：INSERT（插入）、MOVE（移动）、REMOVE（删除）

```javascript
// 示例：key的重要性
// 没有key的情况
<ul>
  <li>A</li>
  <li>B</li>
  <li>C</li>
</ul>
// 插入D到开头
<ul>
  <li>D</li>
  <li>A</li>
  <li>B</li>
  <li>C</li>
</ul>
// React会：重新渲染所有li元素

// 有key的情况
<ul>
  <li key="a">A</li>
  <li key="b">B</li>
  <li key="c">C</li>
</ul>
// 插入D到开头
<ul>
  <li key="d">D</li>
  <li key="a">A</li>
  <li key="b">B</li>
  <li key="c">C</li>
</ul>
// React会：只插入新的D元素，复用其他元素
```

### 🛠️ 实践任务

#### 任务1：手写简版Virtual DOM
```javascript
// 创建Virtual DOM
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child => 
        typeof child === 'object' ? child : createTextElement(child)
      )
    }
  }
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: []
    }
  }
}

// 渲染Virtual DOM到真实DOM
function render(element, container) {
  const dom = element.type === 'TEXT_ELEMENT'
    ? document.createTextNode('')
    : document.createElement(element.type)
    
  const isProperty = key => key !== 'children'
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
    
  element.props.children.forEach(child =>
    render(child, dom)
  )
  
  container.appendChild(dom)
}

// 使用示例
const element = createElement(
  'div',
  { className: 'container' },
  createElement('h1', null, 'Hello'),
  createElement('p', null, 'World')
)

render(element, document.getElementById('root'))
```

#### 任务2：Diff算法模拟
```javascript
// 简化版Diff算法
function diff(oldVNode, newVNode) {
  const patches = []
  
  // 节点类型不同，直接替换
  if (oldVNode.type !== newVNode.type) {
    patches.push({
      type: 'REPLACE',
      node: newVNode
    })
    return patches
  }
  
  // 比较属性
  const propPatches = diffProps(oldVNode.props, newVNode.props)
  if (propPatches.length > 0) {
    patches.push({
      type: 'PROPS',
      patches: propPatches
    })
  }
  
  // 比较子节点
  const childPatches = diffChildren(oldVNode.props.children, newVNode.props.children)
  if (childPatches.length > 0) {
    patches.push({
      type: 'CHILDREN',
      patches: childPatches
    })
  }
  
  return patches
}

function diffProps(oldProps, newProps) {
  const patches = []
  
  // 检查属性变化
  for (let key in newProps) {
    if (oldProps[key] !== newProps[key]) {
      patches.push({
        type: 'SET_PROP',
        key,
        value: newProps[key]
      })
    }
  }
  
  // 检查删除的属性
  for (let key in oldProps) {
    if (!(key in newProps)) {
      patches.push({
        type: 'REMOVE_PROP',
        key
      })
    }
  }
  
  return patches
}
```

### ✅ 学习检验

**理论检验题：**
1. 解释Virtual DOM的工作原理和优势
2. 描述React Diff算法的三大策略
3. 为什么React建议在列表渲染中使用key？
4. 什么情况下React会重新创建组件而不是更新？

**实践检验：**
1. 能够手写简版Virtual DOM创建和渲染函数
2. 理解Diff算法的基本实现思路
3. 能够分析给定场景下的Diff过程

---

## 🗓️ 第二周：React Fiber架构

### 📖 理论学习

#### 1. Fiber架构背景

**React 15的问题：**
- 递归调用栈过深，无法中断
- 长时间占用主线程，导致页面卡顿
- 无法实现优先级调度

```javascript
// React 15 递归渲染（无法中断）
function renderComponent(component) {
  // 渲染当前组件
  const element = component.render()
  
  // 递归渲染子组件
  element.children.forEach(child => {
    renderComponent(child) // 无法中断的递归调用
  })
}
```

#### 2. Fiber核心概念

**Fiber是什么：**
- 一种数据结构，代表一个工作单元
- 一种调度算法，实现可中断的渲染
- React 16的核心架构重构

**Fiber节点结构：**
```javascript
const fiber = {
  // 节点信息
  type: 'div',           // 组件类型
  key: null,             // key
  props: {},             // 属性
  
  // 关系指针
  child: null,           // 第一个子节点
  sibling: null,         // 下一个兄弟节点
  return: null,          // 父节点
  
  // 状态信息
  alternate: null,       // 对应的另一棵树的节点
  effectTag: null,       // 副作用标记
  stateNode: null,       // 对应的DOM节点
  
  // 调度信息
  expirationTime: 0,     // 过期时间
  childExpirationTime: 0 // 子树中最早的过期时间
}
```

#### 3. Fiber工作原理

**双缓存机制：**
```javascript
// current树：当前显示的Fiber树
// workInProgress树：正在构建的Fiber树

// 渲染阶段：构建workInProgress树
function workLoop() {
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress)
  }
}

// 提交阶段：将workInProgress树替换current树
function commitRoot() {
  const finishedWork = root.finishedWork
  root.current = finishedWork
}
```

**时间切片（Time Slicing）：**
```javascript
// 模拟时间切片
function workLoop() {
  while (workInProgress !== null) {
    if (shouldYield()) {
      // 时间片用完，让出控制权
      return
    }
    workInProgress = performUnitOfWork(workInProgress)
  }
}

function shouldYield() {
  // 检查是否还有剩余时间
  return getCurrentTime() >= deadline
}
```

#### 4. 优先级调度

**优先级等级：**
```javascript
const Priority = {
  Immediate: 1,        // 立即执行（用户输入）
  UserBlocking: 2,     // 用户阻塞（点击、滚动）
  Normal: 3,           // 正常优先级（网络请求）
  Low: 4,              // 低优先级（分析统计）
  Idle: 5              // 空闲时执行
}

// 根据优先级计算过期时间
function computeExpirationTime(priority) {
  const currentTime = getCurrentTime()
  
  switch (priority) {
    case Priority.Immediate:
      return currentTime + 1
    case Priority.UserBlocking:
      return currentTime + 250
    case Priority.Normal:
      return currentTime + 5000
    case Priority.Low:
      return currentTime + 10000
    case Priority.Idle:
      return currentTime + 1073741823 // 永不过期
  }
}
```

### 🛠️ 实践任务

#### 任务1：模拟Fiber遍历
```javascript
// Fiber树遍历算法
function traverseFiber(fiber) {
  let currentFiber = fiber
  
  while (currentFiber) {
    // 处理当前节点
    console.log('Processing:', currentFiber.type)
    
    // 深度优先：先处理子节点
    if (currentFiber.child) {
      currentFiber = currentFiber.child
      continue
    }
    
    // 没有子节点，处理兄弟节点
    if (currentFiber.sibling) {
      currentFiber = currentFiber.sibling
      continue
    }
    
    // 没有兄弟节点，回到父节点的兄弟节点
    while (currentFiber) {
      currentFiber = currentFiber.return
      if (currentFiber && currentFiber.sibling) {
        currentFiber = currentFiber.sibling
        break
      }
    }
  }
}

// 创建测试Fiber树
const rootFiber = {
  type: 'div',
  child: {
    type: 'h1',
    sibling: {
      type: 'p',
      child: {
        type: 'span'
      }
    }
  }
}

traverseFiber(rootFiber)
```

#### 任务2：简化版时间切片
```javascript
// 模拟时间切片调度
class Scheduler {
  constructor() {
    this.taskQueue = []
    this.isRunning = false
  }
  
  scheduleWork(callback, priority = 'Normal') {
    const task = {
      callback,
      priority,
      expirationTime: this.computeExpirationTime(priority)
    }
    
    this.taskQueue.push(task)
    this.taskQueue.sort((a, b) => a.expirationTime - b.expirationTime)
    
    if (!this.isRunning) {
      this.flushWork()
    }
  }
  
  flushWork() {
    this.isRunning = true
    
    const workLoop = () => {
      while (this.taskQueue.length > 0 && !this.shouldYield()) {
        const task = this.taskQueue.shift()
        task.callback()
      }
      
      if (this.taskQueue.length > 0) {
        // 还有任务，下一帧继续
        requestIdleCallback(workLoop)
      } else {
        this.isRunning = false
      }
    }
    
    requestIdleCallback(workLoop)
  }
  
  shouldYield() {
    // 简化版：检查是否还有空闲时间
    return performance.now() % 16 > 5 // 模拟5ms时间片
  }
  
  computeExpirationTime(priority) {
    const currentTime = performance.now()
    switch (priority) {
      case 'Immediate': return currentTime + 1
      case 'UserBlocking': return currentTime + 250
      case 'Normal': return currentTime + 5000
      default: return currentTime + 10000
    }
  }
}

// 使用示例
const scheduler = new Scheduler()

scheduler.scheduleWork(() => console.log('High priority task'), 'UserBlocking')
scheduler.scheduleWork(() => console.log('Normal task'), 'Normal')
scheduler.scheduleWork(() => console.log('Low priority task'), 'Low')
```

### ✅ 学习检验

**理论检验题：**
1. 解释Fiber架构解决了React 15的哪些问题？
2. 描述Fiber的双缓存机制
3. 什么是时间切片？如何实现可中断渲染？
4. React如何实现优先级调度？

**实践检验：**
1. 能够描述Fiber树的遍历过程
2. 理解时间切片的基本实现原理
3. 能够分析不同优先级任务的调度顺序

---

## 🗓️ 第三周：React生命周期深度解析

### 📖 理论学习

#### 1. 生命周期演进历史

**React 15 生命周期：**
```javascript
class Component extends React.Component {
  // 挂载阶段
  componentWillMount() {}
  componentDidMount() {}
  
  // 更新阶段
  componentWillReceiveProps(nextProps) {}
  shouldComponentUpdate(nextProps, nextState) {}
  componentWillUpdate(nextProps, nextState) {}
  componentDidUpdate(prevProps, prevState) {}
  
  // 卸载阶段
  componentWillUnmount() {}
}
```

**React 16+ 新生命周期：**
```javascript
class Component extends React.Component {
  // 挂载阶段
  static getDerivedStateFromProps(props, state) {}
  componentDidMount() {}
  
  // 更新阶段
  static getDerivedStateFromProps(props, state) {}
  shouldComponentUpdate(nextProps, nextState) {}
  getSnapshotBeforeUpdate(prevProps, prevState) {}
  componentDidUpdate(prevProps, prevState, snapshot) {}
  
  // 错误处理
  static getDerivedStateFromError(error) {}
  componentDidCatch(error, errorInfo) {}
  
  // 卸载阶段
  componentWillUnmount() {}
}
```

#### 2. 生命周期详解

**挂载阶段（Mounting）：**
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
    console.log('1. constructor')
  }
  
  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps')
    // 根据props计算新的state
    if (props.initialCount !== state.count) {
      return { count: props.initialCount }
    }
    return null
  }
  
  componentDidMount() {
    console.log('4. componentDidMount')
    // 适合：数据获取、订阅、DOM操作
    this.fetchData()
    this.timer = setInterval(() => {
      this.setState(state => ({ count: state.count + 1 }))
    }, 1000)
  }
  
  render() {
    console.log('3. render')
    return <div>{this.state.count}</div>
  }
}
```

**更新阶段（Updating）：**
```javascript
class MyComponent extends React.Component {
  static getDerivedStateFromProps(props, state) {
    // 每次渲染前都会调用
    return null
  }
  
  shouldComponentUpdate(nextProps, nextState) {
    // 性能优化：决定是否重新渲染
    return nextState.count !== this.state.count
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 在DOM更新前获取信息
    if (prevState.count < this.state.count) {
      const list = this.listRef.current
      return list.scrollHeight - list.scrollTop
    }
    return null
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    // DOM更新后调用
    if (snapshot !== null) {
      const list = this.listRef.current
      list.scrollTop = list.scrollHeight - snapshot
    }
  }
}
```

**错误处理（Error Handling）：**
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = { hasError: false, error: null }
  }
  
  static getDerivedStateFromError(error) {
    // 更新state，下次渲染显示错误UI
    return { hasError: true, error }
  }
  
  componentDidCatch(error, errorInfo) {
    // 记录错误信息
    console.error('Error caught by boundary:', error, errorInfo)
    // 发送错误报告
    this.logErrorToService(error, errorInfo)
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong.</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
          </details>
        </div>
      )
    }
    
    return this.props.children
  }
}
```

#### 3. Hooks与生命周期对应关系

```javascript
// Class组件生命周期 vs Hooks
function MyComponent(props) {
  const [count, setCount] = useState(0)
  const [data, setData] = useState(null)
  
  // componentDidMount + componentDidUpdate
  useEffect(() => {
    console.log('组件挂载或count更新')
  }, [count])
  
  // componentDidMount（仅挂载时执行）
  useEffect(() => {
    console.log('组件挂载')
    fetchData().then(setData)
    
    // componentWillUnmount
    return () => {
      console.log('组件卸载')
    }
  }, [])
  
  // getDerivedStateFromProps
  useEffect(() => {
    if (props.initialCount !== count) {
      setCount(props.initialCount)
    }
  }, [props.initialCount])
  
  // shouldComponentUpdate
  const MemoizedChild = useMemo(() => {
    return <ExpensiveChild data={data} />
  }, [data])
  
  return (
    <div>
      <div>{count}</div>
      {MemoizedChild}
    </div>
  )
}
```

### 🛠️ 实践任务

#### 任务1：生命周期完整示例
```javascript
class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      count: 0,
      data: null,
      error: null
    }
    this.listRef = React.createRef()
    console.log('🏗️ Constructor')
  }
  
  static getDerivedStateFromProps(props, state) {
    console.log('📥 getDerivedStateFromProps', { props, state })
    
    // 示例：根据props重置count
    if (props.reset && state.count > 0) {
      return { count: 0 }
    }
    
    return null
  }
  
  componentDidMount() {
    console.log('🎯 componentDidMount')
    
    // 数据获取
    this.fetchData()
    
    // 设置定时器
    this.timer = setInterval(() => {
      this.setState(prevState => ({
        count: prevState.count + 1
      }))
    }, 1000)
    
    // DOM操作
    if (this.listRef.current) {
      this.listRef.current.focus()
    }
  }
  
  shouldComponentUpdate(nextProps, nextState) {
    console.log('🤔 shouldComponentUpdate', {
      currentState: this.state,
      nextState,
      currentProps: this.props,
      nextProps
    })
    
    // 性能优化：只有count变化时才更新
    return nextState.count !== this.state.count ||
           nextState.data !== this.state.data
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('📸 getSnapshotBeforeUpdate', { prevProps, prevState })
    
    // 保存滚动位置
    if (this.listRef.current && prevState.count < this.state.count) {
      return {
        scrollTop: this.listRef.current.scrollTop,
        scrollHeight: this.listRef.current.scrollHeight
      }
    }
    
    return null
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('✅ componentDidUpdate', {
      prevProps,
      prevState,
      snapshot,
      currentState: this.state
    })
    
    // 恢复滚动位置
    if (snapshot && this.listRef.current) {
      const { scrollTop, scrollHeight } = snapshot
      const newScrollHeight = this.listRef.current.scrollHeight
      this.listRef.current.scrollTop = scrollTop + (newScrollHeight - scrollHeight)
    }
    
    // 条件性副作用
    if (prevState.count !== this.state.count && this.state.count % 10 === 0) {
      console.log('🎉 Count reached multiple of 10!')
    }
  }
  
  componentWillUnmount() {
    console.log('💀 componentWillUnmount')
    
    // 清理工作
    if (this.timer) {
      clearInterval(this.timer)
    }
    
    // 取消网络请求
    if (this.abortController) {
      this.abortController.abort()
    }
  }
  
  fetchData = async () => {
    try {
      this.abortController = new AbortController()
      
      const response = await fetch('/api/data', {
        signal: this.abortController.signal
      })
      
      const data = await response.json()
      this.setState({ data })
    } catch (error) {
      if (error.name !== 'AbortError') {
        this.setState({ error: error.message })
      }
    }
  }
  
  render() {
    console.log('🎨 Render', this.state)
    
    const { count, data, error } = this.state
    
    if (error) {
      return <div>Error: {error}</div>
    }
    
    return (
      <div>
        <h2>Lifecycle Demo</h2>
        <div>Count: {count}</div>
        <div ref={this.listRef} style={{ height: '200px', overflow: 'auto' }}>
          {Array.from({ length: count }, (_, i) => (
            <div key={i}>Item {i + 1}</div>
          ))}
        </div>
        {data && <div>Data: {JSON.stringify(data)}</div>}
        <button onClick={() => this.setState({ count: 0 })}>
          Reset Count
        </button>
      </div>
    )
  }
}
```

#### 任务2：Hooks版本对比
```javascript
function LifecycleDemoHooks({ reset }) {
  const [count, setCount] = useState(0)
  const [data, setData] = useState(null)
  const [error, setError] = useState(null)
  const listRef = useRef(null)
  const abortControllerRef = useRef(null)
  
  // getDerivedStateFromProps 等效
  useEffect(() => {
    if (reset && count > 0) {
      setCount(0)
    }
  }, [reset, count])
  
  // componentDidMount 等效
  useEffect(() => {
    console.log('🎯 Component mounted (useEffect [])')
    
    // 数据获取
    fetchData()
    
    // 设置定时器
    const timer = setInterval(() => {
      setCount(prevCount => prevCount + 1)
    }, 1000)
    
    // DOM操作
    if (listRef.current) {
      listRef.current.focus()
    }
    
    // componentWillUnmount 等效
    return () => {
      console.log('💀 Component unmounting (useEffect cleanup)')
      clearInterval(timer)
      if (abortControllerRef.current) {
        abortControllerRef.current.abort()
      }
    }
  }, [])
  
  // componentDidUpdate 等效
  useEffect(() => {
    console.log('✅ Count updated (useEffect [count])', count)
    
    if (count % 10 === 0 && count > 0) {
      console.log('🎉 Count reached multiple of 10!')
    }
  }, [count])
  
  // getSnapshotBeforeUpdate + componentDidUpdate 等效
  const prevCountRef = useRef()
  useLayoutEffect(() => {
    if (listRef.current && prevCountRef.current < count) {
      // 这里可以实现类似getSnapshotBeforeUpdate的逻辑
      const scrollTop = listRef.current.scrollTop
      const scrollHeight = listRef.current.scrollHeight
      
      // 在DOM更新后调整滚动位置
      requestAnimationFrame(() => {
        if (listRef.current) {
          const newScrollHeight = listRef.current.scrollHeight
          listRef.current.scrollTop = scrollTop + (newScrollHeight - scrollHeight)
        }
      })
    }
    prevCountRef.current = count
  }, [count])
  
  const fetchData = async () => {
    try {
      abortControllerRef.current = new AbortController()
      
      const response = await fetch('/api/data', {
        signal: abortControllerRef.current.signal
      })
      
      const result = await response.json()
      setData(result)
    } catch (err) {
      if (err.name !== 'AbortError') {
        setError(err.message)
      }
    }
  }
  
  // shouldComponentUpdate 等效（通过React.memo实现）
  console.log('🎨 Render (Hooks)', { count, data, error })
  
  if (error) {
    return <div>Error: {error}</div>
  }
  
  return (
    <div>
      <h2>Lifecycle Demo (Hooks)</h2>
      <div>Count: {count}</div>
      <div ref={listRef} style={{ height: '200px', overflow: 'auto' }}>
        {Array.from({ length: count }, (_, i) => (
          <div key={i}>Item {i + 1}</div>
        ))}
      </div>
      {data && <div>Data: {JSON.stringify(data)}</div>}
      <button onClick={() => setCount(0)}>
        Reset Count
      </button>
    </div>
  )
}

// 使用React.memo实现shouldComponentUpdate
const MemoizedLifecycleDemoHooks = React.memo(LifecycleDemoHooks, (prevProps, nextProps) => {
  // 返回true表示props相等，不需要重新渲染
  return prevProps.reset === nextProps.reset
})
```

### ✅ 学习检验

**理论检验题：**
1. 解释React 16废弃某些生命周期方法的原因
2. 描述新增生命周期方法的使用场景
3. 如何在Hooks中实现各个生命周期的功能？
4. 错误边界的工作原理和最佳实践

**实践检验：**
1. 能够正确使用所有生命周期方法
2. 理解Hooks与生命周期的对应关系
3. 能够实现错误边界组件
4. 掌握生命周期中的性能优化技巧

---

## 📋 第一阶段总结

### 🎯 学习成果检验

完成以下综合练习，检验第一阶段学习成果：

#### 综合练习：实现一个性能优化的列表组件

**要求：**
1. 使用Virtual DOM概念设计组件结构
2. 实现类似Fiber的可中断渲染（简化版）
3. 正确使用生命周期进行性能优化
4. 支持大量数据的虚拟滚动
5. 包含错误边界处理

**评分标准：**
- **基础分（60分）**：基本功能实现，正确使用生命周期
- **进阶分（80分）**：性能优化，虚拟滚动实现
- **高级分（100分）**：可中断渲染，错误处理完善
- **专家分（120分）**：代码质量优秀，架构设计合理

### 🚀 下一阶段预告

**第二阶段：状态管理原理（3周）**
- Week 1: Flux架构与Redux原理
- Week 2: 现代状态管理库对比
- Week 3: 状态管理最佳实践

准备好进入下一阶段了吗？让我知道你的学习进度和任何问题！