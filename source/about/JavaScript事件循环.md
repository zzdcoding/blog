## 异步执行原理

### （1）单线程的JavaScript

我们知道，JavaScript是一种**单线程**语言，它主要用来与用户互动，以及操作DOM。

JavaScript 有同步和异步的概念，这就解决了代码阻塞的问题：

- **同步**：如果在一个函数返回的时候，调用者就能够得到预期结果，那么这个函数就是同步的；
- **异步**：如果在函数返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

**那单线程有什么好处呢？**

- 在 JS 运行的时候可能会阻止 UI 渲染，这说明了两个线程是互斥的。这是因为 JS 可以修改 DOM，如果在 JS 执行的时候 UI 线程还在工作，就可能导致不能安全的渲染 UI。
- 得益于 JS 是单线程运行的，可以达到节省内存，节约上下文切换时间的好处。

### （2）多线程的浏览器

JS 是单线程的，在同一个时间只能做一件事情，**那为什么浏览器可以同时执行异步任务呢？**

这是因为浏览器是多线程的，当 JS 需要执行异步任务时，浏览器会另外启动一个线程去执行该任务。也就是说，JavaScript是单线程的指的是执行JavaScript代码的线程只有一个，是浏览器提供的JavaScript引擎线程（主线程）。除此之外，浏览器中还有定时器线程、 HTTP 请求线程等线程，这些线程主要不是来执行 JS 代码的。

比如主线程中需要发送数据请求，就会把这个任务交给异步 HTTP 请求线程去执行，等请求数据返回之后，再将 callback 里需要执行的 JS 回调交给 JS 引擎线程去执行。也就是说，浏览器才是真正执行发送请求这个任务的角色，而 JS 只是负责执行最后的回调处理。所以这里的异步不是 JS 自身实现的，而是浏览器为其提供的能力。

下图是Chrome浏览器的架构图：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187132711-3eec1147-95dc-41f0-8188-f1397b2ee391.png)

可以看到，Chrome不仅拥有多个进程，还有多个线程。以渲染进程为例，就包含GUI渲染线程、JS引擎线程、事件触发线程、定时器触发线程、异步HTTP请求线程。这些线程为 JS 在浏览器中完成异步任务提供了基础。

## 浏览器事件循环

JavaScript的任务分为两种同步和异步：

- **同步任务：** 在主线程上排队执行的任务，只有一个任务执行完毕，才能执行下一个任务，
- **异步任务：** 不进入主线程，而是放在任务队列中，若有多个异步任务则需要在任务队列中排队等待，任务队列类似于缓冲区，任务下一步会被移到执行栈然后主线程执行调用栈的任务。

上面提到了任务队列和执行栈，下面就先来看看这两个概念。

#### （1）执行栈与任务队列

**① 执行栈**：从名字可以看出，执行栈使用到的是数据结构中的栈结构， 它是一个存储函数调用的栈结构，遵循**先进后出**的原则。**它主要负责跟踪所有要执行的代码。** 每当一个函数执行完成时，就会从堆栈中弹出（pop）该执行完成函数；如果有代码需要进去执行的话，就进行 push 操作。以下图为例：

![img](https://cdn.nlark.com/yuque/0/2022/gif/32656011/1667187212848-ca634f31-9ae2-4c48-bb63-8453a327471f.gif)

当执行这段代码时，首先会执行一个 main 函数，然后执行我们的代码。根据**先进后出**的原则，后执行的函数会先弹出栈，在图中也可以发现，foo 函数后执行，当执行完毕后就从栈中弹出了。

JavaScript在按顺序执行执行栈中的方法时，每次执行一个方法，都会为它生成独有的执行环境（上下文)，当这个方法执行完成后，就会销毁当前的执行环境，并从栈中弹出此方法，然后继续执行下一个方法。

**② 任务队列：** 从名字中可以看出，任务队列使用到的是数据结构中的队列结构，它用来保存异步任务，遵循**先进先出**的原则。**它主要负责将新的任务发送到队列中进行处理。**

