<properties
   pageTitle="开始使用 SAP 解决方案 | Azure"
   description="了解如何在 Azure 中的虚拟机 (VM) 上运行 SAP 解决方案"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="RicksterCDN"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"
   keywords=""/>  

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="09/23/2016"
   wacn.date="01/05/2017"
   ms.author="rclaus"/>  


# 在 Windows 虚拟机 (VM) 上使用 SAP

[767598]: https://launchpad.support.sap.com/#/notes/767598
[773830]: https://launchpad.support.sap.com/#/notes/773830
[826037]: https://launchpad.support.sap.com/#/notes/826037
[965908]: https://launchpad.support.sap.com/#/notes/965908
[1031096]: https://launchpad.support.sap.com/#/notes/1031096
[1139904]: https://launchpad.support.sap.com/#/notes/1139904
[1173395]: https://launchpad.support.sap.com/#/notes/1173395
[1245200]: https://launchpad.support.sap.com/#/notes/1245200
[1409604]: https://launchpad.support.sap.com/#/notes/1409604
[1558958]: https://launchpad.support.sap.com/#/notes/1558958
[1585981]: https://launchpad.support.sap.com/#/notes/1585981
[1588316]: https://launchpad.support.sap.com/#/notes/1588316
[1590719]: https://launchpad.support.sap.com/#/notes/1590719
[1597355]: https://launchpad.support.sap.com/#/notes/1597355
[1605680]: https://launchpad.support.sap.com/#/notes/1605680
[1619720]: https://launchpad.support.sap.com/#/notes/1619720
[1619726]: https://launchpad.support.sap.com/#/notes/1619726
[1619967]: https://launchpad.support.sap.com/#/notes/1619967
[1750510]: https://launchpad.support.sap.com/#/notes/1750510
[1752266]: https://launchpad.support.sap.com/#/notes/1752266
[1757924]: https://launchpad.support.sap.com/#/notes/1757924
[1757928]: https://launchpad.support.sap.com/#/notes/1757928
[1758182]: https://launchpad.support.sap.com/#/notes/1758182
[1758496]: https://launchpad.support.sap.com/#/notes/1758496
[1772688]: https://launchpad.support.sap.com/#/notes/1772688
[1814258]: https://launchpad.support.sap.com/#/notes/1814258
[1882376]: https://launchpad.support.sap.com/#/notes/1882376
[1909114]: https://launchpad.support.sap.com/#/notes/1909114
[1922555]: https://launchpad.support.sap.com/#/notes/1922555
[1928533]: https://launchpad.support.sap.com/#/notes/1928533
[1941500]: https://launchpad.support.sap.com/#/notes/1941500
[1956005]: https://launchpad.support.sap.com/#/notes/1956005
[1973241]: https://launchpad.support.sap.com/#/notes/1973241
[1984787]: https://launchpad.support.sap.com/#/notes/1984787
[1999351]: https://launchpad.support.sap.com/#/notes/1999351
[2002167]: https://launchpad.support.sap.com/#/notes/2002167
[2015553]: https://launchpad.support.sap.com/#/notes/2015553
[2039619]: https://launchpad.support.sap.com/#/notes/2039619
[2121797]: https://launchpad.support.sap.com/#/notes/2121797
[2134316]: https://launchpad.support.sap.com/#/notes/2134316
[2178632]: https://launchpad.support.sap.com/#/notes/2178632
[2191498]: https://launchpad.support.sap.com/#/notes/2191498
[2233094]: https://launchpad.support.sap.com/#/notes/2233094
[2243692]: https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]: /documentation/articles/xplat-cli-install/
[azure-portal]: https://portal.azure.cn
[azure-ps]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]: https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]: https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits/
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits/#subscription

