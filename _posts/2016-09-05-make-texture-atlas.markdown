---
layout:     post
title:      "How to make a texture atlas in Blender"
subtitle:   ""
date:       2016-09-05
author:     "WunWun"
header-img: "img/in-post/make-a-texture-atlas/texture-atlas.jpg"
tags:
    - Texture Atlas
---


In realtime computer graphics, a texture atlas is a large image containing a collection of sub-images, each of which is a texture map for some part of a 2D or 3D model. A Texture Atlas describes the method of packing many separate textures together into a single texture. 

In my recent project, I need load a lot of 3D models in [cesium](https://cesiumjs.org/) (A WebGL Virtual Globe and Map Engine) which greatly affect the performance of browser. So I want to pack many separate textures of a 3D building model together into a single texture.

Let's go.

0. Import a model,and create a new UV maps.

![java-javascript](/img/in-post/make-a-texture-atlas/0.png)
<small class="img-hint">create a new UV maps</small>

1. Press "Tab" key,enter the Edit Mode.

![java-javascript](/img/in-post/make-a-texture-atlas/1.png)
<small class="img-hint">enter the Edit mode</small>

2. Press "A",select all the faces of the model.

![java-javascript](/img/in-post/make-a-texture-atlas/2.png)
<small class="img-hint">select the model</small>

3. Press "u",and select the "Smart UV project",then click OK.

![java-javascript](/img/in-post/make-a-texture-atlas/3.png)
<small class="img-hint">amart UV project</small>

4. Create a new texture,and save it.

![java-javascript](/img/in-post/make-a-texture-atlas/4.png)
<small class="img-hint">create a new texture</small>

5. In the bake mode,select the "Textures",and click the Bake.You will find a texture atlas.Amazing,isn't it? Now save it.

![java-javascript](/img/in-post/make-a-texture-atlas/5.png)
<small class="img-hint">bake the texture atlas</small>

6. Delete the origin UV map.

![java-javascript](/img/in-post/make-a-texture-atlas/6.png)
<small class="img-hint">delete the origin UV map</small>

7. Delete all the origin materials.

![java-javascript](/img/in-post/make-a-texture-atlas/7.png)
<small class="img-hint">delete all the origin materials</small>

8. Create a new material.

![java-javascript](/img/in-post/make-a-texture-atlas/8.png)
<small class="img-hint">create a new material</small>

9. Create a new texture with the texture atlas which you saved before.

![java-javascript](/img/in-post/make-a-texture-atlas/9.png)
<small class="img-hint">Create a new texture</small>

10. Now you see the magic.

![java-javascript](/img/in-post/make-a-texture-atlas/10.png)
<small class="img-hint">create a new UV maps</small>

Texture atlases can greatly reduce the number of draw calls and state changes, so they are an obvious and necessary optimization. Have a try.

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
