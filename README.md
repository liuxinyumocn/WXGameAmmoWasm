# 微信小游戏中使用Ammo (wasm版)


## 简单了解WebAssembly

微信小游戏目前已经支持WebAssembly能力，可能有的小伙伴还不知道什么WebAssembly（不过能点开本文的应该都已知晓），出于本文结构完整性将先简单介绍。

WebAssembly（简写wasm）是一种可以在浏览器环境中以二进制形式直接进入设备内存执行的浏览器脚本运行方式。以浏览器脚本JavaScript为例，JS在浏览器中运行过程可以理解为：

**下载脚本文件(xxx.js) -> 浏览器词法分析(解释) -> 解释器执行**

而由于wasm文件已经在最初编写后进行了文件编译，因此很明显将直接跳过词法分析过程，直接进入执行，实际上wasm的执行也是基于与内存的直接交互，相对于JS而言，运行效率将会更高。

简单一句话，wasm是使用编译型语言编写，在浏览器内沙箱环境下安全的运行的类原生脚本。wasm比js有更快的计算速度，因此使用wasm来为浏览器承担大量计算的工作是从软件构架角度的优化上很不错的选择。

不过对于Web开发者不要担心，本文所介绍内容不需要开发者掌握非JS/TS语言，仅在说明如何使用目前已提供好的相关版本库的文件。



## Ammo.js & Ammo.wasm

目前知名物理引擎Bullet 3D已经提供对应的JS版本——Ammo，并提供了js与wasm两个版本供Web开发者使用。很明显物理运算是一个极耗计算资源的过程，有关Ammo的js与wasm版本的性能对比请看官移步至本人另一篇文章（[常见游戏引擎在微信小游戏中3D物理性能测试报告](https://github.com/liuxinyumocn/WX3DPhysicsTest) https://github.com/liuxinyumocn/WX3DPhysicsTest）

本文将介绍如何在微信小程序中使用Ammo的wasm版本来提升涉及物理相关内容的游戏性能。

**注：本文不涉及非JS/TS编程语言，Web开发者放心实践。**

有关Ammo网络中资料较少，其中Ammo比较重要的开源仓库地址为：https://github.com/kripken/ammo.js 官方的资料请自行访问查阅，在该仓库中 builds 文件夹中提供了对应的3个文件即：

- ammo.js
- ammo.wasm.js
- ammo.wasm.wasm

其中 ammo.js 为Ammo的JS版本，可单独使用，其余两个文件分别是Ammo的wasm文件以及用于适配wasm的胶水文件，在使用wasm版本时这两个文件需要同时使用。



## 微信小游戏Ammo.wasm使用说明

从上述仓库中获得的Ammo相关文件，在微信小游戏环境中Ammo.js版本可以直接使用，本文将不再介绍。但是在使用wasm版本时相信很多开发者将踩坑无数，本文将给出可用的解决方案，演示Demo（Three.js作为渲染引擎）在本文仓库 **source/threejs_ammowasm** 内，可自行下载体验（附效果图一张）。

<img src="https://github.com/liuxinyumocn/WX3DPhysicsTest/blob/master/image/image-20210410210434254.png?raw=true" alt="image-20210410210434454" style="zoom: 25%;" />

核心插件在 **build** 文件见内，自行下载引入到项目目录内，使用时需使用 **weapp-adapter.js** 类库，自行引用。



### 使用说明：

1.下载 build 内 **ammo_wasm** 文件夹内容至项目目录

2.自行引入 **weapp-adapter.js** 文件

3.在合理的地方开始加载 Ammo_wasm，代码如下：

```JavaScript
import './js/libs/weapp-adapter'
import './ammo_wasm/wx.ammo.wasm'

//others ....

const wasmFilePath = 'ammo_wasm/ammo.wasm.wasm';  //此处必须项目内绝对路径 不可为网络资源
window.Ammo.instantiate( wasmFilePath ).then(
  ()=>{//ammo wasm 装载完成
    
      new Main() //运行其他逻辑
    	
    	//使用Ammo
      var transformAux1 = new window.Ammo.btTransform();
    	//others ...
  }
)
```

加载成功后 window.Ammo 作为全局资源可直接使用，使用方式与Ammojs版本完全一致，值得注意的是 wasm 文件路径必须是微信小游戏中的项目包内绝对路径，不可为相对路径或网络资源，不支持动态加载。



## 踩坑总结

微信小游戏中 WebAssembly 全局对象与浏览器不同，目前版本为 WXWebAssembly，且不支持 compile方式加载wasm，Ammojs那个仓库提供的wasm版本无法直接使用，请使用本文提供的相关文件，此处特别鸣谢Cocos Creator团队开源的项目仓库，本人阅读Cocos Creator仓库代码以及复用其wasm文件最终得以顺利运行。