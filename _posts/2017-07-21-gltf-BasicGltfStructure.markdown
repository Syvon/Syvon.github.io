---
layout:     post
title:      "GLTF 基本结构"
subtitle:   "GLTF系列教程02"
date:       2017-07-21
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - JavaScript
---


# glTF 的基本结构

glTF的核心是一个JSON文件。 此文件描述3D场景的全部内容。它通过定义场景图的节点层次结构来描述场景结构。 使用附加到节点的网格（meshes）定义场景中出现的3D对象。 材质（Materials ）定义对象的外观。 动画（Animations ）描述了3D对象如何随时间变换（例如，旋转到平移），并且皮肤（skins ）定义基于骨架姿势对象的几何如何变形。 摄像机描述了渲染器的视图配置。

## JSON 结构

场景对象存储在JSON文件的字典中。 可以使用ID来访问它们，该ID是字典的键：

```javascript
"meshes": {
    "FirstExampleMeshId": { ... },
    "SecondExampleMeshId": { ... },
    "ThirdExampleMeshId": { ... }
}
```


这些ID也用于定义对象之间的 *关系*。 上面的示例定义了多个网格（meshes），节点可以使用网格ID来引用其中一个，表示网格应当附接到该节点：

```javascript
"nodes:" {
    "FirstExampleNodeId": {
        "meshes": [
            "FirstExampleMeshId"
        ]
    },
    ...
}
```

下面的图片（改编自 [glTF 概念部分](https://github.com/KhronosGroup/glTF/tree/master/specification#concepts)）大致描述了glTF的顶层元素：


![java-javascript](/img/in-post/gltf/gltfJsonStructure.png)
<small class="img-hint">图 2a: glTF 的 JSON 结构</small>


这些元素在此快速总结，以提供一个大概的了解，其中包含指向glTF规范各个部分的链接。 这些元素之间关系的更详细的解释将在以下部分给出。

- [`scene`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-scene) 是用于描述存储在glTF中场景的入口点。它引用了定义场景图的 `node`s 。
- [`node`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-node) 是场景图层次结构中的一个节点。  它可以包含一个变换（如旋转或平移），可以引用其他（子）节点。另外，它可以引用附加到该节点的 `mesh` 或者 `camera` 实例，或者描述网格变形的 `skin` 。
- [`camera`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-camera) 定义用于渲染场景的视图配置。
- [`mesh`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-mesh) 描述了出现在场景中的几何对象。 它引用用于访问实际几何数据的 `accessor` 对象，以及在渲染时定义对象外观的 `material` 对象。
- [`skin`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-skin) 定义了顶点蒙皮所需的参数，它允许基于虚拟角色的姿势的网格变形。 这些参数的值从 `accessor` 获得。
- [`animation`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-animation) 描述了某些节点的变换（如旋转或平移）如何随时间变化。
- [`accessor`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-accessor) 是任意数据的抽象的来源。 它由 `mesh`, `skin` 和 `animation` 使用，并提供几何数据，蒙皮参数和依赖于时间的动画值。 它引用 [`bufferView`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-bufferView), 后者是包含实际原始二进制数据的 [`buffer`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-buffer) 的一部分。
- [`material`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-material) 包含定义对象外观的参数。 它可以引用将被应用于对象的 `texture` ，它也可以引用用给定材料来渲染对象的 `technique` 。 [`technique`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-technique) 指的是包含用于渲染对象的GLSL顶点和片段 [‘着色器‘](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-shader) 的 [`program`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-program) 。  
- [`texture`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-texture) 由一个 [`采样器`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-sampler) 和一个 [`图像`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-image) 定义。 `采样器` 定义了 `图像` 应如何放置在对象上。



## 引用外部数据

二进制数据，如3D对象的几何和纹理，通常不包含在JSON文件中。 相反，它们存储在专用文件中，JSON文件中仅包含指向这些文件的链接。 这允许二进制数据以非常紧凑并且可以有效地在web上传送的形式存储。 另外，数据可以以直接在渲染器中使用的格式存储，而不必解析，解码或预处理数据。   


![java-javascript](/img/in-post/gltf/gltfStructure.png)
<small class="img-hint">图 2b: glTF结构</small>

如上图所示，有三种类型的对象可能包含到外部资源的链接，即 `buffers`， `images` 和 `shaders`。 稍后将更详细地解释这些对象。



### 读取和管理外部数据

读取和处理glTF资源从解析JSON结构开始。 解析完毕后， `buffers`， `images` 和 `shaders` 可用做字典。 字典的键是 IDs, 值分别是相应的 [`buffer`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-buffer), [`image`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-image) 和 [`shader`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-shader) 对象.    

每一个对象都包含URI形式的链接。 为了进一步处理，由这些URI引用的数据必须被读入存储器。 通常它将被存储在字典（或映射）数据结构中，使得它可以使用它所属的对象的ID来查找。


###  `buffers` 中的二进制数据

A [`buffer`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-buffer) 含有指向包含原始二进制缓冲区数据文件的URI：

```javascript
"buffer01": {
    "byteLength": 12352,
    "type": "arraybuffer",
    "uri": "buffer01.bin"
}
```

该二进制数据仅仅是从 `buffer` 的URI读取的存储器的原始块，没有固有含义或结构。  [Buffers, BufferViews and Accessors](gltfTutorial_007_BuffersBufferViewsAccessors.md)部分将介绍如何使用有关数据类型和数据布局的信息扩展原始数据。利用该信息，数据的一部分可能被解释为动画数据，另一部分可能被解释为几何数据。相比于在JSON格式中存储，以二进制形式存储数据可以更有效地在Web上传输，并且二进制数据可以直接传递到渲染器，而不必对其进行解码或预处理。



###  `images` 中的图像数据

[`image`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-image) 引用了可以用作渲染对象的纹理的外部图像文件：

```javascript
"image01": {
    "uri": "image01.png"
}
```

引用通常是指向PNG或JPG文件的URI。 这些文件格式显著减小文件的大小，以便它们可以有效地通过web传输。




###  `shaders` 中的GLSL着色器

顶点或片段 [`着色器`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-shader) 包含一个URI，URI指向包含着色器源代码的文件：

```javascript
"fragmentShader01": {
    "type": 35632,
    "uri": "fragmentShader01.glsl"
}
```

着色器源代码以纯文本格式存储，因此它可以直接使用WebGL，OpenGL或任何其他能够解释GLSL着色器的图形API进行编译。


## URIs 中的二进制数据

通常，`buffer`, `image` 和 `shader` 对象中的URI将指向包含实际数据的文件。作为替代，可以通过使用 [data URI](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs) 以二进制格式将数据 *嵌入* 到JSON中。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
