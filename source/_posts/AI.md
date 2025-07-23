---
title: AI 指北
date: 2025-06-30 22:44:14
categories:
  - 技术
tags:
  - AI
---

<br/>

整理 & 学习了 AI 相关的知识点。这里把 PPT 内容放一下，详细内容可查阅 via [https://yigegongjiang.notion.site/AI](https://yigegongjiang.notion.site/AI-2e72fbdad5bc4ee4b8f2c79cfaf927d2?pvs=4)

## 主题

<table>
<colgroup>
<col width="30%">
<col width="70%">
</colgroup>
<tr>
    <td style="text-align: center; vertical-align: middle; font-size: 1.8em; font-weight: bold;">神经网络抽象了现实世界</td>
    <td style="text-align: center; vertical-align: middle;">
        <img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010329.png" width="50%">
    </td>
</tr>
</table>

## ToC

<table>
<tr><td>1. 数学对现实的抽象</td><td>2. 二元感知机 - 1957</td><td>3. 感知机升维困境 - AI 寒冬</td></tr>
<tr><td>4. 正向传播 & 反向传播 - 1974</td><td>5. 梯度下降 - 古老的算法</td><td>6. 多层感知机 MLP - 1986</td></tr>
<tr><td>7. 卷积神经 CNN - 1998</td><td>8. 自注意力机制 - 2017</td><td>9. Transformer 流程解析</td></tr>
<tr><td>10. AI - 缸中大脑</td><td>11. Prompts 是 AI 入场券</td><td>12. 调参 - 进一步掌控 AI</td></tr>
<tr><td>13. AI 悬停点 - 2025.06</td><td></td><td></td></tr>
</table>
<!-- more -->

## 缩略图预览

<table>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010329.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010346.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010354.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010400.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010408.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010416.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010426.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010435.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010445.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010451.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010455.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010500.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010506.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010510.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010514.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010518.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010522.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010529.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010537.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010544.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010548.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010552.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010557.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010603.png" width="100%"></td>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010609.png" width="100%"></td>
<td><img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010617.png" width="100%"></td>
<td></td>
</tr>
</table>

## 大图预览

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010329.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010346.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010354.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010400.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010408.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010416.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010426.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010435.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010445.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010451.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010455.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010500.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010506.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010510.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010514.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010518.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010522.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010529.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010537.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010544.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010548.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010552.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010557.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010603.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010609.png" width="100%">
<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20250724010617.png" width="100%">

---
