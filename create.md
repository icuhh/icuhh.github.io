## 文档结构



|        文件作用        |   文件名称    |
| :--------------------: | :-----------: |
| 基础配置项（入口文件） |  index.html   |
|      封面配置文件      | _coverpage.md |
|     侧边栏配置文件     |  _sidebar.md  |
|     导航栏配置文件     |  _navbar.md   |
|    主页内容渲染文件    |   README.md   |
|       浏览器图标       |  favicon.ico  |



## 快速开始

推荐全局安装 `docsify-cli` 工具，可以方便地创建及在本地预览生成的文档。

```
npm i docsify-cli -g
```



## 初始化项目

如果想在项目的 `./docs` 目录里写文档，直接通过 `init` 初始化项目。

```
docsify init ./docs
```



##  开始写文档

初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- `index.html` 入口文件
- `README.md` 会做为主页内容渲染
- `.nojekyll` 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 `docs/README.md` 就能更新文档内容，当然也可以[添加更多页面](https://docsify.js.org/#/zh-cn/more-pages)。



## 本地预览

通过运行 `docsify serve` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 [http://localhost:3000](http://localhost:3000/) 。

```
docsify serve docs
```

更多命令行工具用法，参考 [docsify-cli 文档](https://github.com/docsifyjs/docsify-cli)。



## Loading 提示

初始化时会显示 `Loading...` 内容，你可以自定义提示信息。

```
  <!-- index.html -->

  <div id="app">加载中</div>
```

如果更改了 `el` 的配置，需要将该元素加上 `data-app` 属性。

```
  <!-- index.html -->
  <div data-app id="main">加载中</div>

  <script>
    window.$docsify = {
      el: '#main'
    }
  </script>
```



## 封面配置

在根目录创建**封面配置文件（`_coverpage.md`）**，并配置渲染显示内容

```
<!-- _coverpage.md -->

![logo](_media/icon.svg)

# docsify <small>3.5</small>

> 一个神奇的文档网站生成器。

- 简单、轻便 (压缩后 ~21kB)
- 无需生成 html 文件
- 众多主题

[开始使用 Let Go](/README.md)
```

然后在**入口文件（`index.html`）**，开启相关配置

```
<!-- index.html -->

<script>
  window.$docsify = {
    coverpage: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```



## 自定义侧边栏

在根目录创建**侧边栏目录文件（`_sidebar.md`）**，并配置目录菜单

```
* 首页

   * [简介](/)
   * [测试](/test)

* 其他

   * [其他](/other)
```

然后在**入口文件（`index.html`）**，开启相关配置

```
<script>
  window.$docsify = {
	loadSidebar: true
  }
</script>
```



## 导航栏设置

在根目录创建**导航栏配置文件（`_navbar.md`）**，这里可在`index.html`文件直接写入，也可配置文件渲染

```
<body>
  <nav>
    <a href="#/">EN</a>
    <a href="#/zh-cn/">中文</a>
  </nav>
  <div id="app"></div>
</body>
```

然后在**入口文件（`index.html`）**，开启相关配置

```
<script>
  window.$docsify = {
	loadNavbar: true
  }
</script>
```



## 其他个性化主题及插件

- 侧边栏插件

```
<!-- 箭头样式-->
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/sidebar.min.css" />
<!-- 文件夹样式-->
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/sidebar-folder.min.css" />
<script>
  window.$docsify = {
	subMaxLevel: 2,
	sidebarDisplayLevel: 1
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/docsify-sidebar-collapse.min.js"></script>
```

- 代码复制插件

```
  <script src="//unpkg.com/docsify-copy-code"></script>
```

- 字数统计插件

```
<script>
  window.$docsify = {
        count: {
          countable: true,
          position: "top",
          margin: "10px",
          float: "right",
          fontsize: "0.9em",
          color: "#dcdcdc",
          language: "chinese",
          localization: {
            words: "",
            minute: "",
          },
          isExpected: true,
        }
  }
</script>
<script src="https://cdn.jsdelivr.net/npm/docsify-count@latest/dist/countable.min.js"></script>
```

- 上下页切换插件

```
<script>
  window.$docsify = {
 		pagination: {
            previousText: '上一页', // 自定义“上一页”按钮文本
            nextText: '下一页', // 自定义“下一页”按钮文本
            crossChapter: true, // 支持跨章节导航
            crossChapterText: true // 显示跨章节的标题
		}
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify-pagination/dist/docsify-pagination.min.js"></script>
```

- 本地搜索插件

```
<script>
  window.$docsify = {
		search:{
            paths:[
            "/"，
            "/test"
            ],
            maxAge:86400000,
            placeholder: '搜 一 搜',
            depth: 6,
            noData:"没有搜索数据"
      }
  }
</script>
<script src="//unpkg.com/docsify/lib/plugins/search.min.js"></script>
```

 
