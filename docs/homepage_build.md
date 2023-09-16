---
comments: true
---

# 主页搭建
1. 主页搭建借鉴了 [pegasuswang](https://pegasuswang.github.io/booknotes/)。
2. 主页技术栈为
    1. mkdocs: quick start 参考 https://markdown-docs-zh.readthedocs.io/zh_CN/latest/#_8。
    2. Python-Markdown-Math 编写公式
    3. 自动部署 gh-pages: 参考 https://www.cnblogs.com/chinjinyu/p/17610438.html
3. For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## mkdocs 构建命令

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.
* `mkdocs gh-deploy` # 部署到自己的 github pages, 如果是 readthedocs 会自动触发构建

## 项目布局

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

## 选择 material 主题
mkdocs 的 material 主题因为 **极简风格** 与 **配色好看** 十分吸引我。前者能让人专注于内容，后者则能够持续吸引人阅读。

### 配置
material 主题配置可以参考如下两个教程

* [Quick start 教程](https://derrors.github.io/%E9%85%8D%E7%BD%AE%20YAML%20%E6%96%87%E4%BB%B6/), 但是其中在 footer 里面添加邮箱等社交账号的配置方法已经过期了，新配置方法如下
```shell
# mkdocs.yml 相关配置
extra:
    social:
        -   icon: fontawesome/solid/paper-plane
            link: mailto:<shafish_cn@163.com>
            name: 邮箱地址
```
* [详细使用教程](https://shafish.cn/blog/mkdocs/), 这哥们的博客主题就是 material, 这个教程也写的比较详细.
* [其他教程](https://squidfunk.github.io/mkdocs-material/reference/code-blocks/#__codelineno-4-3)

material 主题中可以使用各种小图标(也叫 font)，比如将当前页翻到最后看到的纸飞机和邮箱图标。图标样式和名字可以从下面[链接](https://fontawesome.com/v6/icons?o=r&s=solid)中找到。

### 自定义
虽然 material 主题默认配置已经很好了，但是免不了要做一些自定义修改，比如给页面添加一个统计访问量的功能。网页都是通过 markdown 写的，那怎么修改呢？

我这里先简单介绍下修改思路，只针对 mkdocs 的 material 主题，不一定适用其他主题。material 利用 flask 框架(报错栈中看到了flask关键字)写好网页模板，让后将 markdown 中的内容填到模板当中，然后再利用这个模板生成最终的 html 网页。因此，修改的思路就是修改预定义的 flask 网页模版。详细可以参考 [material 网页自定义教程](https://squidfunk.github.io/mkdocs-material/customization/#overriding-partials)，主要分为如下几步：

1. 添加自定义配置
```shell
theme:
  custom_dir: overrides ## 添加这个配置
```
2. 在 `mkdocs.yml` 文件的同级目录下，新建`overrides` 目录
3. 在 `overrides` 目录下创建需要覆盖的模板文件，并保证文件在 `overrides`中的相对位置和 [material 网页自定义教程](https://squidfunk.github.io/mkdocs-material/customization/#overriding-partials) 中一致。 
4. 修改模板文件，注意区别覆盖原来的模板还是在原来基础上新增内容。

下面以新增网站访问量为例介绍。

## 新增网站访问量功能
1. 如前一小节所述
2. 如前一小节所述
3. 在  `overrides` 目录下新建 `main.html` 文件，这个是所有网页都会用到的主模板文件，按照 [material 网页自定义教程](https://squidfunk.github.io/mkdocs-material/customization/#overriding-partials)要求，它存放的位置如下 
```shell
.
├── README.md
├── docs
│   └── ...
├── mkdocs.yml
└── overrides
    └── main.html
```
4. `main.html` 中内容如下
      1. `{% extends "base.html" %}` 这一句表示继承原模板，含义是 除了新增、修改的内容，其他内容和原模板一样。
      2. `{% block announce %}` 表示接下来修改的是网页中 announce 部分的内容
      3. `{{ super() }}` 表示原模板中的 announce 部分的内容仍然保留
      4. `{% endblock %}` 表示结束修改
      5. 其余的非注释部分内容是新加的
```shell
{% extends "base.html" %}

{% block announce %}
<script async src="https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次, 访客数<span id="busuanzi_value_site_uv"></span>人次
</span>

  <!-- Add scripts that need to run before here -->
  {{ super() }}
  <!-- Add scripts that need to run afterwards here -->
{% endblock %}
```

5. 新加内容中
       1. 第一行的 `<script async src="https:...</script>`表示引用计算网站访问量的脚本，这里用的是 [不蒜子](https://busuanzi.ibruce.info/) 脚本，主要是简单、免费。
       2. 第二行是计算网站访问量：busuanzi_value_site_pv 表示点击次数、busuanzi_value_site_uv表示访客人数

## 添加评论功能
我选 Github Pages 来自建网站写博客，因为

1. 比起用知乎、CSDN等平台，自建博客的社交属性没有那么重，管理起来也更加方便
2. 同时还自带版本管理、域名解析。

Github Pages 是静态网站，所以我一直**狭隘地**以为无法实现交互相关的功能，比如网站访问量统计、评论等。没想到也是有方法的。前面一节已经增加了网站访问量统计功能，这一节介绍下如何给静态网站（特指 Github Pages + mkdocs material）添加评论功能。

网上教程有推荐 Disque、livere 第三方博客评论系统的，注册后看了下价格，直接劝退。

经过多方搜索，找到两套免费实现的方案，都是借助 Github 实现

1. [shafish的material教程](https://shafish.cn/blog/mkdocs/#10-%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F-vssue) 推荐使用 [Vssue](https://vssue.js.org/zh/guide/)
2. squidfunk 推荐的 [基于giscus搭建评论系统] 的教程，同时可以辅助参考 [Giscus 申请教程]

我更倾向于 2，因为是算是官方推荐的。

### 基于giscus搭建评论系统
1.  __安装 [Giscus GitHub App]__ 并将授予某个仓库用 Discussion 存储评论的权限，这个仓库可以不是存放 Github Pages 的 repo；权限授予可以参考 [Giscus 申请教程]
2.  __访问 [Giscus] 并生成如下代码片段__ :

    ``` html
    <script
      src="https://giscus.app/client.js"
      data-repo="<username>/<repository>"
      data-repo-id="..."
      data-category="..."
      data-category-id="..."
      data-mapping="pathname"
      data-reactions-enabled="1"
      data-emit-metadata="1"
      data-theme="light"
      data-lang="en"
      crossorigin="anonymous"
      async
    >
    </script>
    ```

3. 借用 material 主题的自定义扩展功能，将评论模块插入到网站上。
   
    1. 在如下目录结构中新建 `comments.html` [comments]，为什么要是这个目录结构可以参考[overriding partials]
```shell
        .
        ├── README.md
        ├── docs
        │   └── ...
        │      
        ├── mkdocs.yml
        └── overrides
        ├── main.html
        └── partials
            └── comments.html
```
      2. `comments.html` 里的内容如下 
``` html hl_lines="3"
{% if page.meta.comments %}
  <h2 id="__comments">{{ lang.t("meta.comments") }}</h2>
  <!-- Insert generated snippet here -->

  <!-- Synchronize Giscus theme with palette -->
  <script>
    var giscus = document.querySelector("script[src*=giscus]")

    /* Set palette on initial load */
    var palette = __md_get("__palette")
    if (palette && typeof palette.color === "object") {
      var theme = palette.color.scheme === "slate" ? "dark" : "light"
      giscus.setAttribute("data-theme", theme) // (1)!
    }

    /* Register event handlers after documented loaded */
    document.addEventListener("DOMContentLoaded", function() {
      var ref = document.querySelector("[data-md-component=palette]")
      ref.addEventListener("change", function() {
        var palette = __md_get("__palette")
        if (palette && typeof palette.color === "object") {
          var theme = palette.color.scheme === "slate" ? "dark" : "light"

          /* Instruct Giscus to change theme */
          var frame = document.querySelector(".giscus-frame")
          frame.contentWindow.postMessage(
            { giscus: { setConfig: { theme } } },
            "https://giscus.app"
          )
        }
      })
    })
  </script>
{% endif %}
```
      3. 在前面 `comments.html` 的高亮处下面粘贴在第 2 步中获得的 Giscus 授权代码
      4. 按网页开启评论功能：在需要开启评论的 markdown 文档前面添加如下注释
``` yaml hl_lines="1 2 3"
---
comments: true
---

# Page title
...
```

4. 按网页所在的目录批量开启评论功能, 主要流程可以参考 [built-in meta plugin], 但是有个配置过期了
```yml
plugins:
  - meta
```
应该改成
```yml
# 参考 https://pypi.org/project/mkdocs-meta-manager/
plugins:
    - meta-manager
```

  [Giscus]: https://giscus.app/
  [Giscus GitHub App]: https://github.com/apps/giscus
  [overriding partials]:https://squidfunk.github.io/mkdocs-material/customization/#overriding-partials
  [comments]: https://github.com/squidfunk/mkdocs-material/blob/master/src/partials/comments.html
  [built-in meta plugin]: https://squidfunk.github.io/mkdocs-material/plugins/meta/
  [Giscus 申请教程]: https://www.lixueduan.com/posts/blog/02-add-giscus-comment/
  [基于giscus搭建评论系统]: https://squidfunk.github.io/mkdocs-material/setup/adding-a-comment-system/