JavaScript在执行代码时，会将同步的代码按照顺序排在执行栈中，然后依次执行里面的函数。当遇到异步任务时，就将其放入任务队列中，等待当前执行栈所有同步代码执行完成之后，就会从异步任务队列中取出已完成的异步任务的回调并将其放入执行栈中继续执行，如此循环往复，直到执行完所有任务。

JavaScript任务的执行顺序如下：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187228120-c0b43252-08eb-4043-aa3a-edf98d3da8c7.png)

**在事件驱动的模式下，至少包含了一个执行循环来检测任务队列中是否有新任务。通过不断循环，去取出异步任务的回调来执行，这个过程就是事件循环，每一次循环就是一个事件周期。**

### （2）宏任务和微任务

任务队列其实不止一种，根据任务种类的不同，可以分为**微任务（micro task）队列**和**宏任务（macro task）队列**。常见的任务如下：

- **宏任务：** script( 整体代码)、setTimeout、setInterval、I/O、UI 交互事件、setImmediate(Node.js 环境)
- **微任务：** Promise、MutaionObserver、process.nextTick(Node.js 环境)；

任务队列执行顺序如下：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187239414-b580d626-aed0-4b50-810a-83ee409fa624.png)

可以看到，Eventloop 在处理宏任务和微任务的逻辑时的执行情况如下：

1. JavaScript 引擎首先从宏任务队列中取出第一个任务；
2. 执行完毕后，再将微任务中的所有任务取出，按照顺序分别全部执行（这里包括不仅指开始执行时队列里的微任务），如果在这一步过程中产生新的微任务，也需要执行，**也就是说在执行微任务过程中产生的新的微任务并不会推迟到下一个循环中执行，而是在当前的循环中继续执行。**
3. 然后再从宏任务队列中取下一个，执行完毕后，再次将 microtask queue 中的全部取出，循环往复，直到两个 queue 中的任务都取完。

**也是就是说，一次 Eventloop 循环会处理一个宏任务和所有这次循环中产生的微任务。**

下面通过一个例子来体会事件循环：

```javascript
console.log('同步代码1');

setTimeout(() => {
    console.log('setTimeout')
}, 0)

new Promise((resolve) => {
  console.log('同步代码2')
  resolve()
}).then(() => {
    console.log('promise.then')
})

console.log('同步代码3');
```

代码输出结果如下：

```javascript
"同步代码1"
"同步代码2"
"同步代码3"
"promise.then"
"setTimeout"
```

那这段代码执行过程是怎么的呢？

1. 遇到第一个console，它是同步代码，加入执行栈，执行并出栈，打印出 **"同步代码1"；**
2. 遇到setTimeout，它是一个宏任务，加入宏任务队列；
3. 遇到new Promise 中的console，它是同步代码，加入执行栈，执行并出栈，打印出 **"同步代码2"；**
4. 遇到Promise then，它是一个微任务，加入微任务队列；
5. 遇到第三个console，它是同步代码，加入执行栈，执行并出栈，打印出 **"同步代码3"；**
6. 此时执行栈为空，去执行微任务队列中所有任务，打印出 **"promise.then"；**
7. 执行完微任务队列中的任务，就去执行宏任务队列中的一个任务，打印出 **"setTimeout"**

从上面的宏任务和微任务的工作流程中，可以得出以下结论：

- 微任务和宏任务是绑定的，每个宏任务在执行时，会创建自己的微任务队列。
- 微任务的执行时长会影响当前宏任务的时长。比如一个宏任务在执行过程中，产生了 10 个微任务，执行每个微任务的时间是 10ms，那么执行这 10 个微任务的时间就是 100ms，也可以说这 10 个微任务让宏任务的执行时间延长了 100ms。
- 在一个宏任务中，分别创建一个用于回调的宏任务和微任务，无论什么情况下，微任务都早于宏任务执行（优先级更高）。

**那么问题来了，为什么要将任务队列分为微任务和宏任务呢，他们之间的本质区别是什么呢？**

