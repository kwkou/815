<properties
   pageTitle="Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南 | Azure"
   description="Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南"
   services="virtual-machines-windows,virtual-network,storage"
   documentationCenter="saponazure"
   authors="goraco"
   manager="juergent"
   editor=""
   tags="azure-resource-manager"
   keywords=""/>  

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="NA"
   ms.topic="campaign-page"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="08/18/2016"
   wacn.date="10/17/2016"
   ms.author="goraco"/>  


# Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南

[767598]: https://service.sap.com/sap/support/notes/767598
[773830]: https://service.sap.com/sap/support/notes/773830
[826037]: https://service.sap.com/sap/support/notes/826037
[965908]: https://service.sap.com/sap/support/notes/965908
[1031096]: https://service.sap.com/sap/support/notes/1031096
[1139904]: https://service.sap.com/sap/support/notes/1139904
[1173395]: https://service.sap.com/sap/support/notes/1173395
[1245200]: https://service.sap.com/sap/support/notes/1245200
[1409604]: https://service.sap.com/sap/support/notes/1409604
[1558958]: https://service.sap.com/sap/support/notes/1558958
[1585981]: https://service.sap.com/sap/support/notes/1585981
[1588316]: https://service.sap.com/sap/support/notes/1588316
[1590719]: https://service.sap.com/sap/support/notes/1590719
[1597355]: https://service.sap.com/sap/support/notes/1597355
[1605680]: https://service.sap.com/sap/support/notes/1605680
[1619720]: https://service.sap.com/sap/support/notes/1619720
[1619726]: https://service.sap.com/sap/support/notes/1619726
[1619967]: https://service.sap.com/sap/support/notes/1619967
[1750510]: https://service.sap.com/sap/support/notes/1750510
[1752266]: https://service.sap.com/sap/support/notes/1752266
[1757924]: https://service.sap.com/sap/support/notes/1757924
[1757928]: https://service.sap.com/sap/support/notes/1757928
[1758182]: https://service.sap.com/sap/support/notes/1758182
[1758496]: https://service.sap.com/sap/support/notes/1758496
[1772688]: https://service.sap.com/sap/support/notes/1772688
[1814258]: https://service.sap.com/sap/support/notes/1814258
[1882376]: https://service.sap.com/sap/support/notes/1882376
[1909114]: https://service.sap.com/sap/support/notes/1909114
[1922555]: https://service.sap.com/sap/support/notes/1922555
[1928533]: https://service.sap.com/sap/support/notes/1928533
[1941500]: https://service.sap.com/sap/support/notes/1941500
[1956005]: https://service.sap.com/sap/support/notes/1956005
[1973241]: https://service.sap.com/sap/support/notes/1973241
[1984787]: https://service.sap.com/sap/support/notes/1984787
[1999351]: https://service.sap.com/sap/support/notes/1999351
[2002167]: https://service.sap.com/sap/support/notes/2002167
[2015553]: https://service.sap.com/sap/support/notes/2015553
[2039619]: https://service.sap.com/sap/support/notes/2039619
[2121797]: https://service.sap.com/sap/support/notes/2121797
[2134316]: https://service.sap.com/sap/support/notes/2134316
[2178632]: https://service.sap.com/sap/support/notes/2178632
[2191498]: https://service.sap.com/sap/support/notes/2191498
[2233094]: https://service.sap.com/sap/support/notes/2233094
[2243692]: https://service.sap.com/sap/support/notes/2243692

[sap-installation-guides]: http://service.sap.com/instguides

[azure-cli]: /documentation/articles/xplat-cli-install/
[azure-portal]: https://portal.azure.cn
[azure-ps]: /documentation/articles/powershell-install-configure/
[azure-quickstart-templates-github]: https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]: https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits/
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits/#subscription

