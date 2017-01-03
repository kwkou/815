<properties
   pageTitle="Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南 | Azure"
   description="Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南"
   services="virtual-machines-windows,virtual-network,storage"
   documentationCenter="saponazure"
   authors="MSSedusch"
   manager="juergent"
   editor=""
   tags="azure-resource-manager"
   keywords=""/>  

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="11/08/2016"
   wacn.date="12/30/2016"
   ms.author="sedusch"/>  


# Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南

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

[azure-cli]: /documentation/articles/xplat-cli-install/
[azure-portal]: https://portal.azure.cn
[azure-ps]: /documentation/articles/powershell-install-configure/
[azure-quickstart-templates-github]: https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]: https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits/
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits/#subscription

[dbms-guide]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南"
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

[deployment-guide]: /documentation/articles/virtual-machines-windows-sap-deployment-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南"
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

[planning-guide]: /documentation/articles/virtual-machines-windows-sap-planning-guide/ "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南"
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

公司可以使用 Azure 在最短的时间内获取计算和存储资源，而无需经历冗长的采购周期。Azure 虚拟机可让公司将典型的应用程序（例如，基于 SAP NetWeaver 的应用程序）部署到 Azure 中，并提高其可靠性和可用性，且不需要在本地提供其他资源。Azure 还支持跨场地连接，使公司能够主动地将 Azure 虚拟机集成到其本地域、私有云和 SAP 系统布局中。

本白皮书逐步介绍如何为基于 SAP NetWeaver 的应用程序部署准备 Azure 虚拟机。其内容假设已了解 [Planning and Implementation Guide][planning-guide]（规划和实施指南）中包含的信息。如果尚不了解这些信息，应先阅读相关的文档。

此文是对 SAP 安装文档和 SAP 说明的补充，这些文档代表在给定平台上安装和部署 SAP 软件的主要资源。

[AZURE.INCLUDE [windows-warning](../../includes/virtual-machines-linux-sap-warning.md)]

## 介绍
全球有大量的公司都在使用基于 SAP NetWeaver 的应用程序 - 以 SAP Business Suite 为主流 - 来运行其任务关键型业务流程。因此，系统运行状况是一项关键的资产指标，在发生故障（包括性能事件）时提供企业支持的能力也成了一项至关重要的要求。
Azure 提供了优异的平台检测，以满足所有业务关键型应用程序的可支持性要求。本指南可确保将用于部署 SAP 软件的 Azure 虚拟机配置为能够提供企业支持，而不管以哪种方式创建该虚拟机（使用 Azure 应用商店中的映像，或者是使用特定于客户的映像）。
下面将详细说明所有必要的设置步骤。

## 先决条件和资源
### 先决条件
在开始之前，请确保满足以下章节中所述的先决条件。

#### 本地个人计算机
本地个人计算机为 SAP 软件部署设置 Azure 虚拟机的过程包括多个步骤。若要管理 Windows VM 或 Linux VM，需要使用 PowerShell 脚本和 Azure 门户预览。为此，必须准备一台运行 Windows 7 或更高版本的本地个人计算机。如果你只想要管理 Linux VM，并想要使用 Linux 计算机执行此任务，则也可以使用 Azure 命令行接口 (Azure CLI)。

#### Internet 连接
若要下载和执行所需的工具与脚本，需要建立 Internet 连接。此外，运行 Azure 增强型监视扩展的 Azure 虚拟机也需要访问 Internet。如果此 Azure VM 是 Azure 虚拟网络或本地域的一部分，请确保根据本文档的[配置代理][deployment-guide-configure-proxy]一章中所述完成相关的代理设置。

#### Azure 订阅
已有一个 Azure 帐户，并且知道相应的登录凭据。

#### 拓扑注意事项和网络
需要定义 Azure 中 SAP 部署的拓扑和体系结构。与以下各项相关的体系结构：

* 要使用的 Azure 存储帐户
* 要将 SAP 系统部署到的虚拟网络
* 要将 SAP 系统部署到的资源组
* 要部署 SAP 系统的 Azure 区域
* SAP 配置（双层或三层）
* VM 大小以及要装载到 VM 的附加 VHD 数
* SAP 传输与纠正系统配置

应已创建并配置 Azure 存储帐户或 Azure 虚拟网络等。[Planning and Implementation Guide][planning-guide]（规划和实施指南）中介绍了相关的创建和配置操作。

#### SAP 选型
* 已使用 SAP Quicksizer 或工具测定出预计的 SAP 工作负荷，并已知道相应的 SAPS 数
* 应该知道 SAP 系统所需的 CPU 资源和内存消耗
* 应该知道所需的每秒 I/O 运算次数
* 已知道 Azure 中不同 VM 之间最终通信时所需的网络带宽
* 已知道本地资产与 Azure 部署的 SAP 系统之间所需的网络带宽

#### 资源组
资源组是一个新概念，包含具有相同生命周期的所有资源（例如，它们是同时创建和删除的）。有关资源组的详细信息，请阅读[此文][resource-group-overview]。

### <a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>SAP 资源
在配置工作期间需要以下资源：

* SAP 说明 [1928533]
	* 支持部署 SAP 软件的 Azure 虚拟机大小的列表
	* 按 Azure 虚拟机大小提供的重要容量信息
	* 支持的 SAP 软件以及操作系统与数据库的组合
* SAP 说明 [2015553]，其中列出了在将 SAP 软件部署到 Azure 时，要受 SAP 支持而必须满足的先决条件。
* SAP 说明 [1999351]，其中包含有关适用于 SAP 的增强型 Azure 监视的其他故障排除信息。
* SAP 说明 [2178632]，其中包含有关 Azure 上 SAP 的所有可用监视度量值的详细信息。
* SAP 说明 [1409604]，其中包含部署在新的 Azure Resource Manager 中时，Azure 上所需的适用于 Windows 的 SAP 主机代理版本。
* SAP 说明 [2191498]，其中包含部署在新的 Azure Resource Manager 中时，Azure 上所需的适用于 Linux 的 SAP 主机代理版本。
* SAP 说明 [2243692]，其中包含 Azure 上 Linux 中的 SAP 许可信息
* SAP 说明 [1984787]，其中包含有关 SUSE LINUX Enterprise Server 12 的一般信息
* SAP 说明 [2002167]，其中包含有关 Red Hat Enterprise Linux 7.x 的一般信息
* [SCN](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes)，其中包含适用于 Linux 的全部所需的 SAP 说明
* [Azure PowerShell][azure-ps] 中特定于 SAP 的 PowerShell cmdlet
* [Azure CLI][azure-cli] 中特定于 SAP 的 Azure CLI
* [Azure 门户预览][azure-portal]
 
以下指南也包含了有关 Azure 上的 SAP 的主题：

* [Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南][planning-guide]
* [Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南（本文档）][deployment-guide]
* [Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南][dbms-guide]
* [Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性部署指南][ha-guide]

## <a name="b3253ee3-d63b-4d74-a49b-185e76c4088e"></a>Azure 上 SAP 的 VM 部署方案
在本章中，你将了解不同的部署方法，以及针对每种部署类型所要执行的每个步骤。

