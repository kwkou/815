<properties linkid="develop-nodejs-common-tasks-working-with-node-modules" urlDisplayName="Working with Node.js Modules" pageTitle="Working with Node.js Modules" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="Node.js" title="Using Node.js Modules with Azure applications" authors="larryfr" solutions="" manager="paulettm" editor="mollybos" />

# 将 Node.js 模块与 Azure 应用程序一起使用

本文档提供有关将 Node.js 模块与托管在 Azure 中的应用程序一起使用的指南。其中提供有关确保你的应用程序使用特定版本的模块，以及对 Azure 使用本机模块的指南。

如果你已了解如何使用 Node.js 模块、**package.json** 和 **npm-shrinkwrap.json** 文件，可参考本文中所讨论内容的以下快速摘要：

-   Azure 网站了解 **package.json** 和 **npm-shrinkwrap.json** 文件，可基于这些文件中的条目安装模块。
-   Azure 云服务希望所有模块都安装在开发环境中，并将 **node\_modules** 目录包含为部署包的一部分。

<div class="dev-callout">
<strong>说明</strong>
<p>本文不讨论 Azure 虚拟机，因为 VM 中的开发体验将取决于由虚拟机托管的操作系统。</p>
</div>

<div class="dev-callout">
<strong>说明</strong>
<p>可以为使用 <b>package.json</b> 或 <b>npm-shrinkwrap.json</b> 文件在 Azure 上安装模块提供相应支持，但这需要自定义云服务项目使用的默认脚本。有关如何实现此目的的示例，请参阅<a href="http://nodeblog.azurewebsites.net/startup-task-to-run-npm-in-azure">运行 npm 安装以避免部署 Node 模块的 Azure 启动任务</a></p>
</div>

## Node.js 模块

模块是可加载的 JavaScript 包，可为你的应用程序提供特定功能。通常使用 **npm** 命令行工具安装模块，但一些模块（如 http 模块）是作为核心 Node.js 包的一部分提供的。

在安装模块后，这些模块存储在你的应用程序目录结构的根目录下的 **node\_modules** 目录中。**node\_modules** 目录中的每个模块都保留自己的 **node\_modules** 目录，其中包含它依赖的所有模块，依赖项链上的每个模块均是如此。这使得所安装的每个模块都对它所依赖的模块具有自己的版本要求，但这样会生成很大的目录结构。

与使用 **package.json** 或 **npm-shrinkwrap.json** 文件相比，在你的应用程序中部署 **node\_modules** 目录会增加部署的大小；但是，它确实可以保证在生产中使用的模块版本与在开发中使用的模块版本是相同的。

### 本机模块

虽然多数模块都只是纯文本 JavaScript 文件，但一些模块是特定于平台的二进制映像。这些模块通常是在安装时使用 Python 和 node-gyp 编译的。Azure 网站的一个特定限制是，尽管它本来了解如何安装在 **package.json** 或 **npm-shrinkwrap.json** 文件中指定的模块，但它不提供 Python 或 node-gyp，也无法生成本机模块。

由于 Azure 云服务依赖作为应用程序一部分部署的 **node\_modules** 文件夹，因此只要作为已安装模块一部分包含的任何本机模块是在 Windows 开发系统中安装和编译的，那么该模块都应在云服务中运行。

Azure 网站不支持本机模块。一些模块（如 JSDOM 和 MongoDB）具有可选的本机依赖项，可用于托管在 Azure 网站中的应用程序。

### 使用 package.json 文件

可使用 **package.json** 文件来指定你的应用程序所需的顶级依赖项，以便托管平台能够安装这些依赖项，而不是要求你在部署中包含 **node\_packages** 文件夹。部署应用程序后，可使用 **npm install** 命令解析 **package.json** 文件并安装列出的所有依赖项。

在开发期间，你可以在安装模块时使用 **--save**、**--save-dev** 或 **--save-optional** 参数，以便自动将模块条目添加到 **package.json** 文件中。有关详细信息，请参阅 [npm-install][npm-install]。

**package.json** 文件的一个潜在问题是它仅指定顶级依赖项的版本。安装的每个模块不一定会指定它所依赖的模块的版本，所以你最终使用的依赖项链可能与开发过程中使用的不同。

> [WACOM.NOTE]
> 部署到 Azure 网站时，如果 **package.json** 文件引用本机模块，那么在使用 Git 发布应用程序时你会看到如下错误：

>       npm ERR! module-name@0.6.0 install: 'node-gyp configure build'

>       npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1    

### 使用 npm-shrinkwrap.json 文件

**npm-shrinkwrap.json** 文件用于尝试消除 **package.json** 文件的模块版本控制限制。虽然 **package.json** 文件仅包含顶级模块的版本，但 **npm-shrinkwrap.json** 文件包含所有模块依赖项链的版本要求。

你的应用程序准备好生产后，便可锁定版本要求，并使用 **npm shrinkwrap** 命令创建 **npm-shrinkwrap.json** 文件。这将使用当前安装在 **node\_modules** 文件夹中的版本，并将这些信息记录到 **npm-shrinkwrap.json** 文件。将应用程序部署到托管环境后，可使用 **npm install** 命令来解析 **npm-shrinkwrap.json** 文件并安装列出的所有依赖项。有关详细信息，请参阅 [npm-install][npm-install]。

> [WACOM.NOTE]
> 部署到 Azure 网站时，如果 **npm-shrinkwrap.json** 文件引用本机模块，那么在使用 Git 发布应用程序时你会看到如下错误：

>       npm ERR! module-name@0.6.0 install: 'node-gyp configure build'

>       npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1

## 后续步骤

了解如何将 Node.js 模块与 Azure 一起使用后，请了解如何[指定 Node.js 版本][指定 Node.js 版本]、[生成和部署 Node.js 网站][生成和部署 Node.js 网站]以及[如何使用适用于 Mac 和 Linux 的 Azure 命令行工具][如何使用适用于 Mac 和 Linux 的 Azure 命令行工具]。

  [运行 npm 安装以避免部署 Node 模块的 Azure 启动任务]: http://nodeblog.azurewebsites.net/startup-task-to-run-npm-in-azure
  [npm-install]: https://npmjs.org/doc/install.html
  [指定 Node.js 版本]: /zh-cn/documentation/articles/nodejs-specify-node-version-azure-apps/
  [生成和部署 Node.js 网站]: /zh-cn/documentation/articles/web-sites-nodejs-develop-deploy-mac/
  [如何使用适用于 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/documentation/articles/xplat-cli/