[dbms-guide]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南"
[dbms-guide-2.1]: virtual-machines-windows-sap-dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f "VM 和 VHD 的缓存"
[dbms-guide-2.2]: virtual-machines-windows-sap-dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 "软件 RAID"
[dbms-guide-2.3]: virtual-machines-windows-sap-dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 "Azure 存储空间"
[dbms-guide-2]: virtual-machines-windows-sap-dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 "RDBMS 部署的结构"
[dbms-guide-3]: virtual-machines-windows-sap-dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 "Azure VM 的高可用性和灾难恢复"
[dbms-guide-5.5.1]: virtual-machines-windows-sap-dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 "SQL Server 2012 SP1 CU4 和更高版本"
[dbms-guide-5.5.2]: virtual-machines-windows-sap-dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b "SQL Server 2012 SP1 CU3 和更低版本"
[dbms-guide-5.6]: virtual-machines-windows-sap-dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 "使用来自 Azure 应用商店的 SQL Server 映像"
[dbms-guide-5.8]: virtual-machines-windows-sap-dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 "适用于 Azure 上的 SAP 的 SQL Server 总体摘要"
[dbms-guide-5]: virtual-machines-windows-sap-dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 "有关 SQL Server RDBMS 的具体信息"
[dbms-guide-8.4.1]: virtual-machines-windows-sap-dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 "存储配置"
[dbms-guide-8.4.2]: virtual-machines-windows-sap-dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d "备份和还原"
[dbms-guide-8.4.3]: virtual-machines-windows-sap-dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c "备份和还原的性能注意事项"
[dbms-guide-8.4.4]: virtual-machines-windows-sap-dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 "其他"
[dbms-guide-900-sap-cache-server-on-premises]: virtual-machines-windows-sap-dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]: ./media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]: ./media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]: ./media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]: ./media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]: ./media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]: ./media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]: ./media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]: ./media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]: ./media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南"
[deployment-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 "SAP 资源"
[deployment-guide-3.1.2]: virtual-machines-windows-sap-deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab "使用自定义映像部署 VM"
[deployment-guide-3.2]: virtual-machines-windows-sap-deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 "方案 1：为 SAP 部署来自 Azure 应用商店的 VM"
[deployment-guide-3.3]: virtual-machines-windows-sap-deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 "方案 2：使用自定义映像为 SAP 部署 VM"
[deployment-guide-3.4]: virtual-machines-windows-sap-deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 "方案 3：使用包含 SAP 的非通用化 Azure VHD 从本地移动 VM"
[deployment-guide-3]: virtual-machines-windows-sap-deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e "Azure 上 SAP 的 VM 部署方案"
[deployment-guide-4.1]: virtual-machines-windows-sap-deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 "部署 Azure PowerShell cmdlet"
[deployment-guide-4.2]: virtual-machines-windows-sap-deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e "下载并导入 SAP 相关的 PowerShell cmdlet"
[deployment-guide-4.3]: virtual-machines-windows-sap-deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc "将 VM 加入本地域 — 仅限 Windows"
[deployment-guide-4.4.2]: virtual-machines-windows-sap-deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 "Linux"
[deployment-guide-4.4]: virtual-machines-windows-sap-deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d "下载、安装并启用 Azure VM 代理"
[deployment-guide-4.5.1]: virtual-machines-windows-sap-deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 "Azure PowerShell"
[deployment-guide-4.5.2]: virtual-machines-windows-sap-deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f "Azure CLI"
[deployment-guide-4.5]: virtual-machines-windows-sap-deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca "配置适用于 SAP 的 Azure 增强型监视扩展"
[deployment-guide-5.1]: virtual-machines-windows-sap-deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 "适用于 SAP 的 Azure 增强型监视的就绪状态检查"
[deployment-guide-5.2]: virtual-machines-windows-sap-deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 "Azure 监视基础结构配置的运行状况检查"
[deployment-guide-5.3]: virtual-machines-windows-sap-deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 "对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除"

[deployment-guide-configure-monitoring-scenario-1]: virtual-machines-windows-sap-deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b "配置监视"
[deployment-guide-configure-proxy]: virtual-machines-windows-sap-deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d "配置代理"
[deployment-guide-figure-100]: ./media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]: ./media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#figure-11
[deployment-guide-figure-1100]: ./media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]: ./media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]: ./media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#figure-14
[deployment-guide-figure-1400]: ./media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]: ./media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]: ./media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#figure-5
[deployment-guide-figure-50]: ./media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]: ./media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#figure-6
[deployment-guide-figure-600]: ./media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#figure-7
[deployment-guide-figure-700]: ./media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]: ./media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]: ./media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]: virtual-machines-windows-sap-deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]: virtual-machines-windows-sap-deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]: virtual-machines-windows-sap-deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b "Azure 上 SAP 的端到端监视设置的检查和故障排除"

