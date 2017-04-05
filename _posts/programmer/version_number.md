title: 软件版本号
date: 2017-04-05 11:35:20
toc: true
tags: [software]
categories: programmer
keywords: [software, version number, 软件版本号, alpha, beta, rc]
description: 软件版本号概述
---

# 版本号规则

[major].[minor].[revision]

* major，主版本号，软体有重大更新的时候递增，重大更新通常是指功能与介面都有大幅度变动的时候。不向下兼容。
* minor，次版本号，软体发布新功能，但是并不会大幅影响到整个软体的时候递增。可向下兼容。
* revision，修订版本号，通常是在修订软件 bug 后，发布 bug 的修订版时递增。

有些版本编号除了上述三者之外，还会有「build」。也就是只要每次 build 一次软件就会递增一次，有时候甚至会使用 build 的日期或时间。

# 软件版本周期
[软件版本周期](https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E8%99%9F)是指电脑软件的发展及发行过程，如下图，从Pre-alpha（准预览版本）发展到Alpha（预览版本）、Beta（测试版本）、Released candidate （最终测试版本）至最后的Gold（完成版）。
![软件版本周期](https://upload.wikimedia.org/wikipedia/commons/thumb/0/07/Software_dev2.svg/363px-Software_dev2.svg.png)

## pre-alpha（准预览版本）
有时候软件会在Alpha或Beta版本前先发布Pre-alpha版本。一般而言相对于Alpha或Beta版本，Pre-alpha版本是一个功能不完整的版本。

## alpha（预览版本）
Alpha版本仍然需要测试，其功能亦未完善，因为它是整个软件发布周期中的第一个阶段，所以它的名称是“Alpha”，希腊字母中的第一个字母“α”。
Alpha版本通常会送到开发软件的组织或某群体中的软件测试者作内部测试。在市场上，越来越多公司会邀请外部客户或合作伙伴参与其测试。这令软件在此阶段有更大的可用性测试。
在测试的第一个阶段中，开发者通常会进行白盒测试。其他测试会在稍后时间由其他测试团体以黑盒或灰盒技术进行，不过有时会同时进行。

## beta（测试版本）
Beta版本是软件最早对外公开的软件版本，由公众参与测试。一般来说，Beta包含所有功能，但可能有一些已知问题和较轻微的程序错误（BUG）。Beta版本的测试者通常是开发软件的组织的客户，他们会以免费或优惠价钱得到软件。Beta版本亦作为测试产品的支持和市场反应等。
其他情况，例如微软曾以Community Technology Preview（简称CTP，中文称为“社区技术预览”）为发布软件的测试版本之一，微软将这个阶段的软件散布给有需要先行试用的用户或厂商，并收集这些人的使用经验，以便作为进一步修正软件的参考。

## rc（最终测试版本）
Release Candidate（简称RC）指可能成为最终产品的候选版本，如果未出现问题则可发布成为正式版本。在此阶段的产品通常包含所有功能、或接近完整，亦不会出现严重问题。
多数开源软件会推出两个RC版本，最后的RC2则成为正式版本。闭源软件较少公开使用，微软公司在Windows 7上应用此名称。苹果公司把在这阶段的产品称为“Golden Master”（简称GM），而最后的GM即成为正式版本。

## RTM
RTM（Release To Manufacturing）之简称，意思是：发放给生产商。某些计算机程序以“RTM”作为软件版本代号，例如微软Windows 7发行零售版前的RTM版本主要是发放给组装机生产商用，使制造商能够提早进行集成工作或解决软件与硬件设备可能遇到的错误。RTM版本并不一定意味着创作者解决了软件所有问题；仍有可能向公众发布前更新版本。以Windows 7为例：RTM版与零售版的版本号是一样的。
另外一种RTM的称呼是RTW（Release To Web），表示正式版本的软件发布到Web网站上供客户免费下载，这个名词在ASP.NET组件以及Silverlight的发布上很常见。

## Stable
稳定版本来自预览版本释出使用与改善而修正完成。为目前所使用的软件在匹配需求规格的硬件与操作系统中运行不会造成严重的不兼容或是硬件冲突，其已受过某定量的测试无误后所释出者。

# windows 版本

## MSDN
MSDN（Microsoft Developer Network）软件是微软公司面向软件开发者的一种版本。MSDN 涵盖了所有可以被开发扩充的平台和应用程序，如微软公司的百科全书 Encarta，或者是各种游戏，是不包括在 MSDN 之内的，因为这些产品直接面向最终用户，没有进行程序开发的必要。

MSDN 在 Operating System 以上的等级都有附微软的软件授权，根据 MSDN 的使用者授权合约(EULA)，软件只授权给使用 MSDN 的那一位开发人员，专供开发与测试之用，其他人不可使用 MSDN 所附软件。包含企业不能用 MSDN附的 SQL Server Enterprise Edition 做为生产环境中的数据库服务器；秘书不能使用 MSDN 所附的 Office2007 等等。

## OEM

OEM（Original Equipment Manufacturer）软件只能随机器出货，不能零售，所以也叫做随机版。OEM软件只能全新安装，不能从旧有作系统升级。

如果买笔记型计算机或品牌计算机就会有随机版软件。包装不像零售版精美，通常只有一片cd和说明书(授权书)。这种系统通常会少一些驱动，而且目前的OEM软件很少放在光盘里能给你安装，要么就是恢复盘，要么就是硬盘镜像。

## RTL
RTL版 Retail 正式零售版，供市面上架零售。

## VOL
Volume OR Volume Licensing for Organizations，即团体批量许可证，根据这个许可，当企业或者政府需要大量购买一软件时可以获得优惠。这种产品的光盘的卷标都带有"VOL"字样，就取"Volume"前3个字母，以表明是批量。这种版本根据购买数量等又细分为“开放式许可证”(Open License)、“选择式许可证(Select  License)”、“企业协议(Enterprise Agreement)”、“学术教育许可证(Academic Volume Licensing)”等5种版本，上海政府 VOL版XP就是这种批量购买的版本。根据 VOL 计划规定， VOL 产品是不需要激活的(无论升级到SP1还是SP2)。

## EVAL
评估版

## CTP
Community Test Preview

## FPP
FPP就是零售版（盒装软件），这种产品的光盘的卷标都带有"FPP"字样，比如英文WXPPro的FPP版本的光盘卷标就是WXPFPP_EN，其中WX表示是Windows XP，P是Professional（H是Home），FPP表明是零售版本，EN是表明是英语。获得途径除了在商店购买之外，（H是Home），FPP表明是零售版本，EN是表明是英语。获得途径除了在商店购买之外，某些MSDN用户也可以得到。


# 参考文献
1. SLMT's Blog, [版本編號的命名規則](http://www.slmt.tw/2015/07/20/version-number-naming-convention/)
2. zh.wikipedia, [软件版本号](https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E8%99%9F)
3. en.wikipedia, [software versioning](https://en.wikipedia.org/wiki/Software_versioning)
4. zh.wikipedia，[软件版本周期](https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E9%80%B1%E6%9C%9F)
5. gdaily，[查看你的完整 Windows 版本 Retail、VOL、OEM、RTM](https://www.gdaily.org/4275/windows-edition)
6. web duper，[Windows MSDN、OEM、RTM、VOL等各版本的区别](http://materliu.github.io/all/web/ideas/2014/04/23/windows-msdn-oem-rtm-vol.html)
