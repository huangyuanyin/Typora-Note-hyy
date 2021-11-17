### Js 异步处理演进，Callback=>Promise=>Observer

> 异步调用就像是接水管，相互缠绕的管道越多，就越容易漏水。如何将水管巧妙连通，使整个系统有足够的弹性，需要去认真思考 🤔
>
> 对于 JavaScript 异步的理解，不少人感到过困惑：Js 是单线程的，如何做到异步的呢？实际上，Js 引擎通过混用 2 种内存数据结构：**栈和队列**，来实现的。栈与队列的交互也就是大家所熟知的 Js **事件循环**~~

举个栗子🌰：

```JavaScript
function fooB(){
    console.log('fooB: called');
}

function fooA(){
    fooB();
    console.log('fooA: called');
}

fooA();
// -> fooB: called
// -> fooA: called
```

Js 引擎解析如下：

```JavaScript
1. push fooA to stack
<stack>
|fooA| <- push

2. push fooB to stack
<stack>
|fooB| <- push
|fooA|

3. pop fooB from stack and execute
<stack>
|fooB| <- pop
|fooA|

// -> fooB: called
<stack>
|fooA|

4. pop fooA from stack and execute
<stack>
|fooA| <- pop

// -> fooA: called

<stack>
|   | <- stack is empty
```

> 从以上代码可以看出，`fooA`、`fooB` 两个同步函数都被压入 **栈** 中，那么什么样的函数会被放入 **队列** 中呢？

> 当然就是包含异步操作的函数了：
>
> - setTimeout
> - setInterval
> - promise
> - ajax
> - DOM events

举个栗子🌰：

```javascript
function fooB(){
    setTimeout(()=>console.log('API call B'));
    console.log('fooB: called');
}

function fooA(){
    setTimeout(()=>console.log('API call A'));
    fooB();
    console.log('fooA: called');
}

fooA();
/*
fooB: called
fooA: called
API call A
API call B
*/
```

Js引擎解析如下：

```JavaScript
1.push fooA to stack
  <stack>
  |fooA| -- push
2.push 'API call A' to queue
  <queue>|'API call A'| <- push
3.push fooB to stack
  <stack>
  |fooB| <- push
  |fooA|
4.push 'API call B' to queue
  <queue>|'API call A'|'API call B'| <- push
5.pop fooB from stack and execute
  <stack>
  |fooB| <- pop
  |fooA|
  // -> fooB: called
  <stack>
  |fooA|
6.pop fooA from stack and execute
  <stack>
  |fooA| <- pop
  // -> fooA: called
  <stack> <- stack is empty
  |   |
7. pop 'API call A' from queue and execute
  <queue>|'API call A'| <- pop |'API call B'|
  // -> API call A
  <queue>|'API call B'|
8. pop 'API call B' from queue and execute
  <queue>|'API call B'| <- pop
  // -> API call B
  <queue>|   | <- queue is empty
```

##### 一.Callback

> 怎么理解 Callback ？以打电话给客服为例，有两种选择：
>
> 1. 排队等待客服接听；
>
> 2. 选择客服有空时回电给你。
>
>    第 2 种选择就是 JavaScript Callback 回调模式，在等待客服回复的同时，可以做其它事情，一旦客服有空，会主动回电给你~

```javascript
function success(res){
    console.log("API call successful");
}
function fail(err){
    console.log("API call failed");
}
function callApiFoo(success, fail){
    fetch(url)
      .then(res => success(res))
      .catch(err => fail(err));
};
callApiFoo(success, fail);
```

Callback 缺点是：嵌套调用会形成回调地狱，如下：

```JavaScript
callApiFooA((resA)=>{
    callApiFooB((resB)=>{
      callApiFooC((resC)=>{
          console.log(resC)
      },fail)  
    },fail)
},fail)
```

### 二.Promise

> 众所周知，Promise 就是来解决回调地狱的~

```javascript
function callApiFooA(){
    return fetch(url) // JS fetch method returns a Promise
}
function callApiFooB(resA){
    return fetch(url+'/'+resA.id)
}
function callApiFooC(resB){
    return fetch(url+'/'+resB.id)
}
callApiFooA()
	.then(callApiFooB)
	.then(callApiFooC)
	.catch(fail)
```