[dbms-guide]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/ "SAP NetWeaver on Windows virtual machines (VMs) - DBMS Deployment Guide（Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南）"
[dbms-guide-2.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f "VM 和 VHD 的缓存"
[dbms-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#c8e566f9-21b7-4457-9f7f-126036971a91 "软件 RAID"
[dbms-guide-2.3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#10b041ef-c177-498a-93ed-44b3441ab152 "Azure 存储空间"
[dbms-guide-2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#65fa79d6-a85f-47ee-890b-22e794f51a64 "RDBMS 部署的结构"
[dbms-guide-3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#871dfc27-e509-4222-9370-ab1de77021c3 "Azure VM 的高可用性和灾难恢复"
[dbms-guide-5.5.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#0fef0e79-d3fe-4ae2-85af-73666a6f7268 "SQL Server 2012 SP1 CU4 和更高版本"
[dbms-guide-5.5.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#f9071eff-9d72-4f47-9da4-1852d782087b "SQL Server 2012 SP1 CU3 和更低版本"
[dbms-guide-5.6]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 "使用来自 Azure 应用商店的 SQL Server 映像"
[dbms-guide-5.8]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#9053f720-6f3b-4483-904d-15dc54141e30 "适用于 Azure 上的 SAP 的 SQL Server 总体摘要"
[dbms-guide-5]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#3264829e-075e-4d25-966e-a49dad878737 "有关 SQL Server RDBMS 的具体信息"
[dbms-guide-8.4.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#b48cfe3b-48e9-4f5b-a783-1d29155bd573 "存储配置"
[dbms-guide-8.4.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#23c78d3b-ca5a-4e72-8a24-645d141a3f5d "备份和还原"
[dbms-guide-8.4.3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#77cd2fbb-307e-4cbf-a65f-745553f72d2c "备份和还原的性能注意事项"
[dbms-guide-8.4.4]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#f77c1436-9ad8-44fb-a331-8671342de818 "其他"
[dbms-guide-900-sap-cache-server-on-premises]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]: ./media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]: ./media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]: ./media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]: ./media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]: ./media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]: ./media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]: ./media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]: ./media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]: ./media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/ "SAP NetWeaver on Windows virtual machines (VMs) - Deployment Guide（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南）"
[deployment-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 "SAP 资源"
[deployment-guide-3.1.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#3688666f-281f-425b-a312-a77e7db2dfab "使用自定义映像部署 VM"
[deployment-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#db477013-9060-4602-9ad4-b0316f8bb281 "方案 1：为 SAP 部署来自 Azure 应用商店的 VM"
[deployment-guide-3.3]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 "方案 2：使用自定义映像为 SAP 部署 VM"
[deployment-guide-3.4]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#a9a60133-a763-4de8-8986-ac0fa33aa8c1 "方案 3：使用包含 SAP 的非通用化 Azure VHD 从本地移动 VM"
[deployment-guide-3]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#b3253ee3-d63b-4d74-a49b-185e76c4088e "Azure 上 SAP 的 VM 部署方案"
[deployment-guide-4.1]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 "部署 Azure PowerShell cmdlet"
[deployment-guide-4.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e "下载并导入 SAP 相关的 PowerShell cmdlet"
[deployment-guide-4.3]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc "将 VM 加入本地域 — 仅限 Windows"
[deployment-guide-4.4.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 "Linux"
[deployment-guide-4.4]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#c7cbb0dc-52a4-49db-8e03-83e7edc2927d "下载、安装并启用 Azure VM 代理"
[deployment-guide-4.5.1]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#987cf279-d713-4b4c-8143-6b11589bb9d4 "Azure PowerShell"
[deployment-guide-4.5.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#408f3779-f422-4413-82f8-c57a23b4fc2f "Azure CLI"
[deployment-guide-4.5]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#d98edcd3-f2a1-49f7-b26a-07448ceb60ca "配置适用于 SAP 的 Azure 增强型监视扩展"
[deployment-guide-5.1]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 "适用于 SAP 的 Azure 增强型监视的就绪状态检查"
[deployment-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 "Azure 监视基础结构配置的运行状况检查"
[deployment-guide-5.3]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 "对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除"

[deployment-guide-configure-monitoring-scenario-1]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#ec323ac3-1de9-4c3a-b770-4ff701def65b "配置监视"
[deployment-guide-configure-proxy]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#baccae00-6f79-4307-ade4-40292ce4e02d "配置代理"
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
[deployment-guide-figure-azure-cli-version]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/#564adb4f-5c95-4041-9616-6635e83a810b "Azure 上 SAP 的端到端监视设置的检查和故障排除"

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

[planning-guide]: /documentation/articles/virtual-machines-windows-sap-planning-guide/ "SAP NetWeaver on Windows virtual machines (VMs) - Planning and Implementation Guide（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南）"
[planning-guide-1.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#e55d1e22-c2c8-460b-9897-64622a34fdff "资源"
[planning-guide-11]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#7cf991a1-badd-40a9-944e-7baae842a058 "Azure 虚拟机上运行的 SAP NetWeaver 的高可用性 (HA) 和灾难恢复 (DR)"
[planning-guide-11.4.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#5d9d36f9-9058-435d-8367-5ad05f00de77 "SAP 应用程序服务器的高可用性"
[planning-guide-11.5]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4e165b58-74ca-474f-a7f4-5e695a93204f "对 SAP 实例使用 Autostart"
[planning-guide-2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#1625df66-4cc6-4d60-9202-de8a0b77f803 "仅限云 — 在不将依赖项部署到本地客户网络的情况下将虚拟机部署到 Azure 中"
[planning-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 "跨界 — 将单个或多个 SAP VM 部署到 Azure 中并要求完全集成到本地网络"
[planning-guide-3.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#be80d1b9-a463-4845-bd35-f4cebdb5424a "Azure 区域"
[planning-guide-3.2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#df49dc09-141b-4f34-a4a2-990913b30358 "容错域"
[planning-guide-3.2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#fc1ac8b2-e54a-487c-8581-d3cc6625e560 "升级域"
[planning-guide-3.2.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#18810088-f9be-4c97-958a-27996255c665 "Azure 可用性集"
[planning-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 "Azure 虚拟机的概念"
[planning-guide-3.3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"
[planning-guide-5.1.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4d175f1b-7353-4137-9d2f-817683c26e53 "使用非通用化磁盘将 VM 从本地移至 Azure"
[planning-guide-5.1.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#e18f7839-c0e2-4385-b1e6-4538453a285c "使用特定于客户的映像部署 VM"
[planning-guide-5.2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#1b287330-944b-495d-9ea7-94b83aff73ef "准备使用非通用化磁盘将 VM 从本地移到 Azure"
[planning-guide-5.2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 "准备使用特定于客户的映像为 SAP 部署 VM"
[planning-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6ffb9f41-a292-40bf-9e70-8204448559e7 "为 Azure 准备包含 SAP 的 VM"
[planning-guide-5.3.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6e835de8-40b1-4b71-9f18-d45b20959b79 "Azure 磁盘与 Azure 映像之间的差异"
[planning-guide-5.3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#a43e40e6-1acc-4633-9816-8f095d5a7b6a "将 VHD 从本地上载到 Azure"
[planning-guide-5.4.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#9789b076-2011-4afa-b2fe-b07a8aba58a1 "在 Azure 存储帐户之间复制磁盘"
[planning-guide-5.5.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4efec401-91e0-40c0-8e64-f2dceadff646 "SAP 部署的 VM/VHD 结构"
[planning-guide-5.5.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#17e0d543-7e8c-4160-a7da-dd7117a1ad9d "为附加的磁盘设置自动装载"
[planning-guide-7.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#3e9c3690-da67-421a-bc3f-12c520d99a30 "用于 SAP NetWeaver 演示/培训的单一 VM 方案"
[planning-guide-7]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#96a77628-a05e-475d-9df3-fb82217e8f14 "SAP 实例的仅限云部署的概念"
[planning-guide-9.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6f0a47f3-a289-4090-a053-2521618a28c3 "适用于 SAP 的 Azure 监视解决方案"
[planning-guide-azure-premium-storage]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"

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
[planning-guide-storage-microsoft-azure-storage-and-data-disks]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#a72afa26-4bf4-4a25-8cf7-855d6032157f "存储：Azure 存储空间和数据磁盘"

[sap-ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南"
[sap-ha-guide-1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#217c5479-5595-4cd8-870d-15ab00d4f84c "先决条件"
[sap-ha-guide-2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42b8f600-7ba3-4606-b8a5-53c4f026da08 "资源"
[sap-ha-guide-3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42156640c6-01cf-45a9-b225-4baa678b24f1 "Azure Resource Manager 与经典部署模型之间的 SAP HA 差异"
[sap-ha-guide-3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f76af273-1993-4d83-b12d-65deeae23686 "资源组"
[sap-ha-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#3e85fbe0-84b1-4892-87af-d9b65ff91860 "Azure Resource Manager 与经典部署模型的群集比较"
[sap-ha-guide-4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8ecf3ba0-67c0-4495-9c14-feec1a2255b7 "Windows Server 故障转移群集 (WSFC)"
[sap-ha-guide-4.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a3c5408-b168-46d6-99f5-4219ad1b1ff2 "仲裁模式"
[sap-ha-guide-5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fdfee875-6e66-483a-a343-14bbaee33275 "本地环境中的 Windows 故障转移群集"
[sap-ha-guide-5.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#be21cf3e-fb01-402b-9955-54fbecf66592 "共享存储"
[sap-ha-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ff7a9a06-2bc5-4b20-860a-46cdb44669cd "网络/名称解析"
[sap-ha-guide-6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2ddba413-a7f5-4e4e-9a51-87908879c10a "在 Azure 上使用 Windows 故障转移群集"
[sap-ha-guide-6.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a464091-922b-48d7-9d08-7cecf757f341 "使用 SIOS DataKeeper 在 Azure 上创建共享磁盘"
[sap-ha-guide-6.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#44641e18-a94e-431f-95ff-303ab65e0bcb "Azure 上的名称解析"
[sap-ha-guide-7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2e3fec50-241e-441b-8708-0b1864f66dfa "Azure IaaS 上的 SAP NetWeaver 高可用性"
[sap-ha-guide-7.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#93faa747-907e-440a-b00a-1ae0a89b1c0e "SAP 应用程序服务器的高可用性"
[sap-ha-guide-7.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f559c285-ee68-4eec-add1-f60fe7b978db "SAP (A)SCS 实例的高可用性"
[sap-ha-guide-7.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b5b1fd0b-1db4-4d49-9162-de07a0132a51 "在 Azuer 中使用 Windows 故障转移群集实现 SAP (A)SCS 实例的高可用性"
[sap-ha-guide-7.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ddd878a0-9c2f-4b8e-8968-26ce60be1027 "DBMS 实例的高可用性"
[sap-ha-guide-7.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#045252ed-0277-4fc8-8f46-c5a29694a816 "可能的端到端 HA 部署方案"
[sap-ha-guide-8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#78092dbe-165b-454c-92f5-4972bdbef9bf "基础结构准备"
[sap-ha-guide-8.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#c87a8d3f-b1dc-4d2f-b23c-da4b72977489 "部署具有企业网络连接（跨界）的 VM 供生产使用"
[sap-ha-guide-8.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310 "SAP 实例的仅限云部署（用于测试/演示）"
[sap-ha-guide-8.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#47d5300a-a830-41d4-83dd-1a0d1ffdbe6a "Azure 虚拟网络"
[sap-ha-guide-8.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b22d7b3b-4343-40ff-a319-097e13f62f9e "DNS IP 地址"
[sap-ha-guide-8.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#9fbd43c0-5850-4965-9726-2a921d85d73f "SAP ASCS/SCS 群集实例和 DBMS 群集实例的主机名与静态 IP 地址"
[sap-ha-guide-8.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#84c019fe-8c58-4dac-9e54-173efd4b2c30 "为 SAP VM 设置静态 IP 地址"
[sap-ha-guide-8.7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7a8f3e9b-0624-4051-9e41-b73fff816a9e "为内部负载均衡器 (ILB) 设置静态 IP 地址"
[sap-ha-guide-8.8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f19bd997-154d-4583-a46e-7f5a69d0153c "Azure 内部负载均衡器 (ILB) 的默认 ASCS/SCS 负载均衡规则"
[sap-ha-guide-8.9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fe0bd8b5-2b43-45e3-8295-80bee5415716 "更改 Azure 内部负载均衡器 (ILB) 的 ASCS/SCS 默认负载均衡规则"
[sap-ha-guide-8.10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e69e9a34-4601-47a3-a41c-d2e11c626c0c "将 Windows 计算机添加到域"
[sap-ha-guide-8.11]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#661035b2-4d0f-4d31-86f8-dc0a50d78158 "在用于 SAP ASCS/SCS 实例的两个群集节点上添加注册表项"
[sap-ha-guide-8.12]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0d67f090-7928-43e0-8772-5ccbf8f59aab "SAP ASCS/SCS 实例的 Windows Server 故障转移群集设置"
[sap-ha-guide-8.12.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79 "收集群集配置中的群集节点"
[sap-ha-guide-8.12.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e49a4529-50c9-4dcf-bde7-15a0c21d21ca "配置群集文件共享见证"
[sap-ha-guide-8.12.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#06260b30-d697-4c4d-b1c9-d22c0bd64855 "创建文件共享"
[sap-ha-guide-8.12.2.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#4c08c387-78a0-46b1-9d27-b497b08cac3d "在故障转移群集管理器中配置文件共享见证"
[sap-ha-guide-8.12.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5c8e5482-841e-45e1-a89d-a05c0907c868 "为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition"
[sap-ha-guide-8.12.3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1c2788c3-3648-4e82-9e0d-e058e475e2a3 "添加 Microsoft .NET Framework 3.5 功能"
[sap-ha-guide-8.12.3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#dd41d5a2-8083-415b-9878-839652812102 "安装 SIOS DataKeeper"
[sap-ha-guide-8.12.3.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#d9c1fc8e-8710-4dff-bec2-1f535db7b006 "设置 SIOS DataKeeper"
[sap-ha-guide-9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a06f0b49-8a7a-42bf-8b0d-c12026c5746b "SAP NetWeaver 系统安装"
[sap-ha-guide-9.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#31c6bd4f-51df-4057-9fdf-3fcbc619c170 "具有高可用性 ASCS/SCS 实例的 SAP 安装"
[sap-ha-guide-9.1.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a97ad604-9094-44fe-a364-f89cb39bf097 "创建群集 SAP ASCS/SCS 的虚拟主机名"
[sap-ha-guide-9.1.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#eb5af918-b42f-4803-bb50-eff41f84b0b0 "安装 SAP 的第一个群集节点"
[sap-ha-guide-9.1.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 "修改 ASCS/SCS 实例的 SAP 配置文件"
[sap-ha-guide-9.1.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#10822f4f-32e7-4871-b63a-9b86c76ce761 "添加探测端口"
[sap-ha-guide-9.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#85d78414-b21d-4097-92b6-34d8bcb724b7 "安装数据库实例"
[sap-ha-guide-9.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8a276e16-f507-4071-b829-cdc0a4d36748 "安装第二个群集节点"
[sap-ha-guide-9.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#094bc895-31d4-4471-91cc-1513b64e406a "更改 SAP ERS 实例的 Windows 服务启动类型"
[sap-ha-guide-9.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 "安装 SAP 主应用程序服务器 (PAS)"
[sap-ha-guide-9.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 "安装 SAP 附加应用程序服务器 (AAS)"
[sap-ha-guide-10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9 "测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制"
[sap-ha-guide-10.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#65fdef0f-9f94-41f9-b314-ea45bbfea445 "起点 – SAP ASCS/SCS 实例在群集节点 A 上运行"
[sap-ha-guide-10.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5e959fa9-8fcd-49e5-a12c-37f6ba07b916 "从节点 A 到节点 B 的故障转移过程"
[sap-ha-guide-10.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#755a6b93-0099-4533-9f6d-5c9a613878b5 "最终结果 – SAP ASCS/SCS 实例在群集节点 B 上运行"


[sap-ha-guide-figure-1000]: ./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]: ./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]: ./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]: ./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]: ./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]: ./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]: ./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-3000]: ./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]: ./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]: ./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]: ./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]: ./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]: ./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]: ./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]: ./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]: ./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]: ./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]: ./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]: ./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]: ./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]: ./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]: ./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]: ./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]: ./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]: ./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]: ./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]: ./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]: ./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]: ./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]: ./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]: ./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]: ./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]: ./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]: ./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]: ./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]: ./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]: ./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]: ./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]: ./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]: ./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]: ./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]: ./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]: ./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]: ./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]: ./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]: ./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]: ./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]: ./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]: ./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]: ./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]: ./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]: ./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]: ./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]: ./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]: ./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]: ./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]: ./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]: ./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]: ./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]: ./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]: ./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]: ./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png


[powershell-install-configure]: /documentation/articles/powershell-install-configure/
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
[virtual-machines-azure-resource-manager-architecture-benefits-arm]: /documentation/articles/resource-manager-deployment-model/
[virtual-machines-azurerm-versus-azuresm]: /documentation/articles/virtual-machines-windows-compare-deployment-models/
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
[virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]: /documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/
[virtual-machines-windows-portal-sql-alwayson-int-listener]: /documentation/articles/virtual-machines-windows-portal-sql-alwayson-int-listener/
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
 

公司可以使用 Azure 在最短的时间内获取计算、存储和网络资源，而无需经历冗长的采购周期。Azure 虚拟机可让公司将典型的应用程序（例如，基于 SAP NetWeaver 的应用程序，包括 ABAP、Java 和 ABAP+Java 堆栈）部署到 Azure 中，并提高其可靠性和可用性，且不需要在本地提供其他资源。Azure 虚拟机还支持跨场地连接，使公司能够主动地将 Azure 虚拟机集成到其本地域、私有云和 SAP 系统布局中。


本文档详述使用新方法配合新的 Azure Resource Manager 部署模型，在 Azure 中部署高可用性 SAP 系统所要执行的全部步骤。本指南逐步讲解以下主要步骤：


- 查找在稍后标题为[资源][sap-ha-guide-2]的部分中所列的相应 SAP 安装指南和说明。  
  此文是对 SAP 安装文档和 SAP 说明的补充，这些文档代表在给定平台上安装和部署 SAP 软件的主要资源。

- 了解当前 Azure 经典部署模型与新 Azure Resource Manager 部署模型之间的差异。

- 了解 Windows Server 故障转移群集 (WSFC) 仲裁模式，以便选择适用于 Azure 部署的模型

- 了解 Azure 中的 Windows Server 故障转移群集 (WSFC) 共享存储

- 了解在 Azure 中如何保护 SAP 单点故障组件（例如 SAP ASCS/SCS 和 DBMS）和冗余组件（例如 SAP 应用程序服务器）

- 通过循序渐进的方法，了解如何使用 Microsoft Azure 作为平台和新的 Azure Resource Manager，在 Windows Server 故障转移群集 (WSFC) 中安装和配置高可用性 (HA) SAP 系统。

- 需要针对 Azure 中的 WSFC 执行的、但不需要在本地执行的其他步骤


为了简化部署和配置，我们将使用新的 SAP 3 层 HA Azure Resource Manager 模板，此模板以自动化方式部署高可用性 SAP 系统所需的整个基础结构，支持所需的 SAP 系统 SAPS 大小。

[AZURE.INCLUDE [windows-warning](../../includes/virtual-machines-linux-sap-warning.md)]

##  <a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a>先决条件

在开始之前，请确保满足以下章节中所述的先决条件，并已检查“资源”一章所列的所有资源。

将 Azure Resource Manager 模板用于 3 层 SAP NetWeaver：   
[https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image/)

下面提供了 SAP Azure Resource Manager 模板的概述：   
[https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/16/azure-quickstart-templates-for-sap/](https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/16/azure-quickstart-templates-for-sap/)


##  <a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a>资源

我们针对有关 Azure 上的 SAP 部署的主题提供了以下附加指南：

- [SAP NetWeaver on Windows virtual machines (VMs) - Planning and Implementation Guide][planning-guide]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南）
- [SAP NetWeaver on Windows virtual machines (VMs) - Deployment Guide][deployment-guide]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南）
- [SAP NetWeaver on Windows virtual machines (VMs) - DBMS Deployment Guide][dbms-guide]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南）
- [Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南][sap-ha-guide]（本指南）


> [AZURE.NOTE] 在可能的情况下，请访问所述《SAP 安装指南》的链接（请参阅 [SAP Installation Guides][sap-installation-guides]（SAP 安装指南））。在满足先决条件和安装过程中，始终应该仔细阅读《SAP NetWeaver Installation Guides》（SAP NetWeaver 安装指南），因为本文档只包括了有关 Azure 虚拟机中安装的基于 SAP NetWeaver 的系统的具体任务。

以下 SAP 说明与 Azure 上的 SAP 主题相关：


| 说明文档编号 | 标题                                                    
| ------------- |----------------------------------------------------------
| [1928533] | Azure 上的 SAP 应用程序：支持的产品和规模 
| [2015553] | Azure 上的 SAP：支持先决条件         
| [1999351] | 适用于 SAP 的增强型 Azure 监视                        
| [2178632] | Azure 上的 SAP 关键监视度量值        
| [1999351] | Windows 上的虚拟化：增强型监视           


可以在[此文][azure-subscription-service-limits-subscription]中找到有关 Azure 订阅的常规默认限制和最大限制。

##  <a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>Azure Resource Manager 与经典部署模型之间的 SAP HA 差异 

> [AZURE.NOTE] 经典部署模型也称为 Azure 服务管理 (ASM) 模型。

### <a name="f76af273-1993-4d83-b12d-65deeae23686"></a>资源组
资源组是一个新概念，包含具有相同生命周期的所有资源（例如，它们是同时创建和删除的）。有关资源组的详细信息，请阅读此文。

### <a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a>Azure Resource Manager 与经典部署模型的群集比较 

与经典部署模型相比，新 Azure Resource Manager 模型针对 HA 提供以下更改：

- 无需云服务，即可使用 Azure 内部负载均衡器 (ILB)

如果仍要使用旧的 Azure 经典模型，需要遵循 [SAP NetWeaver on Azure - Clustering SAP ASCS/SCS Instances using Windows Server Failover Cluster on Azure with SIOS DataKeeper](http://go.microsoft.com/fwlink/?LinkId=613056)（Azure 上的 SAP NetWeaver - 配合 SIOS DataKeeper 使用 Azure 上的 Windows Server 故障转移群集来组建 SAP ASCS/SCS 实例的群集）一文中所述的过程。

> [AZURE.NOTE] 强烈建议使用新的 Azure Resource Manager 部署模型来进行 SAP 安装，因为与经典部署模型相比，它提供许多优势。
有关详细信息，请参阅[此文][virtual-machines-azure-resource-manager-architecture-benefits-arm]。   


## <a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a>Windows Server 故障转移群集 (WSFC) 

Microsoft WSFC 是 Windows 上高可用性 SAP ASCS/SCS 安装和 DBMS 的技术基础。

故障转移群集是由 1+n 个独立服务器（节点）构成的组，这些服务器配合工作以提高应用程序和服务的可用性。当发生节点故障时，WSFC 必须确定可发生多少错误而仍然能让群集保持正常状态，以便提供定义的应用程序和/或服务。可使用不同的仲裁模式来实现此目的。
 

### <a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a>仲裁模式

在 WSFC 中，可以使用四种不同的仲裁模式：

- **节点多数：**每个节点都可投票。只有获取多数票（也就是过半数）时，群集才能正常运行。当节点数目为奇数时，建议使用此选项。例如：在包含 7 个节点的群集中，可以有 3 个节点故障，在此情况下，群集仍可达到占多数的优势而继续运行。

- **节点和磁盘多数：**每个节点再加上一个群集存储中的指定磁盘（“磁盘见证”）只要是处于可用且通信中状态，都可投票。只有获取多数票（也就是过半数）时，群集才能正常运行。此模式适用于节点数目为偶数的群集环境。只要半数的节点再加上该磁盘在线，群集就保持正常运行状态。

- **节点和文件共享多数：**每个节点再加上一个由管理员所创建的指定文件共享（文件共享见证）只要是处于可用且通信中状态，都可投票。只有获取多数票（也就是过半数）时，群集才能正常运行。此模式适用于节点数目为偶数的群集环境，并且类似于“节点和磁盘多数”模式，只是它使用的是见证文件共享，而不是见证磁盘。它很容易实现，但如果文件共享本身不具有高可用性，则它可能变成单点故障。

- **非大多数：仅磁盘：**只要有一个节点可用且正与群集存储中的特定磁盘进行通信，群集就拥有仲裁。只有也在与该磁盘进行通信的节点能够加ty 群集。不建议使用此模式。 

## <a name="fdfee875-6e66-483a-a343-14bbaee33275"></a>本地环境中的 Windows 故障转移群集

在本示例中，有一个由两个节点组成的群集。如果节点之间的网络连接失败，但两个节点都保持在已启动并正常运行的状态，就有必要澄清哪个节点应该继续提供群集的应用程序和服务。仲裁磁盘或文件共享可用于此用途。有权访问仲裁磁盘或文件共享的节点是可以确保服务可访问性的节点。

在本示例中，使用的是包含两个节点的群集。出于此原因，我们选择了节点与文件共享仲裁模式。节点与磁盘多数也是有效的选项。在生产环境中，建议改用仲裁磁盘，并使用网络和存储系统技术让它变成具有高可用性。

![图 1：针对 Azure 上的 SAP ASCS/SCS 建议使用的 Windows Server 故障转移群集配置][sap-ha-guide-figure-1000]  


_**图 1：**针对 Azure 上的 SAP ASCS/SCS 建议使用的 Windows Server 故障转移群集配置_

### <a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a>共享存储

上图显示了包含两个节点的共享存储群集。在本地环境中的共享存储群集中，有一个该群集内所有节点都看到的共享存储。锁定机制可防止数据损坏。此外，所有节点也可以检测到另一个节点是否发生故障。如果一个节点发生故障，剩余的节点将获取存储资源的所有权并确保服务的可用性。

> [AZURE.NOTE] 就某些 DBMS（例如 SQL Server）而言，实现高可用性不一定需要用到共享磁盘。SQL Server AlwaysOn 将 DBMS 数据和日志从一个群集节点的本地磁盘复制到另一个群集节点的本地磁盘。因此，Windows 群集配置并不需要用到共享磁盘。

### <a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a>网络/名称解析

通过 DNS 服务器提供的虚拟 IP 地址和虚拟主机名即可连接到群集。本地环境中的节点与 DNS 服务器可以处理多个 IP 地址。

在典型设置中，使用两个或更多个网络连接：

- 与存储建立的专用连接；
- 用于检测信号的群集内部网络连接；
- 供客户端用来连接到群集的公共网络。



## <a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a>在 Azure 上使用 Windows 故障转移群集

相比于裸机或私有云部署，Azure 虚拟机要求执行额外的步骤来配置 WSFC。为了构建共享群集磁盘，SAP ASCS/SCS 实例必须要有多个 IP 地址和虚拟主机名。

下面介绍在 Azure 上构建 SAP HA 中心服务群集时所需的其他概念和步骤。这些步骤说明如何设置第三方工具 SIOS DataKeeper，以及配置 Azure 内部负载均衡器。这些工具可让我们使用文件共享见证在 Azure 中创建 Windows 故障转移群集。


![图 2：Azure 中没有共享磁盘的 Windows Server 故障转移群集配置架构][sap-ha-guide-figure-1001]  


_**图 2：**Azure 中没有共享磁盘的 Windows Server 故障转移群集配置架构_


### <a name="1a464091-922b-48d7-9d08-7cecf757f341"></a>使用 SIOS DataKeeper 在 Azure 上创建共享磁盘

自 2016 年 9 月起，Azure 不再提供共享存储来创建共享存储群集。但是，高可用性 SAP ASCS/SCS 实例需要群集共享存储。

第三方软件 SIOS DataKeeper Cluster Edition 可让用户创建镜像存储，模拟群集共享存储。SIOS 解决方案提供实时同步的数据复制。为群集创建共享磁盘资源的方法为：

- 将额外的 Azure VHD 连接到 Windows 群集配置中的每个 VM。
- 在两个 VM 节点上运行 SIOS DataKeeper Cluster Edition
- 配置 SIOS DataKeeper Cluster Edition，使得源 VM 中已附加到其他 VHD 的卷内容能够镜像到目标 VM 中已附加到其他 VHD 的卷。SIOS DataKeeper 将抽象化源和目标本地卷，并以单个共享磁盘形式呈现给 Windows 故障转移群集。

有关 SIOS DataKeeper 产品的详细信息，请参阅以下信息源：[http://us.sios.com/products/datakeeper-cluster/](http://us.sios.com/products/datakeeper-cluster/)

 ![图 2：Azure 中使用 SIOS DataKeeper 的 Windows Server 故障转移群集配置架构][sap-ha-guide-figure-1002]  


_**图 3：**Azure 中使用 SIOS DataKeeper 的 Windows Server 故障转移群集配置架构_

> [AZURE.NOTE] 就某些 DBMS（例如 SQL Server）而言，实现高可用性不一定需要用到共享磁盘。SQL Server AlwaysOn 将 DBMS 数据和日志从一个群集节点的本地磁盘复制到另一个群集节点的本地磁盘。因此，Windows 群集配置并不需要用到共享磁盘。

### <a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a>Azure 上的名称解析

Azure 云平台不允许配置虚拟 IP 地址，例如浮动 IP。出于此原因，需要一个替代解决方案来设置虚拟 IP，以便连接到云中的群集资源。
Azure 提供内部负载均衡器 (ILB)。有了 ILB，便可通过群集虚拟 IP 地址连接到群集
。因此，需要将 ILB 部署在包含群集节点的资源组中。然后，需要使用 ILB 的探测端口来配置所有必要的端口转发规则。
客户端可以通过虚拟主机名进行连接。DNS 服务器解析群集 IP 地址，ILB 处理向活动群集节点的转发。

## <a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a>Azure IaaS 上的 SAP NetWeaver 高可用性

如 HA 章节 [SAP NetWeaver on Windows virtual machines (VMs) - Planning and Implementation Guide][planning-guide]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南）中所述，若要实现 SAP 应用程序的高可用性（例如 SAP 软件组件的 HA），需要保护以下组件：

- SAP 应用程序服务器
- SAP ASCS/SCS 实例
- DBMS 服务器

### <a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a>SAP 应用程序服务器的高可用性

对于 SAP 应用程序服务器/对话实例，通常不需要特定的高可用性解决方案。可以在不同的 Azure 虚拟机上配置多个对话实例，通过冗余来实现高可用性。应该至少有两个 SAP 应用程序实例安装在两个 Azure VM 中。

![图 4：SAP 应用程序服务器 HA][sap-ha-guide-figure-2000]  


_**图 4：**SAP 应用程序服务器 HA_


所有托管 SAP 应用程序服务器的虚拟机都必须放置在同一个 **Azure 可用性集**中。Azure 可用性集将确保：

- 所有 VM 都属于同一个升级域，例如，可避免在计划内维护停机期间更新 VM。
- 所有 VM 都属于同一个容错域，例如，确保 VM 的部署方式避免可能影响所有 VM 可用性的单点故障。

有关详细信息，请参阅 [Manage the availability of virtual machines][virtual-machines-manage-availability]（管理虚拟机的可用性）一文。

由于 Azure 存储帐户可能是潜在的单点故障，因此必须至少要有两个 Azure 存储帐户，并且至少要将两个 VM 分配到其中。理想情况下，每个运行 SAP 对话实例的 VM 都应该部署在不同的存储帐户中。

### <a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a>SAP (A)SCS 实例的高可用性 

![图 5：高可用性 SAP ASCS / SCS 实例 HA][sap-ha-guide-figure-2001]  


_**图 5：**高可用性 SAP ASCS / SCS 实例 HA_


#### <a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a>在 Azuer 中使用 Windows 故障转移群集实现 SAP (A)SCS 实例的高可用性

相比于裸机或私有云部署，Azure 虚拟机要求执行额外的步骤来配置 WSFC。若要构建 Windows 故障转移群集，必须有一个共享磁盘群集、多个 IP 地址和虚拟主机名以及 Azure 内部负载均衡器 (ILB)，组建 SAP ASCS/SCS 实例的群集。

本文稍后将对此提供详细介绍。

![图 6：Azure 中使用 SIOS DataKeeper 的 SAP ASCS/SCS 配置的 Windows Server 故障转移群集架构][sap-ha-guide-figure-1002]  


_**图 6：**Azure 中使用 SIOS DataKeeper 的 SAP ASCS/SCS 配置的 Windows Server 故障转移群集架构_


### <a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a>DBMS 实例的高可用性

DBMS 也是 SAP 系统的 SPOF，需要使用 HA 解决方案来保护它。下面是 Azure 中的 SQL Server AlwaysOn HA 解决方案示例，使用的是 Windows Server 故障转移群集和 Azure 内部负载均衡器。SQL Server AlwaysOn 使用自身的 DBMS 复制功能复制 DBMS 数据和日志文件。因此，不需要群集共享磁盘，简化了整个设置。


![图 7：SAP DBMS 服务器 HA – SQL Server AlwaysOn HA 设置示例][sap-ha-guide-figure-2003]  


_**图 7：**SAP DBMS 服务器 HA – SQL Server AlwaysOn HA 设置示例_


本文档不会介绍如何组建 DBMS 群集。

### <a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a>可能的端到端 HA 部署方案

下面是 Azure 中的完整 SAP NetWeaver HA 架构示例，其中将一个专用群集用于 SAP ASCS/SCS 实例，将另一个群集用于 DBMS。

![图 8：SAP HA 体系结构模板 1 – 包含 ASCS/SCS 的专用群集和 DBMS 实例的专用群集][sap-ha-guide-figure-2004]  


_**图 8：**SAP HA 体系结构模板 1 – 包含 ASCS/SCS 的专用群集和 DBMS 实例的专用群集_

## <a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a>基础结构准备

为了简化 SAP 所需资源的部署，我们为 SAP 开发了 Azure Resource Manager 模板。

这些 3 层模板也支持高可用性方案，例如：

- **体系结构模板 1** – 包含两个群集，分别用于 SAP ASCS/SCS 和 DBMS 的 SAP 单点故障

下面是方案 1 的 Azure Resource Manager 模板：

- Azure 应用商店映像：[https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image)
- 自定义映像：[https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image)

单击 SAP 3 层应用商店映像时，可在 Azure 门户预览中看到以下 UI：

![图 9：指定 SAP HA Azure Resource Manager 参数][sap-ha-guide-figure-3000]  


_**图 9：**指定 SAP HA Azure Resource Manager 参数_


请务必针对“SYSTEMAVAILABILITY”选项选择“HA”。

模板将创建：

- 以下组件所需的一切 **VM**：
    - SAP 应用程序服务器 VM：`<SAPSystemSID>-di-<Number>`
    - ASCS/SCS 群集 VM：`<SAPSystemSID>-ascs-<Number>`
    - DBMS 群集：`<SAPSystemSID>-db-<Number>`
- **所有 VM 的网卡以及关联的 IP 地址**：
    - `<SAPSystemSID>-nic-di-<Number>`

    - `<SAPSystemSID>-nic-ascs-<Number>`
    - `<SAPSystemSID>-nic-db-<Number>`

- **Azure 存储帐户**
- 以下组件的**可用性组**：
    - SAP 应用程序服务器 VM：`<SAPSystemSID>-avset-di`
    - SAP ASCS/SCS 群集 VM：`<SAPSystemSID>-avset-ascs`
    - DBMS 群集 VM：`<SAPSystemSID>-avset-db`
- **Azure 内部负载均衡器 (ILB)** 以及 ASCS/SCS 实例的所有端口和 IP 地址  
  `<SAPSystemSID>-lb-ascs`
-	**Azure 内部负载均衡器 (ILB)** 以及 SQL Server DBMS 的所有端口和 IP 地址
  `<SAPSystemSID>-lb-db`
- **网络安全组**：`<SAPSystemSID>-nsg-ascs-0`  
具有对 `<SAPSystemSID>-ascs-0` VM 开放的外部 RDP 端口


> [AZURE.NOTE]  所有网卡和 Azure ILB 的 IP 地址一开始都创建为**动态**地址。必须如本文稍后所述，将它们更改为**静态** IP 地址。


### <a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a>部署具有企业网络连接（跨界）的 VM 供生产使用

针对生产 SAP 系统，将使用 Azure 站点到站点 (VPN) 或 Azure ExpressRoute 部署具有[企业网络连接（跨界）][planning-guide-2.2]的 Azure VM。

> [AZURE.NOTE]  在此情况下，必须事先创建并准备好 Azure VNET 和子网。


在“NEWOREXISTINGSUBNET”字段中，选择“existing”。

在“SUBNETID”文本字段中，需要添加已准备好的 Azure 网络 SubnetID 完整字符串，这是打算用于部署 Azure VM 的位置。

可以使用以下 PowerShell 命令获取所有 Azure 网络子网的列表：

    (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets

**SUBNETID** 显示在字段 ID 中。

例如，可以使用以下 PowerShell 命令来检索所有 **SUBNETID** 的列表：

    (Get-AzureRmVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id

**SUBNETID** 如下所示：

    /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>

### <a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a>SAP 实例的仅限云部署（用于测试/演示）

也可以在所谓的“仅限云部署”模型中部署 HA SAP 系统。

此部署主要适用于演示用例，但不适用于生产用例。

在“NEWOREXISTINGSUBNET”字段中，选择“new”。将“SUBNETID”字段保留**空白**。

SAP Azure Resource Manager 模板将自动创建 Azure VNET 和子网。

> [AZURE.NOTE] 此外，需要在相同的 VNET 中至少为 AD/DNS 部署一个专用 VM。这些 VM 不是模板创建的。


### <a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a>Azure 虚拟网络

在本例中，Azure VNET 的地址空间是 10.0.0.0/16。有一个名为 _**Subnet**_、地址范围为 10.0.0.0/24 的子网。所有 VM 和 ILB 都部署在此 VNET 中。
  
> [AZURE.NOTE] 请不要对来宾内部的网络设置（例如 IP 地址、DNS 服务器、子网等）进行任何更改。所有网络设置都通过 Azure 完成，通过 DHCP 服务传播。

### <a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a>DNS IP 地址

确保 VNET 的“DNS 服务器”选项设置为“自定义 DNS”。
对于以下情况：

- **[企业网络连接（跨界）][planning-guide-2.2]**：请添加本地 DNS 服务器的 IP 地址。  
  本地 DNS 服务器可以扩展到在 Azure 中运行的 VM。在此情况下，可以添加这些配置为运行 DNS 服务的 Azure VM 的 IP 地址。

-	**[仅限云部署][planning-guide-2.1]**：请在同一 VNET 中部署用作 DNS 服务器的附加 VM。添加这些配置为运行 DNS 服务的 Azure VM 的 IP 地址。


![图 10：配置 Azure VNET 的 DNS 服务器][sap-ha-guide-figure-3001]  


_**图 10：**配置 Azure VNET 的 DNS 服务器_

> [AZURE.NOTE] 如果更改了 DNS 服务器的 IP 地址，则需要重新启动 Azure VM，才能应用更改并传播新的 DNS 服务器。
在本例中，已在以下 Windows VM 上安装并配置 DNS 服务



| VM 角色 | VM 主机名 | 网卡名称 | 静态 IP 地址  
| ---------------|-------------|--------------------|-------------------
| 第 1 台 DNS 服务器 | domcontr-0 | pr1-nic-domcontr-0 | 10\.0.0.10         
| 第 2 台 DNS 服务器 | domcontr-1 | pr1-nic-domcontr-1 | 10\.0.0.11         


### <a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a>SAP ASCS/SCS 群集实例和 DBMS 群集实例的主机名与静态 IP 地址

像在本地一样，需要以下保留的主机名/IP 地址：

| 虚拟主机名角色 | 虚拟主机名 | 虚拟静态 IP 地址 
| ----------------------------------------------------------------------------|------------------|---------------------------
| SAP ASCS/SCS 第 1 个群集虚拟主机名（用于群集管理） | pr1-ascs-vir | 10\.0.0.42                 
| SAP ASCS/SCS **INSTANCE** 虚拟主机名 | pr1-ascs-sap | `10.0.0.43`              
| SAP DBMS 第 2 个群集虚拟主机名（用于群集管理） | pr1-dbms-vir | 10\.0.0.32                 
 

用于管理群集本身的虚拟主机名 _**pr1-ascs-vir**_ 和 _**pr1-dbms-vir**_ 以及关联的 IP 地址是在创建群集时创建的，如[收集群集配置中的群集节点][sap-ha-guide-8.12.1]一章中所述。

群集 SAP ASCS/SCS 实例和群集 DBMS 实例使用的其他两个虚拟主机名 _**pr1-ascs-sap**_ 和 _**pr1-dbms-sap**_ 以及关联的 IP 地址可在 DNS 服务器上手动创建，如[创建群集 SAP ASCS/SCS 的虚拟主机名][sap-ha-guide-9.1.1]一章中所述。

### <a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a>为 SAP VM 设置静态 IP 地址

部署用于群集的虚拟机之后，必须为所有 VM 设置静态 IP 地址。此操作无法在来宾 OS 内完成，必须在 Azure 虚拟网络配置中配置。

为此可以使用 Azure 门户预览。在 Azure 门户预览中，导航到：

    <Resource Group> -> <Network Card> -> Settings -> IP Address

将字段“分配”从“动态”更改为“静态”，然后输入所需的 **IP 地址**。

> [AZURE.NOTE] 如果更改了网卡的 IP 地址，则需要重新启动 Azure VM 才能应用更改。


![图 11：为每个 VM 的网卡配置静态 IP 地址][sap-ha-guide-figure-3002]  


_**图 11：**为每个 VM 的网卡配置静态 IP 地址_

针对所有网络接口（即所有 VM，包括用于 AD/DNS 服务的 VM）重复此步骤。

本例使用以下 VM 和静态 IP 地址：

| VM 角色 | VM 主机名 | 网卡名称 | 静态 IP 地址  
| ----------------------------------------|--------------|--------------------|-------------------
| 第 1 台 SAP 应用程序服务器 | pr1-di-0 | pr1-nic-di-0 | 10\.0.0.50         
| 第 2 台 SAP 应用程序服务器 | pr1-di-1 | pr1-nic-di-1 | 10\.0.0.51         
| ... | ... | ... | ...               
| 最后一台 SAP 应用程序服务器 | pr1-di-5 | pr1-nic-di-5 | 10\.0.0.55         
| ASCS/SCS 实例的第 1 个群集节点 | pr1-ascs-0 | pr1-nic-ascs-0 | 10\.0.0.40         
| ASCS/SCS 实例的第 2 个群集节点 | pr1-ascs-1 | pr1-nic-ascs-1 | 10\.0.0.41         
| DBMS 实例的第 1 个群集节点 | pr1-db-0 | pr1-nic-db-0 | 10\.0.0.30         
| DBMS 实例的第 2 个群集节点 | pr1-db-1 | pr1-nic-db-1 | 10\.0.0.31         

### <a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a>为内部负载均衡器 (ILB) 设置静态 IP 地址

SAP Azure Resource Manager 模板可创建用于 SAP ASCS/SCS 实例和用于 DBMS 实例的 Azure 内部负载均衡器 (ILB)。

初始部署将 ILB 的 IP 地址设置为“动态”。请务必将 IP 地址更改为“静态”。

在本例中，两个 Azure ILB 使用以下静态 IP 地址：

| Azure ILB 角色 | Azure ILB 名称 | 静态 IP 地址 
| ---------------------------|----------------|-------------------
| SAP ASCS/SCS 实例 ILB | pr1-lb-ascs | `10.0.0.43`  
| SAP DBMS ILB | pr1-lb-dbms | 10\.0.0.33         


> [AZURE.NOTE]  
**SAP ASCS/SCS 的虚拟主机名 IP 地址 = SAP ASCS/SCS Azure Load Balancer pr1-lb-ascs 的 IP 地址**  
**DBMS 的虚拟名称 IP 地址 = DBMS Azure Load Balancer pr1-lb-dbms 的 IP 地址**

在本例中，请将内部负载均衡器 _pr1-lb-ascs_ 的 IP 地址设置为 SAP ASCS/SCS 实例的虚拟主机名 IP 地址 (`10.0.0.43`)

![图 12：为 SAP ASCS/SCS 实例的内部负载均衡器 (ILB) 设置静态 IP 地址][sap-ha-guide-figure-3003]  


_**图 12：**为 SAP ASCS/SCS 实例的内部负载均衡器 (ILB) 设置静态 IP 地址_

以相同的方式，将负载均衡器 _pr1-lb-dbms_ 的 IP 地址设置为 DBMS 实例的虚拟主机名 IP 地址（在本例中为 10.0.0.33）。

### <a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a>Azure 内部负载均衡器 (ILB) 的默认 ASCS/SCS 负载均衡规则

默认情况下，SAP Azure Resource Manager 模板为以下组件创建所需的全部端口：

- 默认实例编号为 **00** 的 ABAP ASCS 实例
- 默认实例编号为 **01** 的 Java SCS 实例

因此，在安装 SAP ASCS/SCS 实例时，必须针对 ABAP ASCS 和/或 Java SCS 实例使用这些默认的实例编号 00 和 01。

下面是需要为 SAP NetWeaver ABAP ASCS 端口创建的 Azure ILB 终结点：


| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 00 的 ASCS 实例）（ERS 为 10）的具体端口  
| ---------------------------------------------|-----------------------|--------------------------------------------------------------------------
| 排队服务器 / _lbrule3200_ | 32`<InstanceNumber>` | 3200                                                                     
| ABAP 消息服务器 / _lbrule3600_ | 32`<InstanceNumber>` | 3600                                                                     
| 内部 ABAP 消息 / _lbrule3900_ | 39`<InstanceNumber>` | 3900                                                                     
| 消息服务器 HTTP / _Lbrule8100_ | 81`<InstanceNumber>` | 8100                                                                     
| SAP 启动服务 ASCS HTTP / _Lbrule50013_ | 5`<InstanceNumber>`13 | 50013                                                                    
| SAP 启动服务 ASCS HTTPS / _Lbrule50014_ | 5`<InstanceNumber>`14 | 50014                                                                    
| 排队复制 / _Lbrule50016_ | 5`<InstanceNumber>`16 | 50016                                                                    
| SAP 启动服务 ERS HTTP _Lbrule51013_ | 5`<InstanceNumber>`13 | 51013                                                                    
| SAP 启动服务 ERS HTTP _Lbrule51014_ | 5`<InstanceNumber>`14 | 51014                                                                    
| Win RM _Lbrule5985_ | | 5985                                                                     
| 文件共享 _Lbrule445_ | | 445                                                                      

_**表 1：**SAP NetWeaver ABAP ASCS 实例的端口号_


下面是必须为 SAP NetWeaver Java SCS 端口创建的 Azure ILB 终结点：

| 服务/负载均衡规则名称 | 默认端口号 | （实例编号为 01 的 SCS 实例）（ERS 为 11）的具体端口  
| ---------------------------------------------|-----------------------|--------------------------------------------------------------------------
| 排队服务器 / _lbrule3201_ | 32`<InstanceNumber>` | 3201                                                                     
| 网关服务器 / _lbrule3301_ | 33`<InstanceNumber>` | 3301                                                                     
| Java 消息服务器 / _lbrule3900_ | 39`<InstanceNumber>` | 3901                                                                     
| 消息服务器 HTTP / _Lbrule8101_ | 81`<InstanceNumber>` | 8101                                                                     
| SAP 启动服务 SCS HTTP / _Lbrule50113_ | 5`<InstanceNumber>`13 | 50113                                                                    
| SAP 启动服务 SCS HTTPS / _Lbrule50114_ | 5`<InstanceNumber>`14 | 50114                                                                    
| 排队复制 / _Lbrule50116_ | 5`<InstanceNumber>`16 | 50116                                                                    
| SAP 启动服务 ERS HTTP _Lbrule51113_ | 5`<InstanceNumber>`13 | 51113                                                                    
| SAP 启动服务 ERS HTTP _Lbrule51114_ | 5`<InstanceNumber>`14 | 51114                                                                    
| Win RM _Lbrule5985_ | | 5985                                                                     
| 文件共享 _Lbrule445_ | | 445                                                                      

_**表 2：**SAP NetWeaver Java SCS 实例的端口号_


![图 13：Azure 内部负载均衡器 (ILB) 的默认 ASCS/SCS 负载均衡规则][sap-ha-guide-figure-3004]  


_**图 13：**Azure 内部负载均衡器 (ILB) 的默认 ASCS/SCS 负载均衡规则_

以相同的方式，将负载均衡器 _**pr1-lb-dbms**_ 的 IP 地址设置为 DBMS 实例的虚拟主机名 IP 地址（在本例中为 _**10.0.0.33**_）。

### <a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a>更改 Azure 内部负载均衡器 (ILB) 的 ASCS/SCS 默认负载均衡规则

如果要将其他实例编号用于 SAP ASCS 或 SCS 实例，必须更新这些端口的名称和值。

可以使用 Azure 门户预览更新此设置。

转到 `<SID>-lb-ascs load balancer -> Load Balancing Rules`

针对属于 SAP ASCS 或 SCS 实例的所有负载均衡规则，请更改：

- 名称
- 端口
- 后端端口

例如，如果要将默认 ASCS 实例编号从 00 更改为 31，则必须针对_**表 1：**SAP NetWeaver ABAP ASCS 实例的端口号_中所列的所有端口进行更改。

下面是端口 _lbrule3200_ 的更新示例。

![图 14：更改 Azure 内部负载均衡器 (ILB) 的 ASCS/SCS 默认负载均衡规则][sap-ha-guide-figure-3005]  


_**图 14：**更改 Azure 内部负载均衡器 (ILB) 的 ASCS/SCS 默认负载均衡规则_


### <a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a>将 Windows 计算机添加到域

将静态 IP 地址分配给 VM 后，请将 VM 添加到域。

![图 15：将 VM 添加到域][sap-ha-guide-figure-3006]  


_**图 15：**将 VM 添加到域_


### <a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a>在用于 SAP ASCS/SCS 实例的两个群集节点上添加注册表项

Azure Load Balancer（包括 Azure 内部负载均衡器）在连接空闲达一段特定的时间（空闲超时）之后关闭这些连接。另一方面，对话实例中的 SAP 工作进程在需要发送第一个排队/取消排队请求时，立即打开与 SAP 排队进程的连接。这些连接通常持续保持已建立状态，直到工作进程或排队进程重新启动为止。但是，如果连接空闲达到一段时间，Azure ILB 将关闭这些连接。这实际上不会造成问题，因为如果与排队进程的连接已不存在，SAP 工作进程将重建该连接。但是，系统将这些活动记录在 SAP 进程的开发人员跟踪内，因此会在没有真正合理原因的情况下，在这些跟踪内创建大量内容。因此，我们建议同时更改两个群集节点上的 TCP/IP `KeepAliveTime` 和 `KeepAliveInterval`。更改 TCP/IP 参数需要与 SAP 配置文件参数相结合，我们将在本文稍后进行说明。

在 SAP ASCS/SCS 实例的两个群集节点上添加以下 Windows 注册表项：

| 路径 | HKLM\\SYSTEM\\CurrentControlSet\\Services\\Tcpip\\Parameters   
| ----------------------|-----------------------------------------------------------
| 变量名 | `KeepAliveTime`  
| 变量类型 | REG\_DWORD（十进制）                                        
| 值 | 120000                                                     
| 文档链接 | [https://technet.microsoft.com/zh-cn/library/cc957549.aspx](https://technet.microsoft.com/zh-cn/library/cc957549.aspx) 


_**表 3：**要更改的第一个 TCP/IP 参数_


| 路径 | HKLM\\SYSTEM\\CurrentControlSet\\Services\\Tcpip\\Parameters   
| ----------------------|-----------------------------------------------------------
| 变量名 | `KeepAliveInterval`  
| 变量类型 | REG\_DWORD（十进制）                                        
| 值 | 120000                                                     
| 文档链接 | [https://technet.microsoft.com/zh-cn/library/cc957548.aspx](https://technet.microsoft.com/zh-cn/library/cc957548.aspx)


_**表 4：**要更改的第二个 TCP/IP 参数_

然后**重新启动这两个群集节点**，应用这些更改。

### <a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a>SAP ASCS/SCS 实例的 Windows Server 故障转移群集设置

#### <a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a>收集群集配置中的群集节点

第一步是在两个群集节点上添加故障转移群集功能。为此可以使用“添加角色和功能向导”，在此不再提供其他任何说明。

第二步是使用 Windows 故障转移群集管理器设置故障转移群集。

在故障转移群集管理器 MMC 中，单击“创建群集”，然后只添加第一个群集节点 A 的名称，例如 _**pr1-ascs-0**_。暂时不要添加第二个节点。后面的步骤会添加它。

![图 16：添加故障转移群集配置的第一个步骤 – 添加用作群集节点的第一个节点的服务器/VM 名称][sap-ha-guide-figure-3007]  


_**图 16：**添加故障转移群集配置的第一个步骤 – 添加用作群集节点的第一个节点的服务器/VM 名称_

在后续步骤中，系统将要求提供群集的网络名称（虚拟主机名）。

![图 17：添加故障转移群集配置的第二个步骤 – 定义群集的名称][sap-ha-guide-figure-3008]  


_**图 17：**添加故障转移群集配置的第二个步骤 – 定义群集的名称_


创建群集后，运行群集验证测试

![图 18：添加故障转移群集配置的最后一个步骤 – 运行群集验证检查][sap-ha-guide-figure-3009]  


_**图 18：**添加故障转移群集配置的最后一个步骤 – 运行群集验证检查_


![图 19：添加故障转移群集配置的最后一个步骤 – 群集验证检查将显示有关找不到仲裁磁盘的警告][sap-ha-guide-figure-3010]  


_**图 19：**添加故障转移群集配置的最后一个步骤 – 群集验证检查将显示有关找不到仲裁磁盘的警告_

在此阶段可以忽略任何有关磁盘的警告，稍后将连同 SIOS 共享磁盘一起添加文件共享见证。在此阶段，我们不需要关心仲裁。

![图 20：已定义群集核心资源和 IP 地址 – 但需要添加新 IP 地址][sap-ha-guide-figure-3011]  


_**图 20：**已定义群集核心资源和 IP 地址 – 但需要添加新 IP 地址_


由于服务器的 IP 地址指向某个 VM 节点，因此群集无法启动。现在需要更改核心群集服务的 IP 地址。

例如，需要为群集虚拟主机名 _**pr1-ascs-vir**_ 分配 IP 地址（在本例中为 _**10.0.0.42**_）。可通过核心群集服务的 IP 资源属性页来完成此操作，如下所示

![图 21：使用“属性”更改为正确的 IP 地址][sap-ha-guide-figure-3012]  


_**图 21：**使用“属性”更改为正确的 IP 地址_


![图 22：分配群集的保留 IP 地址][sap-ha-guide-figure-3013]  


_**图 22：**分配群集的保留 IP 地址_

更改 IP 地址后，请将群集虚拟主机名联机。

![图 23：群集核心服务以正确的 IP 地址启动并运行][sap-ha-guide-figure-3014]  


_**图 23：**群集核心服务以正确的 IP 地址启动并运行_

核心群集服务启动并运行后，可以添加第二个群集节点

![图 24：添加第二个群集节点][sap-ha-guide-figure-3015]  


_**图 24：**添加第二个群集节点_

![图 25：添加第二个群集节点主机名，例如 pr1-ascs-1][sap-ha-guide-figure-3016]  


_**图 25：**添加第二个群集节点主机名，例如 pr1-ascs-1_

![图 26：不要选中复选框！][sap-ha-guide-figure-3017]  


_**图 26：**不要选中复选框！_

> [AZURE.NOTE]  
**切勿**选中“将所有符合条件的存储添加到群集”复选框！

![图 27：同样请忽略有关磁盘仲裁的警告][sap-ha-guide-figure-3018]  


_**图 27：**同样请忽略有关磁盘仲裁的警告_

可以忽略有关仲裁和磁盘的警告。仲裁和共享磁盘设置将在稍后配置，如**[为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition][sap-ha-guide-8.12.3]** 一章中所述。

#### <a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a>配置群集文件共享见证

##### <a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a>创建文件共享

请选择“文件共享见证”而不是“仲裁磁盘”。SIOS DataKeeper 支持此选项。

在本文用于演示的配置中，将在 Azure 中运行的名为 _**domcontr-0**_ 的 AD/DNS 服务器上配置文件共享见证。由于已配置了与 Azure 的 VPN 连接（通过站点到站点或 ExpressRoute），而 AD/DNS 驻留在本地环境中，因此它不适合用于运行文件共享见证。

> [AZURE.NOTE] 如果 AD/DNS 只在本地环境中运行，请不要在本地环境中运行的 AD/DNS Windows OS 上配置文件共享见证，因为在 Azure 上运行的群集节点与本地环境中的 AD/DNS 之间，网络延迟可能太大而造成连接问题。请务必在运行位置靠近群集节点的 Azure Windows VM 上配置文件共享见证。

仲裁驱动器至少需要 1024 MB 的可用空间。建议保留 2048 MB 空间

第一步是添加群集名称对象

![图 28：为群集名称对象分配对共享的权限][sap-ha-guide-figure-3019]  


_**图 28：**为群集名称对象分配对共享的权限_

请确保权限包括针对群集名称对象（在本例中为 _**pr1-ascs-vir$**_）更改共享中的数据。若要将群集名称对象添加到上面显示的列表中，必须按“添加”，然后更改筛选器，允许同时检查计算机对象，如下所示。

![图 29：更改对象类型以便同时包括计算机对象][sap-ha-guide-figure-3020]  


_**图 29：**更改对象类型以便同时包括计算机对象_

![图 30：选中计算机对象的复选框][sap-ha-guide-figure-3021]  


_**图 30：**选中计算机对象的复选框_

然后，输入群集名称对象，如图 29 所示。由于现已创建记录，因此可以更改权限，如图 28 所示。

下一步是使用共享的“安全”选项卡，为群集名称对象定义更细微的权限。

![图 31：为群集名称对象设置对文件共享仲裁的安全属性][sap-ha-guide-figure-3022]  


_**图 31：**为群集名称对象设置对文件共享仲裁的安全属性_

##### <a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a>在故障转移群集管理器中配置文件共享见证

下一步是使用故障转移群集管理器将群集配置更改为文件共享见证。

![图 32：如图所示调用“配置群集仲裁设置向导”][sap-ha-guide-figure-3023]  


_**图 32：**如图所示调用“配置群集仲裁设置向导”_

![图 33：不同仲裁配置的选择屏幕][sap-ha-guide-figure-3024]  


_**图 33：**不同仲裁配置的选择屏幕_

在此选项中，需要选择“选择仲裁见证”。

![图 34：文件共享见证的选择屏幕][sap-ha-guide-figure-3025]  


_**图 34：**文件共享见证的选择屏幕_

在本例中，需要选择“配置文件共享见证”。

![图 35：定义见证共享的文件共享位置][sap-ha-guide-figure-3026]  


_**图 35：**定义见证共享的文件共享位置_


输入文件共享的 UNC 路径（在本例中为 `\\domcontr-0\FSW`）

按“下一步”，显示需要进行的更改的列表。请检查该列表，然后再次按“下一步”更改群集配置。

![图 36：显示已在群集中成功完成重新配置的屏幕][sap-ha-guide-figure-3027]  


_**图 36：**显示已在群集中成功完成重新配置的屏幕_

在这最后一个步骤中，群集重新配置应该成功。

### <a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a>为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition

目前的状态是，Azure 中有一个正在运行的 Windows Server 故障转移群集配置。但是，此群集配置目前还没有共享的磁盘资源。若要安装 SAP ASCS/SCS，需要有一个共享磁盘资源。这就是 SIOS DataKeeper Cluster Edition 的作用所在。由于在 Azure 中无法创建具有所需功能的共享磁盘资源，因此必须依赖 SIOS DataKeeper 等程序来提供此功能。

#### <a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a>添加 Microsoft .NET Framework 3.5 功能

最新的 Windows Server 版本上不会自动启用或安装 Microsoft.NET Framework 3.5。但是，SIOS DataKeeper 要求在安装本产品的所有节点上都有 .NET Framework。因此，必须在不同 VM 的所有来宾 OS 上安装 .NET 3.5。

可通过两种方法添加 .Net 3.5 Framework。第一种可能的方法是使用 Windows 中的“添加角色和功能”，如以下所示：

![图 37：通过“添加角色和功能向导”安装 .Net Framework 3.5][sap-ha-guide-figure-3028]  


_**图 37：**通过“添加角色和功能向导”安装 .Net Framework 3.5_

![图 38：通过“添加角色和功能向导”安装 .Net Framework 3.5 时的进度条][sap-ha-guide-figure-3029]  


_**图 38：**通过“添加角色和功能向导”安装 .Net Framework 3.5 时的进度条_

启用 .Net Framework 3.5 功能的第二种可能方法是使用命令行工具 _**dism.exe**_。对于这种类型的安装，需要将 Windows 安装媒体的“sxs”目录设置为可访问。需要在提升权限的命令行窗口中运行以下命令：

    Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess

#### <a name="dd41d5a2-8083-415b-9878-839652812102"></a>安装 SIOS DataKeeper

让我们逐步完成 SIOS DataKeeper Cluster Edition 的安装。需要在群集中的两个节点上安装它。SIOS DataKeeper 通过创建同步镜像和模拟群集共享存储来创建虚拟共享存储。

在安装 SIOS 软件之前，必须先创建域用户 _**DataKeeperSvc**_。

> [AZURE.NOTE] 请将此 _**DataKeeperSvc**_ 用户同时添加到两个群集节点上的“本地管理员”组中。
  
在两个群集节点上安装 SIOS 软件

![SIOS 安装程序][sap-ha-guide-figure-3030]  


![图 39：第一个 SIOS DataKeeper 安装屏幕][sap-ha-guide-figure-3031]  


_**图 39：**第一个 SIOS DataKeeper 安装屏幕_

![图 40：DataKeeper 通知要禁用某个服务][sap-ha-guide-figure-3032]  


_**图 40：**DataKeeper 通知要禁用某个服务_

看到图 40 所示的弹出窗口时，请选择“是”。

![图 41：SIOS DataKeeper 的用户选项][sap-ha-guide-figure-3033]  


_**图 41：**SIOS DataKeeper 的用户选项_


在上述屏幕中，建议选择“域或服务器帐户”。

![图 42：为 SIOS DataKeeper 安装提供域用户和密码][sap-ha-guide-figure-3034]  


_**图 42：**为 SIOS DataKeeper 安装提供域用户和密码_

指定针对 SIOS DataKeeper 创建的域帐户及其密码。

![图 43：提供 SIOS DataKeeper 许可证][sap-ha-guide-figure-3035]  


_**图 43：**提供 SIOS DataKeeper 许可证_

如图 43 所示，安装 SIOS DataKeeper 的许可证密钥。安装结束时，系统会要求**重新启动 VM**。

#### <a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a>设置 SIOS DataKeeper

在两个节点上安装 SIOS DataKeeper 之后，必须开始配置。配置的目的是在连接到每个 VM 的附加 VHD 之间进行同步数据复制。以下步骤说明了两个节点上的配置。

![图 44：DataKeeper 管理和配置工具][sap-ha-guide-figure-3036]  


_**图 44：**DataKeeper 管理和配置工具_


启动 DataKeeper 的管理和配置工具，然后按“连接服务器”链接（上图中带有红圈的部分）

![图 45：插入管理工具应连接到的第一个节点的名称或 TCP/IP 地址，然后在第二个步骤中，插入第二个节点的相关数据][sap-ha-guide-figure-3037]  


_**图 45：**插入管理工具应连接到的第一个节点的名称或 TCP/IP 地址，然后在第二个步骤中，插入第二个节点的相关数据_

下一步是在两个节点之间创建复制作业

![图 46：创建复制作业][sap-ha-guide-figure-3038]  


_**图 46：**创建复制作业_

将有一个向导引导完成此过程

![图 47：定义复制作业的名称][sap-ha-guide-figure-3039]  


_**图 47：**定义复制作业的名称_

![图 48：定义节点（应是当前源节点）的基本数据][sap-ha-guide-figure-3040]  


_**图 48：**定义节点（应是当前源节点）的基本数据_

在第一个步骤中，需要定义源节点的名称、TCP/IP 地址和磁盘卷。第二步是定义目标节点。同样，需要定义目标节点的名称、TCP/IP 地址和磁盘卷。

![图 49：定义节点（应是当前目标节点）的基本数据][sap-ha-guide-figure-3041]  


_**图 49：**定义节点（应是当前目标节点）的基本数据_

下一步是定义要应用的压缩算法。为此，建议压缩复制流。尤其是在重新同步的情况下，压缩复制流可大幅缩短重新同步的时间。请注意，压缩会占用 VM 的 CPU 和 RAM 资源。因此，压缩率越高，CPU 使用率也越高。以后可以调整和更改此设置。

另一个需要检查的设置是复制是以异步还是同步方式运行。**为了保护 SAP ASCS/SCS 配置，必须使用“同步复制”设置**。

![图 50：定义复制详细信息][sap-ha-guide-figure-3042]  


_**图 50：**定义复制详细信息_

最后一步是定义是否应向 WSFC 群集配置指出复制作业所复制的卷是共享磁盘。对于 SAP ASCS/SCS 配置，需要选择“是”，这样，Windows 群集才将复制的卷视为可用作群集卷的共享磁盘。

![图 51：按“是”以便可将复制的卷用作群集卷][sap-ha-guide-figure-3043]  


_**图 51：**按“是”以便可将复制的卷用作群集卷_

创建完成之后，DataKeeper 管理工具将复制作业列为活动状态。

![图 52：SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态][sap-ha-guide-figure-3044]  


_**图 52：**SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态_

这样，该磁盘在 Windows 故障转移群集管理器中显示为 DataKeeper 磁盘，如下所示。

![图 53：DataKeeper 复制的磁盘显示在故障转移群集管理器中][sap-ha-guide-figure-3045]  


_**图 53：**DataKeeper 复制的磁盘显示在故障转移群集管理器中_


## <a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a>SAP NetWeaver 系统安装

本文不会介绍 DBMS 的设置，因为这些设置根据所用 DBMS 系统而有差异。但是，本文假设 DBMS 在高可用性方面的疑虑已通过不同 DBMS 供应商为 Azure 提供的功能支持而获得解决。例如适用于 SQL Server 的 AlwaysOn 或数据库镜像，以及适用于 Oracle 的 Oracle Data Guard。在本文使用的示例方案中，并没有为 DBMS 提供额外保护。

在与 Azure 上的此类群集 SAP ASCS/SCS 配置进行交互方面，不需要根据不同的 DBMS 做出特殊考虑。

> [AZURE.NOTE]  
SAP NetWeaver ABAP 系统、Java 系统和 ABAP+Java 系统的安装过程几乎完全相同。最大的差别在于，SAP ABAP 系统只有一个 ASCS 实例。SAP Java 系统只有一个 SCS 实例，而 SAP ABAP+Java 系统则有一个 ASCS 和一个 SCS 实例在同一 Microsoft 故障转移群集组中运行。本文将明确说明每个 SAP NetWeaver 安装堆栈的所有安装差异。所有其他部件假设相同。

### <a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a>具有高可用性 ASCS/SCS 实例的 SAP 安装

> [AZURE.NOTE]  
切勿将页面文件放在 DataKeeper 镜像卷上，因为 DataKeeper 不支持页面文件。可以将页面文件保留在 Azure VM 的 D:\\ 临时驱动器中，此处是在 Azure 中部署 VM 时就已用于放置页面文件的位置。如果情况并非如此，请更正，然后将 Windows 页面文件放在 Azure VM 的 D:\\ 驱动器上。

#### <a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a>创建群集 SAP ASCS/SCS 的虚拟主机名

第一步是为 ASCS/SCS 实例的虚拟主机名创建所需的 DNS 项。用于执行此操作的工具是 Windows DNS 管理器。除了虚拟主机名以外，还需要定义分配给虚拟主机名的 IP 地址。

> [AZURE.NOTE]  
**请记住，分配给 ASCS/SCS 实例的虚拟主机名的 IP 地址必须与分配给 Azure Load Balancer (`<sid>-lb-ascs`) 的 IP 地址相同。虚拟 SAP ASCS/SCS 主机名 `(pr1-ascs-sap)` 的 IP 地址 = Azure Load Balancer `(pr1-lb-ascs)` 的 IP 地址**

这也意味着，在 Azure 上，一个 Windows Server 故障转移群集中只能运行一个 SAP 故障转移群集角色，例如，对于 ABAP 系统而言，只能运行一个 ASCS 实例；对于 Java 系统而言，只能运行一个 SCS 实例；对于 ABAP+Java 而言，只能运行一个 ASCS 和一个 SCS 实例。

> [AZURE.NOTE] 《SAP 安装指南》（请参阅 [SAP Installation Guides][sap-installation-guides]（SAP 安装指南））中所述的多 SID 群集目前在 Azure 中无法运行。

![图 54：定义 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 项][sap-ha-guide-figure-3046]  


_**图 54：**定义 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 项_

该项显示在 DNS 管理器的域下面，如下图所示。

![图 55：针对 SAP ASCS/SCS 群集配置列出的新虚拟名称和 TCP/IP 地址][sap-ha-guide-figure-3047]  


_**图 55：**针对 SAP ASCS/SCS 群集配置列出的新虚拟名称和 TCP/IP 地址_

#### <a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a>安装 SAP 的第一个群集节点

第一个 ASCS/SCS 群集节点的安装方式与 SAP 安装文档中所述的方式相同：
- 在群集节点 A（在本例中为 _**pr1-ascs-0**_ 主机）上执行“第一个群集节点”选项。

如果要为 Azure 内部负载均衡器保留默认端口，请选择：

- 对于 **ABAP 系统** - **ASCS** 实例编号 **00**
- 对于 **Java 系统** - **SCS** 实例编号 **01**
- 对于 **ABAP + JAVA 系统** - **ASCS** 实例编号 **00** 和 **SCS** 实例编号 **01**

如果要为 ABAP ASCS 实例使用 00 以外的编号，为 Java SCS 实例使用 01 以外的编号，首先需要根据**[更改 Azure 内部负载均衡器 (ILB) 的 ASCS/SCS 默认负载均衡规则][sap-ha-guide-8.9]**中所述，更改 Azure ILB 默认负载均衡规则。

完成此步骤后，需要执行一般 SAP 安装文档中未介绍的几个步骤。

#### <a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a>修改 ASCS/SCS 实例的 SAP 配置文件

需要添加一个新的配置文件参数。此配置文件参数可避免 SAP 工作进程与排队服务器之间的连接在空闲时间太长时关闭。本文档的**[在用于 SAP ASCS/SCS 实例的两个群集节点上添加注册表项][sap-ha-guide-8.11]**一章中已提到了出现该问题的情景。在该部分，我们还介绍了对一些基本 TCP/IP 连接参数所做的两项更改。在第二个步骤中，需要将排队服务器配置为发送 **keep\_alive** 信号，使连接不会达到 Azure ILB 的空闲阈值。为此，请将以下配置文件参数：

    enque/encni/set_so_keepalive = true

添加到 SAP ASCS/SCS 实例配置文件。在本例中，路径为：

`<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`  


例如，添加到 SAP SCS 实例配置文件和相应的路径

`<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`  



#### <a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a>添加探测端口

为使整个群集配置能够与 Azure Load Balancer 配合工作，需要利用内部负载均衡器的探测功能。Azure 内部负载均衡器通常在参与方虚拟机之间平衡和平均分配传入的工作负荷。但是，这种工作方式在此类群集配置中不可行，因为只有一个实例处于主动状态，另一个处于被动状态，无法接受工作负荷。为了启用 Azure 内部负载均衡器只将工作分配给主动实例的配置，建立了探测功能。通过该功能，内部负载均衡器可以检查哪个实例处于主动状态，然后只将工作负荷发送到该实例。首先，让我们在参与群集配置的某个 VM 中执行以下 PowerShell 命令，检查当前的 _**ProbePort**_ 设置：

    Get-ClusterResource „SAP PR1 IP" | Get-ClusterParameter 

![图 56：群集配置的探测端口默认为 0][sap-ha-guide-figure-3048]  


_**图 56：**群集配置的探测端口默认为 0_

探测端口号默认设置为 0。为使配置正常工作，需要定义一个端口。在本例中，必须使用探测端口 _**62300**_，因为 SAP Azure Resource Manager 模板中已定义此端口号。若要分配该端口号，可以使用以下两个命令：

首先，获取 SAP 虚拟主机名群集资源 _**SAP WAC IP**_

    $var = Get-ClusterResource | Where-Object {  $_.name -eq "SAP PR1 IP"  } 

然后，将探测端口设置为 62300

    $var | Set-ClusterParameter -Multiple @{"Address"="10.0.0.43";"ProbePort"=62300;"Subnetmask"="255.255.255.0";"Network"="Cluster Network 1";"OverrideAddressMatch"=1;"EnableDhcp"=0}  

必须将 _**SAP PR1**_ 群集角色停止再启动，才能激活更改。

将 _**SAP PR1**_ 群集角色联机后，检查 _**ProbePort**_ 是否已设置为新值：

    Get-ClusterResource „SAP PR1 IP" | Get-ClusterParameter 

![图 57：更改后的群集探测端口][sap-ha-guide-figure-3049]  


_**图 57：**更改后的群集探测端口_

可以看到，_**ProbePort**_ 现已设置为 _**62300**_。可以从其他主机（例如 _**ascsha-dbas**_）访问文件共享 _**\\\ascsha-clsap\\sapmnt**_。

### <a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a>安装数据库实例

安装数据库实例的方式与 SAP 安装文档中所述的过程没有什么不同。因此，本文不做介绍。

### <a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a>安装第二个群集节点

同样，第二个群集节点的安装方式与 SAP 安装文档所述的方式没有什么不同。请根据安装指南中的步骤安装第二个群集节点。

### <a name="094bc895-31d4-4471-91cc-1513b64e406a"></a>更改 SAP ERS 实例的 Windows 服务启动类型

将两个群集节点上的 SAP ERS Windows 服务的启动类型更改为“自动(延迟启动)”。

![图 58：将 SAP ERS 实例的服务类型更改为自动延迟][sap-ha-guide-figure-3050]  


_**图 58：**将 SAP ERS 实例的服务类型更改为自动延迟_

### <a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a>安装 SAP 主应用程序服务器 (PAS)

在指定用于托管 PAS 的 VM 上执行主应用程序服务器实例 `<sid>-di-0` 安装。这种安装不需要考虑 Azure 还是 DataKeeper。

### <a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a>安装 SAP 附加应用程序服务器 (AAS)

在指定用于托管 SAP 应用程序服务器的所有 VM（例如 `<sid>-di-1` 到 `<sid>-di-<n>`）上执行附加 SAP 应用程序服务器的安装。

## <a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a>测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制

可以使用_**故障转移群集管理器**_和 SIOS DataKeeper UI，轻松测试及监视 SAP ASCS/SCS 实例故障转移与 SIOS 磁盘复制。

### <a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a>起点 – SAP ASCS/SCS 实例在群集节点 A 上运行

_**SAP WAC**_ 群集组在群集节点 A（例如在 _**ascsha-clna**_）上运行。属于 _**SAP WAC**_ 群集组并由 ASCS/SCS 实例使用的共享磁盘 S: 已分配到群集节点 A。

![图 59：故障转移群集管理器：SAP <SID> 群集组在群集节点 A 上运行][sap-ha-guide-figure-5000]  


_**图 59：**故障转移群集管理器：SAP <SID> 群集组在群集节点 A 上运行_

使用 SIOS DataKeeper UI，可以看到共享磁盘数据以同步方式从群集节点 A（例如 _**ascsha-clna [10.0.0.41]**_）上的源卷 S: 复制到群集节点 B（例如 _**ascsha-clnb [10.0.0.42]**_）上的目标卷 _**S:**_

![图 60：SIOS DataKeeper：将本地卷从群集节点 A 复制到群集节点 B][sap-ha-guide-figure-5001]  


_**图 60：**SIOS DataKeeper：将本地卷从群集节点 A 复制到群集节点 B_


### <a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a>从节点 A 到节点 B 的故障转移过程

可以启动将 SAP <SID> 群集组从群集节点 A 故障转转到群集节点 B 的故障转移：

- 使用故障转移群集管理器

- 使用故障转移群集 PowerShell

        Move-ClusterGroup -Name "SAP WAC" 

- 在 Windows 来宾 OS 中重新启动群集节点 A  
（这会启动将 SAP <SID> 群集组从节点 A 故障转转到节点 B 的自动故障转移）  

- 从 Azure 门户预览重新启动群集节点 A  
（这会启动将 SAP <SID> 群集组从节点 A 故障转转到节点 B 的自动故障转移）

- 使用 Azure PowerShell 重新启动群集节点 A  
（这会启动将 SAP <SID> 群集组从节点 A 故障转转到节点 B 的自动故障转移）

        Restart-AzureVM -Name ascsha-clna -ServiceName ascsha-cluster

### <a name="755a6b93-0099-4533-9f6d-5c9a613878b5"></a>最终结果 – SAP ASCS/SCS 实例在群集节点 B 上运行

故障转移后，`SAP <SID>` 群集组在群集节点 B（例如 _**ascsha-clnb**_）上运行。

![图 61：故障转移群集管理器：SAP <SID> 群集组在群集节点 B 上运行][sap-ha-guide-figure-5002]  


_**图 61：**故障转移群集管理器：SAP <SID> 群集组在群集节点 B 上运行_

共享磁盘现已装载到群集节点 B 上。SIOS DataKeeper 正在将数据从群集节点 B（例如 _**ascsha-clnb [10.0.0.42]**_）上的源卷 S: 复制到群集节点 A（例如 _**ascsha-clna [10.0.0.41]**_）上的目标卷 S:。

![图 62：SIOS DataKeeper：将本地卷从群集节点 B 复制到群集节点 A][sap-ha-guide-figure-5003]  


_**图 62：**SIOS DataKeeper：将本地卷从群集节点 B 复制到群集节点 A_

<!---HONumber=Mooncake_1010_2016-->