# Threejs in autonomous driving -（4）不会shader也能写出高级效果

> 在openGL或者webGL用要想实现多彩绚丽的效果，必须要使用到`shader`着色器。着色器需要使用GLSL语言来编写，专业性很强门槛也比较高。我们可以通过内部挖潜来达到同样的目的。

在Threejs的文档中我们可以看到，[CanvasTexture]([https://threejs.org/docs/index.html#api/zh/textures/CanvasTexture](https://threejs.org/docs/index.html#api/zh/textures/CanvasTexture)通过这项技术我们可以把一个canvas当作texture来使用。下面我将实现一个有渐变效果的planning线来作为实例讲解。

![MacHi 2019-09-24 14-15-26.png](https://upload-images.jianshu.io/upload_images/19468284-b9cf889906a18875.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```javascript

import {MeshLine, MeshLineMaterial} from 'three.meshline';

getMaterial(width) {
        let canvas = document.createElement('canvas');
        let ctx = canvas.getContext('2d');
        let colorStop = [
            [0, 'rgba(48,95,255, 1)'],
            [1, 'rgba(40,64,134, 0)']
        ];

        let h = 64;
        let w = 128;

        canvas.width = w;
        canvas.height = h;
        ctx.lineWidth = width;

        let gradient = ctx.createLinearGradient(0, 0, w, 0);

        colorStop.forEach(([position, color]) => gradient.addColorStop(position, color));

        ctx.strokeStyle = gradient;
        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, w, h);
        ctx.shadowBlur = 40;
        ctx.shadowOffsetX = 0;
        ctx.shadowOffsetY = 0;
        ctx.shadowColor = '#00ff00';

        let texture = new THREE.Texture(canvas);

        texture.needsUpdate = true;

        let {far, near} = this.camera;
        let material = new MeshLineMaterial({
            useMap: true,
            map: texture,
            lineWidth: width,
            transparent: true,
            side: THREE.FrontSide,
            depthTest: true,
            opacity: .89
        });

        return material;
    }

draw(point) {
        let geometry = new THREE.Geometry();
        let line = new MeshLine();

        geometry.vertices = point;
        line.setGeometry(geometry);

        let mesh = new THREE.Mesh(line.geometry, this.material);

        mesh.position.setZ(.2);

        this.path = mesh;
        this.scene.add(this.path);
}
```
在safari的canvas面板中可以看到两个两个canvas实例：
![MacHi 2019-09-24 14-21-35.png](https://upload-images.jianshu.io/upload_images/19468284-0cc4c19422b29c09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 结语
  通过内部挖潜使用canvasTexture我们实现了需要用shader才能实现的效果，发散一下思维，很多的高级效果也可以使用这一方式来实现。

---
- 我的blog: [neverland.github.io](https://neverland.github.io/)
- 我的email [enix@foxmail.com](enix@foxmail.com)
