---
title: '搭建一个属于自己的 ChatGPT 镜像网站'
date: '2023-06-08'
description: '教程 - 如何部署自己的私人 ChatGPT 网站'
cover: 'https://www.helloimg.com/images/2023/06/09/osdDQu.webp'
categories:

- 杂谈

tags:

- ChatGPT

---

> 经过三周半的头脑风暴，前几天终于把工作任务做完了，也好久没写博客了，实在是不太想动脑。今天随便写个搭网站的教程吧，我也是跟着别人的教程一步一步走搭建完成的，其实还挺有趣，就当记录一下了。

# 一、准备工作

本教程选用 ChatGPT-Next-Web 项目，Vercel 部署，CloudFlare 进行域名管理与加速，所以需要提前准备好：
 1. Github 账号。
 2. Vercel 账号。
 3. CloudFlare 账号。
 4. OpenAI API KEY。
 5. 一个域名。

# 二、项目部署

## 2.1 ChatGPT-Next-Web

- 目前我接触过的比较好的两个开源项目，一个是[chatgpt-web](https://github.com/Chanzhaoyu/chatgpt-web)，一个是[ChatGPT-Next-Web](https://github.com/Yidadaa/ChatGPT-Next-Web)。
- 两个项目我都尝试搭建过，chatgpt-web 的搭建相对更方便，使用 docker 一键部署，但是需要一台国外的云服务器。这就有点尴尬了，囊中羞涩，最后只能放弃了。
chatgpt-web： 

<img src="https://www.helloimg.com/images/2023/06/09/osdIyv.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

ChatGPT-Next-Web：

<img src="https://www.helloimg.com/images/2023/06/09/osd8mE.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- ChatGPT-Next-Web 流程相对复杂一丢丢，但相对更"实惠"一些，我们项目部署交给免费的 Vercel ，域名解析交给免费的 CloudFlare，这样一套乞丐级流程下来，全程只需要花一个买域名的钱，总花费7快多一点。

<img src="https://www.helloimg.com/images/2023/06/08/oslmMu.jpg"  width="30%">

## 2.2 Fork 项目

- 从 GitHub 上 Fork [ChatGPT-Next-Web](https://github.com/Yidadaa/ChatGPT-Next-Web) 项目到个人仓库

<img src="https://www.helloimg.com/images/2023/06/09/osTOiv.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

## 2.3 Vercel 部署

- [Vercel](https://vercel.com/) 可以直接以 GitHub 账号登录，然后点击`Add New Project`，选择 Project 从 Github 部署项目。

<img src="https://www.helloimg.com/images/2023/06/09/osTws9.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 选择刚刚 Fork 的项目进行导入。

<img src="https://www.helloimg.com/images/2023/06/09/osT0vX.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 配置参数，选择 Environment Variables 并添加两个环境变量参数：
  1. `CODE` 代表网站的访问控制密码，密码不要忘了。
  2. `OPENAI_API_KEY` 你的 OpenAI 账户的 Key。

<img src="https://www.helloimg.com/images/2023/06/09/osT7K6.jpg" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 部署，点击`Deploy`按钮，等待部署成功即可。

<img src="https://www.helloimg.com/images/2023/06/09/osdKcY.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 点击`Continue to DashBoard`按钮，查看部署信息。

<img src="https://www.helloimg.com/images/2023/06/09/osdV8M.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

## 2.4 域名购买

- 其实按道理说 Velcel 部署完成后就OK了。但是使用自动为我们分配好的域名 [chat-gpt-next-web-six-gold-73.vercel.app](https://chat-gpt-next-web-six-gold-73.vercel.app/) 被墙了，访问不了，所以我们得配置一下单独的域名解析。

- 选择购买渠道。可以任意找一个域名供应商进行购买，这里推荐 [Namesilo](https://www.namesilo.com/) ，便宜没有坑，支付也非常方便。

- 注册登录网站后，在搜索框搜索你想要申请的域名，点击`Add`添加到购物车(如果域名已经被注册是无法添加的)，点击`Checkout`进入付款页面。

- 这里可以填写推荐码 `tree1024` 获得 1$ 的优惠。

<img src="https://www.helloimg.com/images/2023/06/09/osdhS6.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;"> 

- 最后填写一些信息，然后选择支付宝付款即可。

## 2.5 CF 域名管理

- 将上面购买到的域名添加到 Cloudflare 中管理，点击`添加站点`按钮，输入域名。

<img src="https://www.helloimg.com/images/2023/06/09/osdy5R.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 选择 Free 计划，点击`Continue`。

<img src="https://www.helloimg.com/images/2023/06/09/osvBdA.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 回到 [Namesilo](https://www.namesilo.com/)，右上角点`Manage My Domains`，然后会看到下图，先勾选你要解析的域名，再点`change Nameservers`。

<img src="https://www.helloimg.com/images/2023/06/09/osvPoc.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 把 CloudFlare 中的名称服务器地址（athena.ns.cloudflare.com、jake.ns.cloudflare.com）填到 Namesilo 里然后保存，解析生效官方的说法是24小时，但一般半个小时之内就OK了。

- 回到 [Vercel](https://vercel.com/) 中，点击项目右上角的`Domains`。添加我们刚才的购买的域名，会提示一同添加 www 开头的域名，按推荐添加即可。

<img src="https://www.helloimg.com/images/2023/06/09/osvZ2m.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 最后回到 CloudFlare 中，点击检查名称服务器，检查通过会发一封邮件。

<img src="https://www.helloimg.com/images/2023/06/09/osves1.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 配置 SSL，点击左侧导航栏的`SSL/TSL`，选择`full`。

<img src="https://www.helloimg.com/images/2023/06/09/osvJeo.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 添加A 记录（可选）和 CNAME 记录，如果你想用根域名访问你的站点，比如 https://leihaochat.top，需要添加一条 A 记录，直接将根域名解析到 Vercel 的服务器地址（76.76.21.21）即可！

<img src="https://www.helloimg.com/images/2023/06/09/osvi5u.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 大功告成！试着访问一下你的网址(收到 CloudFlare 检查通过邮件之后)，可以顺利访问就代表你成功了！

<img src="https://www.helloimg.com/images/2023/06/09/osvnsv.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

# 三、总结

- 如果无法访问，着重检查自己的 CloudFlare 对`DNS`以及`SSL`的配置。
- 总的来说步骤虽然多一点，但是都不复杂，也不会有什么奇奇怪怪的坑，只用花不到一顿饭钱的价格就可以拥有一个自己的ChatGPT网站，还是非常爽的！
- 在部署好的网站的设置里面，有一个访问密码和API Key，访问密码即之前`2.3 Vercel 部署`中`CODE`的值，如果访问密码验证成功，就会消耗你配置的那个API Key，否则会优先使用下方配置的API Key。

<img src="https://www.helloimg.com/images/2023/06/09/osvHvE.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">