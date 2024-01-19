---
title: Linux环境下部署Python脚本
date: 2024-01-10 14:32:41
tags:
---

> 系统用的是Centos，自带了python环境，考虑到后期维护使用virtualenv虚拟环境来为应用创建"隔离"的python运行环境。
>
> virtualenv虚拟环境是一个创建隔绝的Python环境的工具。

![image-20240110124003769](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240110124003769.png)

- **隔离性**：virtualenv可以创建一个隔离的环境，这个环境与其他Python环境（如系统环境或全局环境）完全隔离。这意味着，在这个环境中安装的包不会影响其他环境，反之亦然。这有助于解决不同项目之间的依赖冲突问题。
- **易于管理**：每个虚拟环境都有自己的`pip`版本，这意味着你可以为每个项目安装特定版本的`pip`。此外，每个环境都有自己的Python解释器，这使得版本控制变得简单。
- **安全性**：由于每个环境都是隔离的，因此在一个环境中安装的包不会影响其他环境。这意味着在一个环境中安装的恶意包不会影响其他环境。
- **灵活性**：你可以为不同的项目创建不同的虚拟环境，每个环境都有自己的Python解释器和pip版本。这使得你可以根据项目的需求来配置环境。
- **易于使用**：virtualenv易于安装和使用。你只需要在命令行中输入一些命令就可以创建新的环境，安装包等。这对于初学者来说非常友好。



### 一、部署项目


1.导出项目所需要的依赖包

```xml
pip freeze > requements.txt
```

2.创建python虚拟环境（env_name当前环境的名称）

```xml
python -m venv env_name
```

创建后会生成一个文件夹，里面包含了项目运行所需要的环境

![image-20240110125725202](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240110125725202.png)

3.启动虚拟环境

```xml
source env_name/bin/activate
```

3.启动python脚本

```xml
pip install -r requements.txt # 先导入依赖包
python xx.py # 运行程序文件
```

4.关闭虚拟环境命令

```xml
deactivate
```



## 二、后台运行python脚本

1.nohup命令可以使你在注销账户或关闭终端时，程序仍然保持运行。该命令的基本语法为：

```xml
nohup python your_program.py &
或者：nohup python your_program.py >app.log 2>&1 &
```

&1是一个指向标准输出的指针，2是标准错误输出的文件描述符，该命令会将标准输出和标准错误写入到app.log

2.其中，`your_program.py`为需要后台运行的Python程序文件名，`&`符号表示在后台运行程序。运行上述命令后，你可以看到如下输出：

```xml
nohup: ignoring input and appending output to 'nohup.out'
```

3.这意味着你的Python程序已经在后台运行，并将输出信息保存在nohup.out文件中。你可以使用`tail`命令跟踪该文件以查看程序输出信息。如果你想停止程序的运行，可以使用`kill`命令。示例代码如下：

```xml
tail -f nohup.out
kill PID
```

4.其中，`PID`为你程序的进程ID，在运行`nohup`命令后你可以使用：

```xml
ps -ef | grep python # 命令查找进程ID。
```


## 三、升级python版本

1.[官网下载安装包]([www.python.org/downloads/](https://link.juejin.cn/?target=https%3A%2F%2Fwww.python.org%2Fdownloads%2F))

2.解压：`tar  -xf   Python-3.10.4.tgz`

3.进入解压后的目录：`./configure && make && make install `

4.先找到python文件，准备创建软链接

![image-20240119145905288](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240119145905288.png)

5.创建软连接（/usr/local/bin/python3.10是存放python文件目录）

```xml
ln -sf /usr/local/bin/python3.10 /usr/bin/python
ln -sf /usr/local/bin/python3.10-config /usr/bin/python-config
```


## 四、踩坑记录

1.使用nohup命令后，日志文件内没有日志输出，我遇到的问题是存在输出缓存，通过修改启动命令解决：

```xml
nohup python -u your_program.py >app.log 2>&1 &
```

参考文章：[Python nohup 启动python脚本，后台没有日志](https://blog.csdn.net/xunfengdumo/article/details/132448270)