[Medium: JavaScript Event Loop Explained](https://medium.com/front-end-hacking/javascript-event-loop-explained-4cd26af121d4)

[jsconf: What the heck is the event loop anyway?](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html)

[You Don't Know JS: Async & Performance](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch1.md)

# Event Loop

> event loop 没在ECMAScript标准里提到，而是在HTML标准： https://html.spec.whatwg.org/#event-loops
> ECMAScript标准写的是jobs(microtasks), 为了处理promise callbacks: http://ecma-international.org/ecma-262/6.0/#sec-jobs-and-job-queues
> 虽然是在HTML标准里定义的，但是其他环境，比如Node也用

## 单线程 & 异步

JavaScript is a

single threaded programming language,

single threaded Runtime, it has a

single call stack. And it can

do one thing at a time,

that's what a single thread means, the program can run one piece of code at a time.

JS是单线程，即只有一个call stack, 在一个时间段只能运行一段代码。
同时也是异步的(asynchronous)

## 基本架构 Basic Architecture

![Architecture](/assets/article_images/2018/event_loop.png)

Heap: 对象分配在堆上

Stack: 即提供给JS代码运行的单线程，函数调用组成了栈帧(a stack of frames), 也叫过程活动记录(precedure activation record)

Browser or Web APIs: 并不是JS语言(ECMAScript)的一部分

## Web API

[Web API list](https://developer.mozilla.org/en-US/docs/Web/API)

[setTimeout in Web API](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)

[Introduction to Web APIs
](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)

> Client-side JavaScript has many APIs available to it — these are not part of the JavaScript language itself, rather they are built on top of the core JavaScript language.
客户端JS包含很多API, 这些API不是JS语言(ECMAScript?)的一部分, 而是建立JS语言核心之上

Client-side JS API 大概分为两类:
1. Brower APIs: 内嵌在浏览器中，暴露浏览器&底层环境(surrounding computer environment)数据，比如Geolocation API就封装/抽象了一些底层和GPS硬件通信的C++代码
2. Third party APIs: 不是默认内置在浏览器中, 比如Twitter API这种需要引入的。

## 例子1

```js
function main(){
  console.log('A');
  setTimeout(
    function exec(){
      console.log('B');
    }
  ,0);
  console.log('C');
}
main();
console.log('D');
// A C D B
```

![example](/assets/article_images/2018/event_loop_example.png)

1.1 main() 作为一帧frame被放入栈stack
1.2 main()的第一个声明statement被放入栈: `console.log('A')`
1.3 main()的第一个声明执行，然后这一帧被弹出popped out

2.1 第二个声明(setTimeout)入栈
2.2 setTimeout开始执行，由于setTimeout是Web API（到底是因为这个还是因为是callback, anyway，是进入了消息队列）, 所以这一帧被弹出, 交给浏览器处理，因为浏览器才有timer

3.1 `console.log('C')`入栈, 这时候浏览器的timer还在运行，callback exec()还没有执行.
3.2 由于delay是0s, 理想情况下，回调函数会在浏览器接收到之后立即加入消息队列/回调队列 message queue/callback queue

4 当main()的最后一个声明被执行完后，main()从调用栈中弹出，这时调用栈是空的。这时候浏览器会把消息队列中的消息推到调用栈，**前提是调用栈是空的。** 即使delay是0, 执行exec()回调也需要等到调用栈的所有帧都被执行完成

5 回调exec()被推入调用栈，`log('C')`

> 也就是说setTimeout(function, delayTime)不代表函数执行之后准确的time delay，而是代表了setTimeout函数执行之后的最小等待时间

* 为什么log('D')也在log('B')前面？

> Q: main() has popped out and call stack is empty, why B could not exec ? Is it because Message Queue is not quick enough to push log(‘B’)?
main() 推出的时候调用栈就已经空了

> A: This is because the console.log(‘D’); statement is still left in the stack after main() is popped out. So after executing console.log(‘D’); only then the stack becomes empty. Then the console.log(‘B’); is pushed into the stack from the message Queue. Note that for console.log(‘B’) to taken from the message Queue, the stack has to be empty.
因为main() pop out 的时候立马`log('D')`就进栈了，只有等`log('D')`执行结束后调用栈才是空的，这时候`log('B')`才能从消息队列推到调用栈里。

> But I mean, after main() has popped out and before console.log(‘D’) is pushed into, there is also a time that the stack is empty, right?

不对，在main()和log(‘D’)之上，还有调用他们的函数留在call stack

## 例子2

```js
function main(){
  console.log('A');
  setTimeout(
    function exec(){ console.log('B'); }
  , 0);
  runWhileLoopForNSeconds(3);
  console.log('C');
}
main();
function runWhileLoopForNSeconds(sec){
  // start: 被调用的时间
  let start = Date.now(), now = start;

  // 如果没有跑到sec时长，就一直执行。一个long run task, 占住call stack 3 secends
  while (now - start < (sec*1000)) {
    now = Date.now();
  }
}
// Output
// A
// wait for 3 sec...
// C
// B
```

`while loop`是一个blocking statement: 执行发生在call stack中，并且不需要使用browser APIs, 所以它会blocks所有后面的statements直到它执行完，占住了JS的单线程
在任务队列中生成一个long run task, 所以需要切分成小tasks放在队列中。


# Job Queue

JS中分为两种任务类型：macrotask宏任务和microtask微任务
在ECMAScript中，macrotask可称为task, microtask称为jobs

* macrotask(task)：主代码块，setTimeout，setInterval等
* microtask(jobs)：在浏览器重新渲染UI**之前**执行, 避免了没必要的重绘。Promise callbacks，process.nextTick, DOM mutation changes

event loop的原则：
1. 一次执行一个task (单线程)
2. task在执行中不会被其他task终端

In a single iteration在单次迭代中, event loop先检查macrotask queue,

如果有macrotask, 则执行一个task。如果没有, 直接去microtask queue.

当一个macrotask执行完毕, 去microtask queue。

这时event loop会执行完队列中**所有**的microtasks, 这是他们之间最大的区别。一次iteration中，最多只有一个macrotask会被执行，但是所有microtask queue中的microtasks都会被执行。

![microtask](/assets/article_images/2018/microtask_queue.jpg)

《Secrets of the JavaScript Ninja》