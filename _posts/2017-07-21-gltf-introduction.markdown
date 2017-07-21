---
layout:     post
title:      "GLTF 介绍"
subtitle:   "GLTF系列教程01"
date:       2017-07-21
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - JavaScript
---


又复习了一下GLTF模型，发现了一个很好的英文教程，讲解的非常详细。没有图形学知识背景的人也可以听懂。学习的时候，就顺便翻译成中文，来和大家分享 。当然，更推荐看英文教程。

## 对 glTF 的介绍

基于3D内容的应用和服务越来越多。 在线商店提供3D预览的产品配置器。 博物馆正在使用3D扫描将他们的工件数字化，并允许在虚拟画廊中探索他们的收藏品。 城市规划者使用3D城市模型进行规划和信息可视化。 教育者正在创造人体的交互式动画3D模型。 许多这些应用程序直接在Web浏览器中运行，这是可能的，因为所有现代浏览器都支持使用WebGL高效渲染。

![java-javascript](/img/in-post/gltf/applications.png)
<small class="img-hint">图 1a: 来自不同网站和应用的三维模型截图</small>

因此，在各种应用中，对3D内容有一种强烈且不断增长的需求。 在许多情况下，3D内容必须通过web传输，并且必须在客户端有效地呈现。 但是到目前为止，在3D内容创建和在运行时应用程序中的高效渲染之间存在差距。


### 3D 内容流水线

在客户端应用程序中呈现的3D内容有着不同的来源，并以不同的文件格式存储。[维基百科上的3D图形文件格式列表](https://en.wikipedia.org/wiki/List_of_file_formats#3D_graphics) 显示，3D数据有超过70种不同的文件格式，它们服务于不同的目的和应用案例。

例如，可以使用3D扫描仪获得原始3D数据。 这些扫描器通常提供单个对象的几何数据,并存储在[OBJ](https://en.wikipedia.org/wiki/Wavefront_.obj_file)， [PLY](https://en.wikipedia.org/wiki/PLY_(file_format)) 或 [STL](https://en.wikipedia.org/wiki/STL_(file_format)) 文件中。 但是这些文件格式不包含有关场景结构的信息，或者如何渲染对象。

当然也可以使用创作工具创建更复杂的3D场景。 这些工具不但可以编辑场景中的对象的3D几何，而且允许编辑场景的结构，光设置，相机和动画。 应用程序将此信息存储在自己的自定义文件格式中。 例如， [Blender](https://www.blender.org/) 将场景存储在`.blend`文件中， [LightWave3D](https://www.lightwave3d.com/) 使用`.lws`文件格式， [3ds Max](http://www.autodesk.com/3dsmax) 使用`.max`文件格式， [Maya](http://www.autodesk.com/maya) 场景存储为`.ma`文件。

为了呈现这样的3D内容，运行时应用程序必须能够读取不同的输入文件格式， 解析场景结构，并且必须将3D几何数据转换为图形API所需的格式。 3D数据必须被传送到显卡存储器，然后可以调用图形API来描述呈现过程。 因此，每个运行时应用程序必须为其想要支持的所有文件格式创建导入器，加载器或转换器：


![java-javascript](/img/in-post/gltf/contentPipeline.png)
<small class="img-hint">图 1b: 目前的3D内容流水线</small>


### glTF: 3D场景的传输格式

glTF的目标是定义一种表示3D内容的标准，这种形式适合在运行时应用程序中使用。 现有文件格式不适用于这种情况：其中一些仅包含几何数据，不包含任何场景信息。 其他设计用于在不同应用程序之间交换数据，并且它们的主要目标是保留尽可能多的关于3D场景的信息。 因此，文件通常是巨大复杂的，很难解析。 另外，几何数据可能必须被预处理，才能使得其可以利用客户端应用来呈现。

现有的文件格式都没有设计用于在网络上高效地传输3D场景并尽可能高效地渲染。 但是glTF不是“又一种文件格式”。 它是3D场景的 *传输* 格式的定义：

- 场景结构用JSON描述，非常紧凑，可以很容易地解析
- 对象的3D数据以一种可以由公共图形API直接使用的形式存储，因此没有用于解码或预处理3D数据的开销

不同的内容创建工具现在可以提供glTF格式的3D内容。 如 [图1b](#applications-png)所示，越来越多的客户端应用程序能够使用和渲染glTF。 所以glTF可以帮助弥合内容创建和渲染之间的差距： 


![java-javascript](/img/in-post/gltf/contentPipelineWithGltf.png)
<small class="img-hint">图 1c: glTF的3D内容流水线</small>


越来越多的内容创建工具将能够直接提供glTF。 或者可以使用 [Khronos glTF](https://github.com/KhronosGroup/glTF#converters) 存储库中列出的开源转换实用程序，由其他文件格式来创建glTF资产。 例如，几乎所有创作应用程序都可以以 [COLLADA](https://www.khronos.org/collada/) 格式导出其场景 因此， [COLLADA2GLTF](https://github.com/KhronosGroup/glTF/tree/master/COLLADA2GLTF) 工具可以用于将场景和模型从这些创作应用程序转换为glTF。 `OBJ`文件可以使用 [obj2gltf](https://github.com/AnalyticalGraphicsInc/obj2gltf)转换为glTF。 对于其他文件格式，自定义转换器可用于创建glTF资源，从而使3D内容可用于各种运行时应用程序。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
