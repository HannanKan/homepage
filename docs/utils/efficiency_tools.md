# 效率工具推荐
> 工欲善其事，必先利其器

## vs code

### Paste Image
* 链接：https://github.com/mushanshitiancai/vscode-paste-image
* 推荐理由：vs code 写 markdown 时粘贴截图很不方便，需要先保存图片、再写插入图片的语句，十分繁琐，影响写作速度。有了这个插件之后，经过简单的配置，可以直接使用快捷键往 markdown 里面插入图片了，又少了一个使用 word、知乎写作的理由

### Remote - SSH / Remote Explorer
* 链接: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh
* 推荐理由：vs code 远程连接服务器，代码编写、命令行编译，一站式解决

### Jupyter
* 链接：https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter
* 推荐理由：python是处理数据、编写脚本工具的不二之选，Jupyter 是编写 python 的不二之选，分段执行 code snippet 功能巨方便。

## Ascii 码画图

* 在线工具：
  * https://textik.com
  * https://asciiflow.com
* 桌面程序：
  * mac：https://monodraw.helftone.com/，不支持中文，可以先导入，然后手动插入中文

推荐理由：写代码、博客时，需要画一些简单的架构、示意图，使用 ASCII 画出来的图体积更小，能用于文本编辑的场景。


## 装机必备
最近回购了公司的 intel 版本的 mbp，换了一台新的 m4 pro 的 mbp，由于芯片不兼容，没法使用迁移助手，只能重新装机。总结一下常用的一些工具，以备下次使用。

* zsh
    * oh-my-zsh: 命令行太丑，干活没动力^_^.  **特别注意，不要错装oh-my-posh，它用于 windows powershell**
    * [autosuggestion 插件](https://formulae.brew.sh/formula/zsh-autosuggestions): 命令行自动补全
    * [ zsh-syntax-highlighting](): **命令行**语法高亮

* vim
vim ~/.vimrc 粘贴以下配置，并去掉注释，否则报错
```shell
# 开启 语法高亮
syntax on
# 查找结果 高亮显示
set hlsearch
# 配色方案
colorscheme desert
# 关闭兼容模式
set nocompatible
# 解决vim 退格键（backspace）不能用
set backspace=indent,eol,start
```

* snipest: 截图很方便
* OneNote
* MicroSoft ToDo
* iTerm2