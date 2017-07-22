---
layout:     post
title:      "动画"
subtitle:   "GLTF系列教程07"
date:       2017-07-22
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - JavaScript
---


又复习了一下GLTF模型，发现了一个很好的英文教程，讲解的非常详细。没有图形学知识背景的人也可以听懂。学习的时候，就顺便翻译成中文，来和大家分享 。当然，更推荐看[英文教程](https://github.com/javagl/glTF-Tutorials/tree/master/gltfTutorial#gltf-tutorial)。


# Animations

如[Simple animation](2017-07-22-SimpleAnimation.markdown)所示， [`animation`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-animation)可用于描述节点的`translation`, `rotation` 或 `scale`属性如何随时间变化。

下面是`animation`的另一个例子。 这一次，动画包含两个通道。 一个通道用于动画的平移，另一个通道用于动画节点的旋转：

```javascript
"animation0": {
  "channels": [
    {
      "sampler": "translationSampler",
       "target": {
        "id": "node0",
        "path": "translation"
      }
    },
    {
      "sampler": "rotationSampler",
      "target": {
        "id": "node0",
        "path": "rotation"
      }
    }
  ],
  "samplers": {
    "translationSampler": {
      "input": "timeAccessor",
      "interpolation": "LINEAR",
      "output": "translationAccessor"
    },
    "rotationSampler": {
      "input": "timeAccessor",
      "interpolation": "LINEAR",
      "output": "rotationAccessor"
    }
  }
}
```


### 动画采样器

`samplers`字典包含[`animation.sampler`](https://github.com/KhronosGroup/glTF/blob/master/specification/README.md#reference-animation.sampler)对象，它定义了访问器提供的值如何在关键帧之间插值：


![java-javascript](/img/in-post/gltf/animationSamplers.png)
<small class="img-hint">图 4a: 动画采样器</small>

可以使用以下算法，计算当前动画时间的平移量：

* 将当前动画时间赋值给`currentTime`
* 计算`timeAccessor`中的仅次于`currentTime`的值和仅大于`currentTime`的值:

    `previousTime` = `timeAccessor`中仅次于`currentTime`的值

    `nextTime`  = `timeAccessor`中仅大于`currentTime`的值

* 从`translationAccessor`中获取相应元素

    `previousTranslation` = `translationAccessor`中对应`previousTime`的元素

    `nextTranslation` = `translationAccessor`中对应`nextTime`的元素

* 计算插值（0.0~1.0）。它描述了`previousTime`和`nextTime`之间，当前时间点（`currentTime`）的 *相对* 位置：

    `interpolationValue = (currentTime - previousTime) / (nextTime - previousTime)`

* 利用插值计算当前时间的平移量

    `currentTranslation = previousTranslation + interpolationValue * (nextTranslation - previousTranslation)`


#### 例子:

假设`currentTime` 是 **1.2**。`timeAccessor`中仅次于它的值为 **0.8**，仅大于它的值为 **1.6**。所以

    previousTime = 0.8
    nextTime     = 1.6

`translationAccessor`中可以查到相应的值：

    previousTranslation = (14.0, 3.0, -2.0)
    nextTranslation     = (18.0, 1.0,  1.0)

插值计算如下：

    interpolationValue = (currentTime - previousTime) / (nextTime - previousTime)
                       = (1.2 - 0.8) / (1.6 - 0.8)
                       = 0.4 / 0.8         
                       = 0.5

从插值可以计算出平移量：

    currentTranslation = previousTranslation + interpolationValue * (nextTranslation - previousTranslation)`
                       = (14.0, 3.0, -2.0) + 0.5 * ( (18.0, 1.0,  1.0) - (14.0, 3.0, -2.0) )
                       = (14.0, 3.0, -2.0) + 0.5 * (4.0, -2.0, 3.0)
                       = (16.0, 2.0, -0.5)

因此当前时间是 **1.2**，节点的`translation`为 **(16.0, 2.0, -0.5)**



### 动画通道

动画包含 [`animation.channel`](https://github.com/KhronosGroup/glTF/blob/master/specification/README.md#reference-animation.channel)对象数组。 通道在输入（即从采样器计算的值）和输出（即动画节点属性）之间建立连接。 因此，每个通道利用ID引用一个采样器，并且包含一个 [`animation.channel.target`](https://github.com/KhronosGroup/glTF/blob/master/specification/README.md#reference-animation.channel.target)对象。 `target`利用ID引用了一个节点，并且包含一个 `path`， `path`定义了应该动画化的节点属性的路径。 来自采样器的值将写入此属性。

在上面的示例中，动画有两个通道，两者引用同一节点。第一通道的路径引用的是节点的`translation`，第二通道的路径引用的是节点的`rotation`。 因此，连接到节点的所有对象（网格）将被平移旋转：


![java-javascript](/img/in-post/gltf/animationChannels.png)
<small class="img-hint">图 4b: 动画通道</small>

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
