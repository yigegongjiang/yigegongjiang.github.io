---
title: CPU的制作
date: 2019-04-11 20:14:43
categories:
- 技术
tags:
- 计算机原理
---

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202305300013497.jpg" width="30%">

对 CPU 的一点简要说明。
如果你是对CPU的制作有一些模糊，或者希望通过其他博文来验证你的想法，那么下面一些认知或许对你有一些帮助。
大到宇宙飞船，小到PC、手机、冰箱，无一没有芯片的影子。
各位一定都对芯片有很多认知了，我们不需要多做说明，没有芯片，就没有新时代。从真空三极管到锗晶体管到硅晶体管，每一步都是一次跃迁。

### 集成电路的制作

#### 硅=>晶圆
原材料首当其冲的是高纯度硅。通过把高纯度的硅融化，用一个引子伸入容器，不断的让硅附着生长在引子上面。我们可以想象明矾的制作过程。
引子不断的往上提，最后一个很重的圆柱形硅淀就形成了。
这个圆柱形直径有10厘米+。
然后通过机器切割，从上往下，切割一个豁口或者一个边，这个豁口或者边，是为了客户进行晶圆制作的时候辨认方向用的。因为最终晶圆不是100%利用的，这个豁口或者边，是肯定不会用到的。
然后，对整个圆柱形硅切割成晶圆。每个晶圆对直径也还是10厘米+，但是厚度只有3毫米左右。这样的一片片晶圆，就是后面集成电路的原材料了。
所以，**晶圆其实是很大的一个圆盘，比普通人的一张脸，还是要大一些的**。

<!-- more -->

#### 晶圆光刻腐蚀
这个比较复杂，网上都有详细的说明。Intel和AMD也有公开视频说到一系列的复杂过程。
我不懂，我不做更多说明。大家可以自行查看。网上都是图，还是很方便理解和查看的。
不过这里也是技术难度最大的一环，是技术的巅峰。光刻机，不是一个国家自主创新能够完成的。有时候还是应该想着合作共赢，而不是喊口号和战狼外交。

#### 晶体管数量
很多人看了光刻腐蚀的图文甚至视频，认同CPU里面真的好复杂。为什么呢？很多人想，这样的操作，里面有好多好多个晶体管了吧。
其实，很多人都不敢放大自己的想象。那不是好多好多个，也不是几百万或者几千万甚至几亿个，而是几百亿个。
现在的技术，已经达到**300亿个晶体管**，在一个**指甲盖大小的CPU**里面了。
这些晶体管，就组合成了集成电路，不同数量的晶体管，组合成不同用途的寄存器等。

#### 晶圆良率
请大家一定理解，一个晶圆，可不仅仅只能光刻腐蚀出来一个Die，而是很多个Die。不然这个芯片得多大啊。
我有个同事有次和我说，一个晶圆只能制作一个芯片，这个晶圆利用率为什么不能提高？
其实，我同事就没有理解晶圆的大小。一个比脸还大的盘子，怎么可能只能做一个指甲盖大小的芯片啊！
所以，一个芯片，可以做很多个Die，每个Die，经过测试包装后，就成了CPU。
所以，这里有一个良率的问题。为什么会有良率？因为几百亿个晶体管里面，有一部分的晶体管是不能使用的，而不是全部能正常工作。
很少有晶圆，上面光刻腐蚀的几十个Die都是完好的。
良率有一个现象，越靠近中心的位置，Die的良率越高。同时，越偏离中心，Die的良率越低。（中心，即晶圆的中心）
那每个晶圆里面，肯定有一些Die里面的晶体管是损坏的。是不是这些Die就不能用了呢？
不是的。**同一批次的Die，那些晶体管损坏的产品，就相应的把这些损坏晶体管封死，变成低配的CPU。**
比如，同一批次的I7处理器，可能有些Die里面有两个Core（核心）损坏了。那么在后面测试的时候，就封死这两个核，变成I5处理器。
商人都是无利不起早的，他们为什么要这么做？因为这样做一来商家可以赚更多钱，不然那么多Die不都是扔了嘛。二来降低了用户的价格，东西多一些价格总归会将一些的。
网上有人提到过，那么有没有一个晶圆只做一个Die的呢？
有的。只是这个成本太高的。因为良率的问题，这个Die得多么精密的仪器才能够保障其晶体管不受损坏。因为这个Die占据一整个晶圆，面积大了，坏点几率也相应增加了无数倍。几乎100%。
哪些情况下会使用这样的超大Die？宇宙飞船啊。高级东西当然需要高级的配件。

#### 晶体管数量为什么目前保持300亿多一些，再也上不去了
晶体管数量越多，计算能力也就越强，电脑的处理速度也就越快。那晶体管还能不能提升到上千亿？
不是技术上上不去了，技术上一直在突破。
而是**散热上不去了**。晶体管需要通电才能工作，能量总归是守恒的。CPU内部几百亿个晶体管，总归要散热的。
因为散热真的不能那么有效，晶体管数量再多下去，就不能工作了。晶体管的工作环境温度太高了。
散热有很多计算方式，比如降低电压等方案都可以有效的降低散热量。但是目前的确已经到瓶颈了，好多年没有前进了。都在往多核、超流水线等方向发力。

#### 内存是不是也有很多晶体管？
不仅仅内存有，显卡啥的都有。
一个高速缓存芯片的一比特需要6-8个晶体管，而内存容量比较大，一个比特只需要1个晶体管，但是还需要外部元件来支持，比如需要一个电容，因为晶体管会持续放电，数据会丢失，需要电容来补电。
显卡里面有GPU。GPU和CPU本质上都是一样的，就是里面的运算处理单元因为工作模式不同，量级不同而已。
所以，他们里面都是有晶体管存在。

#### 手机都比电脑还强劲了感觉，手机里面有CPU吗？
手机里面岂止有CPU，手机里面有和PC配件一样的东西。什么CPU、GPU、内存、磁盘、驱动等等，都是齐的。
手机就是缩小版的PC。
手机里面的CPU呢，还不是单独存在的。手机厂商，会把内存、显卡、驱动、CPU等等，全部封装在一个大的芯片里面，叫贴片SOC。他们是非常集中的一体，不像PC上面想换内存条了，再买一个就好了。
___

一个人的思维，决定了一个人的高度。
一举一动，笑或者哭，走或者跳，都是通过大脑发出指令，身体才能够执行。
哪些主观行动，是不通过人的思维控制的呢？神经反射之类的我们除外，它不属于主观行动。
一个人如何学习，如何赚钱，如何理财，如何钱生钱命转命。都是他的思维驱动他的行动的。
所以，努力的提升思维吧！
很多人说的，努力比不上选择，其实努力很廉价的。人只要主观愿意，随时可以很努力。你努力了，更多的人可以比你更努力。努力决定的仅仅是下限！
思维的提升，才能有效的不让努力白费，才能更好的实现人生价值。
都是行人。
