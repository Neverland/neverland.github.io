# Threejs in autonomous driving -（2）merge geometry

> 由于使用场景的关系，我们的产品主要设备是ipad，使用软件为chrome或者safari。对webgl无节制的使用，很容易造成灾难性的后果`崩溃`。所以我们要减少cpu和memory的使用。

## 原理

这里照抄一段`webgl高级编程`的原文。

    任何对WebGL API的调用都会带来开销。每个调用都会要求CPU进行额外的处理和数据复制，这回占用时间并要求CPU做一些额外工作。通常，如果GPU接收到大批可并行处理的数据，运行效率会提高很多。

这里感谢一篇文章，详细的介绍了`merge geometry`。里面的API虽然经过版本迭代有所变化，但是效果依然。通过这种技术成功的使我们可以一次展示成千上万个geometry。
[performance-merging-geometry](http://learningthreejs.com/blog/2011/10/05/performance-merging-geometry/)

## 实现

实现非常的简单，见一下案例代码。官方API [Geometry.merge](https://threejs.org/docs/index.html#api/zh/core/Geometry.merge)

```javascript

let geometry = new THREE.Geometry();

junction.forEach(({point}) => {
    point.push(point[0]);

    let junction = getShapeGeometryFromPoints(point, false);

    geometry.merge(junction);
});

```

## 验证

直观上可以看到cpu和memory使用减少了，怎样可以去量化呢。Threejs的renderer示例提供了一个可以接口，renderer.info属性可以看到render的实时情况。使用renderer.info.calls可以对比一下前后的drawacall情况。

---
- 我的blog: [neverland.github.io](https://neverland.github.io/)
- 我的email [enix@foxmail.com](enix@foxmail.com)