### 为 SAP 部署 VM
Azure 提供多种用于部署 VM 和相关磁盘的方法。因此，请务必了解这些方法之间的差异，因为 VM 的准备工作可能会因部署方法的不同而异。总体而言，我们将探讨以下方案：

#### 部署从 Azure 应用商店中取出的 VM
想要从 Azure 应用商店中获取某个 Microsoft 或第三方映像来部署 VM。将 VM 部署到 Azure 之后，你需要像在本地环境中一样，遵照相同的准则并使用相同的工具在 VM 内部安装 SAP 软件。若要在 Azure VM 内部安装 SAP 软件，SAP 和 Azure 建议将 SAP 安装媒体上载并存储到 Azure VHD 中，或创建一个充当“文件服务器”并包含所有必要 SAP 安装媒体的 Azure VM。

有关详细信息，请参阅[方案 1：为 SAP 部署从 Azure 应用商店中取出的 VM][deployment-guide-3.2] 一章。

#### <a name="3688666f-281f-425b-a312-a77e7db2dfab"></a>使用自定义映像部署 VM
由于 OS 或 DBMS 版本具有特定的补丁要求，从 Azure 应用商店取出的映像可能并不符合需要。因此，你可能需要使用自己的、以后可以多次部署的“专用”OS/DB VM 映像创建一个 VM。
Windows 与 Linux 映像之间的专用映像创建步骤有所不同。

___

> ![Windows][Logo_Windows] Windows
>
> 若要准备可用于部署多个虚拟机的 Windows 映像，必须在本地 VM 上抽象化/通用化 Windows 设置（例如 Windows SID 和主机名）。为此，可以使用 <https://technet.microsoft.com/zh-cn/library/cc721940.aspx> 中所述的 sysprep。
>
> ![Linux][Logo_Linux] Linux
>
> 若要准备可用于部署多个虚拟机的 Linux 映像，必须在本地 VM 上抽象化/通用化某些 Linux 设置。为此，可以使用[此文][virtual-machines-linux-capture-image]或[此文][virtual-machines-linux-agent-user-guide-command-line-options]中所述的 waagent -deprovision。

___

设置数据库内容的方式是使用 SAP Software Provision Manager 安装新的 SAP 系统，从连接到虚拟机的 VHD 还原数据库备份，或直接从 Azure 存储空间还原数据库备份（如果 DBMS 支持此操作）（请参阅 [DBMS Deployment Guide][dbms-guide]（DBMS 部署指南））。如果已在本地 VM 中安装了 SAP 系统（尤其是对于双层系统），则在部署 Azure VM 后，可以通过 SAP Software Provisioning Manager 支持的系统重命名过程来修改 SAP 系统设置（SAP 说明 [1619720]）。否则，可以稍后在部署 Azure VM 之后安装 SAP 软件。

有关详细信息，请参阅[方案 2：使用自定义映像为 SAP 部署 VM][deployment-guide-3.3] 一章。

#### 使用非通用化磁盘将 VM 从本地移至 Azure
你打算将某个特定 SAP 系统从本地移至 Azure。可以通过将包含 OS、SAP 二进制文件和最终 DBMS 二进制文件的 VHD，以及包含 DBMS 数据和日志文件的 VHD 上载到 Azure 来实现此目的。与上面的[使用自定义映像部署 VM][deployment-guide-3.1.2] 一章中所述的方案相反，需要将 Azure VM 中的主机名、SAP SID 和 SAP 用户帐户保留为与本地环境中的配置相同。因此，不需要将操作系统通用化。此情况主要适用于跨界方案，在这类方案中，SAP 布局的一部分在本地运行，其余部分则在 Azure 上运行。

有关详细信息，请参阅[方案 3：使用包含 SAP 的非通用化 Azure VHD 从本地移动 VM][deployment-guide-3.4] 一章。

### <a name="db477013-9060-4602-9ad4-b0316f8bb281"></a>方案 1：为 SAP 部署来自 Azure 应用商店的 VM
Azure 可让用户从 Azure 应用商店部署 VM 实例，该库提供 Windows Server 的一些标准 OS 映像以及不同的 Linux 分发版。你还可以部署包含 DBMS SKU 的映像（例如 SQL Server）。有关使用这些包含 DBMS SKU 的映像的详细信息，请参阅 [DBMS Deployment Guide][dbms-guide]（DBMS 部署指南）。

特定于 SAP 的、从 Azure 应用商店部署 VM 的步骤顺序如下所示：

![使用 Azure 应用商店中的 VM 映像为 SAP 系统部署 VM 的流程图][deployment-guide-figure-100]

根据该流程图，需要执行以下步骤：

#### 使用 Azure 门户预览创建虚拟机
使用 Azure 应用商店中的映像创建新虚拟机的最简单方式是通过 Azure 门户预览。导航到 <https://portal.azure.cn/#create>。在搜索字段中输入要部署的操作系统类型（例如 Windows、SLES 或 RHEL），然后选择版本。请确保选择“Azure Resource Manager”部署模型，然后单击“创建”。

向导将引导你完成创建虚拟机的所需参数以及全部所需资源（例如网络接口或存储帐户）。其中一些参数包括：

1. 基础知识
    1. 名称：资源名称，例如虚拟机名称
    1. 用户名和密码/SSH 公钥：输入在预配期间创建的用户的用户名和密码。对于 Linux 虚拟机，也可以输入使用 SSH 登录计算机时所用的 SSH 公钥。
    1. 订阅：选择要用于预配新虚拟机的订阅。
    1. 资源组：资源组的名称。可以插入新资源组的名称或现有资源组的名称。
    1. 位置：选择新虚拟机应部署到的位置。如果想要将虚拟机连接到本地网络，请确保选择将 Azure 连接到本地网络的虚拟网络位置。有关详细信息，请参阅 [Planning Guide][planning-guide]（规划指南）中的 [Azure Networking][planning-guide-microsoft-azure-networking]（Azure 网络）一章。
1. 大小：有关受支持 VM 类型的列表，请阅读 SAP 说明 [1928533]。如果想要使用高级存储，另请确保选择正确的类型。并非所有 VM 类型都支持高级存储。有关详细信息，请参阅 [Planning Guide][planning-guide]（规划指南）中的 [Storage: Azure Storage and Data Disks][planning-guide-storage-microsoft-azure-storage-and-data-disks]（存储：Azure 存储空间和数据磁盘）及 [Azure Premium Storage][planning-guide-azure-premium-storage]（Azure 高级存储）章节。
1. 设置
    1. 存储帐户：可以选择现有存储帐户，或创建新的存储帐户。有关不同存储类型的详细信息，请阅读 [DBMS Guide][dbms-guide]（DBMS 指南）中的 [Azure Storage][dbms-guide-2.3]（Azure 存储空间）一章。请注意，不一定支持使用所有存储类型来运行 SAP 应用程序。
    1. 虚拟网络和子网：如果要将虚拟机集成到内部网络，请选择连接到本地网络的虚拟网络。
    1. 公共 IP 地址：选择想要使用的公共 IP 地址，或输入参数来创建新的公共 IP 地址。可以使用公共 IP 地址通过 Internet 访问虚拟机。请确保同时创建网络安全组来筛选对虚拟机的访问。
    1. 网络安全组：有关详细信息，请参阅 [What is a Network Security Group (NSG)][virtual-networks-nsg]（什么是网络安全组 (NSG)）。
    1. 监视：可以禁用诊断设置。当运行命令来启用 Azure 增强型监视时，将自动启用诊断，如[配置监视][deployment-guide-configure-monitoring-scenario-1]一章中所述。
    1. 可用性：选择一个可用性集，或输入参数来创建新的可用性集。有关详细信息，请参阅 [Azure 可用性集][planning-guide-3.2.3]一章。
