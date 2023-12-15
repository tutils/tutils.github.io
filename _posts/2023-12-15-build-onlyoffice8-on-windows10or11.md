---
layout: post
title: Build OnllyOffice 8 on Windows 10/11
date: 2023-12-15 18:29:53 +0800
author: t5w0rd
categories: blogs
mermaid: true
---

## 编译步骤
### 1. 更改 Windows 默认编码，从 GBK 改为 UTF-8（解决 python 编译脚本无法正常读取文件问题）
  * a. 在 explorer 地址栏输入：控制面板\时钟和区域\区域
  * b. 管理 > 更改系统区域设置 > 勾选使用 Unicode UTF-8 
  * c. 重启系统
  * d. 验证（可选，完成 python 安装后验证即可）
    ```python
    import locale
    print(locale.getpreferredencoding())
    # 输出 cp936 代表 GBK        [X]
    # 输出 cp65001 代表 UTF-8    [√]
    ```
### 2. 准备编译工作区
  * a. 安装 [Git]()
  * b. 克隆编译工具仓库，注意仓库应存在一级上级目录，因为build_tools在编译过程中会自动下载其他OnlyOffice仓库到其平级目录中
    ```cmd
    cd C:\
    mkdir onlyoffice
    cd onlyoffice
    git clone -b release/v8.0.0 https://github.com/ONLYOFFICE/build_tools.git
    ```
  * c. 对编译工具增加 hack
    * i. v8 单任务作业（解决代码同步超时问题）
        ```python
        os.chdir(base_dir)
        if not base.is_dir("depot_tools"):
        base.cmd("git", ["clone", "https://chromium.googlesource.com/chromium/tools/depot_tools.git"])

        # hack to force 1 jobs
        if base.is_file("depot_tools/gclient.py"):
        base.replaceInFile("depot_tools/gclient.py", "self._options.jobs", "1")
        ```
  * d. 编译工作区备份，由于编译过程可能会破坏工作环境，导致无法重新编译，可以按照下面方法进行准备备份
    * i. 在调用 gn 和 ninja 编译前添加中断编译代码 > 备份整个编译工作区 > 删除中断代码段 > 重新启动编译
        ```python
        # backup workspace
        print("exit, backup workspace")
        sys.exit(0)
        ```
### 3. 准备其他依赖工具
  * a. Python 3
    ```cmd
    pip install setuptools
    ```
  * b. TortoiseSVN，安装时注意勾选上安装命令行工具
  * c. Windows 10 SDK, version 2004 (10.0.19041.0)，该这个版本以解决 v8 编译兼容性问题，同时解决缺失 cdb 的问题
  * d. Visual Studio Community 2019，同时 OnlyOffice8 编译脚本只支持 VS2015 和 VS2019，该版本解决不支持 /std:c++17 选项问题，
    * i. 安装勾选“使用C++的桌面开发”
    * ii. 在右侧安装详细信息中取消勾选全部 Windows SDK
  * e. Qt 5.15.2，安装时勾选 Qt > Qt 5.15.2 > MSVC 2019 64-bit。该版本解决默认 Qt5.9.9 最高只支持到 MSVC2017 的问题
  * f. NodeJS 14
    ```cmd
    npm install -g grunt-cli pkg yarn
    ```
  * g. Perl
### 4. 生成编译配置文件
    ```cmd
    # 如果编译全部模块：--module "desktop builder server"
    python configure.py --branch release/v8.0.0 --module desktop --update 1 --qt-dir C:\Qt\5.15.2 --platform win_64 --vs-version 2019
    ```
### 5. 编译，编译时使用普通 cmd 即可。生成目录在 build_tools\out 下
    ```cmd
    python make.py
    ```
### 6. 打包
