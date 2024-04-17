---
title: 将Python脚本打包成可执行的exe文件
date: 2024-03-14 14:24:09
tags:
---

> 将Python程序打包为可执行的.exe文件有多种方法。以下是其中两种常用的方式。



## 使用PyInstaller库进行打包

1. 首先确保已经安装了PyInstaller库（如果没有，则通过命令`pip install pyinstaller`来安装）；

2. 在命令提示符或终端中导航到要打包的Python脚本所在目录；

3. 运行命令 `pyinstaller --onefile your_script.py`，这会生成一个名为"dist"的文件夹，并在该文件夹内创建一个与脚本同名的.exe文件。

**注意：此方法只适用于单个Python脚本的打包。**



## 使用cx_Freeze库进行打包

1. 首先确保已经安装了cx_Freeze库（如果没有，则通过命令`pip install cx-freeze`来安装）；

2. 创建一个名为setup.py的新文件，并添加以下内容：

   ```python
    from distutils.core import setup
   import sys
   
   if len(sys.argv) == 1 or 'bdist_wininst' not in sys.argv[1]:
       # 设置需要打包的模块及入口点
       modules = ['myapp']
       
       options = {
           "build_exe": {
               "includes": ["atexit"],
               "excludes": [],
               "packages": [],
               "path": "",
            },
         }
       
       setup(name="YourProgram", version="0.1", description="Description of Your Program", author="Author Name", url="http://www.example.com/", console=["myapp.py"])
   else:
       pass
   ```

3. 在命令提示符或终端中导航到包含上述setup.py文件的目录；

4. 运行命令 `python setup.py bdist_wininst`，这会生成一个名为"dist"的文件夹，并在该文件夹内创建一个与脚本同名的.exe文件。

**注意：此方法更适合对多个Python模块进行打包。**
