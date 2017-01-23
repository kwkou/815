<!-- need to be verified -->

<properties
    pageTitle="为 Linux VM 创建 SSH 密钥对 | Azure"
    description="以安全方式为 Linux VM 创建 SSH 公钥和私钥对。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="" />
<tags 
    ms.assetid="34ae9482-da3e-4b2d-9d0d-9d672aa42498"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="12/14/2016"
    wacn.date="01/20/2017"
    ms.author="v-livech" />

# 为 Linux VM 创建 SSH 公钥和私钥对

本文说明如何生成要用于 Linux VM 的 SSH 公钥和私钥对。使用 SSH 密钥对，可以在 Azure 上创建默认为使用 SSH 密钥进行身份验证的虚拟机，从而无需密码就可登录。密码可能被猜到，将 VM 向不间断的强力尝试开放，用于猜测密码。使用 Azure 模板或 `azure-cli` 创建的 VM 可以在部署过程中提供 SSH 公钥，并删除部署后配置步骤（即禁用 SSH 的密码登录）。

## 快速命令

从 Bash shell 运行以下命令，并将示例替换为自己所选项。

SSH 密钥默认保留在 `~/.ssh` 目录中。如果没有 `~/.ssh` 目录，`ssh-keygen` 命令将使用正确的权限为你创建一个。`-N` cli 标志是用于加密 SSH 私钥的密码，*不*是用户密码。

    ssh-keygen \
    -t rsa \
    -b 2048 \
    -C "ahmet@myserver" \
    -f ~/.ssh/id_rsa \
    -N mypassword

现在，`~/.ssh` 目录中包含 `id_rsa` 和 `id_rsa.pub` SSH 密钥对。

    ls -al ~/.ssh

请验证 `ssh-agent` 是否正在运行：

    eval "$(ssh-agent -s)"

将新创建的密钥添加到 `ssh-agent`：

    ssh-add ~/.ssh/id_rsa

如果已创建 VM，可以使用以下命令将新的 SSH 公钥安装到 Linux VM：

    ssh-copy-id -i ~/.ssh/id_rsa.pub ahmet@myserver

使用密钥而不是密码测试登录：

    ssh -o PreferredAuthentications=publickey -o PubkeyAuthentication=yes -i ~/.ssh/id_rsa ahmet@myserver
    Last login: Tue April 12 07:07:09 2016 from 66.215.22.201
    $

如果未提示你输入 SSH 私钥密码或 VM 的登录密码，则 SSH 已成功配置。

## 详细演练

使用 SSH 公钥和私钥是登录到 Linux 服务器的最简单方法。[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)提供了一种比密码安全得多的方法，用于登录到 Azure 中的 Linux 或 BSD VM，密码更容易受到暴力攻击。

