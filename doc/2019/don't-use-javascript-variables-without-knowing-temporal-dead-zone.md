# Don't Use JavaScript Variables Without Knowing Temporal Dead Zone

看一个简单问题。 以下哪个代码片段将产生错误？


第一个使用class定义了一个类然后创建类一个实例：

``` javascript
new Car('red'); // Does it work?

class Car {
  constructor(color) {
    this.color = color;
  }
}

```