1. 摘要：验证摘要页上提供的信息，然后单击“确定”。

完成向导之后，虚拟机将部署在所选的资源组中。

#### 使用模板创建虚拟机
也可以使用 [azure-quickstart-templates github 存储库][azure-quickstart-templates-github]中发布的某个 SAP 模板来创建部署。或者，可以使用 [Azure 门户预览][virtual-machines-windows-tutorial]、[PowerShell][virtual-machines-ps-create-preconfigure-windows-resource-manager-vms] 或 [Azure CLI][virtual-machines-linux-tutorial] 手动创建虚拟机。

* [双层配置（仅一个虚拟机）模板][sap-templates-2-tier-marketplace-image]
如果想要创建仅使用一个虚拟机的双层系统，请使用此模板。
* [三层配置（多个虚拟机）模板][sap-templates-3-tier-marketplace-image]
如果想要创建使用多个虚拟机的三层系统，请使用此模板。

打开上述其中一个模板之后，Azure 门户预览将导航到“编辑参数”面板。输入以下信息：

* **sapSystemId**：SAP 系统 ID
* **osType**：要部署的操作系统，例如 Windows Server 2012 R2、SLES 12 或 RHEL 7.2
    * 该列表仅包含 Azure 上 SAP 支持的版本。
* **sapSystemSize**：SAP 系统的大小
    * 新系统将提供的 SAPS 数量。如果不确定系统需要多少 SAPS，请咨询你的 SAP 技术合作伙伴或系统集成商
* **systemAvailability**：（仅限三层模板）系统可用性
    * 选择适用于 HA 安装的配置的 HA。将为 ASCS 创建两个数据库服务器和两个服务器。
* storageType：（仅限双层模板）应使用的存储类型
    * 对于更大的系统，强烈建议使用高级存储。有关不同存储类型的详细信息，请阅读
        * [DBMS Guide][dbms-guide]（DBMS 指南）中的 [Azure Storage][dbms-guide-2.3]（Azure 存储空间）
        * [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储][storage-premium-storage-preview-portal]
        * [Azure 存储简介][storage-introduction]
* **adminUsername** 和 **adminPassword**：用户名和密码
    * 创建可用于登录计算机的新用户。
* **newOrExistingSubnet**：确定是要创建新的虚拟网络和子网，还是使用现有子网。如果已有连接到本地网络的虚拟网络，请选择现有虚拟网络。
* **subnetId**：虚拟机应连接到的子网的 ID。选择用于将虚拟机连接到本地网络的 VPN 或 Express Route 虚拟网络的子网。ID 通常类似于 /subscriptions/`<subscription id`>/resourceGroups/`<resource group name`>/providers/Microsoft.Network/virtualNetworks/`<virtual network name`>/subnets/`<subnet name`>

输入所有参数后，请选择要使用的订阅和资源组。可以选择现有资源组，或选择下拉菜单中的“+ 新建”来创建新资源组。如果创建新资源组，则还必须选择要在其中创建资源组和虚拟机的区域。

查看并接受法律条款，然后单击“创建”。

请注意，使用 Azure 应用商店中的映像时，将按默认部署 Azure VM 代理。

#### 配置代理设置
根据本地网络配置，如果通过 VPN 或 Express Route 将虚拟机连接到本地网络，则可能需要在虚拟机上配置代理。否则，虚拟机可能无法访问 Internet，因此无法下载所需的扩展或收集监视数据。请参阅本文档的[配置代理][deployment-guide-configure-proxy]一章。

#### 加入域（仅限 Windows）
如果通过 Azure 站点到站点或 Express Route（在 [Planning and Implementation Guide][planning-guide]（规划和实施指南）中也称为跨界）将 Azure 中的部署连接到本地 AD/DNS，则 VM 应该加入本地域。本文档的[将 VM 加入本地域（仅限 Windows）][deployment-guide-4.3]一章说明了此步骤的注意事项。

