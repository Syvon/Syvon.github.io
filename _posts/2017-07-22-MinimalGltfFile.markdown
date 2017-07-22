---
layout:     post
title:      "最小的GLTF文件"
subtitle:   "GLTF系列教程03"
date:       2017-07-22
author:     "WunWun"
header-img: "img/in-post/gltf/gltf.png"
tags:
    - JavaScript
---


又复习了一下GLTF模型，发现了一个很好的英文教程，讲解的非常详细。没有图形学知识背景的人也可以听懂。学习的时候，就顺便翻译成中文，来和大家分享 。当然，更推荐看[英文教程](https://github.com/javagl/glTF-Tutorials/tree/master/gltfTutorial#gltf-tutorial)。


# 一个最小的 glTF 文件

下面的glTF资源包含一个单一的三角形，虽然小但是完整。 您可以将其复制并粘贴到一个`gltf` 文件中，每个基于glTF的应用程序都应该能够加载和渲染它。 本节将基于这个例子，解释glTF的基本概念。

```javascript
{
  "scene" : "scene0",
  "scenes" : {
    "scene0" : {
      "nodes" : [ "node0" ]
    }
  },
  "nodes" : {
    "node0" : {
      "meshes" : [ "mesh0" ]
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

  "buffers" : {
    "buffer0" : {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAAAAgD8AAAAA",
      "byteLength" : 42,
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
    }
  },
  "asset" : {
    "version" : "1.0"
  }
}
```

<p align="center">
<img src="images/triangle.png" /><br>
<a name="triangle-png"></a>图 3a: 一个三角形
</p>


##  `scene` 结构和 `nodes` 结构

 [`scene`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-scene) 是用于描述存储在glTF中的场景的入口点。 当解析glTF JSON文件时，场景结构的遍历将从这里开始。 每个场景包含一个名为 `nodes` 的数组，其中包含 [`node`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-node) 对象的ID。这些节点是场景图层次结构的根节点。

这里的示例包含单个场景 `"scene0"` ，它引用示例中唯一的节点 `"node0"` ，该节点又引用唯一的网格 `"mesh0"`:


```javascript
  "scenes" : {
    "scene0" : {
      "nodes" : [ "node0" ]
    }
  },
  "nodes" : {
    "node0" : {
      "meshes" : [ "mesh0" ]
    }
  },
```

关于场景和节点及其属性的更多细节将在关于 [场景和节点](2017-07-22-ScenesNodes.markdown) 部分中介绍。

## `meshes`

 [`mesh`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-mesh) 表示出现在场景中的实际几何对象。 mesh本身通常没有任何属性，只包含一个 [`mesh.primitive`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-mesh.primitive) 对象的数组，用来构建较大模型。 每个网格图元（mesh.primitive）包含几何数据的描述。

该示例由ID为 `"mesh0"`的单个网格组成，并且具有单个 `mesh.primitive` 对象。 网格图元（mesh.primitive）有一个 `attributes`数组。数组中保存着网格几何的顶点的属性，在这个例子中，只有 `POSITION` 属性，用来描述顶点的位置。 网格图元描述了 *索引* 几何，其通过对 `indices` 的引用来指示。 默认情况下，假设描述一组三角形，三个连续的索引是一个三角形的顶点的索引。

网格图元的实际几何数据由 `attributes` 和 `indices` 给出。 这两者都引用 `accessor` 对象，这将在下面解释。

```javascript
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
```

关于网格和网格图元的更详细的描述可以 [Meshes](2017-07-22-Meshes.markdown) 部分找到。

##  `buffer`， `bufferView` 和 `accessor` 的概念

`buffer`， `bufferView` 和 `accessor` 对象提供关于网格图元所包括的几何数据的信息。基于具体示例，在此快速介绍它们。 关于这些概念的更详细的描述将在 [Buffers, BufferViews and Accessors](2017-07-22-BuffersBufferViewsAccessors.markdown) 部分中给出。

### Buffers

[`buffer`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-buffer) 定义了没有固有含义的原始非结构化数据块。 它包含了一个 `uri`，可以指向包含数据的外部文件，也可以是直接在JSON文件中对二进制数据进行编码的 [data URI](2017-07-21-gltf-BasicGltfStructure.markdown#binary-data-in-data-uris) 。

在这个例子中，用的就是第二种方法： `"buffer0"` 中包含编码为 `data URI` 的42字节数据。

```javascript
  "buffers" : {
    "buffer0" : {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAACAPwAAAAAAAAAAAAAAAAAAgD8AAAAA",
      "byteLength" : 42
    }
  },
```

该数据包含三角形的索引信息和三角形的顶点位置信息。 但是要将这个数据作为网格图元的几何数据使用，就需要关于该数据的结构的附加信息。 关于结构的附加信息被编码在 `bufferView` 和 `accessor` 对象中。

### Buffer views

[`bufferView`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-bufferView) 描述了原始缓冲数据中的一块数据。在这个例子中，有两个 bufferView，它们都引用同一个 `buffer`。第一个bufferView被称为 `"indicesBufferView"`，它引用缓冲区中包含索引数据的部分：byteOffset为0，byteLength为6.第二个bufferView被称为 `"positionsBufferView"`，它引用缓冲区中包含顶点位置的部分：byteOffset为6，byteLength为36，也就是说，它扩展到整个缓冲区的末尾。 

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


### Accessors

构造数据的第二步是使用 [`accessor`](https://github.com/KhronosGroup/glTF/tree/master/specification#reference-accessor) 对象来完成。 它们通过提供关于数据类型和布局的信息，来定义如何解释 `bufferView` 的数据。

在这个例子中，有两个 `accessor`对象。

`"indicesAccessor"` 描述了几何数据的索引。 它引用“indicesBufferView”，后者是包含索引的原始数据的 `buffer` 的一部分。 此外，它指定元素的 `count` ， `type` 及其 `componentType`。 在这个例子中，有3个标量元素，它们的组件类型由一个常数表示，该常量表示`unsigned short` 类型。

`"positionsAccessor"` 描述几何数据的顶点位置，它通过 `"positionsBufferView"`来引用原始数据 `buffer` 的相应部分。在这个例子中，有3个矢量元素，每个元素包含3个浮点数。

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

如上所述， `mesh.primitive` 现在可以使用它们的ID来引用这些访问器：

```javascript
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
```

当要渲染 `mesh.primitive` 时，渲染器可以解析底层缓冲区视图和缓冲区，并将缓冲区的所需部分连同关于数据类型和布局的信息一起发送到渲染器。关于渲染器如何获取和处理访问器数据的更详细描述，可以参见 [Buffers, BufferViews and Accessors](2017-07-22-BuffersBufferViewsAccessors.markdown) 和 [Materials and Techniques]()




##  `asset` 

在glTF 1.0中，此属性仍然是可选的，但在后续的glTF版本中，JSON文件需要包含一个含有 `version` 数字的 `asset` 属性。 这里的示例说明该资源符合glTF版本1.1：


```javascript
  "asset" : {
    "version" : "1.1"
  }
```

`asset` 也可以包含 [`asset` 标准](https://github.com/KhronosGroup/glTF/blob/master/specification/README.md#reference-asset) 中描述的其他元数据。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
