<properties linkid="manage-linux-howto-linux-agent" urlDisplayName="Linux Agent guide" pageTitle="Azure Linux 代理用户指南" metaKeywords="" description="了解如何安装和配置 Linux 代理 (waagent) 以管理您的虚拟机与 Azure 结构控制器的交互。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Azure Linux 代理用户指南" authors="" solutions="" manager="" editor="" />

# Azure Linux 代理用户指南

## 介绍

Azure Linux 代理 (waagent) 管理虚拟机与 Azure 结构控制器的交互。它提供了适用于 Linux IaaS 部署的以下功能：

-   **映像设置**
-   创建用户帐户
-   配置 SSH 身份验证类型
-   部署 SSH 公钥和密钥对
-   设置主机名
-   将主机名发布到平台 DNS
-   将 SSH 主机密钥指纹报告给平台
-   资源磁盘管理
-   格式化并安装资源磁盘
-   配置交换空间
-   **联网**
-   管理路由以提高与平台 DHCP 服务器的兼容性
-   确保网络接口名称的稳定性
-   **内核**
-   配置虚拟 NUMA
-   将 Hyper-V 熵用于 /dev/random
-   为根设备配置 SCSI 超时（可能通过远程方式）
-   **诊断**
-   控制台重定向到串行端口
-   **SCVMM 部署**

    -   当用于 Linux 的 VMM 代理在 System
         Center Virtual Machine Manager 2012 R2 环境中运行时对其进行检测并启动

从平台到代理的信息流通过两个通道进行：

-   用于 IaaS 部署的附加了启动时间的 DVD。此 DVD 包含一个与 OVF 兼容的配置文件，该文件包括除 SSH 密钥对之外的所有配置信息。

-   用于获取部署和拓扑配置的一个公开 REST API 的 TCP 终结点。

### 获得 Linux 代理

可从以下位置直接获取最新的 Linux 代理：

