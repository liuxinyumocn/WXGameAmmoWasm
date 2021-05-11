# 微信小游戏中使用Ammo (WASM版)


## 简单了解WebAssembly

微信小游戏目前已经支持 WebAssembly 能力，可能有的小伙伴还不知道什么 WebAssembly（不过能点开本文的应该都已知晓），出于本文结构完整性将先简单介绍。

WebAssembly（简写 WASM ）是一种可以在浏览器环境中以二进制形式直接进入设备内存执行的浏览器脚本运行方式。以浏览器脚本 JavaScript 为例，JS 在浏览器中运行过程可以理解为：

**下载脚本文件(xxx.js) -> 浏览器词法分析(解释) -> 解释器执行**

而由于 WASM 文件已经在最初编写后进行了文件编译，因此很明显将直接跳过词法分析过程，直接进入执行，实际上 WASM 的执行也是基于与内存的直接交互，相对于 JS 而言，运行效率将会更高。

简单一句话，WASM 是使用编译型语言编写，在浏览器内沙箱环境下安全的运行的类原生脚本。WASM 比 JS 有更快的计算速度，因此使用 WASM 来为浏览器承担大量计算的工作是从软件构架角度优化上很不错的选择。

不过对于 Web 开发者不要担心，本文所介绍内容不需要开发者掌握非 JS/TS 语言，仅在说明如何使用目前已提供好的相关版本库的文件。



## Ammo.js & Ammo.wasm

目前知名物理引擎 Bullet 3D 已经提供对应的 JS 版本——Ammo，并提供了 JS 与 WASM 两个版本供 Web 开发者使用。很明显物理运算是一个极耗计算资源的过程，在注重物理的3D小游戏中选用 WASM 版本的物理引擎可在不需要进行项目重构优化的情况下，有效的提升游戏的运行性能。

本文将介绍如何在微信小程序中使用 Ammo 的 WASM 版本来提升涉及物理相关内容的游戏性能。

**注：本文不涉及非 JS/TS 编程语言，Web 开发者放心实践。**

有关 Ammo 网络中资料较少，其中 Ammo 比较重要的开源仓库地址为：https://github.com/kripken/ammo.js 官方的资料请自行访问查阅，在该仓库中 builds 文件夹中提供了对应的3个文件即：

- ammo.js
- ammo.wasm.js
- ammo.wasm.wasm

其中 ammo.js 为 Ammo 的 JS 版本，可单独使用，其余两个文件分别是 Ammo 的 WASM 文件以及用于适配 WASM 的胶水文件，在使用 WASM 版本时这两个文件需要同时使用。



## 微信小游戏Ammo.wasm使用说明

从上述仓库中获得的 Ammo 相关文件无论是 JS 版本还是 WASM 版本在原生浏览器中均可以根据说明直接使用。在微信小游戏环境中 Ammo.js 版本可以直接使用，本文将不再介绍，但是在使用 WASM 版本时相信很多开发者使用时将遇到很多困难，本文将给出可用的解决方案，演示 Demo（ Three.js 作为渲染引擎）在本文仓库 **source/threejs_ammowasm** 内，可自行下载体验（附效果图一张）。

<img src="https://github.com/liuxinyumocn/WX3DPhysicsTest/blob/master/image/image-20210410210434254.png?raw=true" alt="image-20210410210434454" style="zoom: 25%;" />

核心插件在 **build** 文件见内，自行下载引入到项目目录内，使用时需使用 **weapp-adapter.js** 类库，自行引用。



### 使用说明：

1.下载本文仓库 **build** 内 **ammo_wasm** 文件夹内容至项目目录（本文以根目录为例）

2.自行引入 **weapp-adapter.js** 文件

3.在合理的地方开始加载 **Ammo_wasm**，代码如下：

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

加载成功后 **window.Ammo** 作为全局资源可直接使用，使用方式与 Ammojs 版本完全一致，值得注意的是 WASM 文件路径必须是微信小游戏中的项目包内绝对路径，不可为相对路径或网络资源，不支持动态加载。



## 性能表现

使用 WebAssembly 技术来承担应用内占用算力的模块可有效提升性能，本文使用 Three.js 作为渲染引擎，构建一个场景内刚体逐渐增多的实验场景，观察两个指标的在不同阶段下的数值变化。

指标一：FPS（Frames Per Second），指游戏画面每秒的渲染帧数，通常而言游戏画面稳定的60FPS及以上则表明该游戏渲染流畅。

指标二：物理引擎单步模拟（stepSimulation）耗时，引擎在渲染前需要进行一次场景各个物体的物理属性计算，本文将对每一次计算的前后进行时间戳标记，从而得到每次物理计算的过程中的耗时，场景中的物理随时间推移刚体变多且复杂，由此观测各个环境下的物理运算性能。

本文选用小米10（1档 Android 机型）与iPhone11 Pro Max（1档 iOS 机型）进行测试，并使用 PC 中原生浏览器 Chrome 作为对照组，分别进行 JS 版本与 WASM 版本物理引擎的两个指标的采样，采样间隔为50个方块1组，下表的数据项中如 60FPS - 1ms 指在生成50个方块（刚体）时，渲染帧率为 60FPS，单步物理计算耗时为1毫秒（数据为时刻附近均值）。“ - ”数据代表数据效果较差，不做取样。

| 设备-运行环境              | 50Cube (FPS - ms) | 100        | 150       | 200         | 400         | 800         |
| -------------------------- | ----------------- | ---------- | --------- | ----------- | ----------- | ----------- |
| PC Chrome - JS             | 60FPS - 1ms       | 60 - 1     | 60 - 2    | 60 - 4      | 60 - 7      | 45 - 22     |
| PC Chrome - WASM           | 60 - 1            | 60 - 1     | 60 - 1    | 60 - 2      | 60 - 4      | **57 -13**  |
| iPhone11 Pro Max WX - JS   | 60 - 15           | 25 - 37    | 16 - 63   | 10 - 88     | 4 - 230     | -           |
| iPhone11 Pro Max WX - WASM | 60 - 2            | **60 - 5** | **60 -8** | **49 - 13** | **22 - 28** | **9 - 66**  |
| 小米10 WX - JS             | 60 - 2            | 60 - 5     | 60 - 6    | 60 - 8      | 60 - 12     | **52 - 16** |
| 小米10 WX - WASM           | 60 - 1            | 60 - 4     | 60 - 6    | 60 - 5      | 60 - 7      | 51 - 12     |

由上述实验可知，物理引擎使用 WASM 版相比于 JS 版可有效提升性能。我们重点观察小于 60FPS 时（60FPS 时存在算力过盛因此不做对比），物理引擎的单步模拟耗时在场景内刚体较多时均有不同程度的速度提升，在 iOS 设备中，由于小游戏运行环境没有 JIT，渲染帧率提前开始下降，但可以观察到，使用 WASM 可大幅度提升性能。

详细的性能测试过程与总结请阅读 [几款游戏引擎在微信小游戏中的3d物理性能测试报告](https://github.com/liuxinyumocn/WX3DPhysicsTest) https://github.com/liuxinyumocn/WX3DPhysicsTest

## 总结&鸣谢

微信小游戏中 WebAssembly 全局对象与浏览器不同，目前版本为 WXWebAssembly，且不支持 compile方式加载 WASM，Ammojs仓库提供的 WASM 版本无法直接在小游戏环境中使用，请使用本文提供的相关文件，此处鸣谢 Cocos Creator 团队开源的项目仓库，本文复用 Cocos Creator 仓库部分代码以及 WASM 文件最终适配运行。