归纳记录个人工作环境的配置文件和配置步骤

## 介绍
在2026.4.21日正式拥抱linux

> linux 永远的神

- 系统：Linux Debian 13
- 界面：Gnome
- Shell：Zsh
- 终端：kitty


## 安装系统 + 整合
Debian13无网络安装iso文件 + 拷盘工具 
由b站up准备好的自动化脚本，安装基础环境等
[整合](https://github.com/cold-land/dont-step-linux-pits)
- 包含gcc\g++\tmux\python3\等等需要的环境,可以按需删除


## 需要的包/软件
gh (Github 命令行认证登陆用）

wl-clipboard (在kitty中的nvim同步系统粘贴板需要)

python3-Nautilus  (允许编辑gnome的文件管理器)

nodejs   （nvim pyright补全python所需要，neovim mason使用nodejs将lsp自动下载到mason管理的目录下）

fastfetch (带图形的命令行系统配置展示插件)

Make(c\c++常用自动化编译工具)

YAZI(终端文件管理器)
## npm
使用官方库，还有密钥(会提醒报错)：`curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`

## mysql
数据库应用 
1. 下载配置包`wget https://dev.mysql.com/get/mysql-apt-config_0.8.36-1_all.deb`
2. 添加源目录`sudo dpkg -i mysql-apt-config_0.8.36-1_all.deb`
3. 更新软件包索引`sudo apt update`
4. 安装mysql服务器`sudo apt install mysql-server`
5. 检查mysql状态`sudo systemctl status mysql`
6. 启动mysql服务`sudo systemctl start mysql`

## Redis
数据库应用
1. `sudo apt install redis-server`
2. `sudo systemctl status redis-server`

## dbeaver
数据库图形化界面 
1. 配置公钥匙 `wget -O - https://dbeaver.io/debs/dbeaver.gpg.key | sudo tee /usr/share/keyrings/dbeaver.gpg.key`
2. 添加官方Apt源`echo "deb [signed-by=/usr/share/keyrings/dbeaver.gpg.key] https://dbeaver.io/debs/dbeaver-ce /" | sudo tee /etc/apt/sources.list.d/dbeaver.list` 
3. 安装`sudo apt install dbeaver-ce`


## wps缺失字体
> https://gitcode.com/Premium-Resources/a6802


## 1


## Gnome
- 主题：WhiteSur(MacOsLike)
- 图标：WhiteSur
- 光标：WHiteSur
- 扩展:
  - AppIndicator and KStatusNotifierItem Support   (支持托盘图标)
  - blur my shell                                  (模糊效果)
  - clipboard indicater                            (剪贴板历史插件)
  - Dash to Dock                                   (优化 Dock 栏)
  - Desktop Icons NG                               (支持桌面放置图标)
  - Quick Settings Tweaks                          
  - User Theme X

## Kitty 
见配置文件，特别方便的终端

## Nvim
### 已安装的插件

|插件|用途|状态|
|---|---|---|
|**mason.nvim**|LSP/DAP/linter/formatter 安装管理|✅|
|**blink.cmp**|代码补全引擎|✅|
|**nvim-lspconfig**|LSP 配置桥接|✅|
|**mini.pairs**|自动括号配对|✅|
|**lualine.nvim**|状态栏|✅|
|**catppuccin/nvim**|主题|✅|
|**hop.nvim**|快速跳转|✅|
|**live-preview.nvim**|浏览器实时预览 HTML|✅|
|**nvim-treesitter**|代码语法解析器|✅|
|**nvim-ts-autotag**|自动闭合 HTML 标签|✅|

### 通过 Mason 安装的工具

| 工具                                         | 用途                |
| ------------------------------------------ | ----------------- |
| **tree-sitter-cli**                        | 编译 treesitter 解析器 |
| **html-lsp** (vscode-html-language-server) | HTML 语言服务器        |
| **lua_ls** (lua-language-server)           | Lua 语言服务器         |
| **clangd**                                 | C/C++ 语言服务器       |
| **pyright**                                | Python 语言服务器      |

## python

#todo

linux系统会依赖python包，所以要建立虚拟环境(本质目录),避免pip管理的包在全局影响系统，导致不可预测的崩溃行为。

命令：
