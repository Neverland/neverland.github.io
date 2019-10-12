# Threejs in autonomous driving -（5）图片替代模型

> 在自动驾驶实际中的研发工具中，camera一般是跟随主车进行移动的。那么主车是以固定视角展示给用户的。所以并不需要使用一个很漂亮的模型。由此可见我们完全可以使用图片来代替主车模型。来加速我们的页面加载速度和提升视觉效果。

## geometry + texture尝试

如果简单把图片作为纹理贴在多边形上有多种问题：

1.因为，图片很难达到和模型完全匹配。（uv也非常难以调整）
2.而且一个6面多边形即使把高度设置为1也会看起来很奇怪。
3.通过多边形做主车来贴纹理的方式，主车转弯时会因为矩阵转换导致问题意想不到的效果。

结论：显而易见使用geometry + textrue的方式并不能实现。

## Sprite

我们需要用到一个threejs的特殊的载体， [Sprite](https://threejs.org//docs/index.html#api/zh/objects/Sprite)
Sprite是一个总是面朝着摄像机的平面，通常含有使用一个半透明的纹理。

通过这种方式我们可以轻松的把texture应用在sprite，视觉上看起来也非常的棒。

```javascript

import adcPng from 'assets/model/car.png';

let texture = new THREE.TextureLoader()
    .load(adcPng);
texture.center.set(0, 0);
texture.needsUpdate = true;

let material = new THREE.SpriteMaterial({
    map: texture,
    sizeAttenuation: true,
    transparent: true,
    alphaTest: 0.5,
    opacity: 1
});
let sprite = new THREE.Sprite(material);

car.scale.set(3.5, 3.5, .8); // 需要对sprite进行缩放
car.add(sprite);
scene.add(car);

```

---
- 我的blog: [neverland.github.io](https://neverland.github.io/)
- 我的email [enix@foxmail.com](enix@foxmail.com)
