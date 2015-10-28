<properties pageTitle="使用 CLI 捕获运行 Linux 的虚拟机的映像" description="了解如何捕获运行 Linux 的 Azure 虚拟机 (VM) 的映像。" services="virtual-machines" documentationCenter="" authors="karthmut" manager="madhana" editor="tysonn"/>
<tags ms.service="virtual-machines"
    ms.date="02/20/2015"
    wacn.date="04/15/2015"
    />


# 如何使用 CLI 捕获将用作模板的 Linux 虚拟机##



本文将展示如何捕获运行 Linux 的 Azure 虚拟机，以便你可以将其像模板一样用来创建其他虚拟机。此模板包括 OS 磁盘和附加到虚拟机的任何数据磁盘。它不包括网络配置，因此你在使用此模板创建其他虚拟机时需要配置网络配置。



Azure 将此模板视为一个映像并将其存储在你的映像列表中。这也是你上载和存储任何映像的地方。有关映像的详细信息，请参阅[关于 Azure 中的虚拟机映像] []。



## 开始之前##



这些步骤假定你已创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果你尚未执行此操作，请参阅以下说明：



- [如何创建自定义虚拟机] []

- [如何将数据磁盘附加到虚拟机] []



## 捕获虚拟机##



1. 连接到虚拟机。有关详细信息，请参阅[如何登录到运行 Linux 的虚拟机][]。



2. 只有当 VM 的状态为 **StoppedDeallocated** 时才能捕获 VM。在 SSH 窗口中，键入以下命令来关闭你的 VM：



        vm shutdown [options] <vm-name>



    注意，适用于  `vm shutdown` 的选项之一是 `-p`，这在关闭时将保留计算资源。**不要**启用此选项，因为要捕获 VM，必须将其解除配置。



3. 运行以下命令来捕获你的 VM 映像：



        vm capture <vm-name> <target-image-name> --deleted



    若要捕获映像，必须删除 VM。可以使用 `--deleted` 或 `-t` 来执行此操作。



4. 可以通过检查你的 VM 映像列表来确认是否已捕获了你的映像：



        vm image list | grep <target-image-name>



    `| grep <target-image-name>` 部分是可选的，但是可以帮助你更轻松地在列表中查找映像。



下面是捕获虚拟机的一个示例演练，其中包括终端输出：


    ~$ azure vm list

    info:    Executing command vm list

    + Getting virtual machines

    data:    Name         Status            Location  DNS Name                  IP Address

    data:    -----------  ----------------  --------  ------------------------  ------------

    data:    kmorig       RoleStateUnknown  China North   kmorig.chinacloudapp.cn       100.92.56.16

    info:    vm list command OK

    ~$ azure vm shutdown kmorig

    info:    Executing command vm shutdown

    + Getting virtual machines

    + Shutting down VM

    info:    vm shutdown command OK

    ~$ azure vm list

    info:    Executing command vm list

    + Getting virtual machines

    data:    Name         Status              Location  DNS Name                  IP Address

    data:    -----------  ------------------  --------  ------------------------  ----------

    data:    kmorig       StoppedDeallocated  China North   kmorig.chinacloudapp.cn

    info:    vm list command OK

    ~$ azure vm capture kmorig kmcaptured --deleted

    info:    Executing command vm capture

    + Getting virtual machines

    + Checking image with name kmcaptured exists

    + Capturing VM

    info:    vm capture command OK

    ~$ azure vm image list | grep kmcaptured

    data:    kmcaptured



有关更多详细信息和其他命令，请访问 [Azure CLI 文档页面][]。


[Azure CLI 文档页面]: /documentation/articles/virtual-machines-command-line-tools
[如何登录到运行 Linux 的虚拟机]: /documentation/articles/virtual-machines-linux-how-to-log-on
[关于 Azure 中的虚拟机映像]: http://msdn.microsoft.com/zh-cn/library/azure/dn790290.aspx

[如何创建自定义虚拟机]: /documentation/articles/virtual-machines-create-custom
[如何将数据磁盘附加到虚拟机]: /documentation/articles/storage-windows-attach-disk
<!--HONumber=50-->