[deploy-template-cli]: /documentation/articles/resource-group-template-deploy/#deploy-with-azure-cli-for-mac-linux-and-windows
[deploy-template-portal]: /documentation/articles/resource-group-template-deploy/#deploy-with-the-preview-portal
[deploy-template-powershell]: /documentation/articles/resource-group-template-deploy/#deploy-with-powershell

[dr-guide-classic]: http://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]: /documentation/articles/virtual-machines-windows-sap-get-started/
[getting-started-dbms]: /documentation/articles/virtual-machines-windows-sap-get-started/#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]: virtual-machines-windows-sap-get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]: virtual-machines-windows-sap-get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/
[getting-started-windows-classic-dbms]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]: virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]: virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]: virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]: virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]: http://go.microsoft.com/fwlink/?LinkId=613056

[ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/

[install-extension-cli]: /documentation/articles/virtual-machines-linux-enable-aem/

[Logo_Linux]: ./media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]: ./media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]: https://msdn.microsoft.com/zh-cn/library/azure/mt670598.aspx

[planning-guide]: /documentation/articles/virtual-machines-windows-sap-planning-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南"
[planning-guide-1.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#e55d1e22-c2c8-460b-9897-64622a34fdff "资源"
[planning-guide-11.4.1]: virtual-machines-windows-sap-planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 "SAP 应用程序服务器的高可用性"
[planning-guide-11.5]: virtual-machines-windows-sap-planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f "对 SAP 实例使用 Autostart"
[planning-guide-2.1]: virtual-machines-windows-sap-planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 "仅限云 — 在不将依赖项部署到本地客户网络的情况下将虚拟机部署到 Azure 中"
[planning-guide-2.2]: virtual-machines-windows-sap-planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 "跨界 — 将单个或多个 SAP VM 部署到 Azure 中并要求完全集成到本地网络"
[planning-guide-3.1]: virtual-machines-windows-sap-planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a "Azure 区域"
[planning-guide-3.2.1]: virtual-machines-windows-sap-planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 "容错域"
[planning-guide-3.2.2]: virtual-machines-windows-sap-planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 "升级域"
[planning-guide-3.2.3]: virtual-machines-windows-sap-planning-guide.md#18810088-f9be-4c97-958a-27996255c665 "Azure 可用性集"
[planning-guide-3.2]: virtual-machines-windows-sap-planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 "Azure 虚拟机的概念"
[planning-guide-3.3.2]: virtual-machines-windows-sap-planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"
[planning-guide-5.1.1]: virtual-machines-windows-sap-planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 "使用非通用化磁盘将 VM 从本地移至 Azure"
[planning-guide-5.1.2]: virtual-machines-windows-sap-planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c "使用特定于客户的映像部署 VM"
[planning-guide-5.2.1]: virtual-machines-windows-sap-planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef "准备使用非通用化磁盘将 VM 从本地移到 Azure"
[planning-guide-5.2.2]: virtual-machines-windows-sap-planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 "准备使用特定于客户的映像为 SAP 部署 VM"
[planning-guide-5.2]: virtual-machines-windows-sap-planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 "为 Azure 准备包含 SAP 的 VM"
[planning-guide-5.3.1]: virtual-machines-windows-sap-planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 "Azure 磁盘与 Azure 映像之间的差异"
[planning-guide-5.3.2]: virtual-machines-windows-sap-planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a "将 VHD 从本地上载到 Azure"
[planning-guide-5.4.2]: virtual-machines-windows-sap-planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 "在 Azure 存储帐户之间复制磁盘"
[planning-guide-5.5.1]: virtual-machines-windows-sap-planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 "SAP 部署的 VM/VHD 结构"
[planning-guide-5.5.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#17e0d543-7e8c-4160-a7da-dd7117a1ad9d "为附加的磁盘设置自动装载"
[planning-guide-7.1]: virtual-machines-windows-sap-planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30 "用于 SAP NetWeaver 演示/培训的单一 VM 方案"
[planning-guide-7]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#96a77628-a05e-475d-9df3-fb82217e8f14 "SAP 实例的仅限云部署的概念"
[planning-guide-9.1]: virtual-machines-windows-sap-planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3 "适用于 SAP 的 Azure 监视解决方案"
[planning-guide-azure-premium-storage]: virtual-machines-windows-sap-planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"

[planning-guide-figure-100]: ./media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]: ./media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]: ./media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]: ./media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]: ./media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]: ./media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]: ./media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]: ./media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]: ./media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]: ./media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]: ./media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]: ./media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]: ./media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]: ./media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]: ./media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]: ./media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]: ./media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]: ./media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]: ./media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]: ./media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]: ./media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]: ./media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]: ./media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#61678387-8868-435d-9f8c-450b2424f5bd "Azure 网络"
[planning-guide-storage-microsoft-azure-storage-and-data-disks]: virtual-machines-windows-sap-planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f "存储：Azure 存储空间和数据磁盘"