公钥可与任何人共享；但只有你（或本地安全基础结构）才拥有你的私钥。SSH 私钥应使用[非常安全的密码](https://www.xkcd.com/936/)（源：[xkcd.com](https://xkcd.com)）来保护它。此密码只用于访问 SSH 私钥，**不是**用户帐户密码。向 SSH 密钥添加密码时，它会使用 128 位 AES 对私钥进行加密，因此在没有密码解密的情况下，私钥不可用。如果攻击者窃取了私钥，并且该密钥没有密码，那么他们就能使用该私钥登录到具有相应公钥的任何服务器。如果私钥受密码保护，攻击者就无法使用，它会为 Azure 基础结构提供一个额外的安全层。

本文将创建 *ssh-rsa* 格式的密钥文件，建议将这些文件用于 Resource Manager 上的部署。在[门户](https://portal.azure.cn)上进行经典部署和 Resource Manager 部署时需要 *ssh-rsa* 密钥。

## 通过使用 SSH 密钥禁用 SSH 密码

Azure 需要至少 2048 位采用 ssh-rsa 格式的公钥和私钥。为了创建密钥，将使用 `ssh-keygen`，它会询问一系列问题，然后编写私钥和匹配的公钥。创建 Azure VM 时，会将公钥复制到 `~/.ssh/authorized_keys`。`~/.ssh/authorized_keys` 中的 SSH 密钥用于在 SSH 登录连接时质询客户端以匹配相应的私钥。创建的 Azure Linux VM 使用 SSH 密钥进行身份验证时，Azure 将 SSHD 服务器配置为不允许密码登录，仅允许使用 SSH 密钥。因此，使用 SSH 密钥创建 Azure Linux VM 可帮助保护 VM 部署，并省去了在 sshd\_config 配置文件中禁用密码的典型部署后配置步骤。

## 使用 ssh-keygen

此命令使用 2048 位 RSA 创建密码保护的（加密）SSH 密钥对，并为其加上注释以方便识别。

SSH 密钥默认保留在 `~/.ssh` 目录中。如果没有 `~/.ssh` 目录，`ssh-keygen` 命令将使用正确的权限为你创建一个。

    ssh-keygen \
    -t rsa \
    -b 2048 \
    -C "ahmet@myserver" \
    -f ~/.ssh/id_rsa \
    -N mypassword

*命令解释*

`ssh-keygen` = 用于创建密钥的程序

`-t rsa` = 要创建的密钥类型，即 RSA 格式 [wikipedia](https://en.wikipedia.org/wiki/RSA_(cryptosystem)

`-b 2048` = 密钥的位数

`-C "myusername@myserver"` = 追加到公钥文件末尾以便于识别的注释。通常以电子邮件作为注释，但也可以使用任何最适合基础结构的事物。

## 经典管理门户和 X.509 证书

如果使用的是 Azure [经典管理门户](https://manage.windowsazure.cn/)，它需要将 X.509 证书用作 SSH 密钥。不允许使用任何其他类型的 SSH 公钥，SSH 公钥*必须*是 X.509 证书。

从现有的 SSH-RSA 私钥创建 X.509 证书：

    openssl req -x509 \
    -key ~/.ssh/id_rsa \
    -nodes \
    -days 365 \
    -newkey rsa:2048 \
    -out ~/.ssh/id_rsa.pem

## 使用 `asm` 进行经典部署

如果使用的是经典部署模型（Azure 服务管理 CLI `asm`），则可以在 pem 容器中使用 SSH-RSA 公钥或 RFC4716 格式的密钥。SSH-RSA 公钥已在本文的前面部分使用 `ssh-keygen` 创建。

从现有的 SSH 公钥创建 RFC4716 格式的密钥：

    ssh-keygen \
    -f ~/.ssh/id_rsa.pub \
    -e \
    -m RFC4716 > ~/.ssh/id_ssh2.pem

## ssh-keygen 的示例

    ssh-keygen -t rsa -b 2048 -C "ahmet@myserver"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/ahmet/.ssh/id_rsa): id_rsa
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in id_rsa.
    Your public key has been saved in id_rsa.pub.
    The key fingerprint is:
    14:a3:cb:3e:78:ad:25:cc:55:e9:0c:08:e5:d1:a9:08 ahmet@myserver
    The keys randomart image is:
    +--[ RSA 2048]----+
    |        o o. .   |
    |      E. = .o    |
    |      ..o...     |
    |     . o....     |
    |      o S =      |
    |     . + O       |
    |      + = =      |
    |       o +       |
    |        .        |
    +-----------------+

保存的密钥文件：

`Enter file in which to save the key (/home/ahmet/.ssh/id_rsa): ~/.ssh/id_rsa`  


本文中的密钥对名称。系统默认提供名为 **id\_rsa** 的密钥对，有些工具可能要求私钥文件名为 **id\_rsa**，因此最好使用此密钥对。目录 `~/.ssh/` 是 SSH 密钥对和 SSH 配置文件的默认位置。如果未使用完整路径指定，则 `ssh-keygen` 将在当前工作目录（而不是默认 `~/.ssh`）中创建密钥。

`~/.ssh` 目录的列表。

    ls -al ~/.ssh
    -rw------- 1 ahmet staff  1675 Aug 25 18:04 id_rsa
    -rw-r--r-- 1 ahmet staff   410 Aug 25 18:04 rsa.pub

密钥密码：

`Enter passphrase (empty for no passphrase):`  


`ssh-keygen` 将密码称为“通行短语”。 *强烈*建议在密钥对中添加一个密码。如果不使用密码来保护密钥对，任何人只要拥有私钥文件，就可以用它登录到具有相应公钥的任何服务器。添加密码可提供更多保护以防有人能够获取私钥文件，可让用户有时间更改用于进行身份验证的密钥。

## 使用 ssh-agent 来存储私钥密码

为了避免在每次 SSH 登录时输入私钥文件密码，可以使用 `ssh-agent` 来缓存私钥文件密码。如果用户使用 Mac，OSX Keychain 将在用户调用 `ssh-agent` 时安全存储私钥密码。

验证并使用 ssh-agent 和 ssh-add 通知 SSH 系统有关密钥文件的信息，以便无需以交互方式使用密码。

    eval "$(ssh-agent -s)"

现在，使用命令 `ssh-add` 将私钥添加到 `ssh-agent`。

    ssh-add ~/.ssh/id_rsa

私钥密码现在存储在 `ssh-agent` 中。

## 使用 `ssh-copy-id` 安装新密钥
如果已创建 VM，可以使用以下命令将新的 SSH 公钥安装到 Linux VM：

    ssh-copy-id -i ~/.ssh/id_rsa.pub ahmet@myserver

## 创建并配置 SSH 配置文件

建议的最佳做法是，创建并配置 `~/.ssh/config` 文件以加快登录速度并优化 SSH 客户端行为。

以下示例显示了标准配置。

### 创建文件

    touch ~/.ssh/config

### 编辑文件以添加新的 SSH 配置：

    vim ~/.ssh/config

### 示例 `~/.ssh/config` 文件：

    # Azure Keys
    Host fedora22
      Hostname 102.160.203.241
      User ahmet
    # ./Azure Keys
    # Default Settings
    Host *
      PubkeyAuthentication=yes
      IdentitiesOnly=yes
      ServerAliveInterval=60
      ServerAliveCountMax=30
      ControlMaster auto
      ControlPath ~/.ssh/SSHConnections/ssh-%r@%h:%p
      ControlPersist 4h
      IdentityFile ~/.ssh/id_rsa

此 SSH 配置可以指定每个服务器的部分，以便它们各自获得专用的密钥对。默认设置 (`Host *`) 适用于不匹配配置文件上部列出的任何特定主机的任何主机。

### 配置文件解释

`Host` = 在终端上调用的主机名。`ssh fedora22` 告知 `SSH` 使用标记为 `Host fedora22` 的设置块中的值。注意：Host 可以是符合用途的任何标签，并不代表任何服务器的实际主机名。

`Hostname 102.160.203.241` = 所访问的服务器的 IP 地址或 DNS 名称。

`User ahmet` = 登录到服务器时要使用的远程用户帐户。

`PubKeyAuthentication yes` = 告知 SSH 要使用 SSH 密钥来登录。

`IdentityFile /home/ahmet/.ssh/id_id_rsa` = 要用于身份验证的 SSH 私钥和对应的公钥。

## 在不提供密码的情况下使用 SSH 连接到 Linux

创建 SSH 密钥对并配置 SSH 配置文件后，便可以快速安全地登录到 Linux VM。首次使用 SSH 密钥登录到服务器时，命令将提示用户输入该密钥文件的通行短语。

    ssh fedora22

### 命令解释

执行 `ssh fedora22` 后，SSH 先从 `Host fedora22` 块中找到并加载所有设置，然后从最后一个块 (`Host *`) 中加载所有剩余设置。

## 后续步骤

下一步是使用新 SSH 公钥创建 Azure Linux VM。使用 SSH 公钥作为登录名创建的 Azure VM 可以比使用默认登录方法（即密码）创建的 VM 享受更好的保护。使用 SSH 密钥创建的 Azure VM 默认情况下配置为禁用密码，以避免强力猜测尝试。

* [使用 Azure 模板创建安全 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
* [使用 Azure 门户预览创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
* [使用 Azure CLI 创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)

<!---HONumber=Mooncake_0116_2017-->