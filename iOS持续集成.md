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

创建Job的方式有多种，本次只需要创建`Freestyle project`类型的即可。

> `Main page` -> `New Item` -> `Freestyle project`

对于一个持续集成打包平台，每次打包都由4步组成：触发构建、拉取代码、执行构建、构建后处理。对应的，在每个Job中也对应了这几项的配置。

#配置Git代码仓库
---
要对项目进行构建，配置项目的代码仓库是必不可少的。由于当前我们的项目托管在GitHub私有仓库中，因此需要对Git进行配置。

在`【Source Code Management】`配置栏目下，如果之前`GIT plugin`安装成功，则会出现`Git`选项。

配置Git代码仓库时，有三项是必须配置的：仓库URL地址（`Repository URL`）、仓库权限校验方式（`Credentials`），以及当前Job需要构建的代码分支（`Branches to build`）。

在配置`Repository URL`时，选择`HTTPS URL`或`SSH URL`均可。不过需要注意的是，`Credentials`要和`Repository URL`对应。

在配置`Branches to build`时，可以采用多种形式，包括分支名称`（branchName）`、`tagName`、`commitId`等。其中分支名称的形式用的最多，例如，若是构建`master`分支，则填写`refs/heads/master`，若是构建`develop`分支，则填写`refs/heads/develop`。

除了以上关于Git的必填配置项，有时根据项目的实际情况，可能还需要对Jenkins的默认配置项进行修改。

比较常见的一种情况就是对clone的配置进行修改。

在Jenkins的默认配置中，clone代码时会拉取所有历史版本的代码，而且默认的超时时限只有10分钟。这就造成在某些项目中，由于代码量本身就比较大，历史版本也比较多，再加上网络环境不是特别好，Jenkins根本没法在10分钟之内拉取完所有代码，超时后任务就会被自动终止了（错误状态码143）。

这种问题的解决方式也很简单，无非就是两种思路，要么少拉取点代码（不获取历史版本），要么提高超时时限。对应的配置在`Advanced clone behaviours`中：

- `Shallow clone`：勾选后不获取历史版本；
- `Timeout (in minutes) for clone and fetch operation`：配置后覆盖默认的超时时限。

#配置构建触发器
---
代码仓库配置好了，意味着Jenkins具有了访问GitHub代码仓库的权限，可以成功地拉取代码。

那Jenkins什么时候执行构建呢？

这就需要配置构建触发策略，即构建触发器，配置项位于【`Build Triggers】`栏目。

触发器支持多种类型，常用的有：

- 定期进行构建（Build periodically）
- 根据提交进行构建（Build when a change is pushed to GitHub）
- 定期检测代码更新，如有更新则进行构建（Poll SCM）

构建触发器的选择为复合选项，若选择多种类型，则任一类型满足构建条件时就会执行构建工作。如果所有类型都不选择，则该Jenkins Job不执行自动构建，但可通过手动点击`【Build Now】`触发构建。

#配置构建方式
---
触发策略配置好之后，Jenkins就会按照设定的策略自动执行构建。但如何执行构建操作，这还需要我们通过配置构建方式来进行设定。

常用的构建方式是根据构建对象的具体类型，安装对应的插件，然后采用相应的构建方式。例如，若是构建iOS应用，安装Xcode integration插件之后，就可以选择Xcode，然后选择Xcode进行构建。

不过，如果是采用打包脚本进行构建的话，情况就会简单许多。只要在Jenkins所运行的计算机中安装好开发者证书，打包命令在Shell中能正常工作，那么在Jenkins中执行打包脚本也不会有什么问题。


#构建后处理
---
完成构建后，生成的编译成果物（ipa）会位于指定的目录中。但是，如果要直接在手机中安装ipa文件还比较麻烦，不仅在分发测试包时需要将好几十兆的安装包进行传送，体验用户在安装时也还需要通过数据线将手机与计算机进行连接，然后再使用PP助手或豌豆荚等工具进行安装。

当前比较优雅的一种方式是借助蒲公英或fir.im等平台，将ipa文件上传至平台后由平台生成二维码，然后只需要对二维码链接进行分发，体验用户通过手机扫描二维码后即可实现快速安装，效率得到了极大的提升。

###上传安装包文件，生成二维码

不管是`蒲公英`还是`fir.im`，都有对应的Jenkins插件，安装插件后可以在`Post-build`中实现对安装包的上传。

除了使用Jenkins插件，fir.im还支持命令上传的方式，蒲公英还支持HTTP Post接口上传的方式。

我个人推荐采用命令或接口上传的方法，并在构建脚本中进行调用。灵活是一方面，更大的好处是如果上传失败后还能进行重试，这在网络环境不是很稳定的情况下极其必要。

Jenkins成功完成安装包上传后，`pgyer/fir.im`平台会生成一个二维码图片，并在响应中将图片的URL链接地址进行返回。

###展示二维码图片

二维码图片的URL链接有了，那要怎样才能将二维码图片展示在Jenkins项目的历史构建列表中呢？

这里需要用到另外一个插件，`description setter plugin`。安装该插件后，在`【Post-build Actions】`栏目中会多出`description setter`功能，可以实现构建完成后设置当次build的描述信息。这个描述信息不仅会显示在build页面中，同时也会显示在历史构建列表中。

有了这个前提，要将二维码图片展示在历史构建列表中貌似就可以实现了，能直观想到的方式就是采用`HTML`的`img`标签，将`<img src='qr_code_url'>`写入到build描述信息中。

`Manage Jenkins` -> `Configure Global Security`，将`Markup Formatter`的设置更改为`Safe HTML`即可。更改配置后，我们就可以在build描述信息中采用`HTML`的`img`标签插入图片了。


###收集编译成果物（Artifacts）
每次完成构建后，编译生成的文件较多，但是并不是所有的文件都是我们需要的。

通常情况下，我们可能只需要其中的部分文件，例如.ipa/.app/.plist/.apk等，这时我们可以将这部分文件单独收集起来，并在构建页面中展示出来，以便在需要时进行下载。

要实现这样一个功能，需要在`【Post-build Actions】`栏目中新增`Archive the artifacts`，然后在`Files to archive`中通过正则表达式指定成果物文件的路径。

设置完毕后，每次构建完成后，Jenkins会在`Console Output`中采用设定的正则表达式进行搜索匹配，如果能成功匹配到文件，则会将文件收集起来。

###总结
本文主要是对如何使用Jenkins搭建iOS持续集成打包平台的基础概念和实施流程进行了介绍。



