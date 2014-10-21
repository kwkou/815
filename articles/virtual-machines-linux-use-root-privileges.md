<properties linkid="manage-linux-common-tasks-user-root-privileges" urlDisplayName="Use root privileges" pageTitle="Use root privileges on Linux virtual machines in Azure" metaKeywords="" description="Learn how to use root privileges on a Linux virtual machine in Azure." metaCanonical="" services="virtual-machines" documentationCenter="" title="Using root privileges on Linux virtual machines in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中使用 Linux 虚拟机的根权限

用户可以使用`sudo` 命令在 Linux 虚拟机上使用提升的特权来运行命令。但是，体验可能因系统设置方式而异。

1.  **SSH 密钥和密码或仅密码** - 虚拟机设置时使用的是证书（`.CER` 文件）和密码，或者只是用户名和密码。在这种情况下，`sudo` 在执行命令前将提示用户输入密码。

2.  **仅 SSH 密钥** - 虚拟机设置时使用的是证书（`.cer` 或`.pem` 文件），但没有密码。在这种情况下，`sudo` 在执行命令前**将不**提示用户输入密码。

## SSH 密钥和密码，或仅密码

使用 SSH 密钥或密码身份验证登录到 Linux 虚拟机中，然后使用`sudo`运行命令，例如：

    # sudo <command>
    [sudo] password for azureuser:

在这种情况下，将会提示用户输入密码。输入密码后，`sudo` 将使用`root` 特权来运行命令。

## 仅 SSH 密钥

使用 SSH 密钥身份验证登录到 Linux 虚拟机中，然后使用`sudo`运行命令，例如：

    # sudo <command>

在这种情况下，将**不**会提示用户输入密码。在按`<enter>` 键后，`sudo` 将使用`root` 特权来运行命令。

