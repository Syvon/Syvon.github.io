---
layout:     post
title:      "Buffers,BufferViews,Accessors"
subtitle:   "glTF系列教程05"
date:       2017-07-22
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - glTF
    - WebGL
---


又复习了一下glTF模型，发现了一个很好的英文教程，讲解的非常详细。没有图形学知识背景的人也可以听懂。学习的时候，就顺便翻译成中文，来和大家分享 。当然，更推荐看[英文教程](https://github.com/javagl/glTF-Tutorials/tree/master/gltfTutorial#gltf-tutorial)。


# Buffers, BufferViews, Accessors

`buffer`, `bufferView` 和 `accessor` 的示例已经 [最小glTF文件](http://iwun.github.io/2017/07/22/MinimalGltfFile/)部分中给出。 本节将更详细地解释这些概念。

## Buffers

[`buffer`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-buffer) 表示原始二进制数据的块，没有固有的结构或含义， 可以通过 `uri` 来引用这个数据。 此URI可以指向外部文件，也可以指向直接在JSON文件中对二进制数据进行编码的 [data URI](http://iwun.github.io/2017/07/21/gltf-BasicGltfStructure#binary-data-in-buffers)。[最小glTF文件](http://iwun.github.io/2017/07/22/MinimalGltfFile/) 包含一个 `buffer`示例，它包含42个字节的，编码在 [data URI](http://iwun.github.io/2017/07/21/gltf-BasicGltfStructure#binary-data-in-buffers)中的数据，

```javascript
"buffers" : {
  "buffer0" : {
    "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAAAAgD8AAAAA",
    "byteLength" : 42
  }
},
```


![java-javascript](/img/in-post/gltf/buffer.png)
<small class="img-hint">图 5a: buffer 数据</small>


`buffer` 的部分数据可以作为顶点属性、索引，或者包含蒙皮信息、动画关键帧的数据。传递到渲染器， 为了能够使用该数据，需要关于该数据的结构和类型的附加信息。


## BufferViews

从 `buffer` 构造数据的第一步是使用 [`bufferView`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-bufferView)对象。 `bufferView` 表示一个缓冲区的数据的“切片”， 此切片使用偏移和长度（以字节为单位）定义。 [最小glTF文件](http://iwun.github.io/2017/07/22/MinimalGltfFile/)中定义了两个 `bufferView` 对象：

```javascript
"bufferViews" : {
  "indicesBufferView" : {
    "buffer" : "buffer0",
    "byteOffset" : 0,
    "byteLength" : 6,
    "target" : 34963
  },
  "positionsBufferView" : {
    "buffer" : "buffer0",
    "byteOffset" : 6,
    "byteLength" : 36,
    "target" : 34962
  }
},
```

如图所示：第一个`bufferView`引用缓冲区数据的前6个字节，第二个`bufferView`引用剩余的36个字节，


![java-javascript](/img/in-post/gltf/bufferBufferView.png)
<small class="img-hint">图 5b: 指向部分 buffer 的buffer views</small>


每个 `bufferView` 包含一个 `target` 属性. 这个属性稍后可以被渲染器用来确定数据的类型和属性。 `target` 可以是常量，表示数据用于顶点属性（34962，代表`GL_ARRAY_BUFFER`），或者数据用于顶点索引（34963，代表`GL_ELEMENT_ARRAY_BUFFER`）。

此时，`buffer`数据已被分成多个部分，每个部分由一个`bufferView`描述。 但为了在渲染器中真正使用此数据，还需要有关数据类型和布局的其他信息。


## Accessors

一个 [`accessor`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-accessor)对象引用 `bufferView`，并且包含定义此`bufferView`的数据类型和布局的属性。

### 数据类型

accessor的数据类型信息编码在 `type` 和 `componentType` 属性中。 `type` 属性的值是一个字符串，指定数据元素是标量，向量还是矩阵。例如， `"SCALAR"` 表示标量值； `"VEC3"` 表示三维矢量；`"MAT4"` 表示 4x4 矩阵；

`componentType` 指定数据分量的类型。这是一个GL常量，比如 `5126` (`GL_FLOAT` ) 表示浮点数； `5123` (`GL_UNSIGNED_SHORT`)表示无符号短整型。

这些属性的不同组合可用于描述任意数据类型。 例如， [最小 glTF 文件](http://iwun.github.io/2017/07/22/MinimalGltfFile/)包含两个`accessor`：

```javascript
"accessors" : {
  "indicesAccessor" : {
    "bufferView" : "indicesBufferView",
    "byteOffset" : 0,
    "componentType" : 5123,
    "count" : 3,
    "type" : "SCALAR",
    "max" : [ 2.0 ],
    "min" : [ 0.0 ]
  },
  "positionsAccessor" : {
    "bufferView" : "positionsBufferView",
    "byteOffset" : 0,
    "componentType" : 5126,
    "count" : 3,
    "type" : "VEC3",
    "max" : [ 1.0, 1.0, 0.0 ],
    "min" : [ 0.0, 0.0, 0.0 ]
  }
},
```

第一个accessor引用 `"indicesBufferView"`，它的 `type` 为 `"SCALAR"`， `componentType` 为 `5123` (`GL_UNSIGNED_SHORT`)。这表示该索引以无符号短整型的标量形式存储。第二个accessor引用 `"positionsBufferView"`，它的 `type` 为 `"VEC3"`， `componentType` 为  `5126` (`GL_FLOAT`)。这个访问器描述了一个分量为浮点类型的三维矢量。下图说明了如何使用`bufferView` 对象构造缓冲区的原始数据，并使用 `accessor`对象扩展数据类型信息：


![java-javascript](/img/in-post/gltf/bufferBufferViewAccessor.png)
<small class="img-hint">图 5c: accessors 定义了如何解析 buffer views</small>

### 数据布局

`accessor` 的附加属性还指定数据布局。 `accessor`的 `count` 属性表示它包含多少个数据元素。 在上面的例子中，两个 `accessor` 的`count`都为3，分别代表三角形的3个索引和3个顶点。

每个`accessor`也可以定义 `byteOffset` 和 `byteStride` 属性。 对于作为结构数组存储（Array-Of-Structures）的数据，这些属性是必需的：例如单个 `bufferView` 可以以交织方式存储顶点位置的数据和顶点法线的数据。 `byteOffset`定义第一相关数据元素的开始。 `byteStride`定义直到下一个相关数据元素开始的字节数：


![java-javascript](/img/in-post/gltf/aos.png)
<small class="img-hint">图 5d: 访问器的偏移和步幅</small>


### 数据内容

`accessor`还包含汇总其数据内容的`min` 和 `max`属性。 它们是包含在访问器中的所有数据元素分量的最小值和最大值。 对于顶点位置的情况，`min` 和 `max`属性因此定义对象的包围框。 这对于确定下载的优先级或可见性检测很有用。 一般来说，此信息对于存储和处理量子化数据（*quantized* data）也非常有用（量子化数据在运行时被渲染器解析），但是该量子化的细节超出了本教程的范围。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
