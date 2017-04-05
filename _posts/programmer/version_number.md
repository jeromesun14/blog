title: 软件版本号
date: 2017-03-31 11:39:24
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

# 参考文献
1. SLMT's Blog, [版本編號的命名規則](http://www.slmt.tw/2015/07/20/version-number-naming-convention/)
2. zh.wikipedia, [软件版本号](https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E8%99%9F)
3. en.wikipedia, [software versioning](https://en.wikipedia.org/wiki/Software_versioning)
4. zh.wikipedia，[软件版本周期](https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E9%80%B1%E6%9C%9F)
