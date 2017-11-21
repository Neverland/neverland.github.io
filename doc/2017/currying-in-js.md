# Currying is not idiomatic in JavaScript

## 1.你说的"不常用"是什么意思?

> 稍微激进的标题可能会导致标题党误读了这篇博客。我要澄清一下：我本人喜欢函数式编程。我的主要观点是：

 - 一些核心JavaScript功能与currying相冲突。
 - 在我看来currying还有其他选择（尤其是即将到来的提案）。在partial application(偏函数应用)中使用是非常好的，这没有什么冲突。
 
 JavaScript最好的特性之一就是可以适用多种不同的编程风格。因此：如果你Currying编程到没有任何提到的替代品适合你，那么你就肆意适用Curring吧。 如果你这样做，请保持你的代码风格一致。并且准备好你的代码与生态系统的其他部分稍有不同。

## 2.Currying

 Currying在函数式编程中是一种流行的技术, 它有助于偏函数应用。
 
 其思想如下：如果您不为函数提供所有参数，则返回一个函数。函数的入参是剩余参数，其输出是原始函数的结果。
 
 例如，如果想要数组的所有元素加2，并有一个二进制函数Add（x，y），则可以如下所示：

```javascript
const arr = [1, 2, 3];

arr.map(add(2)); // [3, 4, 5]`
```

 顺便说一下，具有自动currying的函数式编程语言通常具有可以这样使用的加法操作符。

 你有两个选择执行add（）方法来支持currying。

### 2.1 简单 currying  
      
 [ES6箭头函数](http://exploringjs.com/es6/ch_arrow-functions.html)可以很容易地手动编写柯里函数：
 
```javascript
const add = x => y => x + y;

// 等价于（译者注）
function add(x) { 
    return function(y) {
        return x + y;
    }
}
```
 如下所示你可以这样执行add()

```javascript
add(2)(3); // 5
```
 一个函数持有多个需要转化的嵌套函数，多数具有自动currying的函数式编程语言在语法上add(1, 2) 和add(1)(2)没有什么区别。

2.2 重载 currying
  
  一些库提供了重载的函数。 根据参数数量，这些重载函数中的每一个都会有不同的表现：
  
  - 如果只使用一个参数，将会柯里化。
  - 如果使用所有的参数，将会是一个普通的函数调用。
  - 如果使用了多个参数（而不是所有参数），则会返回一个绑定到这些参数的函数。
  
  这种形式的curring的实现如下。
  
```javascript
  function add(...args) {
      if (args.length < 2) {
          return add.bind(this, ...args);
      }
      const [x, y] = args;
      return x + y;
  }
```  

可以编写一个工具函数，将普通函数转换为这种实现。
  
### 2.3 小结

 柯里化是一种有助于偏函数应用的函数转化技术。
 
## 3. Currying 和一些javascript基础有冲突

 如果想要在JavaScript使用currying编程你有两选择：
 
  - 使用真currying
    - 提倡：容易静态输入。
    - 反对：不常用的函数调用语法。
  - 使用重载currying
    - 提倡：优雅的语法。
    - 反对：难以取得的静态类型。

 如果你转换了内置函数或方法来适应你的currying风格，那么你将面对很多问题。
 
 注意：并非所有这些缺点同样重要。
 
### 3.1 具名参数

javascript中具名参数可以通过Object字面量模拟（[Simulating named parameters](http://exploringjs.com/es6/ch_parameter-handling.html#sec_named-parameters) ES6中）我喜欢这种绑定参数的方式，他使得代码更有自我描述性。不是所有currying的javascript库都支持具名参数。

### 3.2 不清楚函数调用和非函数调用

通常情况下在函数的后面使用()就可以执行该函数。Currying后你必须清楚的知道函数函数foo中函数的个数。以便知道 foo(123)执行后得到的是一个函数还是函数的结果。
 
比较下面两个偏函数应用

```javascript
  foo(123, ?)
  _ => foo(123, _)
```  
foo（）支持多种方式调用。

使用具名参数，在curry中会丢失更多信息

```javascript
  arr.map(findCity({ latitude: ‎48.137154 })); // curried
  arr.map(_ => findCity({ latitude: ‎48.137154, longitude: _ }))
```

静态类型系统（TypeScript，Flow，...）会减少错误，但是替代方法仍然更易于阅读。

### 3.3 Currying 与默认参数的冲突

如果忽略参数，就会使用函数的默认参数

```javascript
  function add(x=0, y=0) {
      return x + y;
  }
```
 默认参数可以使的添加参数更加透明并且不会破坏现有调用，这种技术对命名参数特别有用，我经常用来保证通用函数和方法的适用性。

 使用参数默认值省略参数意味着使用默认值，使用currying省略参数意味着返回偏函数， 因此两者有冲突。
 
### 3.4 Currying 友好的函数
 
 使用Currying，参数对函数开始是对配置和结束运算都很重要。而Javascript中并不是这样。
 
 例如parseInt（）的签名是：
 
```javascript
  parseInt(s : string, radix : number);
```
这使得它不可能这样预先填充基数：
 
```javascript
  ['32', '17', '5'].map(parseInt(10)) // doesn’t work
```

## 偏函数编程有那些替代方案？

当谈到Currying，人们通常想要的是偏函数编程。让我们看看下面的例子重载Currying并探索替代方案。
 
```javascript
  [1, 2, 3].map(add(2))
```

替代 1: .bind().
 
```javascript
  [1, 2, 3].map(add.bind(null, 2))
```

替代 2: arrow functions.
 
```javascript
  [1, 2, 3].map(x => add(2, x))
```
替代 3: [an upcoming proposal for partial application.](https://github.com/rbuckton/proposal-partial-application)

```javascript
  [1, 2, 3].map(add(2, ?))
```

注：语法固定，提案处于早期阶段。

这个建议的好处在于，它可以很好地处理任意的签名，并且具有很好的描述性语法。

## 5. 扩展阅读

 - [Currying versus partial application (with JavaScript code)](http://2ality.com/2011/09/currying-vs-part-eval.html)
 - [Arrow functions vs. bind()](http://2ality.com/2016/02/arrow-functions-vs-bind.html)
 - [Uncurrying “this” in JavaScript](http://2ality.com/2011/11/uncurrying-this.html)


[原文地址](http://2ality.com/2017/11/currying-in-js.html)