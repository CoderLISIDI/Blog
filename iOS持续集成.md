# 背景描述
---
根据项目需求，现要在团队内部搭建一个统一的打包平台，实现对iOS项目的打包。而且为了方便团队内部的测试包分发，希望在打包完成后能生成一个二维码，体验用户（产品、运营、测试等人员）通过手机扫描二维码后就能直接安装测试包。

接下来，本文就开始对平台建设的完整实现过程进行详细介绍。

# 安装Jenkins
---
Jenkins依赖于Java运行环境，因此需要首先安装[Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。

安装Jenkins的方式有多种，可以运行对应系统类型的安装包，可以通过docker获取镜像，也可以直接运行war包。

我个人倾向于直接运行war包的形式，只需下载jenkins.war后，运行如下命令即可启动Jenkins。

#Jenkins插件
---

Jenkins有非常多的插件，可以实现各种功能的扩展。

针对搭建的iOS持续集成打包平台，我使用到了如下几个插件。

* GIT plugin
* SSH Credentials Plugin
* Git Changelog Plugin: 获取仓库提交的commit log
* build-name-setter：用于修改Build名称
* description setter plugin：用于在修改Build描述信息，在描述信息中增加显示QRCode（二维码）
* Post-Build Script Plug-in：在编译完成后通过执行脚本实现一些额外功能

#创建项目（Job）
---
在Jenkins中，构建项目以Job的形式存在，因此需要针对每个项目创建一个Job。有时候，一个项目中可能有多个分支同时在进行开发，为了分别进行构建，也可以针对每个分支创建一个Job。

创建Job的方式有多种，本次只需要创建**Freestyle project**类型的即可。

> Main page -> New Item -> **Freestyle project** 

对于一个持续集成打包平台，每次打包都由4步组成：触发构建、拉取代码、执行构建、构建后处理。对应的，在每个Job中也对应了这几项的配置。

