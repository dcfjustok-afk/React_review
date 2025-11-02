React 
    分两步工作：
        1.Reander(调和/构建) 阶段：计算新的 UI（生成新的Fiber树/更新的Fiber节点），这是可中断的
        2.Commit(提交) 阶段：将计算出的变化应用到真实 DOM 上，并执行副作用（生命周期、useEffect的回调等），这是不可中断的
    
    Hooks的职责：把函数组件中的状态（state）、副作用（effect）、引用（ref）等与组件的生命周期和渲染过程联系起来。 底层是靠React对组件的“Hook链表”管理和一个在渲染期间的dispatch函数来实现的。

    Fiber的职责：Fiber是React 的内部节点结构（轻量级）用于描述组件、其状态与子树，它支持分割工作（time-slicing）、中断和优先级调度。Fiber 替换了旧的递归栈实现，使协调/调度更灵活。

Hooks 的基本原理

   概念：
        Hooks是 React 用来在函数组件里保存跨渲染状态和副作用的机制。实现上，React 为每个函数组件维护一条 Hook 链表（保存在该组件的 Fiber 上），每次渲染函数组件时按固定顺序依次“读取或创建”链表上的节点，从而把每个 useXxx 调用绑定到一个唯一的内部节点上。

        Hooks 的头指针（memoizedState）指向链表的第一个节点，每个节点对应一个 useXxx 调用。
        该头指针在 Fiber 节点的 memoizedState 字段中。
        Fiber {
            memoizedState: Hook | null   // 指向该组件第一 Hook 的头指针
            updateQueue: ...             // 整体更新队列（更多在 class / fiber 层）
            // child, sibling, return 等用于树结构
        }
    
    调用流程 Mount vs Update        ****重要***
    React在渲染函数组件时维护两个全局/模块级的状态
        1、currentlyRenderingFiber：指向当前正在渲染的 Fiber 节点。
        2、workInProgressHook：指向当前正在处理的 Hook 节点。

    Mount (首次渲染)
        workInProgressHook 初始值为null
        每遇到一个useXxx （）：
            1、创建一个新的Hook节点
                如果fiber.memoizedState 为null：
                    则将新节点设为头指针
                否则：
                    链接到当前hook的next指针
            2、将workInProgressHook 指向新节点
        返回值（state、dispatch、ref等） 由新创建的hook节点提供
    
    Update (后续渲染)
        workInProgressHook 初始指向 fiber.memoizedState（旧链表头）
        每遇到一个useXxx （）：
            1、React不创建新节点（或说创建新的workInProgressHook节点并把以前的值拷贝/复用），而是从旧链表中遍历找到对应的节点
            2、应用挂起在该 hook.queue 上的更新（如果是 useState，会遍历 pending updates 并计算新的 state）。
            3、将 workInProgressHook = hook.next，并返回当前 state + dispatch。
        返回值（state、dispatch、ref等） 由该节点提供

例子：
组件代码：

function Cmp() {
  const a = useState(1)      // hook #1
  const r = useRef(null)    // hook #2
  const b = useState(2)      // hook #3
  useEffect(() => ...)       // hook #4
  const c = useReducer(...)  // hook #5
  ...
}

第一次渲染（Mount） 内存链：

    fiber.memoizedState -> H1(a) -> H2(r) -> H3(b) -> H4(effect) -> H5(c) -> null


        H1.memoizedState = 1, H1.queue = {}
        H2.memoizedState = { current: null }
        H3.memoizedState = 2
        H4 holds effect create/destroy/deps
        H5 holds reducer state & queue

第二次渲染（Update）：当 re-render 时，React 再次执行 Cmp()，但内部的 useState/useRef/useEffect/useReducer 已经被替换为 UpdateDispatcher：

    执行流程（伪）：

        调用第一个 useState → React 从 H1 读取 memoizedState、消费 H1.queue pending updates → 得到 newA。
        调用 useRef → 读取 H2.memoizedState.current。
        调用第二个 useState → 读取 H3，处理 H3.queue。
        调用 useEffect → 比较 H4.deps，如果需要，标记 effect 以便 commit 阶段运行。
        调用 useReducer → 读取 H5，处理 H5.queue。
        结束后，fiber.memoizedState 的链结构保持不变（或替换为新的 workInProgress 链表，但逻辑上位置 1-5 一一对应）。