<!---- [The different Distribution providers endorsing Linux on Azure](http://support.microsoft.com/kb/2805216)--->

-   或 [Azure Linux 代理的 Github 开放源存储库][Azure Linux 代理的 Github 开放源存储库]

### 支持的 Linux 分发

-   CentOS 6.2+
-   Debian 7.0+
-   Ubuntu 12.04+
-   openSUSE 12.3+
-   SLES 11 SP2+
-   Oracle Linux 6.4+

其他支持的系统：

-   FreeBSD 9+ (WALinuxAgent v2.0.0+)

### 要求

Waagent 依赖一些系统程序包才能正常运行：

-   Python 2.5+
-   Openssl 1.0+
-   Openssh 5.3+
-   文件系统实用工具：sfdisk、fdisk、mkfs
-   密码工具：chpasswd、sudo
-   文本处理工具：sed、grep
-   网络工具：ip-route

## 安装

使用您的分发包存储库中的 RPM 或 DEB 包进行安装是安装和升级 Windows Azure Linux Azure 的首选方法。

如果手动进行安装，应运行以下命令以将 waagent 复制到 /usr/sbin/waagent 并进行安装：

    # sudo chmod 755 /usr/sbin/waagent
    # /usr/sbin/waagent -install -verbose

代理的日志文件保留在 /var/log/waagent.log 中。

## 命令行选项

### 标志

-   verbose:增加指定命令的详细程度
-   force:跳过某些命令的交互式确认

### 命令

-   help:列出支持的命令和标志。

-   install:手动安装代理
-   在系统中查找必需的依赖项

-   创建 SysV init 脚本 (/etc/init.d/waagent) 和 logrotate 配置文件 (/etc/logrotate.d/waagent)，并配置映像以在启动时运行 init 脚本

-   将示例配置文件写入 /etc/waagent.conf

-   任何现有的配置文件都将移到 /etc/waagent.conf.old

-   检测内核版本并应用 VNUMA 解决方法（如有必要）

-   将可能影响网络的 udev 规则（/lib/udev/rules.d/75-persistent-net-generator.rules、/etc/udev/rules.d/70-persistent-net.rules）移至 /var/lib/waagent/

-   uninstall:删除 waagent 和关联的文件
-   从系统中注销 init 脚本并将其删除

-   删除 /etc/waagent.conf 中的 logrotate 配置和 waagent 配置文件

-   还原在安装过程中移动过的任何已移动的 udev 规则

-   不支持 VNUMA 解决方法的自动还原，如果需要，请手动编辑 GRUB 配置文件以重新启用 NUMA。

-   deprovision:尝试清除系统并使其适用于重新配置。此操作已删除以下各项：
-   所有 SSH 主机密钥（如果在配置文件中 Provisioning.RegenerateSshHostKeyPair 为“y”）

-   /etc/resolv.conf 中的 Nameserver 配置

-   /etc/shadow 中的根密码（如果在配置文件中 Provisioning.DeleteRootPassword 为“y”）

-   缓存的 DHCP 客户端租赁

-   将主机名重置为 localhost.localdomain

**警告：**取消设置无法保证清除映像中的所有敏感信息且适用于重新分发。

-   deprovision+user:执行 -deprovision（上述）下面的所有操作，还将删除最后设置的用户帐户（从 /var/lib/waagent 中获得）和关联数据。此参数是取消对以前在 Azure 中设置的映像的设置以便捕获并重新使用该映像时的参数。

-   version:显示 waagent 的版本

-   serialconsole:配置 GRUB 以将 ttyS0（第一个串行端口）标记为
    启动控制台。这可确保将内核启动日志发送到
    串行端口并适用于调试。

-   daemon:将 waagent 作为 daemon 运行以管理与平台的交互。
    在 waagent init 脚本中为 waagent 指定此参数。

## 配置

配置文件 (/etc/waagent.conf) 控制 waagent 的操作。
示例配置文件如下所示：

    #
    # Azure Linux Agent Configuration   
    #
    Role.StateConsumer=None 
    Role.ConfigurationConsumer=None 
    Role.TopologyConsumer=None
    Provisioning.Enabled=y
    Provisioning.DeleteRootPassword=n
    Provisioning.RegenerateSshHostKeyPair=y
    Provisioning.SshHostKeyPairType=rsa
    Provisioning.MonitorHostName=y
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource 
    ResourceDisk.EnableSwap=n 
    ResourceDisk.SwapSizeMB=0
    LBProbeResponder=y
    Logs.Verbose=n
    OS.RootDeviceScsiTimeout=300
    OS.OpensslPath=None

下面详细地描述了各种配置选项。
配置选项分为三类：布尔、字符串或整数。
布尔配置选项可指定为“y”或“n”。
特殊关键字“无”可用于某些字符串类型配置项，详细信息如下所示。

**Role.StateConsumer：**

类型：字符串
默认值：无

如果指定了指向可执行计划的路径，则在 waagent 已配置好映像且将要向结构报告“Ready”状态时调用此计划。为计划指定的参数将为“Ready”。代理不会等待计划返回，便开始继续操作。

**Role.ConfigurationConsumer：**

类型：字符串
默认值：无

如果指定了指向可执行计划的路径，则在结构指示配置文件对虚拟机可用时，将调用此计划。指向 XML 配置文件的路径作为可执行计划的参数提供。这可在配置文件发生更改时多次调用。附录中提供了示例文件。此文件的当前路径为 /var/lib/waagent/HostingEnvironmentConfig.xml。

**Role.TopologyConsumer：**

类型：字符串
默认值：无

如果指定了指向可执行计划的路径，则在结构指示新的网络拓扑布局对虚拟机可用时，将调用此计划。指向 XML 配置文件的路径将作为可执行计划的参数提供。这可在网络拓扑发生更改时（例如，由于服务修复）多次调用。附录中提供了示例文件。此文件的当前位置为 /var/lib/waagent/SharedConfig.xml。

**Provisioning.Enabled：**

类型：布尔
默认值：y

这允许用户在代理中启用或禁用设置功能。有效值为“y”或“n”。如果禁用设置，则会保留映像中的 SSH 主机和用户密钥，并忽略 Azure 设置 API 中指定的所有配置。

**Provisioning.DeleteRootPassword：**

类型：布尔
默认值：n

如果设置此参数，则会在设置过程中清除 /etc/shadow 文件中的根密码。

**Provisioning.RegenerateSshHostKeyPair：**

类型：布尔
默认值：y

如果设置此参数，则会在设置过程中从 /etc/ssh/ 中删除所有 SSH 主机密钥对（ecdsa、dsa 和 rsa）。并且会生成一个全新的密钥对。

此全新密钥对的加密类型可由 Provisioning.SshHostKeyPairType 项进行配置。请注意，在重新启动 SSH 监控程序时（例如，重新启动时），某些分发将为任何缺失的加密类型重新创建 SSH 密钥对。

**Provisioning.SshHostKeyPairType：**

类型：字符串
默认值：rsa

可将其设置为虚拟机上的 SSH 监控程序支持的加密算法类型。通常支持的值为“rsa”、“dsa”和“ecdsa”。请注意，Windows 上的“putty.exe”不支持“ecdsa”。因此，若要在 Windows 上使用 putty.exe 连接到 Linux 部署，请使用“rsa”或“dsa”。

**Provisioning.MonitorHostName：**

类型：布尔
默认值：y

如果设置此参数，则 waagent 将监视 Linux 虚拟机的主机名更改情况（由“hostname”命令返回），并自动更新映像中的网络配置以反映此更改。若要将名称更改推送到 DNS 服务器，可在虚拟机中重新启动网络。这将导致 Internet 连接暂时中断。

**ResourceDisk.Format：**

类型：布尔
默认值：y

如果设置此参数，则当“ResourceDisk.Filesystem”中用户请求的 filesystem 类型是“ntfs”之外的任何值时，平台提供的资源磁盘将通过 waagent 进行格式化和安装。将在磁盘上提供类型 Linux (83) 的单个分区。请注意，如果可以成功安装此分区，则将不会对其进行格式化。

**ResourceDisk.Filesystem：**

类型：字符串
默认值：ext4

这将指定资源磁盘的 filesystem 类型。支持的值随 Linux 分发的不同而不同。如果字符串为 X，则 mkfs.X 应呈现在 Linux 映像上。SLES 11 映像通常应使用“ext3”。FreeBSD 映像在此处应使用“ufs2”。

**ResourceDisk.MountPoint：**

类型：字符串
默认值：/mnt/resource

这将指定资源磁盘的安装路径。

**ResourceDisk.EnableSwap：**

类型：布尔
默认值：n

如果设置此参数，则将在资源磁盘上创建交换文件 (/swapfile) 并将该文件添加到系统交换空间。

**ResourceDisk.SwapSizeMB：**

类型：整数
默认值： 0

交换文件的大小，以兆字节为单位。

**LBProbeResponder：**

类型：布尔
默认值：y

如果设置此参数，则 waagent 将响应来自平台的负载平衡器探测（如果有）。

**Logs.Verbose：**

类型：布尔
默认值：n

如果设置此参数，则将增大日志的详细程度。Waagent 将日志记录到 /var/log/waagent.log 并使用系统 logrotate 功能来循环日志。

**OS.RootDeviceScsiTimeout：**

类型：整数
默认值： 300

这将配置 OS 磁盘和数据驱动器上的 SCSI 超时（以秒为单位）。如果未设置此参数，则使用系统默认值。

**OS.OpensslPath：**

类型：字符串
默认值：无

这可用于指定要用于加密操作的 openssl 二进制文件的替代路径。

## 附录

### 示例角色配置文件

    <?xml version="1.0" encoding="utf-8"?>
    <HostingEnvironmentConfig version="1.0.0.0" goalStateIncarnation="1">
      <StoredCertificates>
        <StoredCertificate name="Stored0Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" certificateId="sha1:C093FA5CD3AAE057CB7C4E04532B2E16E07C26CA" storeName="My" configurationLevel="System" />
      </StoredCertificates>
      <Deployment name="a99549a92e38498f98cf2989330cd2f1" guid="{374ef9a2-de81-4412-ac87-e586fc869923}" incarnation="14">
        <Service name="LinuxDemo1" guid="{00000000-0000-0000-0000-000000000000}" />
        <ServiceInstance name="a99549a92e38498f98cf2989330cd2f1.4" guid="{250ac9df-e14c-4c5b-9cbc-f8a826ced0e7}" />
      </Deployment>
      <Incarnation number="1" instance="LinuxVM_IN_2" guid="{5c87ab8b-2f6a-4758-9f74-37e68c3e957b}" />
      <Role guid="{47a04da2-d0b7-26e2-f039-b1f1ab11337a}" name="LinuxVM" hostingEnvironmentVersion="1" software="" softwareType="ApplicationPackage" entryPoint="" parameters="" settleTimeSeconds="10" />
      <HostingEnvironmentSettings name="full" Runtime="rd_fabric_stable.111026-1712.RuntimePackage_1.0.0.9.zip">
        <CAS mode="full" />
        <PrivilegeLevel mode="max" />
        <AdditionalProperties><CgiHandlers></CgiHandlers></AdditionalProperties></HostingEnvironmentSettings>
        <ApplicationSettings>
          <Setting name="__ModelData" value="&lt;m role=&quot;LinuxVM&quot; xmlns=&quot;urn:azure:m:v1&quot;>&lt;r name=&quot;LinuxVM&quot;>&lt;e name=&quot;HTTP&quot; />&lt;e name=&quot;Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp&quot; />&lt;e name=&quot;Microsoft.WindowsAzure.Plugins.RemoteForwarder.RdpInput&quot; />&lt;e name=&quot;SSH&quot; />&lt;/r>&lt;/m>" />
          <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountEncryptedPassword" value="..." />
          <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountExpiration" value="2015-11-06T23:59:59.0000000-08:00" />
          <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountUsername" value="rdos" />
          <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Enabled" value="true" />
          <Setting name="Microsoft.WindowsAzure.Plugins.RemoteForwarder.Enabled" value="true" />
          <Setting name="startpage" value="Hello World!" />
          <Setting name="Certificate|Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" value="sha1:C093FA5CD3AAE057CB7C4E04532B2E16E07C26CA" />
        </ApplicationSettings>
        <ResourceReferences>
          <Resource name="DiagnosticStore" type="directory" request="Microsoft.Cis.Fabric.Controller.Descriptions.ServiceDescription.Data.Policy" sticky="true" size="1" path="a99549a92e38498f98cf2989330cd2f1.LinuxVM.DiagnosticStore\" disableQuota="false" />
        </ResourceReferences>
      </HostingEnvironmentConfig>

### 示例角色拓扑文件

    <?xml version="1.0" encoding="utf-8"?>
    <SharedConfig version="1.0.0.0" goalStateIncarnation="2">
      <Deployment name="a99549a92e38498f98cf2989330cd2f1" guid="{374ef9a2-de81-4412-ac87-e586fc869923}" incarnation="14">
        <Service name="LinuxDemo1" guid="{00000000-0000-0000-0000-000000000000}" />
        <ServiceInstance name="a99549a92e38498f98cf2989330cd2f1.4" guid="{250ac9df-e14c-4c5b-9cbc-f8a826ced0e7}" />
      </Deployment>
      <Incarnation number="1" instance="LinuxVM_IN_1" guid="{a7b94774-db5c-4007-8707-0b9e91fd808d}" />
      <Role guid="{47a04da2-d0b7-26e2-f039-b1f1ab11337a}" name="LinuxVM" settleTimeSeconds="10" />
      <LoadBalancerSettings timeoutSeconds="32" waitLoadBalancerProbeCount="8">
        <Probes>
          <Probe name="LinuxVM" />
          <Probe name="03F7F19398C4358108B7ED059966EEBD" />
          <Probe name="47194D0E3AB3FCAD621CAAF698EC82D8" />
        </Probes>
      </LoadBalancerSettings>
      <OutputEndpoints>
        <Endpoint name="LinuxVM:Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" type="SFS">
          <Target instance="LinuxVM_IN_0" endpoint="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" />
          <Target instance="LinuxVM_IN_1" endpoint="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" />
          <Target instance="LinuxVM_IN_2" endpoint="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" />
        </Endpoint>
      </OutputEndpoints>
      <Instances>
        <Instance id="LinuxVM_IN_1" address="10.115.38.202">
          <FaultDomains randomId="1" updateId="1" updateCount="2" />
          <InputEndpoints>
            <Endpoint name="HTTP" address="10.115.38.202:80" protocol="tcp" isPublic="true" loadBalancedPublicAddress="70.37.56.176:80" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="80" to="80" />
              </LocalPorts>
            </Endpoint>
            <Endpoint name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" address="10.115.38.202:3389" protocol="tcp" isPublic="false" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="3389" to="3389" />
              </LocalPorts>
              <RemoteInstances>
                <RemoteInstance instance="LinuxVM_IN_0" />
                <RemoteInstance instance="LinuxVM_IN_2" />
              </RemoteInstances>
            </Endpoint>
            <Endpoint name="Microsoft.WindowsAzure.Plugins.RemoteForwarder.RdpInput" address="10.115.38.202:20000" protocol="tcp" isPublic="true" loadBalancedPublicAddress="70.37.56.176:3389" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="20000" to="20000" />
              </LocalPorts>
            </Endpoint>
            <Endpoint name="SSH" address="10.115.38.202:22" protocol="tcp" isPublic="true" loadBalancedPublicAddress="70.37.56.176:22" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="22" to="22" />
              </LocalPorts>
            </Endpoint>
          </InputEndpoints>
        </Instance>
        <Instance id="LinuxVM_IN_0" address="10.115.58.82">
          <FaultDomains randomId="0" updateId="0" updateCount="2" />
          <InputEndpoints>
            <Endpoint name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" address="10.115.58.82:3389" protocol="tcp" isPublic="false" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="3389" to="3389" />
              </LocalPorts>
            </Endpoint>
          </InputEndpoints>
        </Instance>
        <Instance id="LinuxVM_IN_2" address="10.115.58.148">
          <FaultDomains randomId="0" updateId="2" updateCount="2" />
          <InputEndpoints>
            <Endpoint name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Rdp" address="10.115.58.148:3389" protocol="tcp" isPublic="false" enableDirectServerReturn="false" isDirectAddress="false" disableStealthMode="false">
              <LocalPorts>
                <LocalPortRange from="3389" to="3389" />
              </LocalPorts>
            </Endpoint>
          </InputEndpoints>
        </Instance>
      </Instances>
    </SharedConfig>

  [Azure Linux 代理的 Github 开放源存储库]: https://github.com/WindowsAzure/WALinuxAgent
