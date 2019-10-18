# Threejs in autonomous driving -（6）圆矩

> 绘制各种几何体是webgl的强项，相反各种异性几何体就非常麻烦。比如圆角矩形来说在webgl中绘制就相对比较麻烦。在css中给矩形加上border-radius就可以轻易实现。在webgl中就非常麻烦了。

![a5af4149448fa01a17ba1d7fa09f6439026a4469.jpg](https://image-static.segmentfault.com/385/717/3857177158-5da97bafa5c7d_articlex)

### 思路

1.我们可以使用[shape](https://threejs.org/docs/index.html#api/zh/extras/core/Shape)绘制一个圆角矩形的二维平面。
2.使用[ExtrudeGeometry](https://threejs.org/docs/index.html#api/zh/geometries/ExtrudeGeometry)拉起这个二维平面从而形成一个三维几何体。

### 获取一个圆角矩形path的方法

```javascript

function createBoxWithRoundedEdges(width, height, depth, radius0, smoothness) {
        let shape = new THREE.Shape();
        let eps = 0.00001;
        let radius = radius0 - eps;
        shape.absarc(eps, eps, eps, -Math.PI / 2, -Math.PI, true);
        shape.absarc(eps, height -  radius * 2, eps, Math.PI, Math.PI / 2, true);
        shape.absarc(width - radius * 2, height -  radius * 2, eps, Math.PI / 2, 0, true);
        shape.absarc(width - radius * 2, eps, eps, 0, -Math.PI / 2, true);
        let geometry = new THREE.ExtrudeBufferGeometry(shape, {
            depth: depth - radius0 * 2,
            steps: 1,
            bevelSize: radius,
            bevelThickness: radius0
        });
        geometry.center();
        return geometry;
    }
```
---
需要注意的是bevelSegments都不要设置的太大，否则会导致运算量很大，卡顿明显。

原始实现方法来自[round-edged-box](https://discourse.threejs.org/t/round-edged-box/1402)


---
- 我的blog: [neverland.github.io](https://neverland.github.io/)
- 我的email [enix@foxmail.com](enix@foxmail.com)
