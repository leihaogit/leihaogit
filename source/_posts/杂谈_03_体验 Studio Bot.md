---
title: '体验 Studio Bot'
date: '2023-09-01'
description: '教程 - 如何体验谷歌推出的专门用于辅助安卓开发的 AI 编程机器人：Studio Bot'
cover: 'https://www.helloimg.com/images/2023/09/01/oib43m.jpg'
categories:

- 杂谈

tags:

- Blog

---

> 2023年5月11日，谷歌在 I/O 2023 开发者大会上，面向安卓开发者推出了 AI 编程机器人 Studio Bot，距今已经过去3个多月了，但是网上还很少有对它的讨论，今天来看看如何才能体验到这个新出的玩意儿。

# 一、准备工作

- 由于目前 Studio Bot `只提供给美国地区使用`，我们在国内想要体验到它还是有一点点麻烦的：

<img src="https://www.helloimg.com/images/2023/09/01/oibAlh.jpg">

- 所以我们需要准备几个东西：

1. `一个能够全局代理的梯子`。并且能获取代理 ip 和端口。这个一般免费的梯子都不行，建议找个便宜的用用。

<img src="https://www.helloimg.com/images/2023/09/01/oibL6c.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

2. `一个国家/地区版本为美国的谷歌账号`。这个可以在登录谷歌官网后，点击最下方的`条款`，然后就可以看到了。

<img src="https://www.helloimg.com/images/2023/09/01/oibiZq.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

3. `金丝雀版的 Android Studio`，也就是最新的前瞻版，没有经过充分的测试所以可能会存在不少 bug，这里我们只是体验使用，并且它可以和正式版共存，也就无所谓了。[下载](https://developer.android.google.cn/studio/preview)选择 Canary build 版本就可以了。

<img src="https://www.helloimg.com/images/2023/09/01/oibsr0.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

# 二、登录配置

- 安装好最新的 Android Studio preview 版后，我们开启全局代理，然后打开 Android Studio，随便新建一个项目，如果想要使用 Java 编写程序的话，只能选择 No Activity 的那个模版，不然默认就是使用 Jetpack Compose 来进行项目的开发，语言也是 Kotlin 语言，且更换起来有点麻烦。
- 进入项目后，我们可以看到右侧的侧边栏已经有个 Studio Bot：

<img src="https://www.helloimg.com/images/2023/09/01/oib0Qu.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="30%">

- 但是现在还不能直接使用，你点开的话发现他会提示登录，在此之前，我们需要先设置一下`HTTP Proxy`，直接打开 Settings，搜索 HTTP Proxy 并打开，选择 manual proxy configuration，ip 输入 127.0.0.1，port 输入你开启代理的端口：

<img src="https://www.helloimg.com/images/2023/09/01/oibzcY.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

- 接下来，保存设置，打开 Studio Bot，点击 `Log in Google`。他会跳转到浏览器中，我们授权我们的谷歌账号进行登录，登录完成是这样的：

<img src="https://www.helloimg.com/images/2023/09/01/oib7V9.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

- 接下来返回 Android Studio，正常情况下就可以直接点击几个 Next 后开始使用了，如果返回 Android Studio 后没有反应，检查一下前面的 HTTP Proxy 的配置是否正确。

# 三、简单使用

- 上面的步骤都做了之后，就可以和 Studio Bot 聊天了：

<img src="https://www.helloimg.com/images/2023/09/01/oibIXX.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

<img src="https://www.helloimg.com/images/2023/09/01/oib6Mg.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="80%">

- 由于体验时间不长，我感觉比 ChatGPT 稍微弱一些，而且有时候会出现 bug 一直回复一样的话。目前对开发有帮助的点也比较少，一个是可以直接在鼠标处插入生成的代码：

<img src="https://www.helloimg.com/images/2023/09/01/oibkpn.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="50%">

- 一个是可以直接生成新的文件并插入代码：

<img src="https://www.helloimg.com/images/2023/09/01/oibgS6.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;" width="50%">

# 四、总结

- 体验下来感觉目前这个机器人的能力确实还比较一般，对开发并不是很有帮助，当然有可能是我还没有掌握到正确的打开方式。
- 不过，我还是很看好它未来的发展的，由于是直接嵌入 Android Studio 的插件，说不定以后可以一键生成一个项目也不一定呢？