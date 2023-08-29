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