JavaScript在遇到异步任务时，会将此任务交给其他线程来执行（比如遇到setTimeout任务，会交给定时器触发线程去执行，待计时结束，就会将定时器回调任务放入任务队列等待主线程来取出执行），主线程会继续执行后面的同步任务。

对于微任务，比如promise.then，当执行promise.then时，浏览器引擎不会将异步任务交给其他浏览器的线程去执行，而是将任务回调存在一个队列中，当执行栈中的任务执行完之后，就去执行promise.then所在的微任务队列。

所以，宏任务和微任务的本质区别如下：

- 微任务：不需要特定的异步线程去执行，没有明确的异步任务去执行，只有回调；
- 宏任务：需要特定的异步线程去执行，有明确的异步任务去执行，有回调；

## Node.js的事件循环

### （1）事件循环的概念

对于Node.js的事件循环，官网的描述如下：

When Node.js starts, it initializes the event loop, processes the provided input script (or drops into the REPL, which is not covered in this document) which may make async API calls, schedule timers, or call process.nextTick(), then begins processing the event loop.

翻译一下就是：当Node.js启动时，它会初始化一个事件循环，来处理输入的脚本，这个脚本可能进行异步API的调用、调度计时器或调用process.nextTick()，然后开始处理事件循环。

JavaScript和Node.js是基于V8 引擎的，浏览器中包含的异步方式在 NodeJS 中也是一样的。除此之外，Node.js中还有一些其他的异步形式：

- 文件 I/O：异步加载本地文件。
- setImmediate()：与 setTimeout 设置 0ms 类似，在某些同步任务完成后立马执行。
- process.nextTick()：在某些同步任务完成后立马执行。
- server.close、socket.on('close',...）等：关闭回调。

这些异步任务的执行就需要依靠Node.js的事件循环机制了。

Node.js 中的 Event Loop 和浏览器中的是完全不相同的东西。Node.js使用V8作为js的解析引擎，而I/O处理方面使用了自己设计的libuv，libuv是一个基于事件驱动的跨平台抽象层，封装了不同操作系统一些底层特性，对外提供统一的API，事件循环机制也是它里面的实现的，如下图所示：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187334379-8f6bff6f-8513-411d-a661-05c85540cdcd.png)

根据上图，可以看到Node.js的运行机制如下:

1. V8引擎负责解析JavaScript脚本；
2. 解析后的代码，调用Node API；
3. libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎；
4. V8引擎将结果返回给用户；

### （2）事件循环的流程

其中libuv引擎中的事件循环分为 6 个阶段，它们会按照顺序反复运行。每当进入某一个阶段的时候，都会从对应的回调队列中取出函数去执行。当队列为空或者执行的回调函数数量到达系统设定的阈值，就会进入下一阶段。下面 是Eventloop 事件循环的流程：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187345802-bf50c2a3-c440-4cc7-af6b-d142bdea38a7.png)

整个流程分为六个阶段，当这六个阶段执行完一次之后，才可以算得上执行了一次 Eventloop 的循环过程。下面来看下这六个阶段都做了哪些事：

1. timers 阶段：执行timer（setTimeout、setInterval）的回调，由 poll 阶段控制；
2. I/O callbacks 阶段：主要执行系统级别的回调函数，比如 TCP 连接失败的回调；
3. idle, prepare 阶段：仅Node.js内部使用，可以忽略；
4. poll 阶段：轮询等待新的链接和请求等事件，执行 I/O 回调等；
5. check 阶段：执行 setImmediate() 的回调；
6. close callbacks 阶段：执行关闭请求的回调函数，比如socket.on('close', ...)

注意：上面每个阶段都会去执行完当前阶段的任务队列，然后继续执行当前阶段的微任务队列，只有当前阶段所有微任务都执行完了，才会进入下个阶段，这里也是与浏览器中逻辑差异较大的地方。

其中，这里面比较重要的就是第四阶段：poll，这一阶段中，系统主要做两件事：

- 回到 timer 阶段执行回调
- 执行 I/O 回调

