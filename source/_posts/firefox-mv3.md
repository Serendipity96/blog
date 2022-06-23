---
title: Firefox Manifest V3：进展及下一步计划
tags:
- extension
- 翻译
categories:
- extension
- 翻译
# toc: false
date: 2022-05-31 23:27
---

# 译者总结

2022-05-18，Mozilla 官方博客发布了关于扩展（WebExtension）MV3 的进展和计划，以及和 Chrome MV3 不同的地方。值得注意的进展：

-   在 W3C 下成立了一个社区小组来推进跨浏览器 WebExtensions (WECG)
-   background script 可重新启动
-   content scripts 不支持未经允许的跨域请求
-   兼容 blocking WebRequest 和 declarativeNetRequest
-   Event Pages 方案



博客原文链接：[Manifest V3 in Firefox: Recap & Next Steps](https://blog.mozilla.org/addons/2022/05/18/manifest-V3-in-firefox-recap-next-steps/)

如需转载译文，请注明译者：[Serendipity96](http://github.com/Serendipity96)

# 以下是博客译文

---


距离上次讨论 Manifest V3 （[Manifest V3 update](https://blog.mozilla.org/addons/2021/05/27/manifest-v3-update/)）已经过去了一年。这期间最重要的变化是在 W3C 下成立了一个[社区小组](https://github.com/w3c/webextensions)来推进跨浏览器 WebExtensions (WECG)。

在之前的[文章](https://blog.mozilla.org/addons/2021/05/27/manifest-v3-update/)中，Firefox 宣布支持 MV3，在 background pages 中使用 Service Workers。（译者注： MV3 前，background 中使用的是[html 和 js](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Background_scripts) ）直接使用 Service Workers 会使一部分扩展有问题，所以一直在调整方案。其中在 WECG 中提出的 [Event Pages](https://github.com/w3c/webextensions/issues/134) 方案受到了社区的欢迎，并得到了 Safari 浏览器的支持。

我们将启动开发者预览计划来收集 MV3 的反馈。接下来的篇幅会介绍 Firefox MV3 的改进之处，然后谈谈与 Chrome MV3 不同的地方。

## 为什么使用 MV3

2015 年我们使用 WebExtensions model 时，就决定要支持跨浏览器。我们坚信，一个扩展能运行在多个浏览器上，对用户来说是最好的服务。到 2017 年底，不仅支持跨浏览器，而且使用了 WebExtensions model。现在许多扩展只需要微小的改动就能用运行在大多数浏览器上，而且到现在为止依然使用的是 WebExtensions model。

2018 年，Chrome 发布了 Manifest V3，随后微软采用 Chromium 作为新 Edge 浏览器的内核。这意味着，未来凭借 Chromium 内核浏览器在市场上的占比，对 MV3 的支持将成为浏览器扩展的标准。我们相信，在 WECG 的背景下与其他浏览器供应商合作，是平衡用户和开发者需求、建立健康生态的最佳途径。对 Mozilla 来说，这是一项长期的工作。

## 为什么 MV3 对改进 WebExtension 很重要

Manifest V3 是 WebExtensions 的一次技术迭代，这提供了改进 WebExtensions 的机会，支持向后兼容。由于 MV2 架构上的限制，有些问题难以解决，现在可以通过 MV3 来解决这些问题。

扩展架构中一个核心的部分是 [background page](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Background_scripts)，从设计上来说它会一直在运行在浏览器上。但由于内存或平台限制（例如在 Android 上），并不能保证一直运行的状态，不可避免 background page 和扩展本身会被终止运行。MV3 采用了新型架构：background script 可重新启动。为了支持这一特性，我们重新设计了现有 API，并引入了新的 API，使扩展能够声明浏览器应该如何运行，而无需 background script 。

扩展的另一个核心是 [content scripts](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_scripts)，用于与网页交互。我们正阻止不安全的编码行为，并提供更安全的替代方案来提高扩展的安全性。比如：字符串类的 API 已从扩展 API 中删除。此外，为了提升不同 origin 的数据隔离能力，除非目标网站同意跨域（CORS），否则 content scripts 不支持跨域请求。

### 网站访问的用户控制

扩展程序经常需要访问网站上的用户数据，虽然这能让扩展的功能更强大、满足用户的需求，但这也导致了扩展滥用用户隐私。

从 MV3 开始，我们把来自扩展程序的请求作为可选项，并为用户提供透明可选的控制，让用户更容易地管理扩展程序可以访问的网站数据。

同时，我们鼓励扩展使用非永久访问所有网站的模式，如小范围或暂时授权模式。我们正继续探索如何以最优的方式处理要拦截的网站，保护用户的安全（如隐私和安全扩展）。

## Firefox 做了哪些修改

### 网络请求（WebRequest）

Chrome MV3 中最有争议的变化之一是禁止使用 blocking WebRequest，Chrome MV3 提供了更灵活的 [declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/)，这对管理高级隐私和内容屏蔽非常重要。但定义一个范围更小的 API（declarativeNetRequest）替代 blocking WebRequest，会限制一些扩展的能力。

Mozilla MV3 会继续支持 blocking WebRequest。为了最大限度地兼容其他浏览器，也会支持 declarativeNetRequest。Mozilla 将继续与内容拦截器、blocking WebRequest API 主要的消费者合作，确定最合适的方案。内容拦截是扩展程序最重要的功能之一，我们致力于确保 Firefox 用户能够使用到最好的隐私工具。

### Event Pages

Chrome MV3 引入了 Background Service Worker 作为持久化运行 [Background Page](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Anatomy_of_a_WebExtension) 的方案。Mozilla 也支持在 Firefox 扩展中使用 Service Workers，这不仅是出于兼容性的考虑，还因为 Service Workers 是一个[有生命周期事件驱动的环境](https://github.com/w3c/webextensions/issues/51#issuecomment-892702868)，它已经是支持跨浏览器 Web 平台的一部分了。

但我们发现 Service Workers 不支持某些重要的[使用场景](https://github.com/w3c/webextensions/issues/72)，特别是 DOM 相关的功能和 API。此外，普通的 Web 开发者并不熟悉 worker 的环境，我们的开发者社区指出，重写扩展对于[现有扩展](https://addons.mozilla.org/en-US/firefox/extensions/)的开发者来说可能有些困难。

Firefox 决定在 MV3 中支持 Event Pages，我们的开发者预览版不包括 Service Workers（我们还在开发，未来的版本将支持 Service Workers）。这将帮助开发者更容易地迁移现有扩展的 background pages 支持 MV3，同时保留 MV2 中所有 DOM 相关的功能。在即将发布的版本中， MV2 会支持 Event Pages，这有利于 MV2 扩展迁移。

## Firefox 下一步计划

在启动 Manifest V3 开发者预览计划时，希望广大开发者能来测试 MV3 的实现，帮我们找出 bug 或不兼容之处，预计在 2022 年底前推出 MV3 支持版。随着这项工作接近完成，我们将公布更多关于时间节点和迁移的细节。

有关 Manifest V3 开发者预览版的更多信息，请查看[迁移指南](https://extensionworkshop.com/documentation/develop/manifest-v3-migration-guide/)。如果您对 Manifest V3 有任何疑问或反馈，我们很乐意在 [Firefox Add-ons Discourse](https://discourse.mozilla.org/c/add-ons/35) 听到您的意见。
