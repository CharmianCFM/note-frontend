### fiber
fiber: 就是一个数据结构，有很多属性。
如果只用虚拟dom的话有很多的事情做不到；很多真实dom做不到的事情虚拟dom做不到，何况真实dom都做不到的事情虚拟dom就更加做不到了，也就是说单纯地使用虚拟dom可以做到的事情是有限的。
所以新的fiber数据结构给他添加了很多的属性，希望借由这些新的属性来做到一些很有意思的事情。
### fiber 架构
react在早期版本中有一些遗憾或者说不足的地方，于是新版的react中为了弥补这种不足的地方，设计了一些新的算法，用这些新的算法去做一些比较厉害的事情。
这些新的算法的设计就是架构，而为了能让这些算法跑起来就有了fiber这种数据结构。就比如说有了算法但是你没有相应的数据结构那就完不成这个算法。
所以fiber的数据结构加上新的这套算法，就成了fiber架构。
### 三大要素
react应用从始至终管理着最基本的三样东西：

1. Root(整个应用的根儿，一个对象，不同于fiber， 它有属性指向current和workInProgress两颗树)
2. current树(树上的每一个节点都是fiber结构，保存着该节点的上一个状态，并且每个fiber结构都对应着一个jsx节点)
3. workInProgress树(树上每个节点都是fiber，保存着本次更新的状态，并且每个fiber结构都对应着一个jsx节点)
每次更新都会重新创建workInProgress树，然后用这个workInProgress树来和current树作对比，之后进行更新
这三样东西 跟咱们自己传进的一点关系都没有

初次渲染时候没有current也就是没有上一次的状态，react在一开始创建Root的同时就会创建一个 uninitalFiber(未初始化的fiber)，并让Root的current指向这个uninitalFiber，然后再去创建一个本次渲染要用到的workInProgress(RootFiber)，也就是说初次渲染的时候没有上一次状态react内部会构建一个全部状态未初始化的fiber。
### 两个阶段
react中主要有两个阶段
render阶段

1. 为每个节点创建(或复用) 新的workInProgress生产一颗有新状态的fiber树
2. 初次渲染的时候(或新创建了某个节点的时候) 会为这个fiber创建真实的dom节点，并且对创建的dom节点的子节点进行插入append
3. 如果不是初次渲染的话 就对比新旧的fiber的状态 将产生了更新的fiber 最终通过链表的形式 挂载到RootFiber上

commit阶段

