# 掌握 eventLoop

> https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/

```
console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(function () {
    console.log('promise1');
  })
  .then(function () {
    console.log('promise2');
  });

console.log('script end');
```

输出结果：

```
script start
script end
promise1
promise2
setTimeout
```

## 总结

1. 任务分为任务（宏任务）和微任务（Job）。任务按顺序执行，并且浏览器在每个宏任务之间进行渲染。

2. 微任务按顺序执行，并执行:

   （1）在每个回调之后，只要没有其他 JavaScript 在执行中

   （2）在每个任务结束的时候

3. 不同浏览器对于以上输出结果可能会不一样。因为 promise 是来自 ECMAScript 而不是 HTML，但是一般我们会认为 promise 属于微任务的一部分。

4. setTimeout 的 callback 被压入到任务队列，promise 的 callback 被压入到当前队列的微任务队列。