[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[resource-group-authoring-templates]: /documentation/articles/resource-group-authoring-templates/
[resource-group-overview]: /documentation/articles/resource-group-overview/
[resource-groups-networking]: /documentation/articles/resource-groups-networking/
[sap-pam]: https://support.sap.com/pam "SAP 产品可用性对照表"
[sap-templates-2-tier-marketplace-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]: /documentation/articles/storage-azure-cli/
[storage-azure-cli-copy-blobs]: /documentation/articles/storage-azure-cli/#copy-blobs
[storage-introduction]: /documentation/articles/storage-introduction/
[storage-powershell-guide-full-copy-vhd]: /documentation/articles/storage-powershell-guide-full/#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]: /documentation/articles/storage-premium-storage/
[storage-redundancy]: /documentation/articles/storage-redundancy/
[storage-scalability-targets]: /documentation/articles/storage-scalability-targets/
[storage-use-azcopy]: /documentation/articles/storage-use-azcopy/
[template-201-vm-from-specialized-vhd]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]: /documentation/articles/virtual-machines-linux-attach-disk-portal/
[virtual-machines-windows-attach-disk-portal]: /documentation/articles/virtual-machines-windows-attach-disk-portal/
[virtual-machines-azure-resource-manager-architecture]: /documentation/articles/resource-manager-deployment-model/
[virtual-machines-windows-classic-configure-oracle-data-guard]: /documentation/articles/virtual-machines-windows-classic-configure-oracle-data-guard/
[virtual-machines-linux-cli-deploy-templates]: virtual-machines-linux-cli-deploy-templates.md "使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机"
[virtual-machines-deploy-rmtemplates-powershell]: virtual-machines-windows-ps-manage.md "使用 Azure 资源管理器与 PowerShell 来管理虚拟机"
[virtual-machines-linux-agent-user-guide]: /documentation/articles/virtual-machines-linux-agent-user-guide/
[virtual-machines-linux-agent-user-guide-command-line-options]: /documentation/articles/virtual-machines-linux-agent-user-guide/#command-line-options
[virtual-machines-linux-capture-image]: /documentation/articles/virtual-machines-linux-capture-image/
[virtual-machines-linux-capture-image-capture]: /documentation/articles/virtual-machines-linux-capture-image/#capture-the-vm
[virtual-machines-windows-capture-image]: /documentation/articles/virtual-machines-windows-capture-image/
[virtual-machines-windows-capture-image-capture]: /documentation/articles/virtual-machines-windows-capture-image/#capture-the-vm
[virtual-machines-linux-configure-lvm]: /documentation/articles/virtual-machines-linux-configure-lvm/
[virtual-machines-linux-configure-raid]: /documentation/articles/virtual-machines-linux-configure-raid/
[virtual-machines-linux-classic-create-upload-vhd-step-1]: /documentation/articles/virtual-machines-linux-classic-create-upload-vhd/#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]: /documentation/articles/virtual-machines-linux-suse-create-upload-vhd/
[virtual-machines-linux-redhat-create-upload-vhd]: /documentation/articles/virtual-machines-linux-redhat-create-upload-vhd/
[virtual-machines-linux-how-to-attach-disk]: /documentation/articles/virtual-machines-linux-add-disk/
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]: /documentation/articles/virtual-machines-linux-add-disk/#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]: /documentation/articles/virtual-machines-linux-quick-create-cli/
[virtual-machines-linux-update-agent]: /documentation/articles/virtual-machines-linux-update-agent/
[virtual-machines-manage-availability]: /documentation/articles/virtual-machines-windows-manage-availability/
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]: /documentation/articles/virtual-machines-windows-ps-create/
[virtual-machines-sizes]: /documentation/articles/virtual-machines-windows-sizes/
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]: /documentation/articles/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups/
[virtual-machines-windows-classic-ps-sql-int-listener]: /documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]: /documentation/articles/virtual-machines-windows-sql-high-availability-dr/
[virtual-machines-sql-server-infrastructure-services]: /documentation/articles/virtual-machines-windows-sql-server-iaas-overview/
[virtual-machines-sql-server-performance-best-practices]: /documentation/articles/virtual-machines-windows-sql-performance/
[virtual-machines-upload-image-windows-resource-manager]: /documentation/articles/virtual-machines-windows-upload-image/
[virtual-machines-windows-tutorial]: /documentation/articles/virtual-machines-windows-hero-tutorial/
[virtual-machines-workload-template-sql-alwayson]: https://github.com/Azure/azure-quickstart-templates/tree/master/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]: /documentation/articles/virtual-network-deploy-multinic-arm-cli/
[virtual-network-deploy-multinic-arm-ps]: /documentation/articles/virtual-network-deploy-multinic-arm-ps/
[virtual-network-deploy-multinic-arm-template]: /documentation/articles/virtual-network-deploy-multinic-arm-template/
[virtual-networks-configure-vnet-to-vnet-connection]: /documentation/articles/vpn-gateway-vnet-vnet-rm-ps/
[virtual-networks-create-vnet-arm-pportal]: /documentation/articles/virtual-networks-create-vnet-arm-pportal/
[virtual-networks-manage-dns-in-vnet]: /documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/
[virtual-networks-multiple-nics]: /documentation/articles/virtual-networks-multiple-nics/
[virtual-networks-nsg]: /documentation/articles/virtual-networks-nsg/
[virtual-networks-reserved-private-ip]: /documentation/articles/virtual-networks-static-private-ip-arm-ps/
[virtual-networks-static-private-ip-arm-pportal]: /documentation/articles/virtual-networks-static-private-ip-arm-pportal/
[virtual-networks-udr-overview]: /documentation/articles/virtual-networks-udr-overview/
[vpn-gateway-about-vpn-devices]: /documentation/articles/vpn-gateway-about-vpn-devices/
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/
[vpn-gateway-cross-premises-options]: /documentation/articles/vpn-gateway-plan-design/
[vpn-gateway-site-to-site-create]: /documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/
[vpn-gateway-vpn-faq]: /documentation/articles/vpn-gateway-vpn-faq/
[xplat-cli]: /documentation/articles/xplat-cli-install/
[xplat-cli-azure-resource-manager]: /documentation/articles/xplat-cli-azure-resource-manager/

