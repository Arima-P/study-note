# EventLoop

> js 是单线程语言，事件循环 `Event Loop` ，目前浏览器、`NodeJS` 处理 js 的一种机制

任务队列的执行过程是：先执行一个宏任务，执行过程中如果产出新的宏/微任务，就将他们推入相应的任务队列，之后在执行一队微任务，之后再执行宏任务，如此循环。以上不断重复的过程就叫做 Event Loop(事件循环)。

每一次的循环操作被称为 tick。

## 调用栈 `Call Stack`

> js 运行时，主线程会形成一个栈，后进先出，Last in First out

## 任务队伍

#### 同步任务、异步任务

> 每个任务都在做两件事，Ⅰ 发起调用，Ⅱ 得到结果

#### 宏任务、微任务

> 宏任务和微任务都是异步任务。  
> 异步任务的执行也是需要按顺序的，队列的属性就是先进先出（FIFO，First in First out）

#### 任务入队

> js 是单线程，但是浏览器不是单线程，浏览器不同的线程会对不同的事件进行处理，当对应的事件可以执行时，对应线程就会将其放入任务队列

- js 引擎线程：解释执行 js 代码、用户输入、网络请求等
- GUI 渲染线程：绘制用户界面，与 js 主线程互斥
  - 因为 js 可以操作 `DOM` ，进而影响到 GUI 的渲染结果
- http 异步网络请求线程：处理用户的 get 、post 等请求，等返回结果后将回调函数推入到任务队列
- 定时触发器线程：`setInterval`、`setTimeout` 等待时间结束后，会把执行函数推入任务队列中
- 浏览器事件处理线程：将 `click`、`mouse` 等 UI 交互事件发生后，将要执行的回调函数放入到事件队列中

#### 宏任务

|                       | 浏览器 | Node |
| :-------------------: | :----: | :--: |
|  整体代码（script）   |   ✔    |  ✔   |
|      UI 交互事件      |   ✔    |  ×   |
|          I/O          |   ✔    |  ✔   |
|      setInterval      |   ✔    |  ✔   |
|      setTimeout       |   ✔    |  ✔   |
|     setImmediate      |   ×    |  ✔   |
| requestAnimationFrame |   ✔    |  ×   |

#### 微任务

|                            | 浏览器 | Node |
| :------------------------: | :----: | :--: |
|      process.nextTick      |   ×    |  ✔   |
|      MutationObserver      |   ✔    |  ×   |
| Promise.then catch finally |   ✔    |  ✔   |
