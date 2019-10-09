# Don't Use JavaScript Variables Without Knowing Temporal Dead Zone

看一个简单问题。 以下哪个代码片段将产生错误？


第一个，使用class定义了一个类然后创建类一个实例：

``` javascript
new Car('red'); // Does it work?

class Car {
  constructor(color) {
    this.color = color;
  }
}

```

第二个，先引用后定义一个函数：

greet('World'); // Does it work?

``` javascript

function greet(who) {
  return `Hello, ${who}!`;
}

```

正确答案是：第一段用class定义的代码，会产生一个 `ReferenceError` 错误，第二个运行正常。

如果你答错了或者没明白发生了什么，那么你需要掌握暂性死区（TDZ）。TDZ作用于 `let`、`const`和`class`语句。他对于js的变量的工作方式非常重要。

## 1.什么是暂性死区


我们从一个简单的const声明开始。先声明并初始化变量并且访问他，完全ok。

``` javascript

const white = '#FFFFFF';

white; // => '#FFFFFF'

```

下面我们尝试定义前访问变量`white`:

``` javascript
white; // throws `ReferenceError`

const white = '#FFFFFF';

white;

```
直到 `const white = '#FFFFFF'`语句，变量`white`一直在TDZ中。
如果访问了TDZ中的`white`， JavaScript会抛错，ReferenceError: Cannot access 'white' before initialization.