1. 执行生命周期
2. 从RootFiber上 获取到有更新的fiber的那条链表 然后根据每个fiber的更新的状态(Update, Placment, Deletion, ...) 进行真正的修改页面
### setState是同步还是异步?
如果是正常情况下 也就是没有使用Concurrent组件到的情况下 react的更新是同步的，但是! 不会立即获取到最新的state的值。
因为正常情况下调用setState只是单纯的将你传进来的新的状态放入一条链表中，等这个事件执行完毕之后会触发一个回调函数，这个函数中才会真正地执行react的更新状态以及渲染的流程，并且是以完全同步的方式进行的。
当使用了Concurrent组件的时候，才是真正的异步更新模式。同样的，无法在事件中立即获取到最新的状态，并且在执行react的更新和渲染的流程中使用了真正的异步方式(postMessage) 这个才是真正的异步。
3种方式可以立即获取到 state 的值：
1.flushSync API 后 2.setTimeout中 3.addEventListener('click', ()=>{})回调函数中
```javascript
flushSync(() => {
  this.setState({
    ding: xxx
  })
})
console.log(this.state.ding)
// 另外! 当使用用了flushSync这个API的时候 这种情况下 就是完全同步的
// 就是说 一旦执行了flushSync 就会立即触发react的更新以及渲染的过程
// 这样的话 就可以在同一个事件函数中 立即获取到最新的状态了

// unstable_batchedUpdates
setTimeout(() => {
  batchedUpdates(() => {
    this.setState({
      ding: xxx
    })
    this.setState({
      ding: yyy
    })
    console.log(this.state.ding) // 不能获取
  })
  console.log(this.state.ding) // 能获取
})
```
## 代码
### React
```javascript
const REACT_ELEMENT_TYPE = Symbol.for('react.element');

function ReactElement(type, key, props) {
  let element = {
    $$typeof: REACT_ELEMENT_TYPE,
    type, // type: 'div' | type: Ding
    key,
    props
  }
  return element;
}

function createElement(type, props = {}, children) {
  let _props = Object.assign({}, props)
  let _key = _props.key || null;
  let children_length = children.length;
  _props.children = children_length === 0 ? null : children_length === 1 ? children[0] : children
  return ReactElement(type, _key, _props)
}

class Component {
  constructor(props, context, updater) {
    this.props = props;
    this.context = context;
    this.updater = updater || null;
  }
  get isReactComponent() {
    return true;
  }
  setState(partialState, callback) {
    if (partialState instanceof Object || typeof partialState === 'function') {
      let _setState = this.updater.enqueueSetState
      _setState && _setState(this, partialState, callback, 'setState')
    }
  }
}

const React = {
  createElement: function(type, props, ...children) {
    let element = createElement(type, props, children)
    return element;
  },
  Component
}
export default React
```
### ReactDOM
```javascript
let ReactDOM = {
  render: (reactEle, container, callback) => {
    isFirstRender = true;
    let root = new ReactRoot(container);
    container._reactRootContainer = root;
    root.render(reactEle, callback);
    isFirstRender = false;
  },
}

export default ReactDOM
```
### fiber数据结构
```javascript
// Fiber 的类型
let HostRoot = 'HostRoot' // RootFiber的类型
let ClassComponent = 'ClassComponent' // class组件类型
let HostComponent = 'HostComponent' // 原生dom节点类型
let HostText = 'HostText' // 文本类型
// 其他还有各种类型比如: HostPoratl ContextProvider ContextConsumer Fragment FunctionComponent

// effectTag 当前fiber要进行何种更新
let NoWork = 0 // 表示没有更新
let Placement = 'Placement' // 表示要插入
let Deletion = 'Deletion' // 表示要删除
let Update = 'Update' // 更新
let Callback = 'Callback' // 表示有回调
let PlacementAndUpdate = 'Placement|Update' // 表示又要插入又要更新 比如说某个dom属性变了同时还要和某个dom交换位置
let Snapshot = 'Snapshot' // 新周期
let PlacementAndUpdateAndDeletion = 'Placement|Update|Deletion' // 表示所有变化的总和

// 全局变量
let nextUnitOfWork = null;

class FiberNode {
  constructor(tag, pendingProps, key) {
    this.tag = tag // 表示fiber的类型
    this.key = key
    this.type = null // 表示当前fiber对应的节点的类型
    this.stateNode = null // 表示当前fiber对应的实例
    this.child = null // 指向当前fiber的firstChild
    this.sibling = null // 当前fiber的兄弟节点
    this.return = null // 指向当前fiber的父节点
    this.index = 0
    this.memoizedState = null // 表示当前fiber上的state
    this.memoizedProps = null // 表示旧的props的状态
    this.pendingProps = pendingProps // 表示即将挂载的props 或者说是本次更新的新的props
    this.effectTag = NoWork // 用来标识当前fiber要进行何种更新
    // effect应该从子节点一点点往上挂 就是说先挂子节点 再挂父节点
    this.firstEffect = null // 表示需要更新的第一个子节点
    this.lastEffect = null // 表示需要更新的最后一个子节点
    this.nextEffect = null // 表示下一个需要更新的子节点
    this.updateQueue = null // 也是条链表 上面保存着当前fiber的所有的更新状态
    // 这条链表是怎么传递到RootFiber上的呢
    // 是从子节点开始往父节点遍历
    // 如果某个节点本身有更新 就把这个节点作为它的父节点的lastEffect挂载到return上
    this.alternate = null // 用来连接current和workInProgress 互相引用
    // ... 还有很多其他属性
    // expirationTime: 0
  }
}

function createFiber(tag, props, key) {
  return new FiberNode(tag, props, key)
}

```
### class ReactRoot
生成三大件要素：root, current( 初始current: uninitalFiber), workInProgress
```javascript
class ReactRoot {
  
  constructor(container) {
    this._internalRoot = this._createRoot(container)
  }
  
  _createRoot(container) {
    let uninitalFiber = this._createUninitalFiber(HostRoot, null, null)
    let root = {
      container: container,
      current: uninitalFiber,
      finishedWork: null
    }
    uninitalFiber.stateNode = root;
    return root;
  }
  
  _createUninitalFiber(HostRoot, props, key) {
    return createFiber(HostRoot, props, key)
  }
  
  render(element, callback) {
    let root = this._internalRoot;
    // RootFiber的updateQueue比较特殊
    let workInProgress = createWorkInProgress(root.current, null);
    workInProgress.memoizedState = { element: element };
    // 其实react源码中是先把element临时挂到了current上 反正current也用不到
    // 这里直接一点简单一点 直接挂在memoizedState上
    nextUnitOfWork = workInProgress;
    workLoop(nextUnitOfWork);
    root.finishedWork = root.current.alternate; // root的finishedWork指向新生成的workInPress树
    if (!!root.finishedWork) {
      completeRoot(root, root.finishedWork);
    }
  }
  
}
```
### createWorkInProgress
```javascript
function createWorkInProgress(current, pendingProps) {
  let workInProgress = current.alternate; // 复用
  if (!workInProgress) {
    workInProgress = createFiber(current.tag, pendingProps, current.key);
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;
    // 要让这两个fiber相互指向
    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    workInProgress.pendingProps = pendingProps;
    workInProgress.effectTag = NoWork;
    workInProgress.nextEffect = null;
    workInProgress.firstEffect = null;
    workInProgress.lastEffect = null;
  }
  // 本轮复用的workInProgress上不存在update的时候说明fiber在current上，新建的update加在了current上的fiber上
  // 源码中没有这个判断，因为在其他地方进行了状态同步
  if (!!workInProgress &&
      !!workInProgress.updateQueue &&
      !workInProgress.updateQueue.lastUpdate) {
    // current.alternate和current上的updateQueue要保持同步 
    // 在之后会对这个updateQueue进行克隆 保证我们不会操作到current上的状态
    workInProgress.updateQueue = current.updateQueue;
  }
  workInProgress.child = current.child;
  workInProgress.memoizedProps = current.memoizedProps;
  workInProgress.memoizedState = current.memoizedState;
  workInProgress.sibling = current.sibling;
  workInProgress.index = current.index;
  return workInProgress;
}
```
1 `let workInProgress = current.alternate;` 
// current.alternate 上上次的状态，current是页面上正在显示的状态，workInProgress是接下来的。如果可以的话要复用上一次的workInProgress，这里一定要用current.alternate来作为复用的节点。因为如果你直接用current的话相当于之后的任何修改都是直接在current上进行了，而current是上一轮中的workInProgress树的节点，所以current.alternate则是上一轮的current，上一轮的current因为已经完成了更新，所以对于react应用来说是没用的的状态，怎么修改都不会影响啥。所以很适合直接把这个没用的内存中对象拿过来重新复用。
2 `workInProgress.updateQueue = current.updateQueue;`
#### 补：setstate源码![setstate.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1707131816304-5a75f886-855a-43e7-ba3a-c2511afd72b5.jpeg#averageHue=%23393531&clientId=ufd00b788-6fa4-4&from=drop&id=u11bf6ec2&originHeight=1134&originWidth=2068&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1263763&status=done&style=none&taskId=u3e0c5451-5f8b-48ed-8d69-280271bc80c&title=)
图示：
![](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1707137761929-2aa17684-f1ef-455c-b2c4-e4a75c114f2d.jpeg)
new Component的时候new的组件实例中有一个属性会指向对应的 fiber，且不会改变。且每次setState传进来的 updateQueue 会挂载在组件实例上指向的这个fiber上。
（<Ding/>组件有两个 Fiber，一个就是第一次：current 为 uninitialFiber,  创建workInProgress树new Component的时候创建的Fiber；另一个是第一次 setState：`let workInProgress = current.alternate;`（等于uninitialFiber）不能复用，新建的 Fiber。之后会被不断的复用，时而在 current树中，时而作为 workInprogress。（setState的时候要找最初的fiber ，奇数次setState的时候此fiber在current树上，偶数次时此fiber在 current.alternate上））
因为这个fiber一回是在current中一回是current.alternate即上一轮的workInProgres中，所以在复用current.alternate的时候，这个alternate可能是最开始挂到组件实例上的fiber，也可能不是。
所以就导致了本轮复用的current.alternate:workInProgress上的updateQueue可能会存在update，也可能不存在，不存在update的时候说明要找的fiber存在于current树上，这个时候就把current(fiber)上的updateQueue复制过来。
#### 创建 fiber 树![fiber流程.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1706882821568-ca17b425-1ef8-42c4-a83b-5202371faae4.jpeg#averageHue=%23fbfbfb&clientId=ubf289aed-5272-4&from=drop&id=u5380005e&originHeight=922&originWidth=1279&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337566&status=done&style=none&taskId=u140079b3-77c0-45dd-9cbf-7e192796cf5&title=)
先找子节点，返回第一个子节点作为父元素的Child（123）；
再找子节点（第一个）的孩子，没有孩子，再找子节点的兄弟（p）；
都没有（p->h2），则.return 回到父节点，再找父节点的兄弟（h3）；
```javascript
function workLoop(nextUnitOfWork) {
  while(!!nextUnitOfWork) {
    // 让performUnitOfWork返回下一个要工作的work
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
  }
}

// 获取下一个节点
function performUnitWork(workInProgress) {
  let next = beginWork(workInProgress); // 子
  if (next === null) { // 兄弟
    // 说明一侧节点已经遍历完成
    next = completeUnitOfWork(workInProgress);
  }
  return next;
}

// 子节点 Fiber 生成
function beginWork(workInProgress) {
  let next = null;
  let tag = workInProgress.tag;
  if (tag === HostRoot) {
    next = updateHostRoot(workInProgress);
  } 
  else if (tag === HostComponent) {
    next = updateHostComponent(workInProgress);
  } 
  else if (tag === ClassComponent) {
    next = updateClassComponent(workInProgress);
  } 
  else if (tag === HostText) { // 到达最底下的孩子了
    next = null;
  }
  return next;
}

// 兄 || 父 Fiber 生成
function completeUnitOfWork(workInProgress) {
  // 首先肯定是个循环 找自己的兄弟节点或父节点的兄弟节点的过程
  // 如果父节点没有兄弟节点, 那么就直接再找父节点的父节点的兄弟节点
  while(true) {
    let returnFiber = workInProgress.return; //父
    let siblingFiber = workInProgress.sibling; // 兄
    
    // 这里是创建真实dom实例的最佳时期 没有子节点  是最佳的插入时机
    // 这里还要对当前的这个workInProgress执行一些初始化或者插入子节点的操作
    completeWork(workInProgress);
    
   // 第一步 先把当前节点上的有更新的子节点的fiber挂到父节点上
    if (!!returnFiber) {
      if (returnFiber.firstEffect === null) {
        // 说明当前的节点的父节点上 还没有挂任何的有更新的组件
        returnFiber.firstEffect = workInProgress.firstEffect
      }
      if (!!workInProgress.lastEffect) {
        if (!!returnFiber.lastEffect) {
          returnFiber.lastEffect.nextEffect = workInProgress.firstEffect
        }
        returnFiber.lastEffect = workInProgress.lastEffect
      }
    }

    // 第二步 判断当前节点自身是否有更新 如果有更新就挂到当前节点的父节点的effect链表的最后一位
    let effectTag = workInProgress.effectTag;
    let hasChange = (
      effectTag === Update ||
      effectTag === Deletion ||
      effectTag === Placement ||
      effectTag === PlacementAndUpdate
    );
    if (hasChange) {
      if (!!returnFiber.lastEffect) {
        returnFiber.lastEffect.nextEffect = workInProgress;
      } 
      else {
        returnFiber.firstEffect = workInProgress;
      }
      returnFiber.lastEffect = workInProgress;
    }

    
    if (!!siblingFiber) return siblingFiber; // 有兄弟返回兄弟
    if (!!returnFiber) {
      workInProgress = returnFiber; // 没有兄弟，workInProgress指针指向父节点，再次循环，找父的兄弟
      continue; // 继续循环
    }
    return null;
  }
}
```
获取子节点
```javascript
// 获取根节点RootFiber的子节点
function updateHostRoot(workInProgress) { 
  let children = workInProgress.memoizedState.element; 
  return reconcileChildren(workInProgress, children); // workInProgress是 RootFiber，children是Ding
}

function reconcileChildren(workInProgress, nextChildren) {
  // 如果workInProgress是RootFiber,初次渲染时候需要在最外层的组件上挂一个Placement
  if (isFirstRender && !!workInProgress.alternate) {
    workInProgress.child = reconcileChildFiber(workInProgress, nextChildren);
    let effectTag = workInProgress.child.effectTag;
    workInProgress.child.effectTag = effectTag ? `${effectTag}|${Placement}` : Placement;
  } 
  else {
    workInProgress.child = reconcileChildFiber(workInProgress, nextChildren);
  }
  return workInProgress.child;
}

// 分类处理 
function reconcileChildFiber(workInProgress, nextChildren) {
  // 因为传进来的nextChildren可能是单个子节点、一个数组、文本类型
  if (nextChildren instanceof Object && !!nextChildren && !!nextChildren.$$typeof) {
    // 独生子 react元素 对象类型
    return reconcileSingleElement(workInProgress, nextChildren);
  }
  if (nextChildren instanceof Array) {
    // 多个孩子
    return reconcileChildrenArray(workInProgress, nextChildren);
  }
  if (typeof nextChildren === 'string' || typeof nextChildren === 'number') {
    return reconcileSingleTextNode(workInProgress, String(nextChildren));
  }
  return null;
}
  // class bbb extends aaa {}
  // bbb.prototype 是 new aaa()


function reconcileSingleElement(workInProgress, element) {
  // 该函数的主要目的 就是要给当前的workInProgress的child创建fiber
  let type = element.type;
  let flag = null;
  // Symbol.for('react.element') ===  Symbol.for('react.element');
  if (element.$$typeof === Symbol.for('react.element')) {
    // 接下来要根据这个对象的类型来创建不同的fiber
    if (typeof type === 'function') {
      // isReactComponent是 react.component的一个属性，用来区分 class组件 和 function
      if (type.prototype && type.prototype.isReactComponent) {
        flag = ClassComponent;
      }
    } else if (typeof type === 'string') {
      flag = HostComponent;
    }
    let fiber = createFiber(flag, element.props, element.key);
    fiber.type = type;
    fiber.return = workInProgress;
    return fiber;
  }
}


function reconcileSingleTextNode(returnFiber, text) {
  let createdFiber = createFiber(HostText, text, null);
  createdFiber.return = returnFiber;
  return createdFiber;
}

```
![生成dom.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1707143686619-2b4a0ee0-3e56-4e4a-a0b7-1ce36b228624.jpeg#averageHue=%23fafafa&clientId=uc0a19e61-8d28-4&from=drop&id=uba4d97b8&originHeight=902&originWidth=1148&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312382&status=done&style=none&taskId=u8c7e6952-3ab7-4587-bcac-169403bed39&title=)![first渲染.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1707144254354-56147a73-b34e-4b55-9985-301770722744.jpeg#averageHue=%23fafafa&clientId=uc0a19e61-8d28-4&from=drop&id=u77f88c27&originHeight=925&originWidth=1519&originalType=binary&ratio=2&rotation=0&showTitle=false&size=561763&status=done&style=none&taskId=u3fc28da5-fafd-41d8-938d-d465a46cbf6&title=)
```javascript
function updateClassComponent(workInProgress) { // Ding；返回 Ding 的 child
  let component = workInProgress.type;
  let nextProps = workInProgress.pendingProps;
  if (!!component.defaultProps) {
    nextProps = Object.assign({}, component.defaultProps, nextProps)
  }
  let instance = workInProgress.stateNode;
  let shouldUpdate = null;
  if (!instance) {
    // 没有实例的话说明是第一次挂载 创建实例
    instance = new component(nextProps, null);
    workInProgress.memoizedState = instance.state;
    workInProgress.stateNode = instance;
    instance._reactInternalFiber = workInProgress;
    instance.updater = classComponentUpdater;
    // componentWillReceiveProps替代 不能改this了
    let getDerivedStateFromProps = component.getDerivedStateFromProps;
    if (!!getDerivedStateFromProps) {
      let prevState = workInProgress.memoizedState;
      let newState = getDerivedStateFromProps(nextProps, prevState);
      if (!(newState === null || newState === undefined)) {
        workInProgress.memoizedState = Object.assign({}, nextProps, newState)
      }
      instance.state = workInProgress.memoizedState;
    }
    let componentDidMount = instance.componentDidMount;
    if (!!componentDidMount) {
      let effectTag = workInProgress.effectTag;
      // 对于组件来说 Update表示有生命周期
      workInProgress.effectTag = effectTag ? `${effectTag}|${Update}` : Update;
    }
    shouldUpdate = true;
  } else {
    // 在setState时候如果组件写了snapshot方法的话 要给他一个标志
    // if (typeof instance.getSnapshotBeforeUpdate === 'function') {
    //   let effectTag = workInProgress.effectTag
    //   workInProgress.effectTag = effectTag ? `${effectTag}|${Snapshot}` : Snapshot
    // }
  }
  // if (!shouldUpdate) {}
  // 完成挂载class的阶段
  let nextChild = instance.render(); //ding组件的 render
  return reconcileChildren(workInProgress, nextChild);
}

```
react 中处理文本类型的节点的时候分2种情况

1. 一种情况是会直接对文本类型创建fiber的(当某个节点有多个兄弟节点的时候)
2. 另一种情况是不会对文本类型创建fiber 而是直接把文本类型的fiber置为null(当父节点有且只有一个文本类型的子节点的时候)

如果是不创建fiber的这种情况 那么应该怎么将这个文本类型的节点添加到父节点下头，在之后对这个父节点进行创建实例的时候 会把我们传的props们作为属性一一赋值给该节点，当然当判断到children这个属性的时候 如果这个children属性对应的是一个文本类型且是单独子元素的话,会直接添加给父元素。
```javascript
function updateHostComponent(workInProgress) { //div; 返回div的children[]
  // 首先要先拿到children
  let nextProps = workInProgress.pendingProps;
  let nextChildren = nextProps.children;
  if (typeof nextChildren === 'string' || typeof nextChildren === 'number') {
    nextChildren = null;
  }
  return reconcileChildren(workInProgress, nextChildren)
}


function reconcileChildrenArray(workInProgress, nextChildren) { // [h1,h2,h3]
  // 这个方法中要通过index和key去尽可能多的找到可以复用的dom节点
  // 这个函数在react源码中，就是最重要的最最复杂的diff算法 O(n)
  let nowWorkInProgress = null; // 指针
  if (isFirstRender) {
    nextChildren.forEach((reactEle, index) => {
       if (index === 0) {
          if (typeof reactEle === 'string' || typeof reactEle === 'number') {
            workInProgress.child = reconcileSingleTextNode(workInProgress, reactEle);
          } 
          else {
            workInProgress.child = reconcileSingleElement(workInProgress, reactEle);
          };
          nowWorkInProgress = workInProgress.child; // 第一个child
       } 
       else {
          if (typeof reactEle === 'string' || typeof reactEle === 'number') {
            nowWorkInProgress.sibling = reconcileSingleTextNode(workInProgress, reactEle);
          } 
          else {
            nowWorkInProgress.sibling = reconcileSingleElement(workInProgress, reactEle);
          };
          nowWorkInProgress = nowWorkInProgress.sibling; //第一个child的兄弟
       }
    });
    return workInProgress.child; // 返回第一个child
  } else {
    // 执行setState时候进到这里
  }
}
```
#### 创建真实的dom实例
找孩子，找孩子兄弟，找父亲，找父亲兄弟。。。
```javascript
// react事件系统 他把所有的合成事件代理到了根节点上
let eventsName = {
  onClick: 'click',
  onChange: 'change',
  onInput: 'input'
  // ...
};

// 创建自己的真实的dom实例，把子节点添加到dom下，并设置属性
function completeWork(workInprogress) {
  let tag = workInProgress.tag;
  let instance = workInProgress.stateNode;
  if (tag === HostComponent) {
    if(!instance) {
      // 如果没有实例的话 说明这个节点可能是第一次挂载也就是初次渲染，也可能是一个新添加的节点
      // 1 创建真实的dom实例
      let domElement = document.createElement(workInProgress.type);
      domElement.__reactInternalInstance = workInProgress;
      workInProgress.stateNode = domElement;

      // 2 把子节点添加到dom下
      let node = workInProgress.child;
      wrapper: while(!!node) {
        let tag = node.tag;
        if (tag === HostComponent || tag === HostText) {
          domElement.appendChild(node.stateNode); // node是fiber,stateNode才是实例
        } 
        else if (!!node.child) {
          // 可能不是原生的dom节点 比如Ding,得找他的child
          node.child.return = node;
          node = node.child;
          continue;
        };
      };
      // 没有兄弟节点要往上找父节点
      while (node.sibling === null) { 
        if (node.return === null || node.return === workInProgress) { 
          // 找到自己的child说明自己已经插入完成 结束循环
          break wrapper;
        }
        node = node.return;
      };
      // 找完孩子找孩子的兄弟
      node.sibling.return = node.return;
      node = node.sibling;
    }
    // 3 设置属性
    let props = workInProgress.pendingProps;
    for (let propKey in props) {
      let propValue = props[propKey];
      if (propKey === 'children') {
        if (typeof propValue === 'string' || typeof propValue === 'number') {
          domElement.textContent = propValue;
        }
      } 
      else if (propKey === 'style') {
        for (let stylePropKey in propValue) {
          if (!propValue.hasOwnProperty(stylePropKey)) continue;
          let styleValue = propValue[stylePropKey].trim();
          if (stylePropKey === 'float') {
            stylePropKey = 'cssFloat';
          }
          domElement.style[stylePropKey] = styleValue;
        }
      }
      else if (eventsName.hasOwnProperty(propKey)) {
        let event = props[propKey];
        
        // react中 所有写在JSX模板上的事件 都是合成事件 合成事件是react中自己实现的一套事件系统; 
        // 非合成事件：div.addEventListener('onclick', handle, false);监听到事件后handle会立即执行
        // 假如有个onClick事件 当你点击这个元素的时候，react内部不会立即执行传进来的这个函数
        // 而是在执行这个函数之前 进行了很多的操作和处理
        // 比如说 会对事件回调接收的event进行处理
        // 比如说 内部他们自己实现了一个简单的阻止冒泡的方法
        // 很重要的一点就是react中 所有的合成事件都是被注册到了你挂载react应用的那个根节点上
        
        // 简单注册一下
        domElement.addEventListener(eventsName[propKey], event, false)
        // false 表示允许冒泡
        // true 表示捕获 比如div上设置oninput事件，只有捕获才能抓到子节点 input 触发的 oninput事件
      } 
      else {
        domElement.setAttribute(propKey, propValue)
      }
    };
  }

  else if (tag === HostText) {
    // 因为文本节点是不会有子节点的所以无需执行插入子节点的操作
    let oldText = workInProgress.memoizedProps;
    let newText = workInProgress.pendingProps;
    if (!instance) {
      instance = document.createTextNode(newText);
      workInProgress.stateNode = instance;
    } else {
      // if (oldText !== newText) {
      //   // 这里要给这个文本类型的fiber一个Update的标识
      //   let effectTag = workInProgress.effectTag
      //   workInProgress.effectTag = effectTag ? `${effectTag}|Update` : 'Update'
      // }
    }
  }
  
}
```
#### commit 阶段
##### react生命周期
![生命周期.jpeg](https://cdn.nlark.com/yuque/0/2024/jpeg/40468162/1707306047547-d82f9e0e-e47b-41f0-80aa-df488560a19d.jpeg#averageHue=%23f9f8f6&clientId=uc2020c0d-e3b4-4&from=drop&id=u1da17873&originHeight=826&originWidth=1398&originalType=binary&ratio=2&rotation=0&showTitle=false&size=306710&status=done&style=none&taskId=u6c1d3b09-c5aa-4b44-8eec-613e42e05e4&title=)
##### 代码逻辑
```javascript

function completeRoot(root, finishedWork) {
  root.finishedWork = null;
  commitRoot(root, finishedWork)
}


function commitRoot(root, finishedWork) {
  isWorking = true;
  isCommitting = true;

  // 获取到effect这条链表(存储着有变动的 fiber 节点)
  let firstEffect = finishedWork.firstEffect;
  let nextEffect = null;

  // 第一个循环主要用来调用getSnapshot那个生命周期
  // getSnapshotBeforeUpdate在React真正更新dom和ref之前
  // 所以可以用这个周期获取dom更新前的一些属性的值
  nextEffect = firstEffect;
  while (!!nextEffect) {
    let effectTag = nextEffect.effectTag;
    if (effectTag.includes(Snapshot)) {
      if (nextEffect.tag === ClassComponent) {
        let current = nextEffect.alternate;
        let instance = nextEffect.stateNode;
        let prevProps = Object.assign({}, nextEffect.type.defaultProps, current.memoizedProps);
        let prevState = Object.assign({}, current.memoizedState);
        let snapshot = instance.getSnapshotBeforeUpdate(prevProps, prevState) || {};
        instance.__reactInternalSnapshotBeforeUpdate = snapshot;
      }
    }
    nextEffect = nextEffect.nextEffect
  }
  
  
  // 第二个循环真正用来操作界面 主要用来操作有更新的fiber 节点增删改
  nextEffect = firstEffect;
  while (!!nextEffect) {
    let effectTag = nextEffect.effectTag;
    if(effectTag.includes(Placement) {
      // 新插入节点
      // 1.先找到一个能被插进来的父节点 一个可以用来挂载的真实dom节点
      let parentFiber = nextEffect.return;
      let parent = null;
      while (!!parentFiber) {
        let tag = parentFiber.tag;
        if (tag === HostComponent || tag === HostRoot) {
          break;
        }
        parentFiber = parentFiber.returnFiber;
      };
      if (parentFiber.tag === HostComponent) {
        parent = parentFiber.stateNode;
      } 
      else if (parentFiber.tag === HostRoot) {
        parent = parentFiber.stateNode.container // html中的根 container
      };

      // 2.再找到能往父节点中插入的子节点 ding -> dingnew -> div 找到div
      // 简写
      if(isFirstRender) {
        let tag = nextEffect.tag;
        if(tag === HostComponent || tag === HostText) {
            parent.appendChild(nextEffect.stateNode);
        } 
        else {
          let child = nextEffect;
          
          while(true) {
            let tag = child.tag;
            if(tag === HostComponent || tag === HostText) {
              break;
            };
            
            child = child.child;
          };

          parent.appendChild(child.stateNode);
        }
      }
      
    }
    else if(effectTag === Update) {
      // 该节点要更新
    }
    else if(effectTag === Deletion) {
      // 该节点要被删除
    }
    else if(effectTag === PlacementAndUpdate) {
      // 该节点可能是换了位置且属性要更新
    };
    nextEffect = nextEffect.nextEffect;
  }
  
  // 第三个循环主要用来执行剩下的生命周期 didmount didupdate
  nextEffect = firstEffect;
  while (!!nextEffect) {
    let effectTag = nextEffect.effectTag;
    // 当有生命周期方法的时候会给当前组件一个Update
    // 当执行setState给回调的时候
    if (effectTag.includes(Update) || effectTag.includes(Callback)) {
      let tag = nextEffect.tag;
      let instance = nextEffect.stateNode;
      let current = nextEffect.alternate;
      if (tag === ClassComponent) {
        // 当组件有Update时 说明组件上有生命周期方法
        if (effectTag.includes(Update)) {
          if (!!current) {
            let prevProps = current.memoizedProps;
            let prevState = current.memoizedState;
            instance.componentDidUpdate(prevProps, prevState, instance.__reactInternalSnapshotBeforeUpdate)
          } else {
            // 第一次挂载
            instance.componentDidMount()
          }
        }
        // 当effectTag是Callback的时候
        // 说明是setState上传了回调函数
      }
    }
    nextEffect = nextEffect.nextEffect;
  }
  

  isWorking = false;
  isCommitting = false;
}
```
##### getSnapshotBeforeUpdate() 方法：
`getSnapshotBeforeUpdate(prevProps, prevState)`
getSnapshotBeforeUpdate() 方法需要与 componentDidUpdate() 方法一起使用，否则会出现错误。

- 替换componentWillUpdate函数，将参数返回并传递给componentDidUpdate周期函数
- getSnapshotBeforeUpdate() 在最近一次渲染输出（提交到 DOM 节点）之前调用。可以访问更新前的 props 和 state，它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 componentDidUpdate()。
- 此用法并不常见，但它可能出现在 UI 处理中，如需要以特殊方式处理滚动位置的聊天线程等。
```javascript
getSnapshotBeforeUpdate(prevProps, prevState) {
  // 我们是否在 list 中添加新的 items ？
  // 捕获滚动位置以便我们稍后调整滚动位置。
  if (prevProps.list.length < this.props.list.length) {
    const list = this.listRef.current;
    return list.scrollHeight - list.scrollTop;
  }
  return null;
}

componentDidUpdate(prevProps, prevState, snapshot) {
  // 如果我们 snapshot 有值，说明我们刚刚添加了新的 items，
  // 调整滚动位置使得这些新 items 不会将旧的 items 推出视图。
  //（这里的 snapshot 是 getSnapshotBeforeUpdate 的返回值）
  if (snapshot !== null) {
    const list = this.listRef.current;
    list.scrollTop = list.scrollHeight - snapshot;
  }
}
```

##### getDerivedStateFromProps(nextProps, prevState)

- 相当于componentwillmount 和 componentWillReceiveProps合并
- 静态方法，它不希望你在这里头做任何带有副作用的操作包括setState

react中尽量少依赖外部数据源 ，每次更新状态的时候尽量只靠组件自身内部的state，而在componentWillReceiveProps里执行setState的话说明肯定是用了nextProps，这样数据源就不唯一了。
```javascript
let getDerivedStateFromProps = component.getDerivedStateFromProps;
if (!!getDerivedStateFromProps) {
  let newState = getDerivedStateFromProps(nextProps, workInProgress.memoizedState)
  if (!(newState === null || newState === undefined)) {
    workInProgress.memoizedState = Object.assign({}, nextProps, newState)
  }
  instance.state = workInProgress.memoizedState
}
```
