---
title: Mac上使用homebrew安装nvm
date: 2024-07-20 16:04:55
tags:
---

> NVM（Node Version Manager）是一个用于在基于Linux系统上安装和管理Node.js的shell脚本。macOS用户可以使用`homebrew`来安装NVM。

安装Homebrew（如果你还没有安装）：

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

1. 卸载现有node版本

如果你的系统已经安装了node，请先卸载它。

```bash
brew uninstall --ignore-dependencies node 
brew uninstall --force node 
```

2. 使用Homebrew安装nvm：

```sh
brew install nvm
```

3. 将nvm初始化脚本添加到你的shell配置文件中。如果你使用的是zsh，那么你需要在你的`~/.zshrc`文件中添加以下行：

```sh
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

4. 使更改生效，你可以重新启动你的终端，或者运行以下命令：

```sh
source ~/.zshrc
```

5. 验证nvm安装成功：

```sh
nvm --version
```

以上步骤将会在你的macOS系统上安装nvm，并且设置zsh使其能够识别nvm命令。



### 常用命令

```sh
nvm -v //查看nvm版本
nvm ls //查看本机已经安装node版本
nvm use 切换到指定版本node
nvm ls-remote //列出所有可安装的远程node版本
nvm install stable    //安装最新版本
nvm install <version> //安装指定版本
nvm uninstall stable   //卸载最新版本
nvm uninstall <version>   //卸载指定版本
nvm current //查看当前使用的node版本
npm which [current|<version>] //显示已安装node的安装路径。
nvm cache dir //显示nvm的缓存目录
nvm cache clear //清楚nvm的缓存目录
nvm node_mirror [url] //设置node镜像
nvm npm_mirror [url] //设置npm镜像
nvm on //开启node.js版本管理
nvm off //关闭node.js版本管理
 

更新npm到最新版本
npm install -g npm@latest
```



### 踩坑记录：

终端安装好了nvm和node，但是在vscode打开后无法执行命令，需要在` vi ~/.zshrc`加入`export PATH="~/.composer/vendor/bin:$PATH"`



### 参考文章：

[如何在macOS上安装NVM](https://juejin.cn/post/7083026831263137800)

[Mac下安装nvm后vscode输入node -v不起作用](https://www.cnblogs.com/famensaodiseng/p/10427673.html)
