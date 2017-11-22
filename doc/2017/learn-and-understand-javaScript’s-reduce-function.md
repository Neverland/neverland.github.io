# 学习 JavaScript的Array Reduce

> Lear how to use Array.prototype.reduce()

## Reduce 定义 & 语法

 `reduce()`方法将函数应用于数组中的每个项，以将数组缩减为一个值。

 例子：

```javascript

let result = arr.reduce(callback);
// 可选, 你可以指定一个初始值（initial value）
let result = arr.reduce(callback, initValue);

```

 - result — 返回的最终结构.
 - arr — 要使用reduce函数的数组.
 - callback — 这个函数会被尝试用在当前数组的每一项上。如果函数的返回值是true就会使用当前数组项，反之忽略当前数组项.
 - initValue — 可选的初始值. 如果没有设置, 数组的第一项会被作为默认值。

 callback函数有四个参数，通过map(), filter()方法你一定已经熟悉了这些参数，新的参数是累加器（accumulator）。
 
 - accumulator — 累加器会累加callback的返回值。
 - val — 被处理的当前数组项。
 - index — 当前索引。
 - arr — 调用reduce的数组。

## Reduce vs. For Loop


 我们可以把reduce()想象成为一个for循环，专门用数组的项来创建新的值。

```javascript

var arr = [1, 2, 3, 4];
var sum = 0;

for(var i = 0; i < arr.length; i++) {
    sum += arr[i];
}
// sum = 10

```

 如上所见代码的目的是得到数组项的总和。他循环数组并且把值添加到变量sum我们可以得到最终结果10。当然我们可以使用reduce()轻松达到同样的目的。
 
 我们从的简单的数字数组开始讲解reduce()函数的使用。
 
```javascript

let arr = [1,2,3,4];

```
 arr是用来reduce处理的数组，我们期望得到数组项的和。要做到这点数组每次迭代时我们将数组项添加到累加器并且返回，通过返回值将得到新的累加器。
 
```javascript

let sum = arr.reduce((acc, val) => {
  return acc + val;
});

```
 美滋滋！ 用变量acc来表示累加器。 当arr调用reduce函数数时acc值将会增加直到函数完成。

 一旦完成，将会销毁变量得到结果。

```javascript

sum = 10

```

 真棒！

## 指定初始值（Initial Value）

 还记得上面可以指定初始值吗？他设置起来如此简单，我们仍然使用上面的例子去得到数组项的和，但这次我们从100开始。
 
```javascript

let sum = arr.reduce((acc, val) => {
  return acc + val;
}, 100);

```
 如你所见例子相同，变化只是给callback添加了第二个参数。我们以100为初始值当运行这个函数时得到的值是110。
 
## Reduce & ES6
 
 使用箭头函数可以简化代码。箭头函数有返回值时，可以省略括号和return关键字。
 
 上面的代码等价于：

```javascript

 let sum = arr.reduce((acc, val) => acc + val, 100);

```

 很酷吧？只使用一行代码，我们可以把100添加到数组的和中！在for循环上我们已经浪费了很多代码。
 
## 挑战
 
 给出以下数据:
  
```javascript

let data = [
  {
    country: 'China',
    pop: 1409517397
  },
  {
    country: 'India',
    pop: 1339180127
  },
  {
    country: 'USA',
    pop: 324459463
  },
  {
    country: 'Indonesia',
    pop: 263991379
  }
]

```

 使用reduce()方法，如何获得中国以外国家的人口总和？思考答案再滚到下面看解决方案。
 
\***
 
\*****
 
\*******
 
\*****
 
\***

 我们可以这样做：

```javascript 
 
let sum = data.reduce((acc, val) => {
  return val.country == 'China' ? acc : acc + val.pop;
}, 0);

```
 我使用0作为初始值，这里我们检查country是否是China。如果是就返回累加器相当于跳过了China，否则就在累加器上加上当前国家到人口数。
 
 运行函数后，得到 sum = 1927630969。
 
 [原文](https://codeburst.io/learn-understand-javascripts-reduce-function-b2b0406efbdc).
 
## 思考（译者添加）
 
 如何实现一个[].reduce()的es5 shim。