#### <a name="ec323ac3-1de9-4c3a-b770-4ff701def65b"></a>配置监视
如本文档的[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述，配置适用于 SAP 的 Azure 增强型监视扩展。

请查看 SAP 监视的先决条件，以了解本文档的 [SAP 资源][deployment-guide-2.2]一章列出的资源中的 SAP 内核和 SAP 主机代理所需的最低版本。

#### 监视检查
根据 [Azure 上 SAP 的端到端监视设置的检查和故障排除][deployment-guide-troubleshooting-chapter]一章中所述，检查监视是否在正常运行。

#### 部署后步骤
在创建 VM 后，将部署该 VM，然后，你需要自行在该 VM 中安装所有必要的软件组件。因此，这种类型的 VM 部署要求所安装的软件已在 Azure 中的其他某个 VM 中提供，或者以可附加的磁盘形式提供。或者，我们需要研究与本地资产（安装共享）建立连接的跨界方案。

### <a name="54a1fc6d-24fd-4feb-9c57-ac588a55dff2"></a>方案 2：使用自定义映像为 SAP 部署 VM
如 [Planning and Implementation Guide][planning-guide]（规划和实施指南）中的详细步骤所述，可通过一种方法来准备和创建自定义映像，然后使用它来创建多个新的 VM。
流程图中的步骤顺序如下：
 
![使用专用应用商店中的 VM 映像为 SAP 系统部署 VM 的流程图][deployment-guide-figure-300]  


根据该流程图，需要执行以下步骤：

#### 创建虚拟机
若要通过 Azure 门户预览使用专用 OS 映像来创建部署，请使用 [azure-quickstart-templates github 存储库][azure-quickstart-templates-github]中发布的某个 SAP 模板。
也可以使用 [PowerShell][virtual-machines-upload-image-windows-resource-manager] 手动创建虚拟机。

* [双层配置（仅一个虚拟机）模板][sap-templates-2-tier-user-image]
如果想要创建仅使用一个虚拟机和自己的 OS 映像的双层系统，请使用此模板。
* [三层配置（多个虚拟机）模板][sap-templates-3-tier-user-image]
如果想要创建使用多个虚拟机和自己的 OS 映像的三层系统，请使用此模板。

打开上述其中一个模板之后，Azure 门户预览将导航到“编辑参数”面板。输入以下信息：

* **sapSystemId**：SAP 系统 ID
* **osType**：要部署的操作系统类型（Windows 或 Linux）
* **sapSystemSize**：SAP 系统的大小
    * 新系统将提供的 SAPS 数量。如果不确定系统需要多少 SAPS，请咨询你的 SAP 技术合作伙伴或系统集成商
* **systemAvailability**：（仅限三层模板）系统可用性
    * 选择适用于 HA 安装的配置的 HA。将为 ASCS 创建两个数据库服务器和两个服务器。
* **storageType**：（仅限双层模板）应使用的存储类型
    * 对于更大的系统，强烈建议使用高级存储。有关不同存储类型的详细信息，请阅读
        * [DBMS Guide][dbms-guide]（DBMS 指南）中的 [Azure Storage][dbms-guide-2.3]（Azure 存储空间）
        * [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储][storage-premium-storage-preview-portal]
        * [Azure 存储简介][storage-introduction]
* **adminUsername** 和 **adminPassword**：用户名和密码
    * 创建可用于登录计算机的新用户。
* **userImageVhdUri**：专用 OS 映像 VHD 的 URI，例如 https://`<accountname`>.blob.core.chinacloudapi.cn/vhds/userimage.vhd
* **userImageStorageAccount**：存储专用 OS 映像的存储帐户名，例如上述示例 URI 中的 `<accountname`>
* **newOrExistingSubnet**：确定是要创建新的虚拟网络和子网，还是使用现有子网。如果已有连接到本地网络的虚拟网络，请选择现有虚拟网络。
* **subnetId**：虚拟机应连接到的子网的 ID。选择用于将虚拟机连接到本地网络的 VPN 或 Express Route 虚拟网络的子网。ID 通常类似于 /subscriptions/`<subscription id`>/resourceGroups/`<resource group name`>/providers/Microsoft.Network/virtualNetworks/`<virtual network name`>/subnets/`<subnet name`>

输入所有参数后，请选择要使用的订阅和资源组。可以选择现有资源组，或选择下拉菜单中的“+ 新建”来创建新资源组。如果创建新资源组，则还必须选择要在其中创建资源组和虚拟机的区域。

查看并接受法律条款，然后单击“创建”。

#### 安装 VM 代理（仅限 Linux）
如果想要使用上述模板，必须已在用户映像中安装 Linux 代理。否则，部署将失败。如本文档的[下载、安装并启用 Azure VM 代理][deployment-guide-4.4]一章中所述，在用户映像中下载并安装 VM 代理。
如果不使用上述模板，以后也可以安装 VM 代理。

#### 加入域（仅限 Windows）
如果通过 Azure 站点到站点或 Express Route（在 [Planning and Implementation Guide][planning-guide]（规划和实施指南）中也称为跨界）将 Azure 中的部署连接到本地 AD/DNS，则 VM 应该加入本地域。本文档的[将 VM 加入本地域（仅限 Windows）][deployment-guide-4.3]一章说明了此步骤的注意事项。

#### 配置代理设置
根据本地网络配置，如果通过 VPN 或 Express Route 将虚拟机连接到本地网络，则可能需要在虚拟机上配置代理。否则，虚拟机可能无法访问 Internet，因此无法下载所需的扩展或收集监视数据。请参阅本文档的[配置代理][deployment-guide-configure-proxy]一章。

#### 配置监视
如本文档的[配置适用于 SAP 的 Azure 监视扩展][deployment-guide-4.5]一章中所述，配置适用于 SAP 的 Azure 增强型监视扩展。
请查看 SAP 监视的先决条件，以了解本文档的 [SAP 资源][deployment-guide-2.2]一章列出的资源中的 SAP 内核和 SAP 主机代理所需的最低版本。

#### 监视检查
根据 [Azure 上 SAP 的端到端监视设置的检查和故障排除][deployment-guide-troubleshooting-chapter]一章中所述，检查监视是否在正常运行。

### <a name="a9a60133-a763-4de8-8986-ac0fa33aa8c1"></a>方案 3：使用包含 SAP 的非通用化 Azure VHD 从本地移动 VM
此方案可以满足只将 SAP 系统以其当前格式和形式从本地移至 Azure 的需要。这意味着，Windows 或 Linux 主机名和 SAP SID 或类似信息不会发生更改。在此情况下，VHD 在部署期间不是作为映像引用，而是直接用作 OS 磁盘。在部署方面，此方案与前面两种方案的不同之处在于，部署期间无法自动安装 VM 代理。因此，以后需要从 Microsoft 下载 Azure VM 代理，并且需要在 VM 中手动安装并启用该代理。成功完成该任务后，可以继续启动 SAP 主机监视 Azure 扩展及其配置。有关 Azure VM 代理的功能的详细信息，请查看以下文章：

___

> ![Windows][Logo_Windows] Windows
>
> <http://blogs.msdn.com/b/wats/archive/2014/02/17/bginfo-guest-agent-extension-for-azure-vms.aspx>
>
> ![Linux][Logo_Linux] Linux
>
> [Azure Linux 代理用户指南][virtual-machines-linux-agent-user-guide]

___

不同步骤的工作流如下：
 
![使用 VM 磁盘为 SAP 系统部署 VM 的流程图][deployment-guide-figure-400]

假设已在 Azure 中上载并定义磁盘（请参阅 [Planning and Implementation Guide][planning-guide]（规划和实施指南）），请遵循以下步骤

#### 创建虚拟机
若要通过 Azure 门户预览使用专用 OS 磁盘来创建部署，请使用 [azure-quickstart-templates github 存储库][azure-quickstart-templates-github]中发布的 SAP 模板。也可以使用 PowerShell 或 Azure CLI 手动创建虚拟机。

* [双层配置（仅一个虚拟机）模板][sap-templates-2-tier-os-disk]
    * 如果想要创建仅使用一个虚拟机的双层系统，请使用此模板。

打开上述模板之后，Azure 门户预览将导航到“编辑参数”面板。输入以下信息：

* **sapSystemId**：SAP 系统 ID
* **osType**：要部署的操作系统类型（Windows 或 Linux）
* **sapSystemSize**：SAP 系统的大小
    * 新系统将提供的 SAPS 数量。如果不确定系统需要多少 SAPS，请咨询你的 SAP 技术合作伙伴或系统集成商
* **storageType**：（仅限双层模板）应使用的存储类型
    * 对于更大的系统，强烈建议使用高级存储。有关不同存储类型的详细信息，请阅读
        * [DBMS Guide][dbms-guide]（DBMS 指南）中的 [Azure Storage][dbms-guide-2.3]（Azure 存储空间）
        * [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储][storage-premium-storage-preview-portal]
        * [Azure 存储简介][storage-introduction]
* **osDiskVhdUri**：专用 OS 磁盘的 URI，例如 https://`<accountname`>.blob.core.chinacloudapi.cn/vhds/osdisk.vhd
* **newOrExistingSubnet**：确定是要创建新的虚拟网络和子网，还是使用现有子网。如果已有连接到本地网络的虚拟网络，请选择现有虚拟网络。
* **subnetId**：虚拟机应连接到的子网的 ID。选择用于将虚拟机连接到本地网络的 VPN 或 Express Route 虚拟网络的子网。ID 通常类似于 /subscriptions/`<subscription id`>/resourceGroups/`<resource group name`>/providers/Microsoft.Network/virtualNetworks/`<virtual network name`>/subnets/`<subnet name`>

输入所有参数后，请选择要使用的订阅和资源组。可以选择现有资源组，或选择下拉菜单中的“+ 新建”来创建新资源组。如果创建新资源组，则还必须选择要在其中创建资源组和虚拟机的区域。

查看并接受法律条款，然后单击“创建”。

#### 安装 VM 代理
若要使用上述模板，必须已在 OS 磁盘中安装 VM 代理。否则，部署将失败。如本文档的[下载、安装并启用 Azure VM 代理][deployment-guide-4.4]一章中所述，在 VM 中下载并安装 VM 代理。

如果不使用上述模板，以后也可以安装 VM 代理。

#### 加入域（仅限 Windows）
如果通过 Azure 站点到站点或 Express Route（在 [Planning and Implementation Guide][planning-guide]（规划和实施指南）中也称为跨界）将 Azure 中的部署连接到本地 AD/DNS，则 VM 应该加入本地域。本文档的[将 VM 加入本地域（仅限 Windows）][deployment-guide-4.3]一章说明了此步骤的注意事项。

#### 配置代理设置
根据本地网络配置，如果通过 VPN 或 Express Route 将虚拟机连接到本地网络，则可能需要在虚拟机上配置代理。否则，虚拟机可能无法访问 Internet，因此无法下载所需的扩展或收集监视数据。请参阅本文档的[配置代理][deployment-guide-configure-proxy]一章。

#### 配置监视
如本文档的[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述，配置适用于 SAP 的 Azure 增强型监视扩展。

请查看 SAP 监视的先决条件，以了解本文档的 [SAP 资源][deployment-guide-2.2]一章列出的资源中的 SAP 内核和 SAP 主机代理所需的最低版本。

#### 监视检查
根据 [Azure 上 SAP 的端到端监视设置的检查和故障排除][deployment-guide-troubleshooting-chapter]一章中所述，检查监视是否在正常运行。

### 方案 4：更新 SAP 的监视配置
在某些情况下，需要更新监视配置：

* SAP/MS 联合团队已扩展了监视功能，并决定要添加更多的计数器或删除一些计数器。
* Microsoft 引入了可提供监视数据的新版底层 Azure 基础结构，适用于 SAP 的 Azure 增强型监视扩展需要适应这些更改。
* 你要添加装载到 Azure VM 的更多 VHD，或者想要删除 VHD。在此情况下，需要更新存储相关数据的集合。如果通过添加或删除终结点或者通过向 VM 分配 IP 地址来更改配置，则此操作不会影响监视配置。
* 你要更改 Azure VM 大小（例如，将 VM 从 A5 更改为其他任何大小）。
* 你要在 Azure VM 中添加网络接口

若要更新监视配置，请按如下所述继续操作：

* 遵循本文档的[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述的步骤更新监视基础结构。重新运行此章中所述的脚本可以检测是否已部署某个监视配置，并对该监视配置执行所需的更改。

___

> ![Windows][Logo_Windows] Windows
>
> 更新 Azure VM 代理时无需用户干预。VM 代理将自动更新自身，并且不需要重新启动 VM。
>
> ![Linux][Logo_Linux] Linux
>
> 请遵循[此文][virtual-machines-linux-update-agent]中的步骤更新 Azure Linux 代理。

___

## 每个部署步骤的详述

### <a name="604bcec2-8b6e-48d2-a944-61b0f5dee2f7"></a>部署 Azure PowerShell cmdlet
* 转到 <https://www.azure.cn/downloads/>
* 在“命令行工具”部分下，有一个名为“Windows PowerShell”的部分。单击“安装”链接。
* 此时将弹出 Microsoft 下载管理器，其中包含一个以 .exe 结尾的行项。选择“运行”选项。
* 此时将出现一个弹出窗口，询问你是否要运行 Microsoft Web 平台安装程序。按“是”
* 此时将出现如下所示的屏幕：
 
![Azure PowerShell cmdlet 安装屏幕][deployment-guide-figure-500] 
<a name="figure-5"></a>

* 按“安装”并接受 EULA。

经常检查 PowerShell cmdlet 是否已更新。通常，我们每月都会提供更新。执行此操作的最简单方法就是遵循上述安装步骤，直到出现[此图][deployment-guide-figure-5]中所示的安装屏幕为止。此屏幕中显示了 cmdlet 的发行日期及实际版本号。除非 SAP 说明 [1928533] 或 [2015553] 中另有规定，否则建议使用最新版本的 Azure PowerShell cmdlet。

可以使用以下 PS 命令检查台式机/便携式计算机上当前安装的 Azure cmdlet 版本：

    Import-Module Azure
    (Get-Module Azure).Version

结果应如以下[此图][deployment-guide-figure-6]中所示。

![Azure PS cmdlet 版本检查结果][deployment-guide-figure-600] 
<a name="figure-6"></a>

如果台式机/便携式计算机上安装的 Azure cmdlet 版本是最新版本，启动 Microsoft Web 平台安装程序后出现的第一个屏幕将与[此图][deployment-guide-figure-5]中所示的屏幕略有不同。

请注意[下图][deployment-guide-figure-7]中的红圈。
 
![表示已安装最新 Azure PS cmdlet 版本的 Azure PowerShell cmdlet 安装屏幕][deployment-guide-figure-700] 
<a name="figure-7"></a>

如果屏幕与[上图][deployment-guide-figure-7]中所示的相同，则表示已安装最新的 Azure cmdlet 版本，无需继续安装。在这种情况下，你可以在此阶段“退出”安装。

### <a name="1ded9453-1330-442a-86ea-e0fd8ae8cab3"></a>部署 Azure CLI
* 转到 <https://www.azure.cn/downloads/>
* 在“命令行工具”部分下，有一个名为“Azure 命令行接口”的部分。单击你的操作系统对应的“安装”链接。

经常检查 Azure CLI 是否已更新。通常，我们每月都会提供更新。执行此操作的最简单方法是遵循上述安装步骤。

可以使用以下命令检查台式机/便携式计算机上当前安装的 Azure CLI 版本：

    azure --version

结果应如以下[此图][deployment-guide-figure-azure-cli-version]中所示。

![Azure CLI 版本检查结果][deployment-guide-figure-760]  

### <a name="0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda"></a> <a name="31d9ecd6-b136-4c73-b61e-da4a29bbc9cc"></a>将 VM 加入本地域 — 仅限 Windows
如果将 SAP VM 部署到跨界方案，而该方案中的本地 AD 和 DNS 已扩展到 Azure，则这些 VM 应该已加入到本地域。将 VM 加入本地域以及将其他所需软件加入为本地域成员的详细步骤取决于具体的客户。通常，将 VM 加入本地域意味着需要安装附加的软件，例如，恶意软件防护软件，或者备份或监视软件的各种代理。

另外，如果在加入域时强制使用 Internet 代理设置，则需要确保来宾 VM 中的 Windows 本地系统帐户 (S-1-5-18) 也使用这些设置。最简单的方法是针对应用到域中的系统的域组策略强制使用代理。

### <a name="c7cbb0dc-52a4-49db-8e03-83e7edc2927d"></a>下载、安装并启用 Azure VM 代理
从非通用化（例如未针对 Windows 执行 sysprep）的 OS 映像部署适用于 SAP 的 VM 时，需要执行以下步骤。对于从 Azure 应用商店部署的虚拟机，不需要安装代理。这些映像已包含 Azure 代理。

#### <a name="b2db5c9a-a076-42c6-9835-16945868e866"></a>Windows

* 下载 Azure VM 代理：
	* 从以下网页下载 Azure VM 代理安装程序包：<https://go.microsoft.com/fwlink/?LinkId=394789>
	* 将 VM 代理 MSI 包存储在便携式计算机或服务器本地
* 安装 Azure VM 代理：
	* 使用终端服务 (RDP) 连接到已部署的 Azure VM
	* 在 VM 上打开一个 Windows 资源管理器窗口，然后打开 VM 代理的 MSI 文件的目标目录
	* 将 Azure VM 代理安装程序 MSI 文件从本地便携式计算机/服务器拖放到 VM 上的 VM 代理的目标目录中
	* 在 VM 中双击该 MSI 文件
	* 对于已加入本地域的 VM，请确保最终的 Internet 代理设置也适用于 VM 中的 Windows 本地系统帐户 (S-1-5-18)，如[配置代理][deployment-guide-configure-proxy]一章中所述。VM 代理将在此上下文中运行，并且需要能够连接到 Azure。

#### <a name="6889ff12-eaaf-4f3c-97e1-7c9edc7f7542"></a>Linux
使用以下命令来安装适用于 Linux 的 VM 代理

- **SLES**

    sudo zypper install WALinuxAgent

- **RHEL**

    sudo yum install WALinuxAgent

### <a name="baccae00-6f79-4307-ade4-40292ce4e02d"></a>配置代理
在 Windows 和 Linux 中配置代理的步骤有所不同。

#### Windows
这些设置对于用于访问 Internet 的 LocalSystem 帐户也必须有效。如果代理设置不是通过组策略设置的，可以遵循以下步骤为 LocalSystem 帐户配置这些设置。

1.	打开 gpedit.msc
1.	导航到“计算机配置”–>“管理模板”->“Windows 组件”->“Internet Explorer”，然后启用“按计算机(而不是按用户)进行代理设置”
1.	打开“控制台”，然后导航到“网络和 Internet”->“Internet 选项”
1.	打开“连接”选项卡，然后单击“局域网设置”
1.	禁用“自动检测设置”
1.	启用“为 LAN 使用代理服务器”，然后输入代理主机和端口

#### Linux
在 Azure 来宾代理的配置文件（位于 /etc/waagent.conf 中）中配置正确的代理。必须设置以下参数：

    HttpProxy.Host=<proxy host e.g. proxy.corp.local>
    HttpProxy.Port=<port of the proxy host e.g. 80>

更改代理配置后，重新启动代理

    sudo service waagent restart

/etc/waagent.conf 中的代理设置也适用于所需的 VM 扩展。如果你要使用 Azure 存储库，请确保这些存储库的流量不会流经本地 Intranet。如果已创建用户定义的路由来启用强制隧道，请确保添加路由，将发往存储库的流量直接路由到 Internet，而不是通过站点到站点连接路由。

- **SLES** 
此外，需要为 /etc/regionserverclnt.cfg 中列出的 IP 地址添加路由。以下屏幕截图显示了一个示例。

- **RHEL** 
此外，需要为 /etc/yum.repos.d/rhui-load-balancers 中列出的主机 IP 地址添加路由。以下屏幕截图显示了一个示例。

有关用户定义的路由的详细信息，请参阅[此文][virtual-networks-udr-overview]。

![强制隧道][deployment-guide-figure-50]

### <a name="d98edcd3-f2a1-49f7-b26a-07448ceb60ca"></a>配置适用于 SAP 的 Azure 增强型监视扩展
根据 [Azure 上 SAP 的 VM 部署方案][deployment-guide-3]一章所述准备好 VM 之后，即已在计算机上安装 Azure VM 代理。下一个重要步骤是部署 Azure 全球数据中心内的 Azure 扩展存储库中提供的适用于 SAP 的 Azure 增强型监视扩展。有关详细信息，请查看 [Planning and Implementation Guide][planning-guide-9.1]（规划和实施指南）。

可以使用 Azure PowerShell 或 Azure CLI 安装和配置适用于 SAP 的 Azure 增强型监视扩展。如果要使用 Windows 计算机在 Windows 或 Linux VM 上安装扩展，请阅读 [Azure PowerShell][deployment-guide-4.5.1] 一章。若要使用 Linux 台式机在 Linux VM 上安装扩展，请阅读 [Azure CLI][deployment-guide-4.5.2] 一章。

#### <a name="987cf279-d713-4b4c-8143-6b11589bb9d4"></a>适用于 Linux 和 Windows VM 的 Azure PowerShell
若要执行安装适用于 SAP 的 Azure 增强型监视扩展的任务，请执行以下步骤：

* 确保已安装最新版本的 Azure PowerShell cmdlet。请参阅本文档的[部署 Azure PowerShell cmdlet][deployment-guide-4.1] 一章。
* 运行以下 Azure PowerShell cmdlet。有关可用环境的列表，请运行 cmdlet Get-AzureRmEnvironment。如果想要使用公共 Azure，你的环境将是 AzureCloud。对于中国区 Azure，请选择 AzureChinaCloud。

	$env = Get-AzureRmEnvironment -Name <name of the environment>
	Login-AzureRmAccount -Environment $env
	Set-AzureRmContext -SubscriptionName <subscription name>
    
    Set-AzureRmVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>

在你提供帐户数据和 Azure 虚拟机名称后，该脚本将部署所需的扩展，并启用所需的功能。这可能需要几分钟。
有关 Set-AzureRmVMAEMExtension 的详细信息，请阅读[此 MSDN 文章][msdn-set-azurermvmaemextension]。
  
![成功执行特定于 SAP 的 Azure cmdlet Set-AzureRmVMAEMExtension 时的结果屏幕][deployment-guide-figure-900]

成功运行 Set-AzureRmVMAEMExtension 后，将执行所有必要的步骤来为 SAP 配置主机监视功能。

该脚本应提供如下所述的输出：

* 确认已配置基础 VHD（包含 OS）以及装载到 VM 的所有其他 VHD 的监视配置。
* 接下来的两条消息确认已配置特定存储帐户的存储度量值。
* 有一行输出提供了监视配置的实际更新状态。
* 显示的另一行输出确认已部署或已更新配置。
* 最后一行输出是参考性的，指出可以测试监视配置。
* 若要检查是否已成功执行有关 Azure 监视的所有步骤以及 Azure 基础结构是否能够提供所需的数据，请根据本文档的[适用于 SAP 的 Azure 增强型监视的就绪状态检查][deployment-guide-5.1]一章中所述，执行适用于 SAP 的 Azure 监视扩展的就绪状态检查。
* 若要继续执行此操作，请等待 15-30 分钟，直到 Azure 诊断已收集所有相关数据。

#### <a name="408f3779-f422-4413-82f8-c57a23b4fc2f"></a>适用于 Linux VM 的 Azure CLI

若要使用 Azure CLI 执行安装适用于 SAP 的 Azure 增强型监视扩展的任务，请执行以下步骤：

1. 如[此文][azure-cli]中所述安装 Azure CLI
1. 使用 Azure 帐户登录

        azure login -e AzureChinaCloud

1. 切换到 Azure Resource Manager 模式

        azure config mode arm

1. 启用 Azure 增强型监视

        azure vm enable-aem <resource-group-name> <vm-name>

1. 验证 Azure 增强型监视是否在 Azure Linux VM 上处于活动状态。检查文件 /var/lib/AzureEnhancedMonitor/PerfCounters 是否存在。如果存在，请使用以下命令显示 AEM 收集的信息：

        cat /var/lib/AzureEnhancedMonitor/PerfCounters

    然后，将获得类似于下面的输出：

        2;cpu;Current Hw Frequency;;0;2194.659;MHz;60;1444036656;saplnxmon;
        2;cpu;Max Hw Frequency;;0;2194.659;MHz;0;1444036656;saplnxmon;


## <a name="564adb4f-5c95-4041-9616-6635e83a810b"></a>Azure 上 SAP 的端到端监视设置的检查和故障排除
在部署 Azure VM 并设置相关的 Azure 监视基础结构后，请检查 Azure 增强型监视的所有组件是否正常工作。

为此，请根据[适用于 SAP 的 Azure 增强型监视的就绪状态检查][deployment-guide-5.1]一章中所述，执行适用于 SAP 的 Azure 增强型监视扩展的就绪状态检查。如果此项检查的结果为正常并且你已获取所有相关的性能计数器，则表示已成功设置 Azure 监视。在此情况下，请继续安装本文档的 [SAP 资源][deployment-guide-2.2]一章列出的 SAP 说明中提供的 SAP 主机代理。如果就绪状态检查的结果指出缺少计数器，请根据 [Azure 监视基础结构配置的运行状况检查][deployment-guide-5.2]一章中所述，为 Azure 监视基础结构执行运行状况检查。如果 Azure 监视配置存在任何问题，请查看[对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除][deployment-guide-5.3]一章，以获取有关故障排除的更多帮助。

### <a name="bb61ce92-8c5c-461f-8c53-39f5e5ed91f2"></a>适用于 SAP 的 Azure 增强型监视的就绪状态检查
通过此项检查，可以确认底层 Azure 监视基础结构是否能够完全提供 SAP 应用程序中将会显示的度量值。

#### 在 Windows VM 上执行就绪状态检查
若要执行就绪状态检查，请登录 Azure 虚拟机（不需要管理员帐户）并执行以下步骤：

* 打开 Windows 命令提示符，并切换到适用于 SAP 的 Azure 监视扩展的安装文件夹 
C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\`<version`>\\drop

监视扩展路径中提供的版本部分根据具体的情况而有所不同。如果在安装文件夹中看到了监视扩展版本的多个文件夹，请检查 Windows 服务“AzureEnhancedMonitoring”的配置，并切换到“可执行文件的路径”指示的文件夹。
 
![运行适用于 SAP 的 Azure 增强型监视扩展的服务的属性][deployment-guide-figure-1000]

* 在命令窗口中不使用任何参数执行 azperflib.exe。

> [AZURE.NOTE] azperflib.exe 将以循环的方式运行，并每隔 60 秒更新收集的计数器一次。若要结束循环，请关闭命令窗口。

如果 Azure 增强型监视扩展未安装或者服务“AzureEnhancedMonitoring”未运行，则表示未正确配置该扩展。在此情况下，请查看[对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除][deployment-guide-5.3]一章中有关如何重新部署该扩展的详细说明。

##### 检查 azperflib.exe 的输出
azperflib.exe 的输出显示 SAP 的所有已填充 Azure 性能计数器。在收集的计数器的列表底部，可以看到一份摘要，以及指示 Azure 监视状态的运行状况指示器。
 
![通过执行 azperflib.exe 完成的运行状况检查的输出指示不存在问题][deployment-guide-figure-1100] 
<a name="figure-11"></a>

如以[上图][deployment-guide-figure-11]中所示，对于“Counters total”数量报告为 empty 以及对于“Health check”的输出，请检查返回的结果。

可按如下所述解释结果值：

| Azperflib.exe 结果值 | Azure 监视就绪状态 |
| ------------------------------|----------------------------------- |
| **Counters total: empty** | 以下 2 个 Azure 存储计数器可能为空：<ul><li>Storage Read Op Latency Server msec</li><li>Storage Read Op Latency E2E msec</li></ul>所有其他计数器必须包含值。 |
| **运行状况检查** | 仅当返回状态显示为 OK 时才表示正常 |

如果 azperflib.exe 的两个返回值未同时显示已正确返回所有填充的计数器，请根据下面 [Azure 监视基础结构配置的运行状况检查][deployment-guide-5.2]一章中所述，遵循有关针对 Azure 监视基础结构配置执行运行状况检查的说明。

#### 在 Linux VM 上执行就绪状态检查
若要执行就绪状态检查，请使用 SSH 连接到 Azure 虚拟机，然后执行以下步骤：

* 检查 Azure 增强型监视扩展的输出
    * more /var/lib/AzureEnhancedMonitor/PerfCounters
        * 应该提供性能计数器列表。文件不应为空
    * cat /var/lib/AzureEnhancedMonitor/PerfCounters | grep Error
        * 应返回错误为 none 的一行，例如 3;config;Error;;0;0;**none**;0;1456416792;tst-servercs;
    * more /var/lib/AzureEnhancedMonitor/LatestErrorRecord
        * 应为空或不存在
* 如果上述第一次检查不成功，请执行以下附加测试：
    * 确保已安装并启动 waagent
        * sudo ls -al /var/lib/waagent/
            * 应列出 waagent 目录的内容
        * ps -ax | grep waagent
            * 应显示一个类似于“python /usr/sbin/waagent -daemon”的条目
    * 确保已安装并启动 Linux 诊断扩展
        * sudo sh -c 'ls -al /var/lib/waagent/Microsoft.OSTCExtensions.LinuxDiagnostic-*'
            * 应列出 Linux 诊断扩展目录的内容
        * ps -ax | grep diagnostic
            * 应显示一个类似于“python /var/lib/waagent/Microsoft.OSTCExtensions.LinuxDiagnostic-2.0.92/diagnostic.py -daemon”的条目
    * 确保已安装并启动 Azure 增强型监视扩展
        * sudo sh -c 'ls -al /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-*/'
            * 应列出 Azure 增强型监视扩展目录的内容
        * ps -ax | grep AzureEnhanced
            * 应显示一个类似于“python /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-2.0.0.2/handler.py daemon”的条目
* 如 SAP 说明 [1031096] 中所述安装 SAP 主机代理，并检查 saposcol 的输出
    * 执行 /usr/sap/hostctrl/exe/saposcol -d
    * 执行 dump ccm
    * 检查“Virtualization\_Configuration\\Enhanced Monitoring Access”度量值是否为 true
* 如果已安装 SAP NetWeaver ABAP 应用程序服务器，请打开事务 ST06，并检查是否已启用增强型监视。

如果上述任何检查失败，请查看[对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除][deployment-guide-5.3]一章中有关如何重新部署该扩展的详细说明。

### <a name="e2d592ff-b4ea-4a53-a91a-e5521edb6cd1"></a>Azure 监视基础结构配置的运行状况检查
如果前面[适用于 SAP 的 Azure 增强型监视的就绪状态检查][deployment-guide-5.1]一章中所述的测试指出未正确提供某些监视数据，请执行 Test-AzureRmVMAEMExtension 以测试 Azure 监视基础结构和适用于 SAP 的监视扩展的当前配置是否正确。

若要测试监视配置，请执行以下步骤顺序：

* 确保已根据本文档的[部署 Azure PowerShell cmdlet][deployment-guide-4.1] 一章中所述安装了最新版本的 Azure PowerShell cmdlet。
* 运行以下 Azure PowerShell cmdlet。有关可用环境的列表，请运行 cmdlet Get-AzureRmEnvironment。如果想要使用公共 Azure，你的环境将是 AzureCloud。对于中国区 Azure，请选择 AzureChinaCloud。

    $env = Get-AzureRmEnvironment -Name <name of the environment>
    Login-AzureRmAccount -Environment $env
    Set-AzureRmContext -SubscriptionName <subscription name>
    Test-AzureRmVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>

* 在你提供帐户数据和 Azure 虚拟机后，该脚本将测试所选虚拟机的配置。

 
![特定于 SAP 的 Azure cmdlet Test-VMConfigForSAP\_GUI 的输入屏幕][deployment-guide-figure-1200]

在你输入有关帐户的信息和 Azure 虚拟机后，该脚本将测试所选虚拟机的配置。
 
![成功测试适用于 SAP 的 Azure 监视基础结构时的输出][deployment-guide-figure-1300]  


确保每项检查都标记为 OK。如果某些检查的结果不为 ok，请根据本文档的[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述执行 update cmdlet。再次等待 15 分钟，然后重新执行[适用于 SAP 的 Azure 增强型监视的就绪状态检查][deployment-guide-5.1]和 [Azure 监视基础结构配置的运行状况检查][deployment-guide-5.2]章节中所述的检查。如果检查仍然指出部分或所有计数器存在问题，请转到[对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除][deployment-guide-5.3]一章。

### <a name="fe25a7da-4e4e-4388-8907-8abc2d33cfd8"></a>对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除

#### ![Windows][Logo_Windows] Azure 性能计数器根本未显示
Azure 上的性能度量值的收集是由 Windows 服务“AzureEnhancedMonitoring”执行的。如果该服务未正确安装或者未在 VM 中运行，则根本无法收集任何性能度量值。

##### Azure 增强型监视扩展的安装目录为空 

###### 问题
安装目录 C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\`<version`>\\drop 为空。

###### 解决方案
未安装该扩展。请检查是否存在代理问题（根据前面所述）。可能需要重新启动计算机和/或重新运行配置脚本 Set-AzureRmVMAEMExtension

##### 增强型监视的服务不存在 

###### 问题
Windows 服务“AzureEnhancedMonitoring”不存在。 
Azperflib.exe：azperlib.exe 输出中显示了[下图][deployment-guide-figure-14]中所示的错误。
 
![azperflib.exe 的执行结果指出适用于 SAP 的 Azure 增强型监视扩展的服务未运行][deployment-guide-figure-1400]  

###### <a name="figure-14"></a>解决方案
如果该服务如[上图][deployment-guide-figure-14]中所示不存在，则表示未正确安装适用于 SAP 的 Azure 监视扩展。请根据 [Azure 上 SAP 的 VM 部署方案][deployment-guide-3]一章中针对部署方案所述的步骤，重新部署该扩展。

部署该扩展后，请在大约 1 小时后重新检查 Azure VM 中是否提供了 Azure 性能计数器。

##### Azure 增强型监视的服务存在，但无法启动 

###### 问题
Windows 服务“AzureEnhancedMonitoring”存在并已启用，但无法启动。检查应用程序事件日志以了解详细信息。

###### 解决方案
配置错误。根据[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述，为 VM 重新启用监视扩展。

#### ![Windows][Logo_Windows] 缺少某些 Azure 性能计数器
Azure 上的性能度量值的收集是由 Windows 服务“AzureEnhancedMonitoring”执行的，该服务将从多个源中获取数据。某些配置数据是在本地获取的，性能度量值是从 Azure 诊断中读取的，使用的存储计数器来自存储订阅级别的日志记录。

如果借助 SAP 说明 [1999351] 无法排除故障，请重新运行配置脚本 Set-AzureRmVMAEMExtension。可能必须要等待一小时，因为在启用存储分析或诊断计数器后可能不会立即创建这些计数器。如果问题依然存在，请为组件 BC-OP-NT-AZR 创建一条 SAP 客户支持消息。

#### ![Linux][Logo_Linux] Azure 性能计数器根本未显示

Azure 上的性能度量值由一个守护程序收集。如果该守护程序未运行，则根本无法收集性能度量值。

##### Azure 增强型监视扩展的安装目录为空 

###### 问题
/var/lib/waagent/ 目录不包含 Azure 增强型监视扩展的子目录。

###### 解决方案
未安装该扩展。请检查是否存在代理问题（根据前面所述）。可能需要重新启动计算机和/或重新运行配置脚本 Set-AzureRmVMAEMExtension

#### ![Linux][Logo_Linux] 缺少某些 Azure 性能计数器

Azure 上的性能度量值的收集是由一个守护程序执行的，该守护程序将从多个源中获取数据。某些配置数据是在本地获取的，性能度量值是从 Azure 诊断中读取的，使用的存储计数器来自存储订阅级别的日志记录。

有关完整和最新的已知问题列表，请参阅 SAP 说明 [1999351]，其中包含有关适用于 SAP 的增强型 Azure 监视的其他故障排除信息。

如果借助 SAP 说明 [1999351] 无法排除故障，请根据[配置适用于 SAP 的 Azure 增强型监视扩展][deployment-guide-4.5]一章中所述，重新运行配置脚本 Set-AzureRmVMAEMExtension。可能必须要等待一小时，因为在启用存储分析或诊断计数器后可能不会立即创建这些计数器。如果问题依然存在，请为 Windows 虚拟机组件 BC-OP-NT-AZR 或 Linux 虚拟机组件 BC-OP-LNX-AZR 创建一条 SAP 客户支持消息。

<!---HONumber=Mooncake_1010_2016-->