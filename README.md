# Tumo-next-themes

>   A simple and beautiful hexo themes.

## Introduce 

`tumo-next-themes` is a [Hexo](https://hexo.io/zh-cn/) themes, it change from [Next](http://theme-next.iissnan.com/) theme. It's very simple enough, removed bloted style from Next theme.



Theme style reference [https://mrbird.cc/](https://mrbird.cc/).

## Simple

[https://tycoding.cn/](https://tycoding.cn/)

## 操作指令（初次运行没有hexo 执行npm install -g hexo-cli命令）

`hexo init` 命令用于初始化本地文件夹为网站的根目录

`hexo new` 命令用于新建文章，一般可以简写为 hexo n

`hexo generate` 命令用于生成静态文件，一般可以简写为 hexo g

`hexo server` 命令用于启动本地服务器，一般可以简写为 hexo s

`hexo deploy` 命令用于部署网站，一般可以简写为 hexo d

`hexo clean` 命令用于清理缓存文件，是一个比较常用的命令

## Get Started

>   1.  Get theme code

```shell
cd your_blog_root/

git clone https://github.com/TyCoding/tumo-next.git themes/tumo-next
```



>   2.  Edit you_blog_root/_config.yml

```yaml
theme: tumo-next
```

## Document

-   [https://hexo.io/zh-cn/](https://hexo.io/zh-cn/)
-   [http://theme-next.iissnan.com/](http://theme-next.iissnan.com/)

## Config

>    Change Sidebar Menu

Sidebar Menu 对应了Hexo social Links，其中每个链接 `||` 后面是Font Awesome 图标名称，你可以修改 `themes/tumo-next/_config.yml` 配置改变侧边栏菜单: 

```yaml
social:
  GitHub: https://github.com/TyCoding || github
```



>   Change Header Menu

Header Menu 和Next主题一致，但是移除了Font Awesome 图标的显示，你需要按照我以下的配置写，编辑文件 `themes/tumo-next/_config.yml`: 

```yaml
menu:
  home: /
  archives: /archives/
  about: /about/
  links: /links/
```

另外，Header Menu 是相对固定的，如果你需要新增页面（URL），你需要使用`hexo new page xx` 命令在`source`目录下创建新的markdown文件，Hexo将选择这个文件作为新的页面。



>   Change Top Menu 

你也可以设定全局置顶的链接，这里提供一种简洁的方式，仅仅需要修改 `themes/tumo-next/_config.yml`:

```yaml
top:
  links: 
    '🐤SpringCloud分布式项目脚手架': 'https://github.com/TyCoding/cloud-template'
```

## About

-   [https://tycoding.cn/](https://tycoding.cn/)
-   QQ Group: 671017003