---
title: apple frameworks
date: 2024-11-14 13:16:00
categories:
- 技术
tags:
- Swift
- iOS
---

动态库 & 静态库，若单独输出并接入主工程，逻辑倒很清晰。如果牵涉到多个 framework 不同类型并且相互依赖，会增加不少复杂度。
以下说明中，动态库使用 DFramework 表示，静态库使用 SFramework 表示。

# 1 framework
仅需要提供 1 个 framework，直接提供 DFramework 即可。DFramework 优先，必要的时候再提供 SFramework

# n framework
因为代码结构分组，可能需要提供多个 framework：

1.  DFramework 优先，能不提供 SFramework 就不要提供
    * 有 n 个 DFramework 就提供 n 个。相互之间做好依赖文档，App 需要根据依赖文档 **flat** 平铺接入所有需要的 DFramework。
    * 如果 b DFramework 仅仅被 a DFramework 依赖，可以将 b 打入 a DFramework 的 `frameworks`文件夹中，通过 `embed` 实现。
        * 这样就可以不再文档中说明 b DFramework 的存在，对外界透明。
        * 这种方案，最终的 app 中 framework 不是 flat 平铺的，而是具有层级结构，如 ：App - a DFramework - b DFramework
    * 如果 b DFramework 同时被 m DFramework 和 n DFramework 依赖，不能将 b 隐藏（打入 m 和 n 中）。
        * 否则，App 集成 m 和 n 的时候，b 会出现多份。如果 b 的版本不一样，将按照打包顺序只使用其中一个。
<!-- more -->
2.  如果提供 SFramework，则将所有的 framework 做成依赖，输出一份 SFramework 即可。
    * 如果有按需接入的场景，按按照场景输出 n 个 SFramework。

# 依赖开源库

一般这种场景是因为开发 framework 的同时，又使用了开源的库。这个时候如何处理开源库是个问题（开源库一般都是 源码 提供的，不讨论 闭源包 提供方式，这个和上面的场景一致）。
由于 apple 的 Swift Package Manager 三方库管理的介入，一个 module 在最终的 app 中到底是 dymanic 还是 static，已经不可控了。SPM 会自行管理，不像 cocoapods 的时候我们可以自行控制。

> 如果是源码提供者，可以增加 dymanic / static 约束。但很多源码都不会提供这个选项，而是让 SPM 自行管理采用哪一个。

在这种场景下，开发的 framework 如果对开源库有依赖，就必须考虑是否【去开源】事宜了。
另外，除了自己使用开源库，App 主工程开发者，可能也需要使用同一个开源库，这时候还会出现【版本不一致问题】。
以下场景，假设有 T 开源代码需要使用：

## 不封装，让外界接入
framework 需要依赖 T
*   如果不是强依赖，在 xcode 中使用 `-weak_framework`进行标记，这样业务也可以不接入，相当于默认不使用这个功能。
*   如果是强依赖，并且在 文档 中说明业务需要接入 x 的具体版本范围。不接入无法编译或者 app 启动闪退。

## 封装，外界无感
第一步就是需要对 T 进行【模块隔离】，否则外界也接入 T 的时候，App 主工程和我们提供的 framework 同时有 T 代码，会有问题。
虽然 xcode 在编译连接的时候，会处理好这个问题，使得同一份代码尽量在 app 中仅保留一份。
但如果开发人员使用不恰当或者配置错误，很可能导致出现两份 T 源码在项目中。
这个时候如果版本还不兼容，就会有非常大的调试和异常隐患。
模块隔离方案：**修改 Product Name，将 T 变成 TT 模块**
然后，还需要考虑将 TT 变成静态库还是动态库：

### 静态库：
这是完全去开源的方案。变成静态库后，TT 源码将打入 framework 二进制中一起提供。因为模块隔离，外部也无法调用和感知。
* 通过对 framework 二进制字符表进行分析，还是能够看到 T 的踪迹。
* 如果不做模块隔离，也能成功。主 app 也使用 T 模块的时候，链接的时候将自动保留 1 份静态代码。有版本差异的话，出现异常问题将非常难以排查。

### 动态库：
使用动态库的话，如前面【需要提供多个 framework】所描述的，有两种方案：
* TT 可以提供给业务，让业务自行接入。是 flat 平铺 结构。
* 也可以内嵌到 m framework 的 frameworks 文件夹中专门供 m framework 使用。是 内嵌 结构。
  * 如果不做模块隔离，也可以内嵌。但主 app 可能也使用了 T 模块，使得同一个工程包含 2 个版本的 T 模块，运行的时候按照 xcode 编译顺序，只有一份被使用。有版本差异的话，出现异常问题将非常难以排查。

___


