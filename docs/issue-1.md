# JavaScript内存管理

## 简介

JavaScript不像c语言，需要手动去调用底层的内存管理接口，比如 malloc()和free()，来分配和销毁内存，JavaScript是在创建变量（对象，字符串等）时自动进行了分配内存，并且在不使用它们时“自动”释放。 释放的过程称为垃圾回收。

## JavaScript 的内存分配方法

- 值的初始化

```javascript
var s = 'hello world' // 给字符串分配内存
var n = 888 // 给数组分配内存
var array = [1, null, 'abc'] // 给数组及其包含的值分配内存
var obj = {
  a: 1,
  b: 'qwe',
  c: null
} // 给对象及其包含的值分配内存

function f(a){
  return a + 2;
} // 给函数（可调用的对象）分配内存

elementDom.addEventListener('click', function(){
  elementDom.style.color = 'blue';
}, false) // 函数表达式也能分配一个对象
```

- 通过函数调用分配内存

```javascript
// 有些函数调用结果是分配对象内存：
var date = new Date() // 给分配一个Date对象
var dom = document.createElement('p') // 分配一个 DOM 元素

// 有些方法分配新变量或者新对象:
var s = "apple";
var s2 = s.substr(0, 3); // s2 是一个新的字符串,因为字符串是不变量，JavaScript 可能决定不分配内存，只是存储了 [0-3] 的范围。

var array = ["red", "green"];
var array2 = ["apple", "banana"];
var array3 = array.concat(array2); // 新数组有四个元素，是 array 连接 array2 的结果
```

-垃圾回收方法

- 引用计数垃圾收集

这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

```javascript
var obj = {
  a: {
    b: 1
  }
} // 两个对象被创建，a作为'这个对象'属性被引用，'这个对象'被分配给obj
var obj2 = obj // obj2是'这个对象'第2个引用

obj = {c: 8} // '这个对象'只有一个引用obj2
var obja = obj2.a // obja是'这个对象'的第2个引用
obj2 = 3 // '这个对象'只有一个引用obja
obja = null // '这个对象'没有引用，可以被垃圾回收

```

缺陷：陷入引用循环
```javascript
function f() {
  var obj = {}
  var obj2 = {}
  obj.a = obj2
  obj2.a = obj
}
f() // obj和obj2永远不会被垃圾回收
// 还有很多情况由于来不及清除对象的引用，容易会造成内存泄露
```
- 标记-清除算法

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法。所有对JavaScript垃圾回收算法的改进都是基于标记-清除算法的改进。

这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。

这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象。

## Concurrency model and Event Loop

### Javascript是单线程吗？

Javascript是单线程，即同一时刻只能做一件事情。Javascript作为浏览器脚本语言，经常用于操作DOM，如果是多线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

### Javascript执行机制

Javascript的内存模型图
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop
栈(Stack)：函数调用形成了一个由若干帧组成的栈
堆(Heap)：对象被分配在堆中
队列(Queue)：一个 JavaScript 运行时包含了一个待处理消息的消息队列，每一个消息都关联着一个用以处理这个消息的回调函数，只有满足了消息才能触发回调函数

以执行函数为例讲解运行机制

```javascript
function f() {
  console.log('我是f函数')
}

function f2() {
  console.log('我是f2函数')
}

function f3() {
  console.log('我是f3函数')
  setTimeout(f2, 1)
  f();
}

f3()
```
当调用 f3 时，第一个帧被创建并压入栈中，帧中包含了 f3 的参数和局部变量。
当执行setTimeout函数时候，延迟执行f2，此时浏览器不可能等待1ms之后再执行f2，会新增一个消息记录毫秒数并把f2作为消息Callback放入Queue，接着会继续执行下面语句。
当 f3 调用 f 时，第二个帧被创建并被压入栈中，放在第一个帧之上，帧中包含 f 的参数和局部变量。当 f 执行完毕然后返回时，第二个帧就被弹出栈（剩下 f3 函数的调用帧 ）。当f3 也执行完毕然后返回时，第一个帧也被弹出，栈就被清空了。
此时浏览器是空闲的，就会去监听Queue，一旦某个Messahe满足条件，主线程就会从Queue中取出该Message的Callback回调函数压入栈中(如果此时有两个同时满足条件的Message，会先取出先进入Queue的Message的Callback回调执行)，继续执行，执行完弹出帧，继续去监听Queue中是否存在满足条件的Message......直到清空Queue

因此上面代码执行的结果是
```javascript
我是f3函数
我是f函数
我是f2函数
```
**JavaScript运行机制，永远在单线程上按顺序运行，当遇到阻塞执行类型函数(即异步，需要等待某个结果才能执行的函数)时候，线程会把该阻塞执行类型函数塞入Queue等待调用执行，之后会继续依次执行，直到语句执行完成，此时主线程空闲，会会去监听Queue，一旦某个Messahe满足条件，主线程就会从Queue中取出该Message的Callback回调函数压入栈中(如果此时有两个同时满足条件的Message，会先取出先进入Queue的Message的Callback回调执行)到主线程执行，执行完继续去监听Queue中是否存在满足条件的Message，直到Queue为空。**

**当然通过某些手段能让阻塞执行类型函数变成立即执行函数，会被主线程立即执行而不被塞入Queue等待执行队列**


