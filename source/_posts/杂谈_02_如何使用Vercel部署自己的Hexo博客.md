---
title: '如何使用 Vercel 部署自己的 Hexo 博客'
date: '2023-06-10'
description: '教程 - 将自己的 Blog 使用Velcel进行部署，告别 Github Pages 的卡顿'
cover: 'https://www.helloimg.com/images/2023/06/10/osNX56.png'
categories:

- 杂谈

tags:

- Blog

---

# 一、GitHub Pages 部署回顾

- 先看一下之前是怎样部署的，首先在 Hexo 的配置文件`_config.yml`中，修改 deploy 项，type 填写 git，repo 即你的博客仓库地址，branch 是你要部署博客的分支
- 以我的为例：
```text
deploy:
  type: git
  repo: https://github.com/leihaogit/leihaogit
  branch: gh-pages
```
- 分支名字其实无所谓，但是为了统一规范一般都叫这个名字。
- `说明`：博客实体就是 hexo generate 生成的内容，长这样：

<img src="https://www.helloimg.com/images/2023/06/10/osNAih.jpg"  width="50%" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 其实就是博客项目生成的`public`文件夹，里面包括了我们这个静态网站的所有内容。
- 我们在执行`hexo generate`时，会生成当前项目对应的 public 文件夹，然后执行`hexo deploy`时，会将生成的 public 文件夹推送至你配置文件配置的那个仓库的指定分支。
- 最后在 Github Pages 管理页面选择`gh-pages`分支进行部署即可。

# 二、Vercel 部署 - 直接部署

- 通过上面的说明，可以看出，其实我们需要的只有`public`文件夹而已。因此我们可以删除配置文件中的 gh-pages 和对应的仓库分支。这样的话部署完成后，仓库里面是这样的：

<img src="https://www.helloimg.com/images/2023/06/10/osNMeK.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

- 然后直接像部署一个普通的静态网站一样，部署到 Vercel 就行了。
步骤如下：
  1. 在 Vercel 中点击`Add New Project`，选择 Project 从 Github 部署项目，然后导入你的博客仓库：

<img src="https://www.helloimg.com/images/2023/06/10/osNvBQ.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  2. `Framework Preset`就选择`Other`即可。`Root Directory`默认，不用更改。

<img src="https://www.helloimg.com/images/2023/06/10/osNNfv.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  3. 点击`Deploy`，等待部署完成即可。
  4. 其实到这里就可以访问`仓库名.vercel.app`了，但是由于 vercel.app 被墙，所以我们需要配置一下 Domains 了，这个留在后面一起说。
- 我个人不太喜欢这种部署方式，万一哪天电脑坏了，你没有将项目上传到仓库，你就只剩下一个博客了，想再编辑就很麻烦，所以最好还是使用下面介绍的方式进行部署。

# 三、Vercel 部署 - 使用预设框架

- 在上面直接部署的步骤2中，`Framework Preset`其实有个选项就是 Hexo，也就是说，其实 Vercel 是能够直接使用 Hexo 的预设模板进行项目部署的。
步骤如下：
  1. 在 Vercel 中点击`Add New Project`，选择 Project 从 Github 部署项目，然后导入你的博客仓库：

<img src="https://www.helloimg.com/images/2023/06/10/osNvBQ.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  2. `Framework Preset`选择`Hexo`。`Root Directory`默认，不用更改。

<img src="https://www.helloimg.com/images/2023/06/10/osNyHm.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  3. 点击`Deploy`，等待部署完成即可。
  4. 部署完成后，点击`Continue to DashBoard`按钮，查看部署信息。再点击右上角`Domains`，输入自己的域名，点击`Add`。点击之后会提示一同添加带 www 的域名，按推荐添加即可。

<img src="https://www.helloimg.com/images/2023/06/10/osQagb.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  5. 之后会提示你去 DNS 提供商上设置 A 记录或者 CNAME 记录：

<img src="https://www.helloimg.com/images/2023/06/10/osQwIm.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  6. 直接按他提示的，我们前往域名提供商，添加一个 A 记录即可，CNAME 记录当然也是可以的。

<img src="https://www.helloimg.com/images/2023/06/10/osQQAc.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

  7. 回到 Vercel，发现重定向成功！访问网站也没有问题。

<img src="https://www.helloimg.com/images/2023/06/10/osQpNq.png" style="border: 0.1px solid #00BFFF; border-radius: 5px;">

# 四、总结

- 总的来说，部署是很简单的，告别了之前卡顿的GitHub Pages，体验也会好上不少！
- 除了访问更流畅，现在我们每次写完博客，也不需要再执行`Hexo deploy`了，只要我们将代码上传至仓库，Vercel 就会自动帮我们完成更新部署工作。