> 与此同时，Promise 还提供了很多其它更具扩展性的解决方案，比如 `Promise.all`、`Promise.race` 等；

> - Promise.all：并发执行，全部变为 resolve 或 有 reject 状态出现的时候，它才会去调用 .then 方法；

```javascript
function callApiFooA(){
    return fetch(urlA); 
}
function callApiFooB(){
    return fetch(urlB);  
}
function callApiFooC([resA, resB]){
    return fetch(url+'/'+resA.id+'/'+resB.id);  
}
function callApiFooD(resC){
    return fetch(url+'/'+resC.id);  
}
Promise.all([callApiFooA(),callApiFooB()])
	.then(callApiFooC)
	.then(callApiFooD)
	.catch(fail)
```

> Promise 让代码看起来更简洁，但是演进还没结束；如果想处理复杂的数据流，用 Promise 将会很麻烦......

### 三.Observer

> 处理多个异步操作数据流是很复杂的，尤其是当它们之间相互依赖时，我们必须以更巧妙的方式将它们组合；**Observer 登场！**

observer 创建（发布）需更改的数据流，*subscribe* 调用（订阅消费）数据流；以 [RxJs](https://link.juejin.cn/?target=https%3A%2F%2Frxjs.dev%2F) 举例：

```javascript
function callApiFooA(){
    return fetch(urlA); 
} 
function callApiFooB(){
    return fetch( urlB );  
}
function callApiFooC( [resAId, resBId] ){
    return fetch(url +'/'+ resAId +'/'+ resBId);  
} 
function callApiFooD( resC ){
    return fetch(url +'/'+ resC.id);  
} 
Observable.from(Promise.all([callApiFooA(),callApiFooB()])).pipe(
    map(([resA, resB]) => ([resA.id, resB.id])), // <- extract ids
    switchMap((resIds) => Observable.from(callApiFooC( resIds ) )),
    switchMap((resC) => Observable.from(callApiFooD( resC ) )),
    tap((resD) => console.log(resD))
).subscribe();
```

> 详细过程：
>
> 1. Observable.from将一个Promise数组转换为Observable，它是基于callApiFooA和callApiFooB的结果数组。
> 2. map -- 从API 函数A和B的Respond 中提取id；
> 3. switchMap  -- 使用前一个结果的 id 调用 callApiFooC，并返回一个新的 Observable，新 Observable 是 callApiFooC( resIds ) 的返回结果；
> 4. switchMap — 使用函数 callApiFooC 的结果调用 callApiFooD；
> 5. tap — 获取先前执行的结果，并将其打印在控制台中；
> 6. subscribe — 开始监听 observable；

Observable是多数据值的生产者，它在处理异步数据流方面更加强大和灵活，它在 Angular 等前端框架中被使用~~

> **敲！这写法，这模式不就是函数式编程中的函子吗？Observable 就是被封装后的函子，不断传递下去，形成链条，最后调用 subscribe 执行，也就是惰性求值，到最后一步才执行、消费！**

> 这样做有何好处？
>
> 核心原因就是分离**创建（发布）** 和 **调用（订阅消费）**！

```JavaScript
var observable = Rx.Observable.create(function (observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
});

console.log('just before subscribe');
observable.subscribe({
  next: x => console.log('got value ' + x),
  error: err => console.error('something wrong occurred: ' + err),
  complete: () => console.log('done'),
});
console.log('just after subscribe');
/*
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
*/
```

observable 发布（*同步地*）`1`， `2`， `3` 三个值；1秒之后，继续发布`4`这个值，最后结束；

subscribe 订阅，调用执行；subscription.unsubscribe() 可以在过程中中止执行；



> 总结：Js 异步处理演进分为 3 个阶段：Callback=>Promise=>Observer，重点理解也就是 Observer。
>
> Observer 就像是函数编程的函子，封装、传递链、延迟执行，几乎一摸一样，不过它更加强调发布和订阅的思想！分割函数的创建和执行为两个独立的域，对于弹性组装异步水管至关重要！！
>
>  之前的文章就提过，惰性求值似乎能连接 js 最重要的闭包和异步两个要点，现在看来更是如此。。。







