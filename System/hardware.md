# CPU

## 介绍

* 一个超线程的核相当于两个核，但是性能肯定比不上两个核。
* intel cpu系列：i3、i5、i7、E5。其中，型号后有k的表示可以超频

## 频率

* 主频：正常工作频率

* 睿频：根据工作情况可自动上涨的频率

* 超频：人为提升的频率，对cpu伤害大，很多cpu不支持超频

  ----------

* 主频=外频*倍频

* FSB前端总线不等于外频

> 主频低, 也意味着低功耗吧? 

## cpu架构

cpu有很多架构，这三种很常见：**ARM** 、**x86**(IA-32、i386)、 **x86_64**(x64、AMD64、intel64)

架构是一个很抽象的概念，不同cpu产商可以生产同一架构的cpu。操作系统通常有对应不同cpu架构的版本。

x86包括i386、i686等处理器，支持32位地址空间。x64支持64位地址空间。

arm采用精简集指令，x86_64和x86采用复杂集指令。

树莓派是arm架构的。arm架构有很多版本，ARMv3道ARMv7支持32位地址空间，ARMv8-A架构支持64位。

> 参考：
>
> * https://en.wikipedia.org/wiki/ARM_architecture https://en.wikipedia.org/wiki/X86 https://en.wikipedia.org/wiki/X86_64
> * https://serverfault.com/questions/610308/x86-i386-i686-amd64-i5-i7-archtecture-processor-confusion
> * https://stackoverflow.com/questions/14794460/how-does-the-arm-architecture-differ-from-x86
> * https://askubuntu.com/questions/54296/difference-between-the-i386-download-and-the-amd64

# 内存

## 关于LPDDR3与DDR4

在轻薄的笔记本中, 很难腾出一块DDR4内存的空间, 因此使用LPDDR系列, 由于Intel不支持LPDDR4, 所以一般使用LPDDR3, 如华为的MateBook 14.

然而并不是LPDDR3就比DDR4性能差, 相反要好一点, 因为LPDDR3支持双通道, DDR4也可利用双通道, 但是需要两条内存条, 在部分轻薄笔记本中是无法容纳这两条内存条的.

> 参考:[超极本的LPDDR3与笔记本的DDR4性能差多少](<http://zhongce.sina.com.cn/article/view/9040/>)

# USB接口

**USB**，是英文**U**niversal **S**erial **B**us（[通用串行总线](https://baike.baidu.com/item/通用串行总线/8635470)）的缩写，是一个[外部总线](https://baike.baidu.com/item/外部总线)标准，用于规范电脑与[外部设备](https://baike.baidu.com/item/外部设备)的连接和通讯。[Type-C](https://baike.baidu.com/item/Type-C) 是下一代USB接口.

Type-C支持正反面插入, 数据传输信号强, 支持电力传输(充电)

# 主板

* 现在主流出售的主板都集成了声卡和(以太)网卡. 也可以自己加独立声卡,网卡. 加声卡需要在BIOS中将集成声卡屏蔽掉.

  > 参考[现在的网卡和声卡都是主板自带的吗](https://zhidao.baidu.com/question/267370085735553645.html)

# 散热

* 关于风冷和水冷的介绍, 极力推荐!!!

  对于电脑散热方式，风冷和水冷哪种好？ - 浮梁卖茶人的回答 - 知乎 https://www.zhihu.com/question/57695465/answer/440467918

# 机箱

* 含有大量与主板对接的外围设备

# 装机

* 选购

  小白如何从零开始自学装机？ - 老汤的回答 - 知乎 https://www.zhihu.com/question/31362682/answer/55723321

* 参考下别人配置

  [两千以内迷你主机配置推荐](https://baijiahao.baidu.com/s?id=1621464465814192752&wfr=spider&for=pc)

* 装机视频, 极力推荐!!!

  [电脑装机教程 电脑组装视频 电脑详细diy (G4560+H110M) 核显 英雄联盟LOL评测「科技发现」](https://v-wb.youku.com/v_show/id_XMjkyMTU0ODc0NA==.html?refer=seo_operation.liuxiao.liux_00003303_3000_Qzu6ve_19042900)

# 其他

## 笔记本购买

[四机型对比](<http://detail.zol.com.cn/ProductComp_param_1211014-1212030-1233117-1257942-1268272.html>), 我还是觉得华为的matebook性价比高

优点

* 综合性能偏上
* 耗电低, 续航好
* 外观好看, 轻薄便携

缺点

* 接口少了点, 可扩展插槽少了点