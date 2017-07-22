---
layout:     post
title:      "简单的动画"
subtitle:   "GLTF系列教程06"
date:       2017-07-22
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - JavaScript
---


又复习了一下GLTF模型，发现了一个很好的英文教程，讲解的非常详细。没有图形学知识背景的人也可以听懂。学习的时候，就顺便翻译成中文，来和大家分享 。当然，更推荐看[英文教程](https://github.com/javagl/glTF-Tutorials/tree/master/gltfTutorial#gltf-tutorial)。


# 一个简单的动画

如前面章节 [Scenes and Nodes](http://iwun.github.io/2017/07/22/ScenesNodes/)所述，每个节点可以有一个局部变换。这个变换可以由节点的 `matrix` 属性给出，或者是 `translation`，`rotation`和`scale`(TRS)属性。

当变换由TRS属性给出时, 可以用[`animation`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-animation)描述，节点的 `translation`, `rotation` 或 `scale`属性如何随时间改变
以下是先前展示的 [最小glTF文件](http://iwun.github.io/2017/07/22/MinimalGltfFile/)，但是扩展了动画。本节将说明添加此动画所做的更改和扩展。


```javascript
{
  "scenes" : {
    "scene0" : {
      "nodes" : [ "node0" ]
    }
  },
  "nodes" : {
    "node0" : {
      "meshes" : [ "mesh0" ],
      "rotation" : [ 0.0, 0.0, 0.0, 1.0 ]
    }
  },
  "meshes" : {
    "mesh0" : {
      "primitives" : [ {
        "attributes" : {
          "POSITION" : "positionsAccessor"
        },
        "indices" : "indicesAccessor"
      } ]
    }
  },

  "animations" : {
    "animation0" : {
      "samplers" : {
        "rotationSampler" : {
          "input" : "timeAccessor",
          "interpolation" : "LINEAR",
          "output" : "rotationAccessor"
        }
      },
      "channels" : [ {
        "sampler" : "rotationSampler",
        "target" : {
          "id" : "node0",
          "path" : "rotation"
        }
      } ]
    }
  },

  "buffers" : {
    "buffer0" : {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAAAAgD8AAAAA",
      "byteLength" : 42
    },
    "buffer1" : {
      "uri" : "data:application/octet-stream;base64,AAAAAAAAgD4AAAA/AABAPwAAgD8AAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAD0/TQ/9P00PwAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAPT9ND/0/TS/AAAAAAAAAAAAAAAAAACAPw==",
      "byteLength" : 100
    }
  },
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
    },
    "animationsBufferView" : {
      "buffer" : "buffer1",
      "byteOffset" : 0,
      "byteLength" : 100
    }
  },
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
    },
    "timeAccessor" : {
      "bufferView" : "animationsBufferView",
      "byteOffset" : 0,
      "componentType" : 5126,
      "count" : 5,
      "type" : "SCALAR",
      "max" : [ 1.0 ],
      "min" : [ 0.0 ]
    },
    "rotationAccessor" : {
      "bufferView" : "animationsBufferView",
      "byteOffset" : 20,
      "componentType" : 5126,
      "count" : 5,
      "type" : "VEC4",
      "max" : [ 0.0, 0.0, 1.0, 1.0 ],
      "min" : [ 0.0, 0.0, 0.0, -0.707 ]
    }
  },

  "asset" : {
    "version" : "1.1"
  }

}
```


![java-javascript](/img/in-post/gltf/animatedTriangle.gif)
<small class="img-hint">图 5a: 一个动态三角形</small>


## 节点的 `rotation` 属性

示例中的唯一节点现在具有`rotation`属性。 这是一个描述旋转的四元数（[quaternion](https://en.wikipedia.org/wiki/Quaternion)），包含四个浮点数：  

```javascript
"node0" : {
  "meshes" : [ "mesh0" ],
  "rotation" : [ 0.0, 0.0, 0.0, 1.0 ]
}
```

给定值是描述“大约0度旋转”的四元数，因此三角形将以其初始方向显示。

## 动画数据

glTF JSON的顶级字典中添加了三个元素，用于对动画数据进行编码：

- `buffer` 包含原始的动画数据
- `bufferView` 引用 buffer
- `accessor` 用于向动画数据添加结构信息

### 用于原始动画数据的“`buffer` 和 `bufferView` 

glTF中添加了一个新的ID为`"buffer1"`的`buffer`，。 此缓冲区同样使用[data URI](http://iwun.github.io/2017/07/21/gltf-BasicGltfStructure#binary-data-in-data-uris)来编码100个字节的动画数据：

```javascript
"buffers" : {
  ...
  "buffer1" : {
    "uri" : "data:application/octet-stream;base64,AAAAAAAAgD4AAAA/AABAPwAAgD8AAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAD0/TQ/9P00PwAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAPT9ND/0/TS/AAAAAAAAAAAAAAAAAACAPw==",
    "byteLength" : 100
  }
},
"bufferViews" : {
  ...
  "animationsBufferView" : {
    "buffer" : "buffer1",
    "byteOffset" : 0,
    "byteLength" : 100
  }
},
```

还有一个新的ID为`"animationsBufferView"`的`bufferView`。 这个`bufferView`只是简单地引用整个动画缓冲区数据。 进一步的结构信息添加到下面描述的`accessor`对象中。

注意，也可以将动画数据附加到现有的`"buffer0"`中（它已经包含三角形的几何数据）。 在这种情况下，`"animationsBufferView"`将引用

在这个示例中，动画数据作为新的缓冲区添加，以保持几何数据和动画数据分离。


### 用于动画数据的 `accessor` 对象

添加了两个新的 `accessor` 来描述动画数据：第一个accessor（ID为`"timeAccessor"`） 描述动画关键帧的时间。它有5个元素，每个元素是一个标量浮点数（总共20个字节）。 第二个accessor的ID为`"rotationAccessor"`。在前20个字节后，有5个四维向量元素， 这些是对应于动画5个关键帧的旋转四元数。

```javascript
"accessors" : {
  ...
  "timeAccessor" : {
    "bufferView" : "animationsBufferView",
    "byteOffset" : 0,
    "componentType" : 5126,
    "count" : 5,
    "type" : "SCALAR",
    "max" : [ 1.0 ],
    "min" : [ 0.0 ]
  },
  "rotationAccessor" : {
    "bufferView" : "animationsBufferView",
    "byteOffset" : 20,
    "componentType" : 5126,
    "count" : 5,
    "type" : "VEC4",
    "max" : [ 0.0, 0.0, 1.0, 1.0 ],
    "min" : [ 0.0, 0.0, 0.0, -0.707 ]
  }
},
```

如表所示，真实动画数据由`timeAccessor` 和`rotationAccessor`提供：

|timeAccessor|rotationAccessor|含义|
|---|---|---|
|0.0| (0.0, 0.0, 0.0, 1.0 )| 0.0 秒，三角形旋转0度 |
|0.25| (0.0, 0.0, 0.707, 0.707)| 0.25 秒，三角形围绕z轴旋转90度 |
|0.5| (0.0, 0.0, 1.0, 0.0)|  0.5 秒，三角形围绕z轴旋转180度 |
|0.75| (0.0, 0.0, 0.707, -0.707)| 0.75 秒，三角形围绕z轴旋转270度 |
|1.0| (0.0, 0.0, 0.0, 1.0)| 1.0 秒，三角形围绕z轴旋转360度 |

所以这个动画描述了一个三角形在1秒钟内围绕z轴旋转大约360度。


## `animation`  

最后讲一下实际动画的部分：顶级`animations`字典包含一个ID为`"animation0"`的`animation`对象。 它由两个元素组成：

- `samplers`，描述动画数据的来源  
- `channels`，可以被想象为将动画数据的“源”连接到“目标”。

在给定的示例中，有一个采样器，即`"rotationSampler"`。 每个采样器定义输入和输出属性。 它们都指访问器对象，即上面已经描述的`"timeAccessor"`和`"rotationAccessor"`。 另外，采样器定义插值（`interpolation`）类型，在该示例中为`"LINEAR"`。

在示例中还有一个通道（`channel`）。 此频道引用`“旋转采样器”`（`"rotationSampler"`）作为动画数据的源。动画的目标编码在`channel.target` 对象中：它包含一个`id`，它引用（属性应该是动画的）节点。实际的节点属性在`path`中命名。因此这个例子表示`"node0"`节点的`"rotation"`属性是一个动画。


```javascript
"animations" : {
  "animation0" : {
    "samplers" : {
      "rotationSampler" : {
        "input" : "timeAccessor",
        "interpolation" : "LINEAR",
        "output" : "rotationAccessor"
      }
    },
    "channels" : [ {
      "sampler" : "rotationSampler",
      "target" : {
        "id" : "node0",
        "path" : "rotation"
      }
    } ]
  }
},
```

结合以上信息，这个动画对象表示如下：

在动画期间，动画值是从`rotationAccessor`获取的。它根据当前模拟时间和`rotationAccessor`提供的关键帧时间进行线性插值。然后将内插的值写入到ID为`"node0"`的节点的`"rotation"`属性中。

关于内插计算的更详细的描述和实际示例，可以参考[Animations](http://iwun.github.io/2017/07/22/Animations/)部分。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