通过选择 Azure 作为符合 SAP 需求的云合作伙伴，可以在可缩放、兼容且经企业证明的平台上可靠地运行任务关键 SAP 工作负荷。可利用 Azure 的可伸缩性、灵活性和低成本特性。随着 Microsoft 和 SAP 扩大合作伙伴关系，可以在 Azure 的各个开发/测试和生产方案中运行 SAP 应用程序，并能获得完全支持。从 SAP NetWeaver 到 SAP S4/HANA，Linux 到 Windows，SAP HANA 到 SQL，我们都能满足你的需求。

借助 Azure 虚拟机服务和 Azure 上的 SAP HANA 大型实例，Microsoft 提供全面的基础结构即服务 (IaaS) 平台。因为 Azure 支持范围广泛的 SAP 解决方案，此“入门文档”充当当前 SAP 文档集的目录。随着文档库添加更多标题，可在此处看到添加的标题。

## Azure 上的 SAP HANA 认证


SAP 产品 | 支持的操作系统 | Azure 产品/服务 
---------- | ------------ | ------------- 
SAP HANA Developer Edition（包括由 SQLODBC、Windows 专用版 ODBO、ODBC、JDBC 驱动程序、HANA Studio 和 HANA 数据库组成的 HANA 客户端软件） | Red Hat Enterprise Linux、SUSE Linux Enterprise | A7、A8
MHANA One | Red Hat Enterprise Linux、SUSE Linux Enterprise | DS14\_v2（即将推出正式版）
SAP S/4HANA | Red Hat Enterprise Linux、SUSE Linux Enterprise | GS5（受控可用性）、Azure 上的 SAP HANA（大型实例）
Suite on HANA (OLTP) | Red Hat Enterprise Linux、SUSE Linux Enterprise | Azure 上的 SAP HANA（大型实例）
HANA Enterprise for BW (OLAP) | Red Hat Enterprise Linux、SUSE Linux Enterprise | 适用于 Azure 上 SAP HANA 单节点部署的 GS5（大型实例）
SAP BW/4HANA | Red Hat Enterprise Linux、SUSE Linux Enterprise | 适用于 Azure 上 SAP HANA 单节点部署的 GS5（大型实例）


