<properties
  pageTitle="将云服务连接到自定义域控制器 | Azure"
  description="了解如何使用 Powershell 和 AD 域扩展将 Web/辅助角色连接到自定义 AD 域"
  services="cloud-services"
  documentationCenter=""
  authors="Thraka"
  manager="timlt"
  editor=""/>  


  <tags
    ms.service="cloud-services"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/04/2017"
    wacn.date="01/25/2017"
    ms.author="adegeo"/>


# 将 Azure 云服务角色连接到 Azure 中托管的自定义 AD 域控制器

首先，在 Azure 中建立一个虚拟网络 (VNET)。然后，将 Active Directory 域控制器（托管在 Azure 虚拟机上）添加该 VNET。接下来，将现有云服务角色添加预先创建的 VNET，随后将它们连接到域控制器。

在开始之前，请特别注意以下几点：

1.	本教程使用 Powershell，因此请确保已安装 Azure Powershell 并已准备就绪。有关设置 Azure Powershell 的帮助，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

2.	AD 域控制器和 Web/辅助角色实例需要在 VNET 中。

按照以下分步指南操作，如果遇到任何问题，请在下面留言。我们将回复你（没错，我们真的会阅读留言）。

云服务引用的网络<mark>必须是</mark>**经典虚拟网络**。

## 创建虚拟网络

可使用 Azure 经典管理门户或 Powershell 在 Azure 中创建虚拟网络。在本教程中，我们将使用 Powershell。若要使用 Azure 经典管理门户创建虚拟网络，请参阅[创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)。


    #创建虚拟网络
    $vnetStr =
    @"<?xml version="1.0" encoding="utf-8"?>
    <NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
      <VirtualNetworkConfiguration>
        <VirtualNetworkSites>
          <VirtualNetworkSite name="[your-vnet-name]" Location="China North">
            <AddressSpace>
              <AddressPrefix>[your-address-prefix]</AddressPrefix>
            </AddressSpace>
            <Subnets>
              <Subnet name="[your-subnet-name]">
                <AddressPrefix>[your-subnet-range]</AddressPrefix>
              </Subnet>
            </Subnets>
          </VirtualNetworkSite>
        </VirtualNetworkSites>
      </VirtualNetworkConfiguration>
    </NetworkConfiguration>
    "@;

    $vnetConfigPath = "<path-to-vnet-config>"
    Set-AzureVNetConfig -ConfigurationPath $vnetConfigPath;

## 创建虚拟机

完成虚拟网络设置后，需要创建 AD 域控制器。在本教程中，将在 Azure 虚拟机上设置 AD 域控制器。

为此，请使用以下命令通过 Powershell 创建虚拟机。

	# Initialize variables
	# VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.

	$vnetname = '<your-vnet-name>'
	$subnetname = '<your-subnet-name>'
	$vmsvc1 = '<your-hosted-service>'
	$vm1 = '<your-vm-name>'
	$username = '<your-username>'
	$password = '<your-password>'
	$affgrp = '<your- affgrp>'


    # 创建 VM 并将其添加到虚拟网络
	New-AzureQuickVM -Windows -ServiceName $vmsvc1 -name $vm1 -ImageName $imgname -AdminUsername $username -Password $password -AffinityGroup $affgrp -SubnetNames $subnetname -VNetName $vnetname


## 将虚拟机提升为域控制器
若要将虚拟机配置为 AD 域控制器，需要登录到 VM 并对其进行配置。

若要登录到 VM，可使用以下命令通过 Powershell 获取 RDP 文件。

    # 获取 RDP 文件
	Get-AzureRemoteDesktopFile -ServiceName $vmsvc1 -Name $vm1 -LocalPath <rdp-file-path>

登录 VM 后，请根据[如何设置客户 AD 域控制器](http://social.technet.microsoft.com/wiki/contents/articles/12370.windows-server-2012-set-up-your-first-domain-controller-step-by-step.aspx)中的分步指导，将虚拟机设置为 AD 域控制器。

## 将云服务添加到虚拟网络

接下来，需要将云服务部署添加到刚刚创建的 VNET。为此，请使用 Visual Studio 或选择的编辑器将相关节添加到 cscfg，以修改云服务 cscfg。

    <ServiceConfiguration serviceName="[hosted-service-name]" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="[os-family]" osVersion="*">
        <Role name="[role-name]">
        <Instances count="[number-of-instances]" />
      </Role>
      <NetworkConfiguration>

        <!--optional-->
        <Dns>
          <DnsServers><DnsServer name="[dns-server-name]" IPAddress="[ip-address]" /></DnsServers>
        </Dns>
        <!--optional-->

    <!--VNet settings
        VNet and subnet must be classic virtual network resources, not Azure Resource Manager resources.-->
    <VirtualNetworkSite name="[virtual-network-name]" />
    <AddressAssignments>
        <InstanceAddress roleName="[role-name]">
        <Subnets>
            <Subnet name="[subnet-name]" />
        </Subnets>
        </InstanceAddress>
    </AddressAssignments>
    <!--VNet settings-->

      </NetworkConfiguration>
    </ServiceConfiguration>

接下来，请生成云服务项目并将其部署到 Azure。有关将云服务包部署到 Azure 的帮助，请参阅[如何创建和部署云服务](/documentation/articles/cloud-services-how-to-create-deploy/#deploy)

## 将 Web/辅助角色连接到域

在 Azure 上部署云服务项目后，请使用 AD 域扩展将角色实例连接到自定义 AD 域。若要将 AD 域扩展添加到现有云服务部署并加入自定义域，请在 Powershell 中执行以下命令：


    # 初始化域变量
	$domain = '<your-domain-name>'
	$dmuser = '$domain<your-username>'
	$dmpswd = '<your-domain-password>'
	$dmspwd = ConvertTo-SecureString $dmpswd -AsPlainText -Force
	$dmcred = New-Object System.Management.Automation.PSCredential ($dmuser, $dmspwd)


    # 将 AD 域扩展添加到云服务角色
	Set-AzureServiceADDomainExtension -Service <your-cloud-service-hosted-service-name> -Role <your-role-name> -Slot <staging-or-production> -DomainName $domain -Credential $dmcred -JoinOption 35

这就是所有的操作。

云服务现在应已加入自定义域控制器。如果想要详细了解可用于配置 AD 域扩展的其他选项，请如下所示使用 PowerShell 帮助。

    help Set-AzureServiceADDomainExtension
    help New-AzureServiceADDomainExtensionConfig


<!---HONumber=Mooncake_Quality_Review_1215_2016-->
<!--Update_Description:update meta properties-->