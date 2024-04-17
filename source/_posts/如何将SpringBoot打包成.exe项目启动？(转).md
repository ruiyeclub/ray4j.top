---
title: 如何将SpringBoot打包成.exe项目启动？(转)
date: 2024-03-05 12:39:48
tags:
---

## 前言

近期做了一个前后端合并的spring boot项目，但是要求打包城exe文件，提供给不懂电脑的小白安装使用，就去研究了半天，踩了很多坑，写这篇文章，是想看到这篇文章的人，按照我的步骤走，能少踩坑。

## 准备

准备工作：

1. 一个jar包，没有bug能正常启动的jar包
2. [exe4j](https://exe4j.apponic.com/)，一个将jar转换成exe的工具，链接：`https://pan.baidu.com/s/1J30uUMJcYnqWCJSr6gkM5w `提取码：6esr
   注册码：L-g782dn2d-1f1yqxx1rv1sqd
3. [inno setup](https://jrsoftware.org/isdl.php)，一个将依赖和exe一起打成一个安装程序的工具，链接：`https://pan.baidu.com/s/1DgFo1ceM\_8Bqx\_b-veibbQ`提取码：g9jd

## 开始

以我为例子，我将jar包放在了桌面

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111543565-1396048921.png)

 

jar包放在桌面打开安装好的exe4j

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111600449-1354345087.png)

 

打开exe4j直接下一步进入界面，选择JAVA转EXE

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111610780-2132812247.png)

 

JAVA转EXE然后点下一步，输入名称和输出路径

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111619283-1606783867.png)

 

输入名称和输出路径继续点击下一步，选择启动模式

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111647166-1055422528.png)

 

选择启动模式下方有个选项，需要设置打包后的程序兼容32和64位系统

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111656096-898994566.png)

 

兼容32和64位系统进来后勾选上

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111704684-1345085588.png)

 

勾选然后一直下一步，一直出现如下界面，开始选择jar包以及配置在VM参数配置的地方加上：`-Dfile.encoding=utf-8`

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111739256-1971491709.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111747934-1026729198.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111755257-424525494.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111800109-1087786170.png)

 

点击下一步，配置JRE

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111827329-840979154.png)

 

配置JRE下拉框点击后进入如下界面

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111838992-1945360511.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111848124-1110985164.png)

 

下拉框点击

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111901893-628430600.png)

 

照着这个样子写的目的是，最终会把本地jre目录和exe一起打包，让exe文件自己去根据路径去查找一起打包的jre，可不用再安装jdk

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111937871-1409793481.png)

 

接着下一步，选择Client VM

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509111955182-1164832840.png)

 

选择Client VM然后一直下一步，最终出现如下界面![图片](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112509693-1948806647.png)

 

这个时候你会发现桌面多了一个`demo.exe`文件，这个时候先别着急点开，接下来就是将jre和exe文件再打个包合并，达到在没有jdk电脑环境下也能运行打开inno setup，左上角File - New

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112041717-1500637696.png)

 

打开inno setup直接点下一步，填写配置，应用名称，版本等，随意

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112057607-1301322721.png)

 

版本然后点击下一步，这个地方默认就行，直接下一步

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112112186-624409517.png)

 

直接下一步接着选择生成好的exe文件

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112120569-1427010105.png)

 

接着选择生成好的exe文件然后下一步，进入这个界面保持默认，直接下一步

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112129814-1671272672.png)

 

保持默认依旧下一步，不用管

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112138411-293405841.png)

 

下一步继续下一步，这里是选择语言

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112146171-409628643.png)

 

选择语言然后就是选择输出路径和填写安装程序的名字了

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112155246-276457879.png)

 

选择输出路径然后下一步，直接点Next，然后结束配置到最后一步了，脚本文件，到这里会弹出问你是否马上编译，选择否，先把脚本写好再自己编译

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112233707-1556689797.png)

 

编译然后到了最后一步了，把本地的JRE写进脚本

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112247175-713534534.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112313666-1584429067.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112325941-471763229.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112334907-851084806.png)

 

``

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112343729-1930111580.png)

 

```xml
Source: "自己本地JRE路径\*"; DestDir: "{app}\{#MyJreName}"; Flags: ignoreversion recursesubdirs createallsubdirs
```

然后直接编译就好了，会提示保存当前脚本，随便起个名字，下个还可以继续用

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112358190-36465892.png)

 

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112415031-718978884.png)

 

然后等待绿色滚动条结束

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112426523-1804405720.png)

 

等待绿色滚动条结束当绿色滚动条结束后，桌面会多了一个`setup.exe`文件

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112437772-311845179.png)

 

Java打包ext也同时会跳出一个安装的，因为程序帮你自动启动生成的安装程序了，安装就可以了，安装的时候记得勾选创建快捷方式

![img](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/1608160-20230509112448655-1218116363.png)

 　

创建快捷方式这个就是最后的程序了，双击运行就可以看到结果了，把`setup.exe`文件给别人安装，就都可以看到自己的程序了。