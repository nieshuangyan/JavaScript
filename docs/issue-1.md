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

-通过函数调用分配内存

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