## SAP NetWeaver 认证

在 Microsoft 和 SAP 的全面支持下，Azure 已针对以下 SAP 产品进行认证。

SAP 产品 | 来宾 OS | RDBMS | 虚拟机类型 
---------- | ------------ | ------------- | ------------- 
SAP 业务套件软件 | Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux | SQL Server、Oracle、DB2、SAP ASE | A5 至 A11、D11 至 D14、DS11 至 DS14、GS1 至 GS5 
多合一 SAP 业务软件 | Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux | SQL Server、Oracle、DB2、SAP ASE | A5 至 A11、D11 至 D14、DS11 至 DS14、GS1 至 GS5 
SAP BusinessObjects BI | Windows | 不适用 | A5 至 A11、D11 至 D14、DS11 至 DS14、GS1 至 GS5 
SAP NetWeaver | Windows、SUSE Linux Enterprise、Red Hat Enterprise Linux | SQL Server、Oracle、DB2、SAP ASE | A5 至 A11、D11 至 D14、DS11 至 DS14、GS1 至 GS5 



[AZURE.INCLUDE [windows-warning](../../includes/virtual-machines-linux-sap-warning.md)]

## 在 Azure 上开始使用 SAP HANA

标题：在 Azure VM 上手动安装 SAP HANA 的快速入门指南

摘要：本快速入门指南帮助通过手动安装 SAP NetWeaver 7.5 和 SAP HANA SP12，在 Azure VM 上设置单实例 SAP HANA 原型/演示系统。本指南假设读者熟悉 Azure IaaS 的基本概念，例如如何通过 Azure 门户预览或 Powershell/CLI 部署虚拟机或虚拟网络，包括使用 json 模板的选项。此外还预期读者熟悉 SAP HANA、SAP NetWeaver 以及本地安装方式。

更新时间：2016 年 9 月

## Azure 上 SUSE Linux 中 NetWeaver 的快速入门指南

标题：在 Azure SUSE Linux VM 上测试 SAP NetWeaver

