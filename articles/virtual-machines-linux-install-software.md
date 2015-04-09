<properties linkid="manage-linux-commontasks-install-software" urlDisplayName="Install software on VM" pageTitle="Install software on a Linux virtual machine - Azure" metaKeywords="" description="Learn how to install software on your Linux virtual machine in Azure by using CentOS/Red Hat or Ubuntu." metaCanonical="" services="virtual-machines" documentationCenter="" title="Install software on your Linux virtual machine in Azure" authors="" solutions="" manager="" editor="" />

# 通过 Azure 在 Linux 虚拟机上安装软件

Linux 分发倾向于使用软件“程序包”安装软件。通常使用一系列命令管理这些程序包，例如`apt` 或`yum`。你还可以不使用程序包来安装程序，例如使用源代码的 *tarball*。

我们将介绍如何使用程序包管理器完成一些常见的 Linux 分发。相关步骤因你所使用的 Linux 分发而异。

**注意：**根据你的环境的设置方式，可能需要使用根权限（通过`sudo`）运行这些命令。

## CentOS/Red Hat

CentOS 带有`yum` ，可用于程序包管理。使用此工具，你可以安装、卸载、更新、列出已安装程序包，还可执行更多操作。有关这些命令的语法，请参阅以下内容。

### 安装

将安装一个程序包，以及它所依赖的任何程序包。由于存在依赖项，可能会安装多个程序包。

    yum install [package name]

### 卸载

将从你的计算机中卸载程序包。请记住，它不会删除任何依赖项。

    yum remove [package name]

### 正在更新…

这会将程序包更新至最新版本。必须先安装程序包，之后才能更新该程序包。

    yum update [package name]

### 列出已安装程序包

将在你的计算机上显示所有已安装程序包的列表。

    yum list installed

## Ubuntu

Ubuntu 带有`apt` （高级打包工具），可用于程序包管理。使用此工具，你可以安装、卸载、更新、列出已安装程序包，还可执行更多操作。有关这些命令的语法，请参阅以下内容。

### 安装

将安装一个程序包，以及它所依赖的任何程序包。由于存在依赖项，可能会安装多个程序包。

    apt-get install [package name]

### 卸载

将从你的计算机中卸载程序包。请记住，它不会删除任何依赖项。

    apt-get remove [package name]

### 更新/升级

若要升级至新版本，你将需要使用两个命令：一个命令用于更新你的程序包索引，另一个用于升级程序包。运行以下命令以更新程序包索引：

    apt-get update

更新程序包索引后，请运行以下命令升级程序包：

    apt-get upgrade
