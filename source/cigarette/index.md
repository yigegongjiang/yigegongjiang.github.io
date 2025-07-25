---
title: 日本香烟（帮带回国）
date: 2025-06-24 12:06:05
---

> 「每月回国，按例带烟，长期有效」。

> Release 1.7: 价格微调（非利润增加，完全是成本波动）
>
> - Reason：1. 七星日税也不好买，7 月份跑了 7 个便利店才买齐 20 包 2. 拼包场景，从订单整理到最后发货，增加了很多人工成本。
> - 七星日税每条增加 10 rmb(跑路成本)，日免不受影响。-> 日本便利店也断货，要去好几个店凑货
> - 和平铁盒每条增加 20 rmb(快递成本)，纸盒不受影响。-> 铁盒情况特别，后面会先囤货。这样预定就一定有货，客户体验好一些。囤货需要同事先快递给我，这里增加了快递成本。
> - 【拼包】场景，每包增加 0.5 rmb。-> 拼包比整条发，增加的复杂度超过预期：(
>
> Release 1.6: a. 增加【拼包】了。 b. 收尾款时间延后了，收货后再付款。
> Release 1.5: 有需要可[点击](https://www.yigegongjiang.com/cigarette/v1-5)查看。

> 8 月初可以预定了。8 月是 中日游客 往返最高峰，机票昂贵，回国同事少，总量也少。有需要请及早预定。

## 类型 & 价格

1. 七星 SevenStars 日税（480 rmb/条），日免（370 rmb/条）
2. 和平 Peace 纸盒款（370 rmb/条），铁盒款（660 rmb/条，限量）

### 焦油量

a. 七星 & 和平，默认均为 10mg。如预定非 10mg，一定提前说明（按以往客户预定分布，非 10mg 预定极少）。
b. 七星有 14mg、和平有 21mg，价格同 10mg。

### 价格

a. 不同类型，帮带费用均保持一致，和往期没有变化。总价格不同仅为成本价的差异导致。

    a-i. 七星日税，7 月份之前是 470 rmb，当前增加了 10 rmb。原因参见 Release 1.7。
    a-j. 和平铁盒，7 月份之前是 640 rmb，当前增加了 20 rmb。原因参见 Release 1.7。

b. 各品类在日本零售店的价目：

    b-i. SevenStars 系列 & 纸盒款 Peace：6000 日元/条。不同时间点汇率成 rmb 大约 300-310 rmb /条。
    b-j. 两款香烟，都是日本最贵的香烟。零售店一般都放在烟柜最顶层，或单独有存放区域。
    >>> 汇率口径为 Alipay 汇率，Google 检索到的国际汇率是参考标准，没有人能按照国际汇率进行换汇的。<<<

c. 价格其实很低的，转手就能...

    c-i. 免责声明：仅为表达价格低，一定请自购自用。香烟非普通商品。
    c-j. 职业代购商的价格会很高，总之就是很高很高，一两句也解释不清楚。

## 口味 & 测评

**七星 SevenStars 系列**，和国内香烟相比，核心是口味上的增强。口感更加醇厚、紧实。
a. 烟草味浓郁，和国内高端烟比较接近，网评最好，也是最符合国内抽烟习惯的一款高级香烟。海关违规通报的全都是它。
b. 实测：日税和日免，没感觉出来有啥差异。不少网评说日税的口味是最好的。

**和平 Peace 系列**，无国内香烟做对比，它主打『烟草香』。烟草叶自带香味(非后期香料)，且抽烟过后口腔没有异味。
a. 铁盒款：是天花板，除了贵，找不到缺点。
b. 纸盒款：性价比最高，铁盒款的缩减版。相比铁盒款，它的香味淡了很多。

     // Peace 香味浓郁程度
     铁盒款：开盒，扑面而来。
     纸盒款：烟在鼻子上嗅能清晰闻到香味。抽的过程中能感受到一丝香味在口腔蔓延。

c. 实测：抽过铁盒款，就回不到纸盒款了。没接触过铁盒款，那纸盒款就是非常棒的。

### 测评

a. 日税 SevenStars 在国内是最出名。
b. 日免 SevenStars & 纸盒款 Peace 性价比最高。
c. 铁盒款 Peace 是【天花板】，也是最贵 & 最难购买的。

## 预定方案

```
// 【铁盒 和平 Peace】因为量少，只能限量。
a. 有概率断货，影响很大。解决方案：囤货。有同事回国，就会请帮忙带铁盒。
b. 每位顾客能预定的量不多，但每位顾客一定能预定到铁盒。
c. 限量公式：m = floor(n / 7) ，简单理解：逢 7 加 1，每 7 包可预定 1 盒铁盒。
  - m: 铁盒数量（单位：盒），n：预定总数量(单位：包)，floor：向下取整。
  - n = sum(拼包数量 + 整条数量*10)。一条香烟包含 10 包。
  - e.g.：6÷7=0、7÷7=1、8÷7=1、14÷7=2、20÷7=2
```

### 整条预定

无注意事项，提前预定即可。

### 拼包预定

> 拼包即『按包预定』，对新客户友好，可以熟悉不同香烟的口味。

a. 单价 = 一条价格 / 10.0 + ¥0.5。（即：按条折价。例和平 Peace：370/10 + 0.5 = 37.5）

    a-i. 相比 7 月份，每包增加 0.5 rmb。拼包订单，从整理到发货，增加了很多复杂度 & 人工成本，希望理解。

b. 总拼包数量 >= 4
c. 拼包就不接受退换货啦。

    c-i. 一直以来，只要客户觉得不满意，我这边就会安排退货退款的，拆包装了也没问题，退回来我自己抽。退款也是我出运费、按包折价。
    c-j. 到目前为止，还没有出现退货退款案例。但是有对货不喜欢的案例，暂无退货完全是托客户的厚爱了（感谢支持）。
    >>> 你们都想象不到我听闻客户不满意货后，主动退货退款是多么的努力，抽着不喜欢的香烟，是非常难过的事情。<<<

### 预定建议

新老客户都建议使用【拼包】熟悉不同种类的口味。完全可以在【整条】预定的同时，叠加一些【拼包】。

    // !!以下示例完全是参考作用，具体数量 & 焦油量等，完全自行决定。

    示例 0：整条 1 + 拼包 10 包
    => 纸盒 Peace (1条) + 日税七星 (2) + 日免七星 (6) + 铁盒 Peace (2)
    => 370*1 + 48.5*2 + 37.5*6 + 66.5*2 = 825 rmb

    示例 1：拼包 8 包
    => 日免七星 (3) + 纸盒 Peace (4) + 铁盒 Peace (1)
    => 37.5*3 + 37.5*4 + 66.5*1 = 329 rmb

    示例 2：拼包 16 包
    => 日税七星 (2) + 日免七星 (5) + 纸盒 Peace (7) + 铁盒 Peace (2)
    => 48.5*2 + 37.5*5 + 37.5*7 + 66.5*2 = 680 rmb

## 下单流程

欢迎补仓 🌹

快递费用说明：
a. 快递费用需要客户承担（顺丰到付 **12-18 rmb**），一个包裹最多 2.2 条（22 包），多出数量需要拆包裹。
b. 快递默认无纸箱，小概率香烟会有挤压，不影响食用。需要纸箱外包装请提前告知，增加 1-2 元 到付 包装费。

预定流程说明：

1. 通过 Email 预定（无定金），Email：<strong><em>one.gongjiang&#64;gmail.com</em></strong>
   a. 回复：当前是否有车位，或者下一趟时间
   b. 如确认预定，请告知收件地址
2. 回国后顺丰发货，大部分地区都是次日或者次次日抵达
3. 回日本后，再联系客户收取尾款事宜(alipay)。此时大概率已经抽上了 ：）

以上交易为【君子约定】，因为信任，所以简单。

收尾款时间非常延迟，属于 0 风险购物，『慢节奏的 Email』可以代替『快节奏的 IM』完成交易流程了。如有需要，依旧可以通过 IM 联系的。
每次回国都很忙，就不主动联系客户、告知快递单号了。家里没电脑，手机操作很不方便。
返回日本后，再联系收取尾款事宜，大家相互信任 ：）

近一年无违约案例产生，感谢往期顾客。如不愿遵守流程约定，请不要主动预定。

## 信息补充

- 七星 SevenStars 有 14mg 焦油量，分日税和日免。可直接联系，价格同 10mg。
- 和平 Peace 有 21mg 黄色纸盒款。可直接联系，价格同 10mg。
- 老顾客 75%+ 会回购 Peace 纸盒款。主要是它的口感在国内没有平替。SevenStars 也是巅峰，有中华等作为平替。
- Q：怎么判断是正品？A：日本所有香烟，用物品轻碰燃烧点，燃烧点会轻松掉落（灭烟）。燃烧到烟屁股后，燃烧点会自然掉落。
- Q：有生产日期吗？A：普遍都没有。日本没有囤货场景，小到餐饮大到汽车。判断是正品，既可以保证生产日期是很靠近的。
- Q：七星的日税、日免是什么？A：Peace 和 七星 生产地都在日本，但 七星 在不同海关使用不同的外包装，就有了不同的名称。
- Q：铁盒 Peace 为啥难带？A：重（1.4KG）、体积大、廉价航空、旅途累、商场缺货、没空间、汇率波动...
- Q：你抽的啥？A：我有15年烟龄，目前只抽七星 & 和平，量分布：30% 日税、20% 中免、30% 铁盒、20% 纸盒。
- Q：除了烟，其他能带吗？A：能带，成交量不多，大多被劝退了。拼多多等海购平台很棒了，价格还便宜，到货时间也快。
- Q：药能带吗？A：处方药不行。日本药妆店里面的，可以带。

## 图资料

![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250722141618043.jpg)

---

做这个业务回血机票费用已经一年多了，虽然客户不多，但回的血都是实打实的，感谢所有顾客。
到目前没有出现【尾款违约】、【退货退款】案例，完全托各位 IT 行业客户厚爱了，感谢大家。

> IT 行业的伙伴们，依旧是繁杂社会中，最保持初心的群体。
> 工作再忙，请多休息。道阻且艰，与君共勉。