摘要：本文介绍在 Azure SUSE Linux 虚拟机 (VM) 上运行 SAP NetWeaver 时应注意的事项。自 2016 年 5 月 19 日起，Azure 上的 SUSE Linux VM 已正式支持 SAP NetWeaver。有关 Linux 版本、SAP 内核版本等等的所有详细信息，请参阅 SAP 说明 1928533“Azure 上的 SAP 应用程序：支持的产品和 Azure VM 类型”。

更新时间：2016 年 9 月

[可在此处找到此指南](/documentation/articles/virtual-machines-linux-sap-on-suse-quickstart/)

## 在 Azure 上为 SAP ERP 6.0 部署 SAP IDE EHP7 SP3

标题：在 Azure VM 上手动安装 SAP HANA 的快速入门指南

摘要：本文介绍如何通过 SAP Cloud Appliance Library 3.0，在 Azure 上部署运行 SQL Server 和 Windows OS 的 SAP IDES。

更新时间：2016 年 9 月

[可在此处找到此指南](/documentation/articles/virtual-machines-windows-sap-cal-ides-erp6-ehp7-sp3-sql/)
##  <a name="3da0389e-708b-4e82-b2a2-e92f132df89c"></a>规划和实现

标题：Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南

摘要：如果你正在考虑在 Azure 虚拟机中运行 SAP NetWeaver，可从本文开始着手。此规划和实现指南可帮助你评估现有或计划的基于 SAP NetWeaver 的系统是否可以部署到 Azure 虚拟机环境。它介绍了多个 SAP NetWeaver 部署方案，并包括特定于 Azure 的 SAP 配置。该文列出并说明了在 SAP/Azure 端运行混合 SAP 布局产品时所需的所有必要配置信息。此外，还介绍了为确保 IaaS 上基于 SAP NetWeaver 的系统实现高可用性可以采取的措施。

更新时间：2016 年 8 月

[可在此处找到此指南][planning-guide]
## <a name="6aadadd2-76b5-46d8-8713-e8d63630e955"></a>部署

标题：Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南

摘要：本文档提供将 SAP NetWeaver 软件部署到 Azure 中虚拟机的过程指导。此文重点介绍三种特定部署方案，主要侧重于启用 SAP 的 Azure 监视扩展，包括针对 SAP 的 Azure 监视扩展的故障排除建议。本文假设你已阅读规划和实施指南。

更新时间：2016 年 8 月

[可在此处找到此指南][deployment-guide]

## <a name="1343ffe1-8021-4ce6-a08d-3a1553a4db82"></a>DBMS 部署指南

标题：Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南

摘要：此文介绍了应与 SAP 一起运行的 DBMS 系统的规划和实现注意事项。在第一部分中，列出并提供了一般注意事项。此文的以下部分与在 Azure 中部署 SAP 所支持的不同 DBMS 相关。提供的不同 DBMS 为 SQL Server、SAP ASE 和 Oracle。在这些特定部分中，讨论了在 Azure 上将 SAP 系统与这些 DBMS 一起运行时必须考虑的注意事项。提供了 Azure 上的不同 DBMS 支持的备份和高可用性方法等主题，以便用于 SAP 应用程序。

更新时间：2016 年 8 月

[可在此处找到此指南][dbms-guide]

## <a name="63dab028-2c4f-4636-8f99-90bbb264eaba"></a>高可用性部署指南

标题：Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性部署指南

摘要：本文档介绍如何在 Azure 中保护 SAP 单点故障组件，如 SAP ASCS/SCS 和 DBMS。SAP ASCS/SCS、DBMS 和应用程序服务器的组合对于 SAP NetWeaver 系统（例如 SAP NetWeaver ABAP、SAP NetWeaver Java、SAP NetWeaver ABAP+Java 系统）的功能至关重要。因此，高可用性功能需要落实到位，以确保这些组件可以承受服务器或 VM 失败，就像祼机和 Hyper-V 环境的 Windows 群集配置所做的那样。

更新时间：2016 年 8 月

[可在此处找到此指南][ha-guide]

<!---HONumber=Mooncake_1121_2016-->