在进入该阶段时如果没有设定了 timer 的话，会出现以下情况：

（1）如果 poll 队列不为空，会遍历回调队列并同步执行，直到队列为空或者达到系统限制；

（2）如果 poll 队列为空时，会出现以下情况：

- 如果有 setImmediate 回调需要执行，poll 阶段会停止并且进入到 check 阶段执行回调；
- 如果没有 setImmediate 回调需要执行，会等待回调被加入到队列中并立即执行回调，这里同样会有个超时时间设置防止一直等待下去；

当设定了 timer 且 poll 队列为空，则会判断是否有 timer 超时，如果有的就会回到 timer 阶段执行回调。

这一过程的具体执行流程如下图所示：

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187363206-19bda891-0838-4ea3-af0f-ce716f301fe3.png)
 

### （3）宏任务和微任务

Node.js事件循环的异步队列也分为两种：宏任务队列和微任务队列。

- 常见的宏任务：setTimeout、setInterval、 setImmediate、script（整体代码）、 I/O 操作等。
- 常见的微任务：process.nextTick、new Promise().then(回调)等。

### （4）process.nextTick()

上面提到了process.nextTick()，它是node中新引入的一个任务队列，它会在上述各个阶段结束时，在进入下一个阶段之前立即执行。

Node.js官方文档的解释如下：

process.nextTick()is not technically part of the event loop. Instead, thenextTickQueuewill be processed after the current operation is completed, regardless of the current phase of the event loop. Here, an operation is defined as a transition from the underlying C/C++ handler, and handling the JavaScript that needs to be executed.

例如下面的代码：

```javascript
setTimeout(() => {
    console.log('timeout');
}, 0);

Promise.resolve().then(() => {
    console.error('promise')
})

process.nextTick(() => {
    console.error('nextTick')
})
// 输出
nextTick
promise
timeout
```

可以看到，process.nextTick()是优先于promise的回调执行。

### （5）setImmediate 和 setTimeout

上面还提到了setImmediate 和 setTimeout，这两者很相似，主要区别在于调用时机的不同：

- **setImmediate**：在poll阶段完成时执行，即check阶段；
- **setTimeout**：在poll阶段为空闲时，且设定时间到达后执行，但它在timer阶段执行；

例如下面的代码：

```javascript
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('setImmediate');
});
// 输出
timeout
setImmediate
```

在上面代码的执行过程中，第一轮循环后，分别将 setTimeout  和 setImmediate 加入了各自阶段的任务队列。第二轮循环首先进入**timers 阶段**，执行定时器队列回调，然后 **pending callbacks**和**poll 阶段**没有任务，因此进入**check 阶段**执行 setImmediate 回调。所以最后输出为timeout、setImmediate。###= 4. Node与浏览器事件循环的差异 Node.js与浏览器的事件循环的差异如下：

- **Node.js**：microtask 在事件循环的各个阶段之间执行；
- **浏览器**：microtask 在事件循环的 macrotask 执行完之后执行；

![img](https://cdn.nlark.com/yuque/0/2022/png/32656011/1667187462318-c5a8e1ad-1bfe-4c21-97f6-be5f30f67d32.png)

Nodejs和浏览器的事件循环流程对比如下：

1. 执行全局的 Script 代码（与浏览器无差）；
2. 把微任务队列清空：注意，Node 清空微任务队列的手法比较特别。在浏览器中，我们只有一个微任务队列需要接受处理；但在 Node 中，有两类微任务队列：next-tick 队列和其它队列。其中这个 next-tick 队列，专门用来收敛 process.nextTick 派发的异步任务。**在清空队列时，优先清空 next-tick 队列中的任务，随后才会清空其它微任务**；
3. 开始执行 macro-task（宏任务）。注意，Node 执行宏任务的方式与浏览器不同：在浏览器中，我们每次出队并执行一个宏任务；而在 Node 中，我们每次会尝试清空当前阶段对应宏任务队列里的所有任务（除非达到系统限制）；
4. 步骤3开始，会进入 3 -> 2 -> 3 -> 2…的循环。