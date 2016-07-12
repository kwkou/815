<properties
	pageTitle="使用负载平衡集群集化 Linux 上的 MySQL"
	description="该文章作为示例说明了在 Azure 中使用 MySQL 设置负载平衡的高可用性 Linux 群集的模式"
	services="virtual-machines"
	documentationCenter=""
	authors="bureado"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/14/2015"
	wacn.date="01/21/2016"/>

# 使用负载平衡集群集化 Linux 上的 MySQL

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

## 介绍

本文旨在探索并说明可用于在 Azure 上部署基于 Linux 的高度可用服务的不同方法，并作为入门书探索 MySQL Server 的高可用性。

我们将基于 DRBD、Corosync 和 Pacemaker 概述无共享双节点单主机 MySQL 高可用性解决方案。一次只有一个节点运行 MySQL。读取和写入 DRBD 资源也限制为一次只有一个节点。

无需使用类似 LVS 的 VIP 解决方案，因为我们使用 Azure 负载平衡集来同时提供轮循功能和 VIP 的终结点检测、删除和正常恢复功能。VIP 是在首次创建云服务时由 Azure 分配的全局可路由 IPv4 地址。

MySQL 的其他可能体系结构包括 NBD 群集、Percona 和 Galera 以及多个中间件解决方案，其中至少有一个以 [VM Depot](http://vmdepot.msopentech.com) 中的 VM 的形式提供。只要这些解决方案可以复制到单播、多播或广播上，并且不要依赖于共享存储或多个网络接口，这些方案就应该很容易在 Azure 上部署。

当然，这些群集体系结构可以类似方式扩展到其他产品（如 PostgreSQL 和 OpenLDAP）。例如，此无共享的负载平衡过程已成功在多主机 OpenLDAP 上测试，并可以在我们的第 9 频道博客上观看。

## 做好准备

你将需要一个具有有效订阅可以创建至少两 (2) 个虚拟机的 Azure 帐户（在此示例中使用 XS）、一个网络和一个子网、一个地缘组和一个可用性集，以及能够在同一区域中创建新的 VHD 作为云服务，并将这些 VHD 附加到 Linux 虚拟机。

### 已测试的环境

- Ubuntu 13.10
  - DRBD
  - MySQL Server
  - Corosync 和 Pacemaker

### 地缘组

解决方案的地缘组是通过登录到 Azure 经典管理门户，向下滚动到“设置”并创建新地缘组来创建的。稍后创建的已分配资源将分配给此地缘组。

### 网络

将创建新网络，并在该网络内部创建子网。我们选择了其中只有一个 /24 子网的 10.10.10.0/24 网络。

### 虚拟机

第一个 Ubuntu 13.10 VM 是使用 Endorsed Ubuntu 库映像并调用 `hadb01` 创建的。在此过程中，将创建名为 hadb 的新云服务。我们这样命名它是为了说明当我们添加更多资源时该服务所具有的共享的负载平衡性质。创建 `hadb01` 是平淡无奇的，可以使用经典管理门户完成。将自动创建 SSH 的终结点，并选择我们创建的网络。我们还选择为虚拟机创建新的可用性集。

创建第一个虚拟机后（从技术上讲，创建云服务时），我们将继续创建第二个虚拟机 `hadb02`。对于第二个虚拟机，我们还将通过经典管理门户使用库中的 Ubuntu 13.10 VM，但我们将选择使用现有的云服务 `hadb.chinacloudapp.cn`，而不是创建一个新的。应为我们自动选择网络和可用性集。也将创建 SSH 终结点。

创建两个 VM 后，我们将记下 `hadb01` 的 SSH 端口 (TCP 22) 和 `hadb02`（由 Azure 自动分配）

### 附加存储

我们将新磁盘附加到这两个 VM，并在进程中创建新的 5 GB 磁盘。磁盘将在 VHD 容器中托管，用作我们的主要操作系统磁盘。创建并附加磁盘后，无需重启 Linux，因为内核将会发现新设备（通常为 `/dev/sdc`，你可以检查 `dmesg` 的输出）

在每个虚拟机上，我们继续使用 `cfdisk` 创建新分区（主 Linux 分区），并写入新的分区表。**不要在此分区上创建文件系统**。

## 设置群集

在这两个 Ubuntu VM 中，将需要使用 APT 安装 Corosync、Pacemaker 和 DRBD。使用 `apt-get`：

    sudo apt-get install corosync pacemaker drbd8-utils.

**这次不安装 MySQL**。Debian 和 Ubuntu 安装脚本将在 `/var/lib/mysql` 上初始化 MySQL 数据目录，但由于该目录将由 DRBD 文件系统取代，因此我们需要稍后执行此操作。

此时我们还应该验证（使用 `/sbin/ifconfig`）这两个 VM 是否使用 10.10.10.0/24 子网中的地址，并且它们是否可以用名称彼此 ping 通。如果需要，还可以使用 `ssh-keygen` 和 `ssh-copy-id` 以确保这两个 VM 可以通过 SSH 通信而无需密码。

### 设置 DRBD

我们将创建一个 DRBD 资源，该资源使用基础 `/dev/sdc1` 分区生成能够使用 ext3 格式化并在主节点和辅助节点中使用的 `/dev/drbd1` 资源。若要执行此操作，请打开 `/etc/drbd.d/r0.res` 并复制以下资源定义。在这两个 VM 中执行此操作：

    resource r0 {
      on `hadb01` {
        device  /dev/drbd1;
        disk   /dev/sdc1;
        address  10.10.10.4:7789;
        meta-disk internal;
      }
      on `hadb02` {
        device  /dev/drbd1;
        disk   /dev/sdc1;
        address  10.10.10.5:7789;
        meta-disk internal;
      }
    }

执行此操作后，在这两个 VM 中使用 `drbdadm` 初始化资源：

    sudo drbdadm -c /etc/drbd.conf role r0
    sudo drbdadm up r0

最后，在主节点 (`hadb01`) 中强制实施 DRBD 资源的所有权（主）：

    sudo drbdadm primary --force r0

如果你在这两个 VM 中检查 /proc/drbd (`sudo cat /proc/drbd`) 的内容，你应在 `hadb01` 上看到 `Primary/Secondary`，在 `hadb02` 上看到 `Secondary/Primary`，与此时的解决方案保持一致。5 GB 磁盘将通过 10.10.10.0/24 网络进行同步，而不会向客户收取费用。

同步磁盘后，便可以在 `hadb01` 上创建文件系统了。出于测试目的，我们使用了 ext2，但以下指令将创建一个 ext3 文件系统：

    mkfs.ext3 /dev/drbd1

### 装载 DRBD 资源

在 `hadb01` 上，我们现已准备好装载 DRBD 资源。Debian 和派生物使用 `/var/lib/mysql` 作为 MySQL 的数据目录。由于我们尚未安装 MySQL，我们将创建该目录并装载 DRBD 资源。在 `hadb01` 上：

    sudo mkdir /var/lib/mysql
    sudo mount /dev/drbd1 /var/lib/mysql

## 设置 MySQL

现在你已准备好在 `hadb01` 上安装 MySQL：

    sudo apt-get install mysql-server

对于 `hadb02`，可以使用两个选项。现在可以安装 mysql-server 了，这将创建 /var/lib/mysql 并填入一个新的数据目录，然后继续执行以删除内容。在 `hadb02` 上：

    sudo apt-get install mysql-server
    sudo service mysql stop
    sudo rm –rf /var/lib/mysql/*

第二个选项用于故障转移到 `hadb02`，然后在该处安装 mysql-server（安装脚本会注意到现有安装并且不会动它）

在 `hadb01` 上：

    sudo drbdadm secondary –force r0

在 `hadb02` 上：

    sudo drbdadm primary –force r0
    sudo apt-get install mysql-server

如果你现在不打算故障转移 DRBD，则第一个选项虽说算不上极好但更容易。设置此项后，但可以开始处理 MySQL 数据库了。在 `hadb02`（或根据 DRBD 处于活动状态的服务器之一）上：

    mysql –u root –p
    CREATE DATABASE azureha;
    CREATE TABLE things ( id SERIAL, name VARCHAR(255) );
    INSERT INTO things VALUES (1, "Yet another entity");
    GRANT ALL ON things.* TO root;

**警告**：此最后一条语句有效地对该表中的根用户禁用身份验证。这应替换为生产级别 GRANT 语句，并且仅为说明目的才包括在内。

如果要从 VM 外部进行查询，则还需要为 MySQL 启用网络，这是本指南的目的。在这两个 VM 上，打开 `/etc/mysql/my.cnf` 并浏览到 `bind-address`，将它从 127.0.0.1 更改为 0.0.0.0。保存该文件之后，在当前主节点上发出 `sudo service mysql restart`。

### 创建 MySQL 负载平衡集

我们将返回到 Azure 经典管理门户并浏览到 `hadb01` VM，然后终结点。我们将创建一个新的终结点，从下拉列表中选择 MySQL (TCP 3306)，并勾选*“新建负载平衡集”*框。我们将调用我们的负载平衡终结点 `lb-mysql`。我们将保留大多数选项不动，但时间除外，我们会将其减少到 5（秒，最小值）

创建终结点后，我们将转到 `hadb02` 终结点并创建新的终结点，但我们将选择 `lb-mysql`，然后从下拉菜单中选择 MySQL。还可以将 Azure CLI 用于此步骤。

此时，我们已具备对群集执行手动操作所需的一切条件。

### 测试负载平衡集

可以从外部计算机使用任何 MySQL 客户端以及应用程序（例如，作为 Azure Web 应用运行的 phpMyAdmin）执行测试。在这种情况下，我们在另一台 Linux 计算机上使用 MySQL 的命令行工具：

    mysql azureha –u root –h hadb.chinacloudapp.cn –e "select * from things;"

### 手动故障转移

现在可以通过关闭 MySQL、切换 DRBD 的主节点并重启 MySQL 来模拟故障转移。

在 hadb01 上：

    service mysql stop && umount /var/lib/mysql ; drbdadm secondary r0

然后，在 hadb02 上：

    drbdadm primary r0 ; mount /dev/drbd1 /var/lib/mysql && service mysql start

手动故障转移后，你可以重复执行远程查询，并且它应该能够完美运行。

## 设置 Corosync

Corosync 是使 Pacemaker 工作所需的基础群集基础结构。对于检测信号 v1 和 v2 用户（以及 Ultramonkey 等其他方法），Corosync 是 CRM 功能的拆分，而 Pacemaker 将保持更类似于功能中的检测信号。

Azure 上 Corosync 的主要约束是 Corosync 首选多播，其次广播，再其次单播通信，但 Azure 网络仅支持单播。

幸运的是，Corosync 具有一种工作单播模式，并且唯一的真正约束是，由于并非所有节点之间都*自动*互相通信，因此需要在配置文件中定义节点，包括其 IP 地址。我们可以对单播使用 Corosync 示例文件，并只需更改绑定地址、节点列表和日志记录目录（Ubuntu 使用 `/var/log/corosync`，而示例文件使用 `/var/log/cluster`），并启用仲裁工具。

**请记下下面的 `transport: udpu` 指令以及为节点手动定义的 IP 地址**。

在两个节点的 `/etc/corosync/corosync.conf` 上：

    totem {
      version: 2
      crypto_cipher: none
      crypto_hash: none
      interface {
        ringnumber: 0
        bindnetaddr: 10.10.10.0
        mcastport: 5405
        ttl: 1
      }
      transport: udpu
    }

    logging {
      fileline: off
      to_logfile: yes
      to_syslog: yes
      logfile: /var/log/corosync/corosync.log
      debug: off
      timestamp: on
      logger_subsys {
        subsys: QUORUM
        debug: off
        }
      }

    nodelist {
      node {
        ring0_addr: 10.10.10.4
        nodeid: 1
      }

      node {
        ring0_addr: 10.10.10.5
        nodeid: 2
      }
    }

    quorum {
      provider: corosync_votequorum
    }

我们将此配置文件复制到这两个 VM 中，并在这两个节点中启动 Corosync：

    sudo service start corosync

启动该服务后不久，群集应在当前环中建立，并应构成仲裁。我们可以通过查看日志或以下项来检查此功能：

    sudo corosync-quorumtool –l

应符合类似于下图所示的输出：

![corosync-quorumtool -l sample output](./media/virtual-machines-linux-classic-mysql-cluster/image001.png)

## 设置 Pacemaker

Pacemaker 使用群集监视资源、定义主节点何时停机，并将这些资源切换到辅助节点。可以通过一组可用脚本或 LSB（类似 init）脚本以及其他选项定义资源。

我们想要 Pacemaker“拥有”DRBD 资源、装入点和 MySQL 服务。如果在主节点出现问题时 Pacemaker 可以启用和关闭 DRBD，请按正确的顺序装载/卸载它，并启动/停止 MySQL，我们的设置已完成。

首次安装 Pacemaker 时，你的配置应足够简单，如下所示：

    node $id="1" hadb01
      attributes standby="off"
    node $id="2" hadb02
      attributes standby="off"

通过运行 `sudo crm configure show` 来检查它。现在，使用以下资源创建一个文件（假设 `/tmp/cluster.conf`）：

    primitive drbd_mysql ocf:linbit:drbd \
          params drbd_resource="r0" \
          op monitor interval="29s" role="Master" \
          op monitor interval="31s" role="Slave"

    ms ms_drbd_mysql drbd_mysql \
          meta master-max="1" master-node-max="1" \
            clone-max="2" clone-node-max="1" \
            notify="true"

    primitive fs_mysql ocf:heartbeat:Filesystem \
          params device="/dev/drbd/by-res/r0" \
          directory="/var/lib/mysql" fstype="ext3"

    primitive mysqld lsb:mysql

    group mysql fs_mysql mysqld

    colocation mysql_on_drbd \
           inf: mysql ms_drbd_mysql:Master

    order mysql_after_drbd \
           inf: ms_drbd_mysql:promote mysql:start

    property stonith-enabled=false

    property no-quorum-policy=ignore

并立即将其加载到配置中（只需在一个节点上执行此操作）：

    sudo crm configure
      load update /tmp/cluster.conf
      commit
      exit

此处，请确保 Pacemaker 在这两个节点中引导时启动：

    sudo update-rc.d pacemaker defaults

几秒钟后，请使用 `sudo crm_mon –L` 验证你的节点之一是否已成为群集的主机并且正在运行所有资源。可以使用 mount 和 PS 来检查资源是否正在运行。

下面的屏幕截图显示一个节点已停止的 `crm_mon`（使用 Control-C 退出）

![crm\_mon 节点已停止](./media/virtual-machines-linux-classic-mysql-cluster/image002.png)

并且此屏幕截图显示这两个节点（一个主节点和一个从节点）：

![crm\_mon 操作主/从](./media/virtual-machines-linux-classic-mysql-cluster/image003.png)

## 测试

我们已准备好进行自动故障转移模拟。可通过两种方式执行此操作：软方式和硬方式。软方式使用群集的关闭功能：``crm_standby -U `uname -n` -v on``。在主节点上使用此功能时，从节点将接管。记住将此功能重新设为 off（crm_mon 会指示一个节点处于启用状态，其他节点处入待机状态）

硬方式是通过经典管理门户关闭主虚拟机 (hadb01) 或更改虚拟机上的运行级别（即，停止、关闭），然后，我们将通过指示主节点将要关闭来帮助 Corosync 和 Pacemaker。我们可以对此进行测试（适用于维护窗口），但我们也可以通过只冻结虚拟机来强制实施该方案。

## STONITH

它应该能够通过 Azure CLI 代替用于控制物理设备的 STONITH 脚本向 VM 发出关闭命令。可以使用 `/usr/lib/stonith/plugins/external/ssh` 作为基础并在群集的配置中启用 STONITH。应全局安装 Azure CLI 并应为群集的用户加载发布设置/配置文件。

资源的示例代码可在 [GitHub](https://github.com/bureado/aztonith) 上找到。你需要更改群集的配置，方法是将以下代码添加到 `sudo crm configure`：

    primitive st-azure stonith:external/azure \
      params hostlist="hadb01 hadb02" \
      clone fencing st-azure \
      property stonith-enabled=true \
      commit

**注意：** 该脚本不执行向上/向下检查。原始 SSH 资源使用 15 次 ping 检查，但 Azure VM 的恢复时间可能更多变。

## 限制

以下限制适用：

- 管理 DRBD 的 linbit DRBD 资源脚本作为 Pacemaker 中的资源在关闭节点时使用 `drbdadm down`，即使该节点只是转为待机状态也是如此。这并不理想，因为当主节点获得写入时，从节点将不会同步 DRBD 资源。如果主节点未优雅地失败，则从节点可以接受较旧的文件系统状态。可通过两种可能方式来解决此问题：
  - 在所有群集节点上通过本地（非群集化）监视程序强制执行 `drbdadm up r0`，或者
  - 编辑 linbit DRBD 脚本以确保未在 `/usr/lib/ocf/resource.d/linbit/drbd` 中调用 `down`。
- 负载平衡器至少需要 5 秒钟才能做出响应，因此应用程序应是群集感知的并更容忍超时；其他体系结构也会有帮助，例如应用程序中队列、查询中间件等。
- 有必要进行 MySQL 优化以确保以合理的速度完成写入，并且尽可能频繁地将缓存刷新到磁盘
- 写入性能将依赖于虚拟交换机中的 VM 互连，因为这是 DRBD 用于复制设备的机制

<!---HONumber=79-->