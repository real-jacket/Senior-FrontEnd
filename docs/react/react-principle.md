## 5.4 React-hooks

### 5.4.1 hooks使命

#### 逻辑组件复用

- 逻辑与UI组件分离

  React 官方推荐在开发中将逻辑部分与视图部分结耦，便于定位问题和职责清晰

- 函数组件拥有state

  在函数组件中如果要实现类似拥有state的状态，必须要将组件专成class组件

- 逻辑组件复用

 社区一直致力于逻辑层面的复用，像 render props / HOC，不过它们都有对应的问题，Hooks是目前为止相对完美的解决方案

#### hooks 解决的问题

render props

Avator 组件是一个渲染头像的组件，里面包含其中一些业务逻辑，User组件是纯ui组件，用于展示用户昵称

```jsx
export default funtion APP(){
    return (
        <div className="App">
            <Avatar>
               {data=> <User name={data}/>}
            </Avatar>
        </div>
    )
}
```

- 通过渲染props来实现逻辑组件复用
- render props 通过嵌套组件实现，在真实的业务中，会出现嵌套多层，以及梭理props不清晰的问题

Hoc

```jsx
class Avatar extends Component {
    render(){
        return <div>{this.props.name}</div>
    }
}
funtion HocAvatar(Component){
    return ()=> <Component name='王一瑾'/>
}
```
- 通过对现有组件进行扩展、增强的方式来实现复用，通常采用包裹方法来实现
- 高阶组件的实现会额外地增加元素层级，使得页面元素的数量更加臃肿

Hooks

```jsx
import React,{useState} from 'react'

export function HooksAvatar (){
    const [name,setName]=useState('王一瑾')
    return <>{name}</>
}
```
- React 16.8引入的Hooks，使得实现相同功能而代码量更少成为现实
- 通过使用Hooks，不仅在编码层面减少代码的数量，同样在编译之后的代码也会更少

### 5.4.2 hooks原理

hooks不是一个新ap也不是一个黑魔法，就是单纯的一个数组，看上面的例子hook api返回一个数组，一个是当前值，一个是设置当前值的函数
#### hooks的demo 

```jsx
import React ,{useState}from 'react';

const App = () => {
    const [name,setName]=useState('王一瑾')
    return (<div>
             <div>{name}</div>
             <button
                onClick={()=> setName('张艺凡')}
               >切换</button>
           </div>
       );
}
export default App;
```
- 上边是一个非常简单的Hook API，创建了name和setName，在页面上展示name，按钮的点击事件修改name

- 那么在这个过程中setState是如何实现的呢？

#### useState源码解析

```ts
useState<S>(initialState:(()=>S) | S) :[S,Dispatch<BasicStateAction<S>>] {

    currentHookNameInDev='useState'
    mountHookTypesDev()
    const preDispatcher=ReactCurrentDispatcher.current
    ReactCurrentDispatcher.current=InvalidNestedHooksDispatcherOnMountInDEV
    try{
        return mountState(initialState)
    }finally {
        ReactCurrentDispatcher.current=prevDispatcher
    }
}
```
- useState API 虽然是在react中引入的，其内部实现是在react-reconciler包中完成的
- 在try/catch代码部分，调用了mountState方法
- 顺着这个方法，我们去探寻一下mountState的实现

#### mountState解析

```ts
function mountState<S>(initialState:(()=>S | S,):[S,Dispatch<BasicStateAction<S>]{
    // 返回当前正在运行的hook对象
    const hook=mountWorkInProgressHook()
    // 初始值如果是函数，现执行函数
    if(typeof initialState==='function'){
        initialState=initialState()
    }
    // 如果是字符串就赋值给hook对象，hook.baseState和hook.memoizedState
    hook.memoizedState=hook.baseState=initialState
    // 定义一个队列
    const queue=(hook.queue={
        pending:null,
        dispatch:null,
        lastRenderedReducer:basicStateReducer,
        lastRenderedState:(initialState:any)
    })
    // dispatch挂载的queue，
    const dispatch:Dispatch<BasicStateAction<S>>=(
        dispatchAction.bind(
            null,
            currentlyRenderingFiber,
            queue // 传入queue与dispatch关联起来
         ):any))
        //  2个值以数值的形式返回
         return [hook.memoizedState,dispatch]
}
```
如果方法里面有多个useState方法，如何让这些按期望顺序执行呢？怎样维护queue对象？

![](~@/react/mountState.png)

- 在初始化时，每一次申明useState就图上所示，会生成一对state/setter映射。
接着每次渲染都会按照这个序列从数组最小下标遍历到最大值
- 在前面代码（mountState）中，我们说会先返回一个hook对象，state值（memoizedState）和返回的setXXX都会关联到这个hook对象，因此在触发某一个setXXX方法的时候可以正确地设置memoizedState值

### 5.4.3 hooks实践

#### Hook官方APi（大概率用到的）
- useState 
 函数组件中的state方法
- useEffect
函数组件处理副作用的方法
什么是副作用？异步请求、订阅原生的dom实事件、setTimeoutd等
- useReducer
另一种"useState"，跟redux有点类似
- useRef
返回一个突变的ref对象，对象在函数的生命周期内一直存在
-useCustom
自定义Hooks组件

#### test

