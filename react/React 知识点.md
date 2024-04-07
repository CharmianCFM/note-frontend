### 1 render方法的原理？在什么时候被触发？
**原理**
1  两种形式  
首先，render函数在react中有两种形式：在类组件中，指的是render方法；在函数组件中，指的是函数组件本身。
2 渲染页面
在render中，我们会编写jsx，jsx通过babel编译后就会转化成createElement的形式，用于生成虚拟DOM，最终转化成真实DOM.
3 更新页面
在render过程中，React 将新调用的 render 函数返回的树与旧版本的树进行 diff 比较，更新 DOM 树。

**触发时机**
state变化

- 类组件调用 setState 修改状态，只要调用 setState就会执行 render
- 函数组件通过useState hook修改状态，useState 会判断当前值有无发生改变来确定是否执行render方法

父组件重新渲染

- 类组件重新渲染的时候，子组件也一定会重新render渲染
- 函数组件重新渲染的时候，也会导致子组件重新render

context 变更导致引用的组件重新渲染

> **避免重新渲染的优化方案  **[**https://blog.renwangyu.com/2022/10/07/react-rerender/**](https://blog.renwangyu.com/2022/10/07/react-rerender/)
> **1 状态隔离，传递不变**
> 状态下放、组件原子化：尽可能把逻辑和状态放在相关的组件内，而不要上浮到父组件，避免因为一个地方的状态变更，导致子组件的全量更新渲染。
> 子组件作为 children 传递
> 子组件作为 props 传递
> **2 记忆组件**
> 通常，我们只需要使用React为我们提供的API（[**React.memo**](https://reactjs.org/docs/react-api.html#reactmemo)），就可以把组件变为记忆组件。在属性不变（或没有属性）的情况下，记忆组件可以阻止重新渲染。
> 如果是带参数的记忆组件，那么就要保证每个参数都是初始值，我们可以用另一个API（[useMemo](https://reactjs.org/docs/hooks-reference.html#usememo)）来记忆属性值。
一句话描述，**如果想要避免重新渲染，组件memo，属性memo**。
> 记忆组件作为 props 或 children 传递：这个其实和上述的属性参数初始值一样，既然值不能变，那么如果是组件传递，也需要memo一下，不变才能不重新渲染。
> **3 列表中的重新渲染-记忆组件**
> **4 context中的优化**
> 记忆 provider 的值；拆分 context 的功能层；高阶组件记忆属性
> 注：useMemo可以帮我们记忆值，避免在重新渲染中再次计算（尤其是大计算量的值）。
但是useMemo本身也是有性能代价的，并不推荐滥用，应该尽可能去**记忆React的元素**，尤其是那些要作为组件一部分的组件元素。
类似组件元素的缓存。


### 2 React事件系统 -合成事件和原生事件
**为什么要有事件系统：**

1. 跨平台兼容：在传统的事件里，不同的浏览器需要兼容不同的写法，而React提供统一的事件对象，抹平了浏览器的兼容性差异
2. 事件可控性（设想一下，如果给真实的DOM绑定事件的话，那么用户触发DOM事件，React就不能及时感知到有事件触发了，即便是可以通过事件监听器的方式，但是也很难改变事件触发的上下文）
> 能够解决上面问题的就是，让React能够感知到事件的触发，并且让事件变成可控的。这样给onClick绑定的事件处理函数handleClick就不能直接绑定在原生 DOM 上，而是由外层 App 统一做事件代理，再主动去改变上下文状态，并且执行事件处理函数。

**事件系统设计**
> 1 事件可控性
> 2 跨平台兼容
> 3 事件合成机制
> 在React中有一套事件系统来处理DOM事件，React的事件系统大致可以分为三个部分来消化。第一个部分是事件合成系统，根据运行的平台，做事件的初始化操作。第二个就是在一次渲染过程中，收集并处理标签中的事件。第三个就是一次用户交互，事件触发，到事件执行一系列过程。
> React把事件绑定在应用对应的容器根节点root上，将事件绑定在同一容器统一管理。事件绑定采用的是 addEventListener 的方式。
> 4 事件统一处理函数
> 以React中点击事件为例子，本质上都是通过addEventListener进行监听的，但是处理点击事件的函数只有一个，这个函数就是dispatchEvent。在事件处理函数中，可以通过事件源来找到点击事件到底发生在哪个 DOM 上，这个方式在传统的事件流中叫作事件委托。
> 5 冒泡和捕获的处理
> addEventListener在绑定事件的时候，可以通过第三个参数来确定是在冒泡阶段执行，还是在捕获阶段执行
> addEventListener('click',dispatchEvent$1,true)；addEventListener('click',dispatchEvent$2,false)
> 6 收集预处理事件
> 在整个应用渲染阶段的时候，遍历fiber节点的时候，会对比props中的属性，来对事件做预处理，在老版本 React 事件系统中，事件是在这个阶段绑定和收集的。
> 7 事件执行
> 如果触发一次点击事件，那么在新版React中会触发两次React的统一处理函数：第一次是捕获执行，onClick就会在此执行。第二次是冒泡执行，onClickCapture也会执行了。这样就保证了事件处理函数（例如onClick和onClickCapture）与原生的事件流保持一致。

**几个概念：**
合成事件：
> 在React应用中，元素绑定的事件并不是原生事件，而是React合成的事件，比如onClick是由click合成，onChange是由 blur、change、focus 等多个事件合成，原生 dom中是没有 onchange事件的。底层React用一个对象registrationNameDependencies保存React事件和合成的原生事件的映射关系。

事件委托
事件委托的意思就是可以通过给父元素绑定事件，通过事件对象的target属性可以获取到当前触发目标阶段的dom元素，来进行统一管理。
事件代理的好处是有两点：
第一点：可以大量节省内存占用，减少事件注册事件。
第二点：当新增子对象时无需再次对其绑定
事件监听 & 事件绑定
事件监听主要用到了addEventListener这个函数，事件监听和事件绑定的最大区别就是事件监听可以给一个事件设置多个处理函数，而事件绑定只有一次。
```javascript
// 可以监听多个，不会被覆盖 
eventTarget.addEventListener('click',()=>{}); 
eventTarget.addEventListener('click',()=>{}); 

// 第二个会把第一个覆盖
eventTarget.onclick=function(){}; 
eventTarget.onclick=function(){};
```
dom事件流
描述的是从页面中接收事件的顺序。事件冒泡流/事件捕获流。
三个阶段：事件捕获阶段、事件目标阶段、事件冒泡阶段。
事件执行顺序
捕获阶段 => 目标阶段 => 冒泡阶段
![截屏2024-03-10 下午1.14.14.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710047693570-f8c74325-6716-4a2e-b9fd-956506ef189c.png#averageHue=%23f9f9f9&clientId=u9318b0ce-66c3-4&from=drop&height=272&id=u2a697b3d&originHeight=678&originWidth=1146&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87251&status=done&style=none&taskId=uef671eb9-92fe-43f8-b738-96a288e21bf&title=&width=460)
**react 事件系统不同表现**
react 16 版本
在react17之前事件系统是将所有合成事件都挂载在document上。通过dispatchEvent代理所有的事件处理逻辑。当事件触发时，会收集当前事件源到根元素的所有同类型事件（此处包括捕获和冒泡）。生成事件列表，最后遍历事件列表，执行事件函数。
![截屏2024-03-10 下午1.24.58.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710048306943-766d32f5-e0b3-409b-a34c-ed6466b2920a.png#averageHue=%23fefefe&clientId=u9318b0ce-66c3-4&from=drop&height=285&id=u44576631&originHeight=1244&originWidth=2254&originalType=binary&ratio=2&rotation=0&showTitle=false&size=333137&status=done&style=none&taskId=u14958378-2383-4daa-a153-20b2856ad6e&title=&width=516)
![截屏2024-03-10 下午4.53.12.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710060800599-60c793e9-75cd-4b93-84a0-b0ea26fb0f5f.png#averageHue=%23333437&clientId=u9318b0ce-66c3-4&from=drop&height=202&id=u0214efc3&originHeight=522&originWidth=670&originalType=binary&ratio=2&rotation=0&showTitle=false&size=83847&status=done&style=none&taskId=u5303fec5-e52e-4c38-b51a-9f72bc9e2a5&title=&width=259)
> 1 当我们把上面的demo的原生div的stopPropagation() 方法调用阻止捕获和冒泡阶段中当前事件的进一步传播，会阻止后续的所有事件执行
> const divClickCapFunc = (e) => {
    e.stopPropagation(); // 增加原生捕获阶段的阻止事件
    logFunc("div", false, true);
};
> 当阻止之后，我们点击h1，事件流运行到div的捕获阶段就不触发了，后续的所有的捕获冒泡包括合成事件也都不会触发。
> 2 而当我们给合成事件的事件流中断stopPropagation() 时，合成事件运行到捕获阶段的div之后被阻止传播了，后续的所有合成事件都不会执行了，但是原生的document冒泡还是会执行完。

**react 17**
![截屏2024-03-10 下午4.52.10.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710060735134-1171746f-ec06-4983-8364-9557c60c1178.png#averageHue=%23e7e7e7&clientId=u9318b0ce-66c3-4&from=drop&height=227&id=ub8ddd7b3&originHeight=424&originWidth=522&originalType=binary&ratio=2&rotation=0&showTitle=false&size=77649&status=done&style=none&taskId=ud05eee44-b59b-4234-a181-8e573de8975&title=&width=279)
react 18
在新的事件系统中，createRoot方法会一次性注册完全部的事件。即在点击之前 就在 root 容器上监听了 JavaScript 所有原生事件的冒泡和捕获。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710081403932-9037a483-2674-4e64-8448-6b6af7c3a173.png#averageHue=%23f2f2f2&clientId=u9318b0ce-66c3-4&from=paste&height=306&id=u8c2a9dca&originHeight=822&originWidth=1507&originalType=binary&ratio=2&rotation=0&showTitle=false&size=181866&status=done&style=none&taskId=uf821436a-cc86-4993-b168-4907990e2c4&title=&width=561)
**总结：**

- 16版本将所有合成事件都挂载在document上，通过dispatchEvent代理所有的事件处理逻辑。它会先执行原生事件，当冒泡到document时，才统一执行合成事件，--但这是不符合预期的！与原生的事件流顺序差别较大（e.stopPropagation不能阻止事件冒泡）
- 17版本，React把事件挂载在了render的根节点上（id为app的div上）,修改了dispatchEvent的逻辑，避免了上述的问题。在原生事件执行前先执行合成事件捕获阶段，原生事件执行完毕执行冒泡阶段的合成事件, 通过根节点来管理所有的事件。
- 而到了[react18事件系统](https://juejin.cn/post/7156225347879436318#heading-0)上有了较大的改变。大概的过程是：
   - （事件绑定）首先事件初始化合成事件 -> 捕获和冒泡分别绑定事件 -> 
   - （事件触发）进行事件收集 -> 执行捕获阶段的事件 -> 执行冒泡阶段的事件。
   - 因为事件源是react自己合成的。这里如果一个事件中执行了e.stopPropagation那么事件源就能感知到。接下来就可以阻止事件冒泡。

**新老版本事件系统的差异**
![截屏2024-03-10 下午11.05.23.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710083132972-bf38391d-f8ce-4565-b053-7e88bf47d2b6.png#averageHue=%23efefef&clientId=u9318b0ce-66c3-4&from=drop&height=400&id=uaf683ea5&originHeight=906&originWidth=1682&originalType=binary&ratio=2&rotation=0&showTitle=false&size=287007&status=done&style=none&taskId=uc756659c-fdce-48a8-bf42-705801c3f2a&title=&width=742)
1 初始化差异
老版本事件系统初始化过程中，并没有直接注册事件，取而代之的是**形成了一个事件插件对象** registrationNameModules。const registrationNameModules = { onBlur: SimpleEventPlugin, onClick: SimpleEventPlugin, ... }
> 为什么要用不同的事件插件处理不同的React事件? 首先对于不同的事件，有不同的处理逻辑；对应的事件源对象也有所不同，React的事件和事件源是自己合成的，所以对于不同事件需要不同的事件插件处理。

在新的事件系统中，createRoot方法会**一次性注册完全部的事件**。捕获和冒泡分别绑定事件。
2 事件收集差异
**在老版本事件系统中，在render阶段会执行事件的绑定和收集**。会处理props，比如发现了onClick事件，那么才向外层容器中绑定 click 事件，如果发现了onChange事件，才向容器中绑定 blur、change、focus 等事件，而不是在初始化过程中统一绑定的。
3 事件执行差异
**新版本事件系统会触发两次事件**，分别是冒泡和捕获事件，优先执行捕获事件，onClickCapture等事件。接下来执行冒泡事件，onClick事件。
在老版本事件系统中，**只会执行一次事件，本质上是在冒泡阶段执行的**。而捕获阶段执行的事件，是事件系统模拟的。
> 具体如何模拟的呢？React 会在事件底层用一个数组队列来收集fiber 树上一条分支上的所有的onClick和onClickCapture事件，遇到捕获阶段执行的事件，比如onClickCapture，就会通过unshift放在数组的前面，如果遇到冒泡阶段执行的事件，比如onClick，就会通过push放在数组的后面，最后依次执行队列中的事件处理函数，模拟事件流。这个就是为什么老版本的事件系统执行时机和真实的事件流相差很大的原因。


API 补充：
> 默认情况下，addEventListener第三个参数为false， 默认传播方向是冒泡。
> div1.addEventListener("click",function (e) {console.log("div1被点击"); e.stopPropagation(); // e.stopImmediatePropagation(); // 也能阻止捕获 }, true );
>  div2.addEventListener("click", function (e) {console.log("div2被点击"); e.stopPropagation();// e.stopImmediatePropagation(); // 也能阻止冒泡 });
>     - event.preventDefault();  // 阻止默认事件 <a href=''/> 阻止默认的为空跳转到自己页面
>     - event.stopPropagation();  // 阻止事件传播 即阻止捕获和冒泡阶段中当前事件的进一步传播
>     - event.stopImmediatePropagation();  //阻止监听同一事件的其他事件监听器被调用。
> 如果多个事件监听器被附加到相同元素的相同事件类型上，当此事件触发时，它们会按其被添加的顺序被调用。如果在其中一个事件监听器中执行 stopImmediatePropagation() ，那么剩下的事件监听器都不会被调用。

### 简述react的事件代理机制
React@18
React 并不会把所有的处理函数直接绑定在真实的节点上。而是把所有的事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监听器上**维持了一个映射**来保存所有组件内部的事件监听和处理函数。
当组件挂载或卸载时，只是在这个**统一的事件监听器**上插入或删除一些对象。
当事件发生时，首先被这个统一的事件监听器处理，然后在映射里找到真正的事件处理函数并调用。
这样做的优点是解决了兼容性问题，并且简化了事件处理和回收机制（不需要手动的解绑事件，React 已经在内部处理了）。但是有些事件 React 并没有实现，比如 window 的 resize 事件。
在React@17.0.3版本中：

- 所有事件都是委托在id = root的DOM元素中（网上很多说是在document中，17版本不是了）；
- 在应用中所有节点的事件监听其实都是在id = root的DOM元素中触发；
- React自身实现了一套事件冒泡捕获机制；
- React实现了合成事件SyntheticEvent；
- React在17版本不再使用事件池了（网上很多说使用了对象池来管理合成事件对象的创建销毁，那是16版本及之前）；
- 事件一旦在id = root的DOM元素中委托，其实是一直在触发的，只是没有绑定对应的回调函数；

按官方解释，之所以会将事件委托从document中移到id = root的DOM元素，是为了**可以更加安全地进行新旧版本 React 树的嵌套**。
###  3 受控组件和非受控组件及应用场景
大部分时候推荐使用受控组件来实现表单，因为在受控组件中，表单数据由React组件负责处理
如果选择非受控组件的话，控制能力较弱，表单数据就由DOM本身处理，但更加方便快捷，代码量少
针对两者的区别，其应用场景如下图所示：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710125403492-9be710a2-12ad-4324-b80c-3bb099e3bc1d.png#averageHue=%23f6f4f4&clientId=u9318b0ce-66c3-4&from=paste&height=318&id=ucc404968&originHeight=636&originWidth=882&originalType=binary&ratio=2&rotation=0&showTitle=false&size=83415&status=done&style=none&taskId=ube064097-482f-4a06-b0dc-99153ed1059&title=&width=441)
### 4 你在 react项目中是如何使用redux的，项目结构是如何划分的？
** 1 背景**
redux是用于数据状态管理的，而react是一个视图层面的库
如果将两者连接在一起，可以使用官方推荐react-redux库，其具有高效且灵活的特性
react-redux将组件分成：

- 容器组件：存在逻辑处理
- UI 组件：只负责现显示和交互，内部不处理逻辑，状态由外部控制

通过redux将整个应用状态存储到store中，组件可以派发dispatch行为action给store
其他组件通过订阅store中的状态state来更新自身的视图
**2 如何做**
使用react-redux分成了两大核心：

- Provider
- connection

**Provider**
在redux中存在一个store用于存储state，如果将这个store存放在顶层元素中，其他组件都被包裹在顶层元素之下
那么所有的组件都能够受到redux的控制，都能够获取到redux中的数据。
使用方式如下：
```javascript
<Provider store ={store}>
    <App />
<Provider>
```
**connection**
connect方法将store上的getState 和 dispatch 包装成组件的props
导入conect如下：
import{ connect }from"react-redux";
用法如下：
connect(mapStateToProps, mapDispatchToProps)(MyComponent)
可以传递两个参数：

- mapStateToProps
- mapDispatchToProps

**mapStateToProps**
把redux中的数据映射到react中的props中去，组件内部就能够通过props获取到store中的数据
```javascript
const mapStateToProps = (state) => {
    return {
        // prop : state.xxx  | 意思是将state中的某个数据映射到props中
        foo: state.bar
    }
}
```
**mapDispatchToProps**
将redux中的dispatch映射到组件内部的props中
```javascript
const mapDispatchToProps = (dispatch) => { // 默认传递参数就是dispatch
  return {
    onClick: () => {
      dispatch({
        type: 'increatment'
      });
    }
  };
}
```
```javascript
class Foo extends Component {
    constructor(props){
        super(props);
    }
    render(){
        return(
             <button onClick = {this.props.onClick}>点击increase</button>
        )
    }
}
Foo = connect(mapStateToProps, mapDispatchToProps)(Foo);
export default Foo;
```
**3 项目结构**

1. 按角色组织（MVC） ---scratch ---react16  "react-redux": "5.0.7"
- reducers
- actions
- components  // 视图+交互  使用 container 提供的数据和函数
- containers    // connect+逻辑
```javascript
reducers/
  todoReducer.js
  filterReducer.js
actions/
  todoAction.js
  filterActions.js
components/
  todoList.js
  todoItem.js
  filter.js
containers/
  todoListContainer.js
  todoItemContainer.js
  filterContainer.js
```

2. 按功能组织

就是把完成同一应用功能的代码放在一个目录下，一个应用功能包含多个角色的代码
Redux中，不同的角色就是reducer、actions和视图，而应用功能对应的就是用户界面的交互模块
```javascript
todoList/
  actions.js
  actionTypes.js
  index.js
  reducer.js
  views/
    components.js
    containers.js
filter/
  actions.js
  actionTypes.js
  index.js
  reducer.js
  views/
    components.js
    container.js
```
每个功能模块对应一个目录，每个目录下包含同样的角色文件：

- actionTypes.js 定义action类型
- actions.js 定义action构造函数
- reducer.js 定义这个功能模块如何响应actions.js定义的动作
- views 包含功能模块中所有的React组件，包括展示组件和容器组件
- index.js 把所有的角色导入，统一导出

其中index模块用于导出对外的接口
```javascript
import*as actions from'./actions.js';
import reducer from'./reducer.js';
import view from'./views/container.js';

export{ actions, reducer, view };
```
导入方法如下
```javascript
import{ actions, reducer, view as TodoList }from'./xxxx'
```
### 5 说说你对 redux的理解，其工作原理
**1 定义：**
React是用于构建用户界面的，帮助我们解决渲染DOM的过程
而在整个应用中会存在很多个组件，每个组件的state是由自身进行管理，包括组件定义自身的state、组件之间的通信通过props传递、使用Context实现数据共享
如果让每个组件都存储自身相关的状态，理论上来讲不会影响应用的运行，但在开发及后续维护阶段，我们将花费大量精力去查询状态的变化过程
这种情况下，如果将所有的状态进行集中管理，当需要更新状态的时候，仅需要对这个管理集中处理，而不用去关心状态是如何分发到每一个组件内部的
redux就是一个实现上述集中管理的容器，遵循三大基本原则：

- 单一数据源
- state 是只读的
- 使用纯函数来执行修改

注意的是，redux并不是只应用在react中，还与其他界面库一起使用，如Vue。
**2 工作原理：**
redux 要求我们把数据都放在 store 公共存储空间
一个组件改变了 store 里的数据内容，其他组件就能感知到 store 的变化，再来取数据，从而间接的实现了这些数据传递的功能。
工作流程图如下：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710131120965-4c4b3910-7c4c-4bb5-9887-dd7e321e9f6a.png#averageHue=%23f9f7f5&clientId=u9318b0ce-66c3-4&from=paste&height=247&id=u07b67053&originHeight=494&originWidth=783&originalType=binary&ratio=2&rotation=0&showTitle=false&size=20174&status=done&style=none&taskId=u7edb1f1d-9c32-4b29-aded-4ec72add44b&title=&width=391.5)
**3 如何使用：**
创建一个store的公共数据区域
```javascript
import { createStore } from 'redux'// 引入一个第三方的方法
const store = createStore()// 创建数据的公共存储区域（管理员）
```
还需要创建一个记录本去辅助管理数据，也就是reduecer，本质就是一个函数，接收两个参数state，action，返回state
```javascript
// 设置默认值
const initialState ={
  counter:0
}

const reducer = (state = initialState, action)=>{}
```
然后就可以将记录本传递给store，两者建立连接。如下：
```javascript
const store =createStore(reducer)
```
如果想要获取store里面的数据，则通过store.getState()来获取当前state。
```javascript
console.log(store.getState());
```
下面再看看如何更改store里面数据，是通过dispatch来派发action，通常action中都会有type属性，也可以携带其他的数据
```javascript
store.dispatch({
  type:"INCREMENT"
})

store.dispath({
  type:"DECREMENT"
})

store.dispatch({
  type:"ADD_NUMBER",
  number:5
})
```
下面再来看看修改reducer中的处理逻辑：
```javascript
const reducer = (state = initialState, action)=>{
  switch(action.type){
    case"INCREMENT":
      return {...state, counter: state.counter + 1};
    case"DECREMENT":
      return {...state, counter: state.counter - 1};
    case"ADD_NUMBER":
      return {...state, counter: state.counter + action.number}
    default: 
      return state;
  }
}
```
注意，reducer是一个纯函数，不需要直接修改state
这样派发action之后，既可以通过store.subscribe监听store的变化，如下：
```javascript
store.subscribe(()=>{
  console.log(store.getState());
})
```
在React项目中，会搭配react-redux进行使用
完整代码如下：
```javascript
const redux =require('redux');

const initialState ={
counter:0
}

// 创建reducer
constreducer=(state = initialState, action)=>{
  switch(action.type){
    case"INCREMENT":
      return{...state, counter: state.counter +1};
    case"DECREMENT":
      return{...state, counter: state.counter -1};
    case"ADD_NUMBER":
      return{...state, counter: state.counter + action.number}
    default: 
      return state;
  }
}

// 根据reducer创建store
const store = redux.createStore(reducer);

store.subscribe(()=>{
  console.log(store.getState());
})

// 修改store中的state
store.dispatch({
  type:"INCREMENT"
})
// console.log(store.getState());

store.dispatch({
  type:"DECREMENT"
})
// console.log(store.getState());

store.dispatch({
  type:"ADD_NUMBER",
  number:5
})
// console.log(store.getState());
```
**4 小结**

- createStore可以帮助创建 store
- store.dispatch 帮助派发 action , action 会传递给 store
- store.getState 这个方法可以帮助获取 store 里边所有的数据内容
- store.subscrible 方法订阅 store 的改变，只要 store 发生改变， store.subscrible 这个函数接收的这个回调函数就会被执行
### 6 redux中间件
**1 定义**
在上篇文章中，了解到了Redux整个工作流程，当action发出之后，reducer立即算出state，整个过程是一个同步的操作。
那么如果需要支持异步操作，或者支持错误处理、日志监控，这个过程就可以用上中间件。
Redux中，中间件就是放在就是在dispatch过程，在分发action时进行拦截处理，如下图：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710132344170-a7444e09-7f92-4c28-a4d7-5601e279ea28.png#averageHue=%23f7c932&clientId=u9318b0ce-66c3-4&from=paste&height=224&id=ufa49d5a3&originHeight=348&originWidth=496&originalType=binary&ratio=2&rotation=0&showTitle=false&size=46150&status=done&style=none&taskId=u4ba298ae-4801-4b7e-ad25-ad17d174a43&title=&width=319)
其本质上一个函数，对store.dispatch方法进行了改造，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能
** 2 常用中间件**
有很多优秀的redux中间件，如：

- redux-thunk：用于异步操作
- redux-logger：用于日志记录

上述的中间件都需要通过applyMiddlewares进行注册，作用是将所有的中间件组成一个数组，依次执行
然后作为第二个参数传入到createStore中
```javascript
const store =createStore(
    reducer,
    applyMiddleware(thunk, logger)
);
```
**redux-thunk**
redux-thunk是官网推荐的异步处理中间件
默认情况下的dispatch(action)，action需要是一个JavaScript的对象
redux-thunk中间件会判断你当前传进来的数据类型，如果是一个函数，将会给函数传入参数值（dispatch，getState）

- dispatch函数用于我们之后再次派发action
- getState函数考虑到我们之后的一些操作需要依赖原来的状态，用于让我们可以获取之前的一些状态

所以dispatch可以写成下述函数的形式：
```javascript
const getHomeMultidataAction = () => {
  return (dispatch) => {
    axios.get("http://xxx.xx.xx.xx/test").then(res => {
      const data = res.data.data;
      dispatch(changeBannersAction(data.banner.list));
      dispatch(changeRecommendsAction(data.recommend.list));
    })
  }
}
```
**redux-logger**
如果想要实现一个日志功能，则可以使用现成的redux-logger
```javascript
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);
```
这样我们就能简单通过中间件函数实现日志记录的信息
**3 实现原理**
```javascript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```
所有中间件被放进了一个数组chain，然后嵌套执行，最后执行store.dispatch。可以看到，中间件内部（middlewareAPI）可以拿到getState和dispatch这两个方法
redux-thunk
```javascript
function patchThunk(store) {
    let next = store.dispatch;

    function dispatchAndThunk(action) {
        if (typeof action === "function") {
            action(store.dispatch, store.getState);
        } else {
            next(action);
        }
    }

    store.dispatch = dispatchAndThunk;
}
```
实现一个日志输出的原理也非常简单，如下：
```javascript
function patchLogger(store) {
  let next = store.dispatch;
  
  function dispatchAndLog(action) {
    console.log("dispatching:", addAction(10));
    next(addAction(5));
    console.log("新的state:", store.getState());
  }
  
  store.dispatch = dispatchAndLog;
}
```
### 7 说说对 immutable的理解？如何应用在react项目中？
**1 定义**
Immutable，不可改变的，在计算机中，即指一旦创建，就不能再被更改的数据
对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象
Immutable 实现的原理是 Persistent Data Structure（持久化数据结构）:

- 用一种数据结构来保存数据
- 当数据被修改时，会返回一个新对象，但是新的对象会尽可能的利用之前的数据结构而不会对内存造成浪费

也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变，同时为了避免 deepCopy 把所有节点都复制一遍带来的性能损耗，Immutable 使用了 Structural Sharing（结构共享）
如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710149533477-ac256b29-68ba-4bb2-9c66-a1ae73a21b27.png#averageHue=%2331373d&clientId=u9318b0ce-66c3-4&from=paste&height=288&id=uf9fcd0f4&originHeight=575&originWidth=613&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319756&status=done&style=none&taskId=u181072a6-2741-4e24-a394-5794bf84e87&title=&width=306.5)
**2 使用**
使用Immutable对象最主要的库是immutable.js
immutable.js 是一个完全独立的库，无论基于什么框架都可以用它
**其出现场景在于弥补 Javascript 没有不可变数据结构的问题，通过 structural sharing来解决的性能问题**
内部提供了一套完整的 Persistent Data Structure，还有很多易用的数据类型，如Collection、List、Map、Set、Record、Seq，其中：

- List: 有序索引集，类似 JavaScript 中的 Array
- Map: 无序索引集，类似 JavaScript 中的 Object
- Set: 没有重复值的集合

主要的方法如下：

- fromJS()：将一个js数据转换为Immutable类型的数据
```javascript
const obj = Immutable.fromJS({a:'123',b:'234'})
```

- toJS()：将一个Immutable数据转换为JS类型的数据
- is()：对两个对象进行比较
```javascript
import{ Map, is }from'immutable'
const map1 =Map({ a:1, b:1, c:1})
const map2 =Map({ a:1, b:1, c:1})
map1 === map2 //false
Object.is(map1, map2)// false
is(map1, map2)// true
```

- get(key)：对数据或对象取值
- getIn([]) ：对嵌套对象或数组取值，传参为数组，表示位置
```javascript
let abs = Immutable.fromJS({a:{b:2}});
abs.getIn(['a','b'])// 2
abs.getIn(['a','c'])// 子级没有值

let arr = Immutable.fromJS([1,2,3:{a:5}]);
arr.getIn([3,'a']);// 5
arr.getIn([3,'c']);// 子级没有值
```

- 如下例子：使用方法如下：
```javascript
import Immutable from'immutable';
foo = Immutable.fromJS({a:{b:1}});
bar = foo.setIn(['a','b'],2);// 使用 setIn 赋值
console.log(foo.getIn(['a','b']));// 使用 getIn 取值，打印 1
console.log(foo === bar);// 打印 false
```
如果换到原生的js，则对应如下：
```javascript
let foo ={a:{b:1}};
let bar = foo;
bar.a.b =2;
console.log(foo.a.b);// 打印 2
console.log(foo === bar);// 打印 true
```
**3 在 react中应用**
1 使用 Immutable 可以给 React 应用带来性能的优化，主要体现在减少渲染的次数。
在做react性能优化的时候，为了避免重复渲染，我们会在shouldComponentUpdate()中做对比，当返回true执行render方法
Immutable通过is方法则可以完成对比，而无需像一样通过深度比较的方式比较
2 在使用redux过程中也可以结合Immutable，不使用Immutable前修改一个数据需要做一个深拷贝
```javascript
import '_' from 'lodash';

const Component = React.createClass({
  getInitialState() {
    return {
      data: { times: 0 }
    }
  },
  handleAdd() {
    let data = _.cloneDeep(this.state.data);
    data.times = data.times + 1;
    this.setState({ data: data });
  }
}

  // 使用 Immutable 后：
  getInitialState() {
    return {
      data: Map({ times: 0 })
    }
  },
  handleAdd() {
    this.setState({ data: this.state.data.update('times', v => v + 1) });
    // 这时的 times 并不会改变
    console.log(this.state.data.get('times'));
  }                                    
```
同理，在redux中也可以将数据进行fromJS处理
```javascript
import * as constants from './constants'
import {fromJS} from 'immutable'
const defaultState = fromJS({ //将数据转化成immutable数据
    home:true,
    focused:false,
    mouseIn:false,
    list:[],
    page:1,
    totalPage:1
})
export default(state=defaultState,action)=>{
    switch(action.type){
        case constants.SEARCH_FOCUS:
            return state.set('focused',true) //更改immutable数据
        case constants.CHANGE_HOME_ACTIVE:
            return state.set('home',action.value)
        case constants.SEARCH_BLUR:
            return state.set('focused',false)
        case constants.CHANGE_LIST:
            // return state.set('list',action.data).set('totalPage',action.totalPage)
            //merge效率更高，执行一次改变多个数据
            return state.merge({
                list:action.data,
                totalPage:action.totalPage
            })
        case constants.MOUSE_ENTER:
            return state.set('mouseIn',true)
        case constants.MOUSE_LEAVE:
            return state.set('mouseIn',false)
        case constants.CHANGE_PAGE:
            return state.set('page',action.page)
        default:
            return state
    }
}
```
python 文件系统数据结构？？优化
```javascript
const filedata: any = {
    children: [
      {
          name: 'main.py',
          type: 'py',
          path: 'main.py',
          select: true,
      }
    ],
    name: 'root',
    type: 'dir',
    path: '',
};
```
###  8 说说 react JSX 转换成真实 dom的过程

- 使用React.createElement或JSX编写React组件，实际上所有的 JSX 代码最后都会转换成React.createElement(...) ，Babel帮助我们完成了这个转换的过程。
- createElement函数对key和ref等特殊的props进行处理，并获取defaultProps对默认props进行赋值，并且对传入的孩子节点进行处理，最终构造成一个虚拟DOM对象
- ReactDOM.render将生成好的虚拟DOM渲染到指定容器上，其中采用了批处理、事务等机制并且对特定浏览器进行了性能优化，最终转换为真实DOM
```javascript
<div>
  <img src="avatar.png" className="profile" />
  <Hello />
</div>

// babel转化
React.createElement(
  "div",
  null,
  React.createElement("img", {
    src: "avatar.png",
    className: "profile"
  }),
  React.createElement(Hello, null)
);

// createElement结果：生成虚拟 dom对象
{
  type: 'div',
  props: {
    children: [
      {
        type: 'img',
        props: {
          src: "avatar.png",
          className: "profile",
          children: []
        }
      },
      {
        type: "Hello",
        children: []
      }
    ]
  }
}

// 渲染虚拟dom到页面
ReactDOM.render(element, container[, callback])
// 如果提供了可选的回调函数callback，该回调将在组件被渲染或更新之后被执行
```
在react中，节点大致可以分成四个类别：

- 文本节点
- 原生标签节点
- 函数组件
- 类组件

渲染的时候会分别渲染不同的节点。
### 9 说说你在 react 项目中是如何捕获错误的
**1 **为了解决出现的错误导致整个应用崩溃的问题，react16引用了**错误边界**新的概念
错误边界是一种 React 组件，这种组件可以捕获发生在其子组件树任何位置的 JavaScript 错误，并打印这些错误，同时展示降级 UI，而并不会渲染那些发生崩溃的子组件树
错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误
形成错误边界组件的两个条件：

- 使用了 static getDerivedStateFromError()
- 使用了 componentDidCatch()

抛出错误后，请使用 static getDerivedStateFromError() 渲染备用 UI ，使用 componentDidCatch() 打印错误信息，如下：
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

// 然后就可以把自身组件的作为错误边界的子组件，如下：
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
下面这些情况无法捕获到异常：

- 事件处理
- 异步代码
- 服务端渲染
- 自身抛出来的错误

在react 16版本之后，会把渲染期间发生的所有错误打印到控制台。
除了错误信息和 JavaScript 栈外，React 16 还提供了组件栈追踪。现在你可以准确地查看发生在组件树内的错误信息：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710167314749-b1ac8e69-13f8-4f9b-9101-42148e3c9d9e.png#averageHue=%23fde4e4&clientId=ud289c408-fb23-4&from=paste&height=102&id=ubd27f05c&originHeight=204&originWidth=1248&originalType=binary&ratio=2&rotation=0&showTitle=false&size=33435&status=done&style=none&taskId=u4d0d163f-6b1f-428c-a67f-9a89cf483b3&title=&width=624)
可以看到在错误信息下方文字中存在一个组件栈，便于我们追踪错误
对于错误边界无法捕获的异常，如事件处理过程中发生问题并不会捕获到，是因为其不会在渲染期间触发，并不会导致渲染时候问题。
**2 **这种情况可以使用js的**try...catch...**语法，如下：
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    try {
      // 执行操作，如有错误则会抛出
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>
    }
    return <button onClick={this.handleClick}>Click Me</button>
  }
}
```
try-catch属于同步代码块，因此无法捕获异步（重新开辟的线程，例如定时器，异步请求）代码中的异常，即能被try-catch捕获的异常，必须是在报错时候，线程的执行进入了try-catch代码块时，才能被捕获异常。
通常，promise的异常都是由reject以及Promise.prototype.catch来捕获的，Promise在执行回调中都用try-catch包裹了，错误被内部捕获，不会往上抛。try-catch捕获不到。
**3 **除此之外还可以通过监听**onerror**事件捕获异常
```javascript
window.addEventListener('error', function(event) { ... })
// js error 或者 resource error
window.addEventListener('unhandledRejection', function(event) { ... })
// promise error
```
###  10 react 服务端渲染怎么做？原理是什么？
服务端渲染（Server-Side Rendering ，简称SSR），指由服务侧完成页面的 HTML 结构拼接的页面处理技术，发送到浏览器，然后为其绑定状态与事件，成为完全可交互页面的过程
其解决的问题主要有两个：

- SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面
- 加速首屏加载，解决首屏白屏问题![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710169503379-60ae5bd9-78c3-46d6-b300-c2bae8634a03.png#averageHue=%23fcfcfc&clientId=ud289c408-fb23-4&from=paste&height=263&id=ua14fceac&originHeight=525&originWidth=930&originalType=binary&ratio=2&rotation=0&showTitle=false&size=63121&status=done&style=none&taskId=u1fc5d520-4ccf-4075-acf6-7bf3959f560&title=&width=465)

在react中，实现SSR主要有两种形式：

- 手动搭建一个 SSR 框架
- 使用成熟的SSR 框架，如 Next.JS

这里主要以手动搭建一个SSR框架进行实现
首先通过express启动一个app.js文件，用于监听3000端口的请求，当请求根目录时，返回HTML，如下：
```javascript
const express = require('express')
const app = express()
app.get('/', (req,res) => res.send(`
<html>
   <head>
       <title>ssr demo</title>
   </head>
   <body>
       Hello world
   </body>
</html>
`))

app.listen(3000, () => console.log('Exampleapp listening on port 3000!'))
```
然后再服务器中编写react代码，在app.js中进行引用
```javascript
import React from 'react'

const Home = () =>{

    return <div>home</div>

}

export default Home
```
为了让服务器能够识别JSX，这里需要使用webpakc对项目进行打包转换，创建一个配置文件webpack.server.js并进行相关配置，如下：
```javascript
const path = require('path')    //node的path模块
const nodeExternals = require('webpack-node-externals')

module.exports = {
    target:'node',
    mode:'development',           //开发模式
    entry:'./app.js',             //入口
    output: {                     //打包出口
        filename:'bundle.js',     //打包后的文件名
        path:path.resolve(__dirname,'build')    //存放到根目录的build文件夹
    },
    externals: [nodeExternals()],  //保持node中require的引用方式
    module: {
        rules: [{                  //打包规则
           test:   /\.js?$/,       //对所有js文件进行打包
           loader:'babel-loader',  //使用babel-loader进行打包
           exclude: /node_modules/,//不打包node_modules中的js文件
           options: {
               presets: ['react','stage-0',['env', { 
                                  //loader时额外的打包规则,对react,JSX，ES6进行转换
                    targets: {
                        browsers: ['last 2versions']   //对主流浏览器最近两个版本进行兼容
                    }
               }]]
           }
       }]
    }
}
```
接着借助react-dom提供了服务端渲染的 renderToString方法，负责把React组件解析成html
```javascript
import express from 'express'
import React from 'react'//引入React以支持JSX的语法
import { renderToString } from 'react-dom/server'//引入renderToString方法
import Home from'./src/containers/Home'

const app= express()
const content = renderToString(<Home/>)
app.get('/',(req,res) => res.send(`
<html>
   <head>
       <title>ssr demo</title>
   </head>
   <body>
        ${content}
   </body>
</html>
`))

app.listen(3001, () => console.log('Exampleapp listening on port 3001!'))
```
上面的过程中，已经能够成功将组件渲染到了页面上
但是像一些事件处理的方法，是无法在服务端完成，因此需要将组件代码在浏览器中再执行一遍，这种服务器端和客户端共用一套代码的方式就称之为**同构**
通俗讲，“同构”就是一套React代码在服务器上运行一遍，到达浏览器又运行一遍：

- 服务端渲染完成页面结构
- 浏览器端渲染完成事件绑定

浏览器实现事件绑定的方式为让浏览器去拉取JS文件执行，让JS代码来控制，因此需要引入script标签
通过script标签为页面引入客户端执行的react代码，并通过express的static中间件为js文件配置路由，修改如下
```javascript
import express from 'express'
import React from 'react'//引入React以支持JSX的语法
import { renderToString } from'react-dom/server'//引入renderToString方法
import Home from './src/containers/Home'

const app = express()
app.use(express.static('public'));
//使用express提供的static中间件,中间件会将所有静态文件的路由指向public文件夹
const content = renderToString(<Home/>)

app.get('/',(req,res)=>res.send(`
<html>
   <head>
       <title>ssr demo</title>
   </head>
   <body>
        ${content}
   <script src="/index.js"></script>
   </body>
</html>
`))

 app.listen(3001, () =>console.log('Example app listening on port 3001!'))
```
然后再客户端执行以下react代码，新建webpack.client.js作为客户端React代码的webpack配置文件如下：
```javascript
const path = require('path')                    //node的path模块

module.exports = {
    mode:'development',                         //开发模式
    entry:'./src/client/index.js',              //入口
    output: {                                   //打包出口
        filename:'index.js',                    //打包后的文件名
        path:path.resolve(__dirname,'public')   //存放到根目录的build文件夹
    },
    module: {
        rules: [{                               //打包规则
           test:   /\.js?$/,                    //对所有js文件进行打包
           loader:'babel-loader',               //使用babel-loader进行打包
           exclude: /node_modules/,             //不打包node_modules中的js文件
           options: {
               presets: ['react','stage-0',['env', {     
                    //loader时额外的打包规则,这里对react,JSX进行转换
                    targets: {
                        browsers: ['last 2versions']   //对主流浏览器最近两个版本进行兼容
                    }
               }]]
           }
       }]
    }
}
```
这种方法就能够简单实现首页的react服务端渲染，过程对应如下图：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710170971094-fb0eca27-1a75-4f0f-aa82-267097640b22.png#averageHue=%23b6ea82&clientId=ud289c408-fb23-4&from=paste&height=380&id=u9df84592&originHeight=759&originWidth=506&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53485&status=done&style=none&taskId=u8150c001-f93c-4b39-bd7b-b25011cdf8c&title=&width=253)
路由的服务端渲染：只需要将路由信息在服务端执行一遍，使用使用StaticRouter来替代BrowserRouter，通过context进行参数传递
```javascript
import express from 'express'
import React from 'react'//引入React以支持JSX的语法
import { renderToString } from 'react-dom/server'//引入renderToString方法
import { StaticRouter } from 'react-router-dom'
import Router from '../Routers'

const app = express()
app.use(express.static('public'));
//使用express提供的static中间件,中间件会将所有静态文件的路由指向public文件夹

app.get('/',(req,res)=>{
    const content  = renderToString((
        //传入当前path
        //context为必填参数,用于服务端渲染参数传递
        <StaticRouter location={req.path} context={{}}>
           {Router}
        </StaticRouter>
    ))
    res.send(`
   <html>
       <head>
           <title>ssr demo</title>
       </head>
       <body>
       <div id="root">${content}</div>
       <script src="/index.js"></script>
       </body>
   </html>
    `)
})


app.listen(3001, () => console.log('Exampleapp listening on port 3001!'))
```
**原理**
整体react服务端渲染原理并不复杂，具体如下：
node server 接收客户端请求，得到当前的请求url 路径，然后在已有的路由表内查找到对应的组件，拿到需要请求的数据，将数据作为 props、context或者store 形式传入组件
然后基于 react 内置的服务端渲染方法 renderToString()把组件渲染为 html字符串，在把最终的 html 进行输出前需要将数据注入到浏览器端
浏览器开始进行渲染和节点对比，然后执行完成组件内事件绑定和一些交互，浏览器重用了服务端输出的 html 节点，整个流程结束。
### 11 react是如何实现更新过程可控的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710171991848-aa07a129-e7be-4dc8-b61a-ae2fde6051eb.png#averageHue=%23faf8f8&clientId=ud289c408-fb23-4&from=paste&height=634&id=ucd48024c&originHeight=1267&originWidth=1017&originalType=binary&ratio=2&rotation=0&showTitle=false&size=97340&status=done&style=none&taskId=u06bd7257-885a-4f8f-8421-a54cdd91e8e&title=&width=508.5)
更新过程的可控主要体现在下面几个方面：

- 任务拆分
- 任务挂起、恢复、终止
- 任务具备优先级

**任务拆分**
在 React Fiber 机制中，它采用"化整为零"的思想，将调和阶段（Reconciler）递归遍历 VDOM 这个大任务分成若干小任务，每个任务只负责一个节点的处理。
**任务挂起、恢复、终止**

- workInProgress tree

workInProgress 代表当前正在执行更新的 Fiber 树。在 render 或者 setState 后，会构建一颗 Fiber 树，也就是 workInProgress tree，这棵树在构建每一个节点的时候会收集当前节点的副作用，整棵树构建完成后，会形成一条完整的副作用链。

- currentFiber tree

currentFiber 表示上次渲染构建的 Filber 树。在每一次更新完成后 workInProgress 会赋值给 currentFiber。在新一轮更新时 workInProgress tree 再重新构建，新 workInProgress 的节点通过 alternate 属性和 currentFiber 的节点建立联系。
在新 workInProgress tree 的创建过程中，会同 currentFiber 的对应节点进行 Diff 比较，收集副作用。同时也会复用和 currentFiber 对应的节点对象，减少新创建对象带来的开销。
也就是说无论是创建还是更新、挂起、恢复以及终止操作都是发生在 workInProgress tree 创建过程中的。workInProgress tree 构建过程其实就是循环的执行任务和创建下一个任务。
**挂起**
当第一个小任务完成后，先判断这一帧是否还有空闲时间，没有就挂起下一个任务的执行，记住当前挂起的节点，让出控制权给浏览器执行更高优先级的任务。
**恢复**
在浏览器渲染完一帧后，判断当前帧是否有剩余时间，如果有就恢复执行之前挂起的任务。如果没有任务需要处理，代表调和阶段完成，可以开始进入渲染阶段。

- 如何判断一帧是否有空闲时间的呢？

使用前面提到的 RIC (RequestIdleCallback) 浏览器原生 API，React 源码中为了兼容低版本的浏览器，对该方法进行了 Polyfill。

- 恢复执行的时候又是如何知道下一个任务是什么呢？

答案是在前面提到的链表。在 React Fiber 中每个任务其实就是在处理一个 FiberNode 对象，然后又生成下一个任务需要处理的 FiberNode。
**终止**
其实并不是每次更新都会走到提交阶段。当在调和过程中触发了新的更新，在执行下一个任务的时候，判断是否有优先级更高的执行任务，如果有就终止原来将要执行的任务，开始新的 workInProgressFiber 树构建过程，开始新的更新流程。这样可以避免重复更新操作。这也是在 React 16 以后生命周期函数 componentWillMount 有可能会执行多次的原因。
**任务具备优先级**
React Fiber 除了通过挂起，恢复和终止来控制更新外，还给每个任务分配了优先级。
具体点就是在创建或者更新 FiberNode 的时候，通过算法给每个任务分配一个到期时间（expirationTime）。在每个任务执行的时候除了判断剩余时间，如果当前处理节点已经过期，那么无论现在是否有空闲时间都必须执行该任务。过期时间的大小还代表着任务的优先级。
任务在执行过程中顺便收集了每个 FiberNode 的副作用，将有副作用的节点通过 firstEffect、lastEffect、nextEffect 形成一条副作用单链表 A1(TEXT)-B1(TEXT)-C1(TEXT)-C1-C2(TEXT)-C2-B1-B2(TEXT)-B2-A。
其实最终都是为了收集到这条副作用链表，有了它，在接下来的渲染阶段就通过遍历副作用链完成 DOM 更新。
这里需要注意，更新真实 DOM（commit） 的这个动作是一气呵成的，不能中断，不然会造成视觉上的不连贯。
### 
12 Fiber 为什么是 react 性能的一个飞跃
**什么是 Fiber**
Fiber 的英文含义是“纤维”，它是比线程（Thread）更细的线，比线程（Thread）控制得更精密的执行模型。在广义计算机科学概念中，Fiber 又是一种协作的（Cooperative）编程模型（协程），帮助开发者用一种【既模块化又协作化】的方式来编排代码。
在 React 中，Fiber 就是 React 16 实现的一套新的更新机制，让 React 的更新过程变得可控，避免了之前采用递归需要一气呵成影响性能的做法。
**React Fiber 中的时间分片**
把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，虽然总时间依然很长，但是在每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。
React Fiber 把更新过程碎片化，每执行完一段更新过程，就把控制权交还给 React 负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。
**Stack Reconciler**
基于栈的 Reconciler，浏览器引擎会从执行栈的顶端开始执行，执行完毕就弹出当前执行上下文，开始执行下一个函数，直到执行栈被清空才会停止。然后将执行权交还给浏览器。由于 React 将页面视图视作一个个函数执行的结果。每一个页面往往由多个视图组成，这就意味着多个函数的调用。
如果一个页面足够复杂，形成的函数调用栈就会很深。每一次更新，执行栈需要一次性执行完成，中途不能干其他的事儿，只能"一心一意"。结合前面提到的浏览器刷新率，JS 一直执行，浏览器得不到控制权，就不能及时开始下一帧的绘制。如果这个时间超过 16ms，当页面有动画效果需求时，动画因为浏览器不能及时绘制下一帧，这时动画就会出现卡顿。不仅如此，因为事件响应代码是在每一帧开始的时候执行，如果不能及时绘制下一帧，事件响应也会延迟。
**Fiber Reconciler**
**链表结构**
在 React Fiber 中用链表遍历的方式替代了 React 16 之前的栈递归方案。在 React 16 中使用了大量的链表。

- 使用多向链表的形式替代了原来的树结构；
```javascript
<div id="A">
A1
<div id="B1">
  B1
  <div id="C1"></div>
</div>
<div id="B2">
  B2
</div>
</div>
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710173125428-d94c5764-d23e-42ae-8aff-5b64f3a05149.png#averageHue=%23fefefe&clientId=u5fe52490-db47-4&from=paste&height=595&id=u42baa34e&originHeight=1190&originWidth=1786&originalType=binary&ratio=2&rotation=0&showTitle=false&size=39531&status=done&style=none&taskId=uf0d7e787-6ede-4f4d-966d-2bde4d663e6&title=&width=893)

- 副作用单链表；

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710173178708-1297751d-d204-4a93-b747-35efabc57dff.png#averageHue=%23fafafa&clientId=u5fe52490-db47-4&from=paste&height=165&id=u6a37561a&originHeight=330&originWidth=1600&originalType=binary&ratio=2&rotation=0&showTitle=false&size=10244&status=done&style=none&taskId=u76316d40-a2e1-47eb-ad3a-d94c00fe3e5&title=&width=800)

- 状态更新单链表；

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710173210957-6eced48f-8d2d-4510-98ca-bb920f1c4785.png#averageHue=%23fafafa&clientId=u5fe52490-db47-4&from=paste&height=169&id=u77ac211f&originHeight=338&originWidth=1596&originalType=binary&ratio=2&rotation=0&showTitle=false&size=10405&status=done&style=none&taskId=u77101460-b3e2-4204-a3c0-1abf8afe3cc&title=&width=798)链表是一种简单高效的数据结构，它在当前节点中保存着指向下一个节点的指针；遍历的时候，通过操作指针找到下一个元素。
链表相比顺序结构数据格式的好处就是：

- 操作更高效，比如顺序调整、删除，只需要改变节点的指针指向就好了。
- 不仅可以根据当前节点找到下一个节点，在多向链表中，还可以找到他的父节点或者兄弟节点。

但链表也不是完美的，缺点就是：

- 比顺序结构数据更占用空间，因为每个节点对象还保存有指向下一个对象的指针。
- 不能自由读取，必须找到他的上一个节点。

React 用空间换时间，更高效的操作可以方便根据优先级进行操作。同时可以根据当前节点找到其他节点，在下面提到的挂起和恢复过程中起到了关键作用。
### 13 react生命周期
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710218286326-0bf3d4c1-de6b-4749-a479-ccd95939c5cd.png#averageHue=%23f9f8f7&clientId=u5fe52490-db47-4&from=paste&height=661&id=u4ca6a7b2&originHeight=1322&originWidth=2190&originalType=binary&ratio=2&rotation=0&showTitle=false&size=68181&status=done&style=none&taskId=ua3797c93-ad55-4fe6-ab1f-07878bcc38e&title=&width=1095)
**挂载**
当组件实例被创建并插入 DOM 中时，其生命周期调用顺序如下：

- constructor()
- static getDerivedStateFromProps()
- render()
- componentDidMount()

**更新**
当组件的 props 或 state 发生变化时会触发更新。组件更新的生命周期调用顺序如下：

- static getDerivedStateFromProps()
- shouldComponentUpdate()
- render()
- getSnapshotBeforeUpdate()
- componentDidUpdate()

**卸载**
当组件从 DOM 中移除时会调用如下方法：

- componentWillUnmount()

**错误处理**
渲染过程，生命周期，或子组件的构造函数中抛出错误时，会调用如下方法：

- static getDerivedStateFromError()
- componentDidCatch()

### 14 常用的 react hooks 有哪些
React 提供了一系列的 Hooks，用于在函数组件中添加和管理状态、副作用等功能。
以下是一些常用的 React Hooks：

1. useState：用于在函数组件中添加状态管理。
2. useEffect：用于处理副作用操作（如数据获取、订阅、事件监听等）。
3. useContext：用于在组件树中获取和使用共享的上下文。
4. useReducer：用于管理复杂状态逻辑的替代方案，类似于 Redux 的 reducer。
5. useCallback：用于缓存回调函数，以便在依赖未变化时避免重复创建新的函数实例。
6. useMemo：用于缓存计算结果，以便在依赖未变化时避免重复计算。
7. useRef：用于在函数组件之间保存可变的值，并且不会引发重新渲染。
8. useLayoutEffect：与 useEffect 类似，但在浏览器完成绘制之前同步执行。
9. useImperativeHandle：用于自定义暴露给父组件的实例值或方法。
10. useDebugValue：用于在开发者工具中显示自定义的钩子相关标签。

### 15 react中实现父组件调用子组件的方法
要实现父组件调用子组件中的方法，需要通过以下步骤进行操作：

1. 在子组件中，创建一个公开的方法。这可以通过在子组件类中定义一个方法，或者使用 React Hooks 中的 useImperativeHandle 来实现。
   - 如果是类组件，可以在子组件类中定义一个方法，并将其挂载到实例上。例如：
```javascript
class ChildComponent extends React.Component {
  childMethod() {
    // 子组件中需要执行的操作
  }

  render() {
    // 子组件的渲染逻辑
  }
}
```

   - 如果是函数式组件，可以使用 useImperativeHandle Hook 将指定的方法暴露给父组件。例如：
```javascript
import { forwardRef, useImperativeHandle } from 'react';

function ChildComponent(props, ref) {
  useImperativeHandle(ref, () => ({
    childMethod() {
      // 子组件中需要执行的操作
    }
  }));

  // 子组件的渲染逻辑
}

export default forwardRef(ChildComponent);

```

2. 在父组件中，首先引用或创建对子组件的引用。可以使用 ref 对象来保存对子组件的引用。
- 如果是类组件，可以使用 createRef 创建一个 ref 对象，并将其传递给子组件的 ref prop。例如：
```javascript
class ParentComponent extends React.Component {
  constructor(props) {
    super(props);
    this.childRef = React.createRef();
  }

  handleClick() {
    // 调用子组件的方法
    this.childRef.current.childMethod();
  }

  render() {
    return (
      <div>
        <ChildComponent ref={this.childRef} />
        <button onClick={() => this.handleClick()}>调用子组件方法</button>
      </div>
    );
  }
}
```
如果是函数式组件，可以使用 useRef 创建一个 ref 对象，并将其传递给子组件的 ref prop。例如：
```javascript
function ParentComponent() {
  const childRef = useRef(null);

  const handleClick = () => {
    // 调用子组件的方法
    childRef.current.childMethod();
  };

  return (
    <div>
      <ChildComponent ref={childRef} />
      <button onClick={handleClick}>调用子组件方法</button>
    </div>
  );
}
```
通过以上步骤，父组件就能够成功调用子组件中暴露的方法了。请注意，在函数式组件中，需要使用 forwardRef 来包裹子组件，并通过 ref 参数来定义暴露的方法。
### 16 useref、ref和 forwardref的区别
useRef 和 ref 都是 React 中用于操作 DOM 元素或自定义组件实例的工具，而 forwardRef 则是用于访问嵌套子组件中的 DOM 元素或自定义组件实例。
它们之间的区别如下：

1. useRef 是一个 hook 函数，可以在函数组件中使用；ref 是一个对象属性，只能在类组件中使用。
2. useRef 返回一个可变的 ref 对象，可以在组件的整个生命周期内保持不变，也就是说不会因为重新渲染而改变。而 ref 每次渲染都会被重新创建。
3. useRef 主要用于存储和更新组件内部状态，以及操作 DOM 元素。而 ref 主要用于获取 DOM 元素或自定义组件实例。
4. forwardRef 是用于将 ref 属性“向下传递”给一个函数式子组件或自定义组件的工具函数。它允许父组件调用子组件中的 DOM 元素或自定义组件实例。

综上所述，useRef 和 ref 都是用于操作 DOM 元素或自定义组件实例的工具，与之相比，forwardRef 则是一个更高级的工具，用于处理专门的情况，即访问嵌套子组件中的 DOM 元素或自定义组件实例。
### 17 useContext的理解
context（上下文）可以看成是扩大版的props，它可以将全局的数据通过provider接口传递value给局部的组件，让包围在provider中的局部组件可以获取到全局数据的读写接口。
用法：

- 用creacteContext创建一个上下文     const X = createContext(); 
- 设置provider并通过value接口传递state     <X.Provider value={{ state, setState }}>
- 局部组件从value接口中传递的数据对象中获取读写接口    const { state, setState } = useContext(X);
```javascript
import React, { createContext, useContext, useState } from "react";
const initialState = { m: 100, n: 50 }; // 定义初始state
const X = createContext(); // 创建Context
let a = 0;

export default function App() {
  console.log(`render了${a}次`);//用来检查执行App函数多少次
  const [state, setState] = useState(initialState); // 创建state读写接口
  a += 1;
  return (
    <X.Provider value={{ state, setState }}> // 通过provider提供value给包围里内部组件，只有包围里的组件才有效
      <Father></Father>
    </X.Provider>
  );
}

const Father = (props) => {
  const { state, setState } = useContext(X);//拿到 名字为X的上下文的value，用两个变量来接收读写接口
  const addN = () => {
    setState((state) => {
      return { ...state, n: state.n + 1 };
    });
  };
  const addM = () => {
    setState((state) => {
      return { ...state, m: state.m + 1 };
    });
  };
  return (
    <div>
      爸爸组件
      <div>n:{state.n}</div>
      <Child />
      <button onClick={addN}>设置n</button>
      <button onClick={addM}>设置m</button>
    </div>
  );
};

const Child = (props) => {
  const { state } = useContext(X); // 读取state
  return (
    <div>
      儿子组件
      <div>m:{state.m}</div>
    </div>
  );
};
```
> tips：注意到最上层的变量a没？这是我用来测试的，我发现点击按钮后会触发App函数并更新页面，说明react下使用context来修改数据的时候，都会重新进行全局执行，而不是数据响应式的。

### 18 对 useMemo的理解
在class的时代，我们一般是通过pureComponent来对数据进行一次浅比较，引入Hook特性后，我们可以使用Memo进行性能提升。
```javascript
import React, { useCallback, useMemo, useState } from "react";
import ReactDOM from "react-dom";

import "./styles.css";

function App() {
  const [n, setN] = useState(0);
  const [m, setM] = useState({ m: 1 });
  console.log("执行最外层盒子了");
  const addN = useMemo(() => {
    return () => {
      setN(n + 1);
    };
  }, [n]);
  const addM = useCallback(() => {
    setM({ m: m.m + 1 });
  }, [m]);
  return (
    <>
      <div>
        最外层盒子
        <Child1 value={n} click={addN} />
        <Child2 value={m} click={addM} />
        <button onClick={addN}>n+1</button>
        <button onClick={addM}>m+1</button>
      </div>
    </>
  );
}

const Child1 = React.memo((props) => {
  console.log("执行子组件1了");
  return <div>子组件1上的n：{props.value}</div>;
});

const Child2 = React.memo((props) => {
  console.log("执行子组件2了");
  return <div>子组件2上的m：{props.value.m}</div>;
});

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

- 使用memo可以帮助我们优化性能，让react没必要执行不必要的函数
- **由于复杂数据类型的地址可能发生改变**，于是传递给子组件的props也会发生变化，这样还是会执行不必要的函数，所以就用到了useMemo这个api
- useCallback是useMemo的语法糖
- 父组件 re-render子组件也会 re-render，优化方法：组件 memo属性 memo。
#### useMemo和 useCallback的使用场景
useMemo 和 useCallback 是 React 的内置 Hook，通常作为优化性能的手段被使用。他们可以用来缓存函数、组件、变量，以避免两次渲染间的重复计算。但是实践过程中，他们经常被过度使用：担心性能的开发者给每个组件、函数、变量、计算过程都套上了 memo，以至于它们在代码里好像失控了一样，无处不在。
我们先从 useMemo/useCallback 的目的说起。
**为什么使用 useMemo 和 useCallback**
使用 memo 通常有三个原因：

1. ✅ 防止不必要的 effect。
2. ❗️防止不必要的 re-render。
3. ❗️防止不必要的重复计算。

后两种优化往往被误用，导致出现大量的无效优化或冗余优化。下面详细介绍这三个优化方式。
**1 防止不必要的 effect**
如果一个值被 useEffect 依赖，那它可能需要被缓存，这样可以避免重复执行 effect。
```javascript
const Component=()=>{
  // 在 re-renders 之间缓存 a 的引用
  const a = useMemo(()=>({ test:1}),[]);
  
  useEffect(()=>{
    // 只有当 a 的值变化时，这里才会被触发
    doSomething();
  },[a]);
  
  // the rest of the code
};
```
useCallback 同理：
```javascript
constComponent=()=>{
  // 在 re-renders 之间缓存 fetch 函数
  const fetch = useCallback(()=>{
    console.log('fetch some data here');
  },[]);
  
  useEffect(()=>{
    // 仅fetch函数的值被改变时，这里才会被触发
    fetch();
  },[fetch]);
  
  // the rest of the code
};
```
当变量直接或者通过依赖链成为 useEffect 的依赖项时，那它可能需要被缓存。这是 useMemo 和 useCallback 最基本的用法。
**2 防止不必要的 re-render**
1. 组件什么时候会 re-render
三种情况：

1. 当本身的 props 或 state 改变时。
2. Context value 改变时，使用该值的组件会 re-render。
3. 当父组件重新渲染时，它所有的子组件都会 re-render，形成一条 re-render 链。

第三个 re-render 时机经常被开发者忽视，**导致代码中存在大量的无效缓存**。
2. 如何防止子组件 re-render
必须同时满足以下两个条件，子组件才不会 re-render：

- 子组件自身被缓存。
- 子组件所有的 prop 都被缓存。

3. 如何判断子组件需要缓存
我们已经了解，为了防止子组件 re-render，需要以下成本：

1. **开发者工作量的增加**： 一旦使用缓存，就必须保证组件本身以及所有 props 都缓存，后续添加的所有 props 都要缓存。
2. **代码复杂度和可读性的变化**：代码中出现大量缓存函数，这会增加代码复杂度，并降低易读性。

除此之外还有另外一个成本：**性能成本**。 组件的缓存是在初始化时进行，虽然每个组件缓存的性能耗费很低，通常不足1ms，但大型程序里成百上千的组件如果同时初始化缓存，成本可能会变得很可观。
所以局部使用 memo，比全局使用显的更优雅、性能更好，坏处是需要开发者主动去判断是否需要缓存该子组件。
**3 防止不必要的重复计算**
useMemo 的基本作用是，避免在每次渲染时都进行高开销的计算。
**高开销的计算其实极少出现。相比计算，组件渲染才是性能的瓶颈****，应该把 useMemo 用在程序里渲染昂贵的组件上**，而不是数值计算上。当然，除非这个计算真的很昂贵，比如阶乘计算。
至于为什么不给所有的组件都使用 useMemo，上文已经解释了。useMemo 是有成本的，它会增加整体程序初始化的耗时，并不适合全局全面使用，它更适合做局部的优化。
**结论**
讲到这里我们可以总结出 useMemo/useCallback 使用准则了：

1. **大部分的 useMemo 和 useCallback 都应该移除**，他们可能没有带来任何性能上的优化，反而增加了程序首次渲染的负担，并增加程序的复杂性。
2. 使用 useMemo 和 useCallback 优化子组件 re-render 时，**必须同时满足以下条件才有效**。
   1. 子组件已通过 React.memo 或 useMemo 被缓存
   2. 子组件所有的 prop 都被缓存
3. **不推荐默认给所有组件都使用缓存**，大量组件初始化时被缓存，可能导致过多的内存消耗，并影响程序初始化渲染的速度。
#### useMemo、useCallback区别
在 React 中，useMemo 和 useCallback 都是用来优化性能的钩子函数，但它们的用途和作用稍有不同。

1. **useMemo**: useMemo 的主要作用是在组件重新渲染时，用来缓存计算结果，以避免不必要的重复计算。它接收两个参数：一个回调函数和一个依赖数组。回调函数用于进行计算，而依赖数组用于指定在数组中列出的依赖项发生变化时，才重新计算并返回新的值，否则会返回上一次缓存的值。
```javascript
const memoizedValue =useMemo(()=>{
  // 进行耗时的计算
  return someValue;
},[dependency1, dependency2]);
```
在上面的示例中，只有当 dependency1 或者 dependency2 发生变化时，useMemo 才会重新计算并返回新的值，否则会复用之前的值。

1. **useCallback**: useCallback 的作用是在组件重新渲染时，返回一个记忆化的回调函数，以避免不必要的函数重新创建。它也接收两个参数：一个回调函数和一个依赖数组。当依赖项发生变化时，会返回一个新的回调函数，否则会复用之前的回调函数。
```javascript
const memoizedCallback =useCallback(()=>{
  // 处理事件的回调函数
},[dependency1, dependency2]);
```
在这个示例中，只有当 dependency1 或者 dependency2 发生变化时，useCallback 才会返回一个新的回调函数，否则会返回之前的回调函数。
总结区别：

- useMemo 主要用于缓存计算结果，适用于任何需要缓存值的场景。
- useCallback 主要用于缓存回调函数，适用于需要传递给子组件的事件处理函数，以避免不必要的重新渲染。

**另外，在大多数情况下，你不必在每个函数组件中都使用 ****useMemo**** 或 ****useCallback****。**
**只有当你在性能测试中发现了性能问题，或者在特定情况下需要优化函数的创建和计算时，再考虑使用这些钩子。**
### 19 useRef 常见用法
```javascript
import { useRef} from "react";
import React from "react";
export default function App() {
  const ref1 = useRef();
  const ref2 = useRef(2021);
  console.log("if rendered, I am showing");
  console.log(ref1, ref2);
  return (
    <div>
      <h2>{ref1.current}</h2>
      <h2>{ref2.current}</h2>
    </div>
  );
}

// 值为undefined 页面展示空白
// 2021
```
```javascript
import { useRef } from "react";
import "./styles.css";
const App = () => {
  const countRef = useRef(0);
  console.log("render");
  return (
    <div className="App">
      <h2>count: {countRef.current}</h2>
      <button
        onClick={() => {
          countRef.current = countRef.current + 1;
          console.log(countRef.current);
        }}
      >
        increase count
      </button>
    </div>
  );
};
// 控制台打印1
// 2
// 3
//但是页面显示始终是 0
```
我们的目标是定义一个名为countRef的ref，用0初始化该值，并在每次单击按钮时该计数器变量会增加。呈现的计数值应该更新。令人惊讶的是:页面上的计数值没有更新,但是:控制台的输出证明了当前属性是保存了正确的更新的; 也就是说**useRef不会触发重新渲染。**
那么useRef应该怎么用？
其实它与其他触发重新渲染的 Hook 结合起来很方便，例如useState,useReducer和useContext。
**通过该ref属性，React 提供了对 React 组件或 HTML 元素的直接访问**
```javascript
import { useRef, useState } from "react";
import React from "react";
export default function App() {
  const [value, setValue] = useState("");
  const valueRef = useRef();
  console.log("render");
  
  const handleClick = () => {
    console.log(valueRef);
    setValue(valueRef.current.value);
  };
  
  return (
    <div className="App">
      <h4>Value: {value}</h4>
      <input ref={valueRef} />
      <button onClick={handleClick}>click</button>
    </div>
  );
}
```
```javascript
import { useRef, useState, useEffect } from "react";
import React from "react";
export default function App() {
  const inputRef = useRef();
  console.log("render");
  useEffect(() => {
    console.log("running in useEffect");
    inputRef.current.focus()//注意有无这句话刷新前后的区别
  }, []);
  return (
    <div className="App">
      <input ref={inputRef} placeholder="this is input" />
    </div>
  );
}
```
需要**上一个渲染周期的状态值**时也要运用到useRef。
```javascript
import { useRef, useState, useEffect } from "react";
import React from "react";
export default function App() {
  console.log("render");
  const [count, setCount] = useState(0);
  //获取上一个值(在上次渲染时传递给钩子的)
  const ref = useRef();
  
  // 存储ref中的current值
  useEffect(() => {
    console.log("useEffect");
    ref.current = count;
  }, [count]); // count变化时才会重新渲染
  
  return (
    <div className="App">
      <h1>
        Now: {count}, before: {ref.current}
      </h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
// 初始化时count为0，before为空（先 render 再 didmount），mount 之后 ref为 0 了
// 点击更新：count 变为 1， before 显示为 0
```
### 20 useEffect
```javascript
useEffect(() => {
  // 只在挂载和卸载时执行
}, []);

useEffect(() => {
  // 在挂载、依赖列表变化及卸载时执行
}, [dep1, dep2]);
```
下面是这两种情况的总结：

- 当传递空数组 [] 时，useEffect 只会在组件挂载和卸载时调用一次，不会对组件进行重新渲染。
- 当传递依赖数组时，useEffect 会在组件挂载和依赖项更新时调用，每次更新时都会检查依赖项列表是否有变化，如果有变化则重新执行。

如果 useEffect 中使用了闭包函数，则应该确保所有引用的变量都在依赖项中被显示声明，否则可能会导致不必要的重新渲染或者无法获取最新的状态。
### 21 react hook 闭包陷阱
**闭包**（closure）是一个函数以及其捆绑的周边环境状态（lexical environment，词法环境）的引用的组合。换而言之，闭包让开发者可以从内部函数访问外部函数的作用域。在 JavaScript 中，闭包会随着函数的创建而被同时创建。
React Hooks 的**闭包陷阱**是指在使用 Hooks 时可能遇到的一种常见问题，特别是在使用 useState 或 useEffect 等 Hook 时。这个问题通常是由于 JavaScript 中闭包的特性所引起的。
当你在函数组件中使用 Hooks 时，每次组件重新渲染时，函数组件内部的所有东西都会被重新创建。这包括函数内部的任何变量或函数。如果在某个 Hook 的回调函数中使用了某些在该 Hook 之外声明的变量，那么这些变量将会形成闭包，而且由于闭包的特性，它们会捕获到每次渲染时的那个特定值，而不是期望的最新值。
**useState中的闭包陷阱**
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setTimeout(() => {
      setCount(count + 1);
    }, 1000);
  };
  const handleReset = () => {
    setCount(0);
  };
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```
每次点击触发 handleClick 函数，1s 后 setCount 会将 count 值加 1，在这 1s 内，无论我们点击多少次 Increment 按钮，count 值都只会加 1。这是因为 setCount 所接收的 count 值是在闭包中被缓存的 count 值，这个值是始终不变的。
解决方案：使用React Hooks提供的更新函数的形式来更新状态
```javascript
const handleClick = () => {
  setTimeout(() => {
    setCount(currentCount => currentCount + 1);
  }, 1000);
};
```
在这个版本的handleClick函数中，我们使用了setCount的更新函数形式。这个函数会接收count的当前值作为参数，这样我们就可以在闭包中使用这个值，而不需要担心它被缓存。
在React中，useState hook返回的更新state的函数，即setCount函数，**可以接受一个回调函数作为参数**。这个回调函数会接受当前state的值作为参数，然后返回一个新的state值。React会使用这个新的state值来更新组件的状态。
在上述代码中，通过使用回调函数的形式来更新count的值，这个回调函数会接受 currentCount 作为参数，即当前的count值，而不是从外部直接引用count变量。这样，**即使在闭包中使用了count变量，也不会受到影响，因为回调函数内部的 currentCount 变量是函数作用域内的局部变量**，不会受到外部变量的影响。这种方式可以避免闭包陷阱，保证组件可以正确更新状态。
**实现每隔一秒加一：**
```javascript
import React, { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // 这里的 count 将会是每次渲染时的最新值
      console.log(count);
      setCount(prevCount => prevCount + 1); // 正确的更新方式
    }, 1000);

    return () => clearInterval(intervalId);
  }, [count]);

  return <div>{count}</div>;
}
```
**useEffect中的闭包陷阱**
在useEffect中使用闭包的问题则是因为useEffect中的函数是在每次组件更新时都会执行一次。如果我们在useEffect中使用闭包，那么这个闭包中的变量值也会被缓存，这样就可能会导致一些问题。
```javascript
function App() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
    }, 1000);
    return () => clearInterval(timer);
  }, []);
  
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```
在这个例子中，我们使用了 useState 和 useEffect Hooks。在 useEffect 回调函数内部，我们使用了一个 setInterval 函数来输出 count 状态变量。然而，由于 useEffect 只会在组件首次渲染时执行一次，因此闭包中的 count 变量始终是首次渲染时的变量，而不是最新的值。
解决如下：
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(timer);
}, [count]);
```
此处依赖 了 count，是count改变的时候新建了定时器了，逻辑上不太好。本意相当于挂载的时候新建一个定时器，定时器每隔一秒输出一次，setCount了之后，定时器输出最新的 count，此时用下文的useRef理解上更好一些。
再例如
```javascript
function App(){
    const [count, setCount] = useState(1);
    useEffect(()=>{
        setInterval(()=>{
            console.log(count)
        }, 1000)
    }, [])
}
```
在这个定时器里面去打印 count 的值，会发现，不管在这个组件中的其他地方使用 setCount 将 count 设置为任何值，还是设置多少次，打印的都是1。
一个熟悉的闭包场景
```javascript
for ( var i=0; i<5; i++ ) {
    setTimeout(()=>{
        console.log(i)
    }, 0)
}
```
解决
```javascript
for ( var i=0; i<5; i++ ) {
   (function(i){
         setTimeout(()=>{
            console.log(i)
        }, 0)
   })(i)
}
```
这个原理其实就是使用闭包，定时器的回调函数去引用立即执行函数里定义的变量，形成闭包保存了立即执行函数执行时 i 的值，异步定时器的回调函数才如我们想要的打印了顺序的值。
其实，useEffect 的那个场景的原因，跟这个简直是一样的，useEffect 闭包陷阱场景的出现，是 react 组件更新流程以及 useEffect 的实现的自然而然的结果。
#### hooks原理
Fiber对象上有一个 memoizedState 用于存放组件的 state。ok，现在看 hooks 所针对的 FunctionComponnet。 无论开发者怎么折腾，**一个对象都只能有一个 state 属性或者 memoizedState 属性**。谁知道可爱的开发者们会在 FunctionComponent 里写上多少个 useState，useEffect 等等 ? 所以，react用了**链表**这种数据结构来存储 FunctionComponent 里面的 hooks。比如
```javascript
function App(){
    const [count, setCount] = useState(1)
    const [name, setName] = useState('chechengyi')
    useEffect(()=>{

    }, [])
    const text = useMemo(()=>{
        return 'ddd'
    }, [])
}
```
在组件第一次渲染的时候，为每个hooks都创建了一个对象
```javascript
type Hook = {
  memoizedState: any,
  baseState: any,
  baseUpdate: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null,
  next: Hook | null,
};
```
最终形成了一个链表。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710385280351-53e81bb6-bfc4-4f64-b527-14d44ee43700.png#averageHue=%23f4e7bf&clientId=u8cef728a-2d40-4&from=paste&height=26&id=u0fc6fe96&originHeight=51&originWidth=571&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1679&status=done&style=none&taskId=uf04b1649-b1bb-4465-9216-9e227416224&title=&width=285.5)
这个对象的memoizedState属性就是用来存储组件上一次更新后的 state,next毫无疑问是指向下一个hook对象。在组件更新的过程中，hooks函数执行的顺序是不变的，就可以根据这个链表拿到当前hooks对应的Hook对象，函数式组件就是这样拥有了state的能力。当前，具体的实现肯定比这三言两语复杂很多。
所以，知道为什么不能将hooks写到**if else语句中了把？因为这样可能会导致顺序错乱**，导致当前hooks拿到的不是自己对应的Hook对象。
```javascript
function App(){
  const [count, setCount] = useState(1);

  useEffect(()=>{
      setInterval(()=>{
          console.log(count)
      }, 1000)
  }, [])
  
  function click(){ setCount(count+1) }
  
  return (
    <div>
        <h1>{count}</h1>
        <button onClick={click}>click</button>
    </div>
  )
}
```
好，开动脑袋开始想象起来，组件第一次渲染执行 App()，执行 useState 设置了初始状态为1，所以此时的 count 为1。然后执行了 useEffect，回调函数执行，设置了一个定时器每隔 1s 打印一次 count。
接着想象如果 click 函数被触发了，调用 setCount(2) 肯定会触发react的更新，更新到当前组件的时候也是执行 App()，之前说的链表已经形成了哈，此时 useState 将 Hook 对象 上保存的状态置为2， 那么此时 count 也为2了。然后在执行 useEffect 由于依赖数组是一个空的数组，所以此时回调并不会被执行。
ok，这次更新的过程中根本就没有涉及到这个定时器，这个定时器还在坚持的，默默的，每隔1s打印一次 count。 注意这里打印的 count ，是组件第一次渲染的时候 App() 时的 count， count的值为1，**因为在定时器的回调函数里面被引用了，形成了闭包一直被保存**。
> **依赖数组里写上值  可行**

```javascript
export default function App(){
    const [count, setCount] = useState(1);
  
    useEffect(()=>{
        const intervalA = setInterval(()=>{
            console.log(count)
        }, 1000);

        return () => {
            clearTimeout(intervalA);
        }
    }, [count])
  
    function click(){ setCount(count+1) }
  
    return (
        <div>
            <h1>{count}</h1>
            <button onClick={click}>click</button>
        </div>
    )
}
```
```javascript
// 不依赖 count, 修改同一个对象也可
 const [count, setCount] = useState({sum: 1});

  useEffect(()=>{
      setInterval(()=>{
          console.log(count.sum)
      }, 1000)
  }, [])

function click(){ 
    setCount((prevState)=> {
        var nowObj = Object.assign(prevState, {
          sum: prevState.sum + 1
        })
        return nowObj
      })
}
```
**2 难道真的只能在依赖数组里写上的值，才能拿到新鲜的值？**
仿佛都习惯性都去认为，只有在依赖数组里写上我们所需要的值，才能在更新的过程中拿到最新鲜的值。那么看一下这个场景：
```javascript
function App() {
  return <Demo1 />
}

function Demo1(){
  const [num1, setNum1] = useState(1)
  const [num2, setNum2] = useState(10)

  const text = useMemo(()=>{
    return `num1: ${num1} | num2:${num2}`
  }, [num2])

  function handClick(){
    setNum1(2)
    setNum2(20)
  }

  return (
    <div>
      {text}
      <div><button onClick={handClick}>click!</button></div>
    </div>
  )
}
```
text 是一个 useMemo ，它的依赖数组里面只有num2，没有num1，却同时使用了这两个state。当点击button 的时候，num1和num2的值都改变了。
为什么呢，再说一遍，这个依赖数组存在的意义，是react为了判定，在**本次更新**中，是否需要执行其中的回调函数，这里依赖了的num2，而num2改变了。回调函数自然会执行， 这时形成的闭包引用的就是最新的num1和num2，所以，自然能够拿到新鲜的值。**问题的关键，在于回调函数执行的时机，闭包就像是一个照相机，把回调函数执行的那个时机的那些值保存了下来。**之前说的定时器的回调函数我想就像是一个从1000年前穿越到现代的人，虽然来到了现代，但是身上的血液、头发都是1000年前的。
**3 为什么使用useRef能够每次拿到新鲜的值？**
大白话说：因为初始化的 useRef 执行之后，返回的都是同一个对象。
```javascript
var A = {name: 'chechengyi'}
var B = A
B.name = 'baobao'
console.log(A.name) // baobao
```
对，这就是这个场景成立的最根本原因。
也就是说，在组件每一次渲染的过程中。 ref = useRef() 所返回的都是同一个对象，每次组件更新所生成的ref指向的都是同一片内存空间， 那么当然能够每次都拿到最新鲜的值了。这个同一个对象将这些个被保存于不同闭包时机的变量联系了起来。
所以，提出一个合理的设想。只要我们能保证每次组件更新的时候，useState 返回的是同一个对象的话？我们也能绕开闭包陷阱这个情景。
```javascript
function App() {
  return <Demo2 />
}

function Demo2(){
  const [obj, setObj] = useState({name: 'chechengyi'})

  useEffect(()=>{
    setInterval(()=>{
      console.log(obj)
    }, 2000)
  }, [])

  function handClick(){
    setObj((prevState)=> {
      var nowObj = Object.assign(prevState, {
        name: 'baobao',
        age: 24
      })
      console.log(nowObj == prevState)
      return nowObj
    })
  }
  return (
    <div>
      <div>
        <span>name: {obj.name} | age: {obj.age}</span>
        <div><button onClick={handClick}>click!</button></div>
      </div>
    </div>
  )
}
```
简单说下这段代码，在执行 setObj 的时候，传入的是一个函数。这种用法就不用我多说了把？然后 Object.assign 返回的就是传入的第一个对象。总儿言之，就是在设置的时候返回了同一个对象。
执行这段代码发现，确实点击button后，定时器打印的值也变成了：
```javascript
{
    name: 'baobao',
    age: 24 
}
```
**用useRef解决方法：**
```javascript
export default function App(){
    const [count, setCount] = useState(1);
    const ref = useRef(count);
  
    useEffect(()=>{
        const intervalA = setInterval(()=>{
            console.log(ref.current)
        }, 1000);

        return () => {
            clearTimeout(intervalA);
        }
    }, [])
  
    function click(){ 
        setCount(count + 1);
        ref.current = count + 1;
    }
  
    return (
        <div>
            <h1>{count}</h1>
            <button onClick={click}>click</button>
        </div>
    )
}
```
### 22 useReducer
useReducer 是 React Hooks 中的一个函数，用于管理和更新组件的状态。它可以被视为 useState 的一种替代方案，适用于处理更复杂的状态逻辑。
使用 useReducer，我们首先需要定义一个 reducer 函数，该函数接收当前状态（state）和动作（action）作为参数，并返回新的状态。在组件中，可以通过调用 useReducer 来创建一个状态值以及与之配套的派发（dispatch）方法。
```javascript
import { useReducer } from 'react';

const initialState = {
  count: 0,
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unsupported action type');
  }
};

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  const increment = () => {
    dispatch({ type: 'increment' });
  };

  const decrement = () => {
    dispatch({ type: 'decrement' });
  };

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```
上面的代码定义了一个初始状态对象 initialState 和一个 reducer 函数 reducer。reducer 接收当前状态和动作类型，然后根据动作类型返回新的状态对象。
组件中使用 useReducer 创建了一个名为 state 的状态值和一个 dispatch 方法。通过调用 dispatch 方法，我们可以向 reducer 发送一个动作，从而触发状态的更新。在示例中，点击 "Increment" 或 "Decrement" 按钮会分别派发 increment 和 decrement 动作。
最后，组件渲染时会展示当前计数器的值以及两个按钮，用于增加或减少计数器的值。
相比于 useState，useReducer 在处理复杂状态逻辑时更有优势，因为它允许我们将状态更新的逻辑封装在 reducer 函数中，并根据不同的动作类型执行相应的逻辑。这样可以使代码更具可读性和可维护性，并且更容易进行状态追踪和调试。
### 23 如何让 useEffect支持 async await
大家在使用 useEffect 的时候，假如回调函数中使用 async...await... 的时候，会报错如下。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710471431392-11cc81ea-d10b-40af-8e85-2d17a6ce0f7e.png#averageHue=%23fee4e3&clientId=u8cef728a-2d40-4&from=paste&height=399&id=u725958eb&originHeight=798&originWidth=1366&originalType=binary&ratio=2&rotation=0&showTitle=false&size=68919&status=done&style=none&taskId=u1d84ab48-e668-4c62-a620-abe670f2e72&title=&width=683)
看报错，我们知道 effect function 应该返回一个销毁函数（return返回的 cleanup 函数），如果 useEffect 第一个参数传入 async，返回值则变成了 Promise，会导致 react 在调用销毁函数的时候报错。
**React 为什么要这么做？**
useEffect 作为 Hooks 中一个很重要的 Hooks，可以让你在函数组件中执行副作用操作。
它能够完成之前 Class Component 中的生命周期的职责。它返回的函数的执行时机如下：

- 首次渲染不会进行清理，会在下一次渲染，清除上一次的副作用。
- 卸载阶段也会执行清除操作。

不管是哪个，我们都不希望这个返回值是异步的，这样我们无法预知代码的执行情况，很容易出现难以定位的 Bug。
所以 React 就直接限制了不能 useEffect 回调函数中不能支持 async...await...
**useEffect 怎么支持 async...await...**
竟然 useEffect 的回调函数不能使用 async...await，那我直接在它内部使用。
做法一：创建一个异步函数（async...await 的方式），然后执行该函数。
```javascript
useEffect(() => {
  const asyncFun = async () => {
    setPass(await mockCheck());
  };
  asyncFun();
}, []);
```
做法二：也可以使用 IIFE，如下所示：
```javascript
useEffect(() => {
  (async () => {
    setPass(await mockCheck());
  })();
}, []);
```
**自定义 hooks**
既然知道了怎么解决，我们完全可以将其封装成一个 hook，让使用更加的优雅。我们来看下 ahooks 的 useAsyncEffect，它支持所有的异步写法，包括 generator function。
思路跟上面一样，入参跟 useEffect 一样，一个回调函数（不过这个回调函数支持异步），另外一个依赖项 deps。**内部还是 useEffect，将异步的逻辑放入到它的回调函数里面。**
```javascript
function useAsyncEffect(
  effect: () => AsyncGenerator<void, void, void> | Promise<void>,
  // 依赖项
  deps?: DependencyList,
) {
  
  // 判断是 AsyncGenerator
  function isAsyncGenerator(
    val: AsyncGenerator<void, void, void> | Promise<void>,
  ): val is AsyncGenerator<void, void, void> {
    // Symbol.asyncIterator: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator
    // Symbol.asyncIterator 符号指定了一个对象的默认异步迭代器。如果一个对象设置了这个属性，它就是异步可迭代对象，可用于for await...of循环。
    return isFunction(val[Symbol.asyncIterator]);
  }

  useEffect(() => {
    const e = effect();
    // 这个标识可以通过 yield 语句可以增加一些检查点
    // 如果发现当前 effect 已经被清理，会停止继续往下执行。
    let cancelled = false;
    // 执行函数
    async function execute() {
      // 如果是 Generator 异步函数，则通过 next() 的方式全部执行
      if (isAsyncGenerator(e)) {
        while (true) {
          const result = await e.next();
          // Generate function 全部执行完成
          // 或者当前的 effect 已经被清理
          if (result.done || cancelled) {
            break;
          }
        }
      } else {
        await e;
      }
    }
    execute();
    return () => {
      // 当前 effect 已经被清理
      cancelled = true;
    };
  }, deps);
}
```
async...await 我们之前已经提到了，重点看看实现中变量 cancelled 的实现的功能。 它的作用是**中断执行**。
试想一下，有一个场景，用户频繁的操作，可能现在这一轮操作 a 执行还没完成，就已经开始开始下一轮操作 b。这个时候，操作 a 的逻辑已经失去了作用了，那么我们就可以停止往后执行，直接进入下一轮操作 b 的逻辑执行。这个 cancelled 就是用来取消当前正在执行的一个标识符。
**还可以支持 useEffect 的清除机制么？**
你可能会觉得，我们将 effect(useAsyncEffect 的回调函数)的结果，放入到 useAsyncEffect 中不就可以了？
实现最终类似如下：
```javascript
function useAsyncEffect(effect: () => Promise<void | (() => void)>, dependencies?: any[]) {
  return useEffect(() => {
    const cleanupPromise = effect();
    return () => { cleanupPromise.then(cleanup => cleanup && cleanup()) }
  }, dependencies)
}
```
github有大神认为这种**延迟清除机制**是不对的，应该是一种**取消机制**。否则，在钩子已经被取消之后，回调函数仍然有机会对外部状态产生影响。他的实现和例子我也贴一下，跟 useAsyncEffect 其实思路是一样的，如下：
```javascript
function useAsyncEffect(effect: (isCanceled: () => boolean) => Promise<void>, dependencies?: any[]) {
  return useEffect(() => {
    let canceled = false;
    effect(() => canceled);
    return () => { canceled = true; }
  }, dependencies)
}
```
Demo
```javascript
useAsyncEffect(async (isCanceled) => {
  const result = await doSomeAsyncStuff(stuffId);
  if (!isCanceled()) {
    // TODO: Still OK to do some effect, useEffect hasn't been canceled yet.
  }
}, [stuffId]);
```
其实归根结底，**我们的清除机制不应该依赖于异步函数，否则很容易出现难以定位的 bug**。
**总结与思考**
由于 useEffect 是在函数式组件中承担执行副作用操作的职责，它的返回值的执行操作应该是可以预期的，而不能是一个异步函数，所以不支持回调函数 async...await 的写法。
我们可以将 async...await 的逻辑封装在 useEffect 回调函数的内部，这就是 ahooks useAsyncEffect 的实现思路，而且它的范围更加广，它支持的是所有的异步函数，包括 generator function。
### 24 自定义 hook
**自定义Hook**
通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。
可以理解成Hook就是用来放一些重复代码的函数。
下面我将做手动实现一个列表渲染、删除的组件，然后把它做成自定义Hook。
**示例**
定义数据列表
```javascript
const initialState = [
  { id: 1, name: "qiu" },
  { id: 2, name: "yan" },
  { id: 2, name: "xi" }
];
```
创建一个App组件并渲染它
```javascript
function App(props) {
  const [state, setState] = useState(initialState);
  const deleteLi = (index) => {
    setState((state) => {
      const newState = JSON.parse(JSON.stringify(state));//深拷贝数据
      newState.splice(index, 1);
      return newState;
    });
  };
  return (
    <>
      <ul>
        {state
          ? state.map((v, index) => {
              return (
                <li key={index}>
                  {index + "、"}
                  {v.name}
                  <button
                    onClick={() => {
                      deleteLi(index);
                    }}
                  >
                    X
                  </button>
                </li>
              );
            })
          : \"加载中\"}
      </ul>
    </>
  );
}
```
上面的代码，我对一个数组进行渲染+删除操作，当点击按钮时，就会删除数组的对应index的数据，从而执行页面更新
**封装成 hook**
```javascript
const useList = () => {
  const [state, setState] = useState(initialState);
  const deleteLi = (index) => {
    setState((state) => {
      const newState = JSON.parse(JSON.stringify(state));
      newState.splice(index, 1);
      return newState;
    });
  };
  return { state, setState, deleteLi };//返回查、改、删
};
```
我把上面的业务逻辑都放在useList这个函数中，并将查、改、删的API给放在一个对象中return出去。这样就形成了一个自定义Hook
**使用自定义Hook**
一般可以将自定义Hook给单独放在一个文件中，如果要使用，就引过来
```javascript
+import useList from "./useList";
```
在需要使用的App组件中执行自定义Hook并接收API
```javascript
function App(props) {
  const { state, deleteLi } = useList();//这里接收return出来的查、删API
  return (
     ... //这里跟最开始的App组件里是一样的，为了页面整洁，就不贴代码了
  );
}
```
**总结**
所谓的自定义Hook，实际上就是把很多重复的逻辑都放在一个函数里面，通过闭包的方式给return出来，这是非常高级的方式，程序员崇尚代码简洁，如果说以后业务开发时需要大量的重复代码，我们就可以将它封装成自定义Hook。
### 25 实现 useUpdate方法，调用时强制组件重新渲染
可以利用 useReducer。Reducer接受一个 state和 action，返回新的 state。  
const [state, dispatch] = useReducer(reducer, initialState);
```javascript
import { useReducer } from 'react';

const updateReducer = (num: number): number => (num + 1) % 1_000_000;

export default function useUpdate(): () => void {
  const [, update] = useReducer(updateReducer, 0);

  return update;
}

//使用
const update = useUpdate();
```
### 26 实现一个useTimeout Hook
useTimeout 是可以在函数式组件中，处理 setTimeout 计时器函数
**解决了什么问题？**
如果直接在函数式组件中使用 setTimeout ，会遇到以下问题：

- 多次调用setTimeout
```javascript
 function App() {  
    const [state, setState] = useState(1);  
    setTimeout(() => {  
        setState(state + 1);  
    }, 3000);  
    return (  
        // 我们原本的目的是在页面渲染完3s后修改一下state，但是你会发现当state+1后，触发了页面的重新渲染，就会重新有一个3s的定时器出现来给state+1，既而变成了每3秒+1。  
        <div> {state} </div>  
    );  
  }; 
```

- hooks 的闭包缺陷
```javascript
function App() {  
  const [count, setCount] = useState(0)  
  const [countTimeout, setCountTimeout] = useState(0)  
  useEffect(() => {  
      setTimeout(() => {  
          setCountTimeout(count)  
      }, 3000)  
      setCount(5)  
  }, [])  
  return (  
       //count发生了变化，但是3s后setTimout的count却还是0  
      <div>  
          Count: {count}  
          
  
          setTimeout Count: {countTimeout}  
      </div>  
  )  
}
```
**useTimeout 实现**
只执行一次
```javascript
function useTimeout(callback, delay) {
  
  const memorizeCallback = useRef();

  useEffect(() => {
    memorizeCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay !== null) {
      const timer = setTimeout(() => {
        memorizeCallback.current();
      }, delay);
      
      return () => {
        clearTimeout(timer);
      };
    }
  }, [delay]);
  
};
```
使用
```javascript
 // callback 回调函数， delay 延迟时间
  useTimeout(callback, delay);
```
如果不使用 **useRef**，而是直接在 **useEffect** 中使用 **callback** 变量，那么在 **useEffect** 中，**callback** 变量会指向最初的回调函数，而不是最新的。这可能会导致使用过期的回调函数，从而引发意外行为或错误。
使用 **useRef** 创建的 **memorizeCallback** 是一个可变的引用，它的 **current** 属性可以存储任意值，并且不会引发组件的重新渲染。这意味着，即使在组件重新渲染时，**memorizeCallback.current** 的值也会保持最新的回调函数。
















