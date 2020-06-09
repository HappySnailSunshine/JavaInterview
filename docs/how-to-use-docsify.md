还记得我在上周发的[《V2.0 版本的 JavaGuide 面试突击版来啦！带着它的在线阅读版本来啦！》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486494&idx=1&sn=a17e8278bd9fc1354449f925ef990c25&chksm=cea243d5f9d5cac3b0b3b55769e162363256eb7fa483997c21b62cddb4ef2d583a2bcae0ac1a&token=1910196174&lang=zh_CN#rd)这篇文章中答应手把手教大家搭建一下下面一样的文档类型网站不？


![](https://imgkr.cn-bj.ufileos.com/bf58b379-2006-47a3-b722-5c65dbd1b0f6.png)

这篇文章 Guide 哥就手把手教大家搭建一个像下面这样的文档类型的网站，你可以用来当做项目的说明文档，也还可以当做自己专属的知识小仓库。

![](https://imgkr.cn-bj.ufileos.com/c190b25c-e3c0-4c2c-a6b6-60a9a9dbb99e.png)

官网教程的也很详细了，地址在这里：[https://docsify.js.org/#/zh-cn/quickstart](https://docsify.js.org/#/zh-cn/quickstart) ，不过我的这篇教程比较贴合实际使用。

下面演示的所有内容的源文件在这里：https://github.com/Snailclimb/docsify-demo

最终效果展示地址：https://snailclimb.gitee.io/docsify-demo/#/

## 一.前置条件

1. 确保自己电脑下载安装了 NPM 并且使用这个命令： `npm i docsify-cli -g`安装了 docsify-cli 这个工具 。
2. 确保自己有一个 Github 账号（码云账号为非必选项，有的话更好）。

## 二.初始化项目并预览

**1.新建一个文件夹：`mkdir docsify-demo`**

**2.进入文件夹并运行 docsify 初始化命令：`cd docsify-demo` -> `docsify init ./`**

你会发现 docsify-demo 文件夹下面多了下面这些文件，一一为你解释一下它们是干嘛的！

![](https://imgkr.cn-bj.ufileos.com/db8f8dfb-00fb-4608-a400-9a6b036b9a25.png)

**3.本地预览网站：`docsify serve ./` 然后访问：`http://localhost:3000/`**

![](https://imgkr.cn-bj.ufileos.com/cd56465f-c1cf-4e2f-ac0f-f6178b1dd57d.png)

## 三.给我们的项目增加点颜色

![](https://imgkr.cn-bj.ufileos.com/c91ce762-2948-4455-af28-a330bc7e4120.png)

建议 clone 一下我的仓库： [https://github.com/Snailclimb/docsify-demo](https://github.com/Snailclimb/docsify-demo) ，在本地运行一下 ，这是一个比较典型的使用 docsify 搭建的网站，可以作为参考。如果你们想搭建一个不错的文档网站的话，可以在我的这个基础上去改。

### 3.1 修改配置文件 index.html

主要配置了文档网站的名字以及开启了一些配置选项。

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>docsify-demo</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport"
    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//unpkg.com/docsify/lib/themes/vue.css">
</head>

<body>
  <div id="app"></div>
  <!-- docsify-edit-on-github -->
  <script src="//unpkg.com/docsify-edit-on-github/index.js"></script>
  <script>
    window.$docsify = {
      name: 'docsify-demo',
      repo: 'https://github.com/Snailclimb/JavaGuide-Interview',
      maxLevel: 5,//最大支持渲染的标题层级
      subMaxLevel: 3,
      homepage: 'README.md',
      coverpage: true,
      loadSidebar: true,
      auto2top: true,//切换页面后是否自动跳转到页面顶部
    }
  </script>
  <script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
</body>

</html>
```

### 3.2 添加侧边栏文件

在第一步中，我们在已经开启了侧边栏选项：

```js
loadSidebar: true
```

但是，仅仅这样还不行，我们需要定义一个名为 `_sidebar.md`的文件，文件的内容就是我们侧边栏的内容。

如下图所示，我们定义了一个侧边栏，并且为它添加了一些内容：

_一般建议将文档放进 docs 文件下面，可以参考我的仓库：[https://github.com/Snailclimb/docsify-demo](https://github.com/Snailclimb/docsify-demo)_

![](https://imgkr.cn-bj.ufileos.com/d2a6900f-82a1-443b-a3be-fdc1a523486d.png)

修改完成之后，你就会发现我们的文档网站多了侧边栏，你点击侧边栏对应的内容在右边显示。

![](https://imgkr.cn-bj.ufileos.com/3183d9b4-04b4-4a60-b7b5-d18e0b4a609c.png)

### 3.3 修改主页内容

修改 REDME.md，内容如下：

![](https://imgkr.cn-bj.ufileos.com/a4489ab1-523e-4770-bf2b-bcb663503cfa.png)

然后我们的文档网站的主页就变成了这样：

![](https://imgkr.cn-bj.ufileos.com/30fdd3c0-cda4-4f8d-8a70-9c51666a9ab5.png)

### 3.4 添加一个封面

第一步中，我们在已经开启了封面选项：

```js
 coverpage: true
```

为了能让我们的文档网站有封面，我们还需要新建一个名字为 `_coverpage.md`的文件，内容如下

![](https://imgkr.cn-bj.ufileos.com/9f55da9e-b949-40e1-8a5e-f44a4cad3bd7.png)

然后，我们再打开网站的时候，就有了封面，如下图所示：

![](https://imgkr.cn-bj.ufileos.com/e4d4ae6f-1436-49a0-80d8-d57708b528e0.png)


## 四.一些有用的插件

我下面简单介绍几个我觉得比较实用的插件，更多插件的话在这里：[https://docsify.js.org/#/zh-cn/plugins](https://docsify.js.org/#/zh-cn/plugins) 。

![](https://imgkr.cn-bj.ufileos.com/ca7372df-0b0e-4004-95f6-d86c82605dcf.png)

### 4.1 增加 Java 代码高亮

**手动引入 Java 代码高亮的插件：**

```js
<!--Java代码高亮-->
<script src="//unpkg.com/prismjs/components/prism-java.js"></script>
```

![](https://imgkr.cn-bj.ufileos.com/623d61c0-11af-4765-b1b0-23883f546a72.png)

### 4.2 增加全文搜索功能

引入插件：

```js
  <!--全文搜索,直接用官方提供的无法生效-->
  <script src="https://cdn.bootcss.com/docsify/4.5.9/plugins/search.min.js">
```

配置一下：

```js
  <script>
    window.$docsify = {
      ......
      //全文搜索
      search: {
        maxAge: 86400000, // 过期时间，单位毫秒，默认一天
        paths: 'auto',
        placeholder: '搜索',
        noData: '找不到结果',
        // 搜索标题的最大程级, 1 - 6
        depth: 3,
      },
    }
  </script>
```

然后我们的文档网站就有全文搜索功能了：

![](https://imgkr.cn-bj.ufileos.com/223816e3-e911-48f0-b85b-5ecccb7598ee.png)

### 4.3 复制代码到剪切板

引入插件即可！

```js
<!-- 复制代码到剪贴板 -->
<script src="//unpkg.com/docsify-copy-code"></script>
```

![](https://imgkr.cn-bj.ufileos.com/2059a8e3-f124-4433-b159-ac0457b14fd9.png)

### 4.4 图片缩放和字数统计

引入下面两个插件即可！

```js
<!-- 图片缩放 -->
<script src="//unpkg.com/docsify/lib/plugins/zoom-image.js"></script>
<!-- 字数统计 -->
<script src="//unpkg.com/docsify-count/dist/countable.js"></script>
```

![](https://imgkr.cn-bj.ufileos.com/ffd0a2d6-8c0a-4760-b98a-b3f07134691c.png)

### 4.5 edit on github

做如下配置，注意修改为你自己的路径。

```js
  <script>
    window.$docsify = {
      ......
      plugins: [
        EditOnGithubPlugin.create('https://github.com/Snailclimb/JavaGuide-Interview/blob/master/')
      ],
    }
  </script>
```

然后我们的每个页面都出来了 "Edit on github" 选项，你点击之后就会跳到 Github 对应的页面编辑。

![](https://imgkr.cn-bj.ufileos.com/07e627ee-31de-47cf-b0a3-3e5c8dfc1ac1.png)

## 五.部署

### 5.1 部署到 Github

**1.Github 新建一个仓库，这一步就不说了。**

**2.把项目提交上去**

**3.开启 Github Pages**

![](https://imgkr.cn-bj.ufileos.com/9a080df1-dbca-4d7b-b94c-c76e3349c3c6.png)

![](https://imgkr.cn-bj.ufileos.com/8957722b-ccf2-4565-9a0c-86cc4bba25ca.png)

然后你的项目就能在线访问了！

### 5.2 同步到码云提高国内访问速度

**1.导入 Github 项目**

![image-20200411170453064](/Users/shuang.kou/Library/Application Support/typora-user-images/image-20200411170453064.png)

注意把下面的是否开源勾选为公开，不然别人无法访问。

![](https://imgkr.cn-bj.ufileos.com/90c323a9-07d1-4a2b-bde8-7ac61d99941a.png)

**2.选择 Gitee Pages 服务**

![](https://imgkr.cn-bj.ufileos.com/e1869e66-9c25-44c4-91c8-a04ee2e509b6.png)

![](https://imgkr.cn-bj.ufileos.com/ba11f6fe-102a-452c-b5d6-c47a0c451e58.png)**3.收获喜悦的成果**

![](https://imgkr.cn-bj.ufileos.com/31aa3b38-f583-464d-b2e0-a78228b8c486.png)