<properties
    pageTitle="在现有群集配置中配置附加的 SAP ASCS/SCS 实例以创建 SAP 多 SID 配置 - Azure Resource Manager | Azure"
    description="Windows 虚拟机上的多 SID 高可用性 SAP NetWeaver 指南"
    services="virtual-machines-windows,virtual-network,storage"
    documentationcenter="saponazure"
    author="goraco"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    keywords="" />
<tags 
    ms.assetid="0b89b4f8-6d6c-45d7-8d20-fe93430217ca"
    ms.service="virtual-machines-windows"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="12/09/2016"
    wacn.date="01/13/2017"
    ms.author="goraco" />

# 在现有群集中配置附加的 SAP ASCS/SCS 实例以创建 SAP 多 SID 配置
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

[sap-installation-guides]: http://service.sap.com/instguides

[azure-cli]: /documentation/articles/xplat-cli-install/
[azure-portal]: https://portal.azure.cn
[azure-ps]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]: https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]: https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits/
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits/

[dbms-guide]: virtual-machines-windows-sap-dbms-guide.md "Windows 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南"
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

[deployment-guide]: virtual-machines-windows-sap-deployment-guide.md "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南"
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
[getting-started-deployment]: /documentation/articles/virtual-machines-windows-sap-get-started/#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]: /documentation/articles/virtual-machines-windows-sap-get-started/#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/
[getting-started-windows-classic-dbms]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]: /documentation/articles/virtual-machines-windows-classic-sap-get-started/#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]: http://go.microsoft.com/fwlink/?LinkId=613056

[ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/

[install-extension-cli]: /documentation/articles/virtual-machines-linux-enable-aem/

[Logo_Linux]: ./media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]: ./media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]: https://msdn.microsoft.com/zh-cn/library/azure/mt670598.aspx

[planning-guide]: virtual-machines-windows-sap-planning-guide.md "Windows 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南"
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
[planning-guide-5.5.1]: virtual-machines-windows-sap-planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 "SAP 部署的 VM/VHD 结构"
[planning-guide-5.5.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#17e0d543-7e8c-4160-a7da-dd7117a1ad9d "为附加的磁盘设置自动装载"
[planning-guide-7.1]: virtual-machines-windows-sap-planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30 "用于 SAP NetWeaver 演示/培训的单一 VM 方案"
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

[sap-ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/ "Windows 虚拟机上的高可用性 SAP NetWeaver"
[sap-ha-guide-1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#217c5479-5595-4cd8-870d-15ab00d4f84c "先决条件"
[sap-ha-guide-2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42b8f600-7ba3-4606-b8a5-53c4f026da08 "资源"
[sap-ha-guide-3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42156640c6-01cf-45a9-b225-4baa678b24f1 "Azure Resource Manager 与经典部署模型中的高可用性 SAP"
[sap-ha-guide-3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f76af273-1993-4d83-b12d-65deeae23686 "资源组"
[sap-ha-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#3e85fbe0-84b1-4892-87af-d9b65ff91860 "Azure Resource Manager 与经典部署模型中的群集"
[sap-ha-guide-4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8ecf3ba0-67c0-4495-9c14-feec1a2255b7 "Windows Server 故障转移群集"
[sap-ha-guide-4.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a3c5408-b168-46d6-99f5-4219ad1b1ff2 "仲裁模式"
[sap-ha-guide-5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fdfee875-6e66-483a-a343-14bbaee33275 "本地 Windows Server 故障转移群集"
[sap-ha-guide-5.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#be21cf3e-fb01-402b-9955-54fbecf66592 "共享存储"
[sap-ha-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ff7a9a06-2bc5-4b20-860a-46cdb44669cd "网络和名称解析"
[sap-ha-guide-6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2ddba413-a7f5-4e4e-9a51-87908879c10a "Azure 中的 Windows Server 故障转移群集"
[sap-ha-guide-6.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a464091-922b-48d7-9d08-7cecf757f341 "使用 SIOS DataKeeper 在 Azure 中创建共享磁盘"
[sap-ha-guide-6.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#44641e18-a94e-431f-95ff-303ab65e0bcb "Azure 中的名称解析"
[sap-ha-guide-7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2e3fec50-241e-441b-8708-0b1864f66dfa "Azure 基础结构即服务 (IaaS) 中的高可用性 SAP NetWeaver"
[sap-ha-guide-7.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#93faa747-907e-440a-b00a-1ae0a89b1c0e "高可用性 SAP 应用程序服务器"
[sap-ha-guide-7.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f559c285-ee68-4eec-add1-f60fe7b978db "高可用性 SAP ASCS/SCS 实例"
[sap-ha-guide-7.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b5b1fd0b-1db4-4d49-9162-de07a0132a51 "在 Azure 中使用 Windows 故障转移群集创建高可用性 SAP ASCS/SCS 实例"
[sap-ha-guide-7.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ddd878a0-9c2f-4b8e-8968-26ce60be1027 "高可用性 DBMS 实例"
[sap-ha-guide-7.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#045252ed-0277-4fc8-8f46-c5a29694a816 "端到端高可用性部署方案"
[sap-ha-guide-8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#78092dbe-165b-454c-92f5-4972bdbef9bf "准备基础结构"
[sap-ha-guide-8.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#c87a8d3f-b1dc-4d2f-b23c-da4b72977489 "部署具有企业网络连接（跨界连接）的虚拟机以便在生产环境中使用"
[sap-ha-guide-8.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310 "用于测试和演示的仅限云 SAP 实例部署"
[sap-ha-guide-8.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#47d5300a-a830-41d4-83dd-1a0d1ffdbe6a "Azure 虚拟网络"
[sap-ha-guide-8.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b22d7b3b-4343-40ff-a319-097e13f62f9e "DNS IP 地址"
[sap-ha-guide-8.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#9fbd43c0-5850-4965-9726-2a921d85d73f "SAP ASCS/SCS 群集实例和 DBMS 群集实例的主机名与静态 IP 地址"
[sap-ha-guide-8.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#84c019fe-8c58-4dac-9e54-173efd4b2c30 "为 SAP 虚拟机设置静态 IP 地址"
[sap-ha-guide-8.7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7a8f3e9b-0624-4051-9e41-b73fff816a9e "为内部负载均衡器创建静态 IP 地址"
[sap-ha-guide-8.8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f19bd997-154d-4583-a46e-7f5a69d0153c "Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则"
[sap-ha-guide-8.9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fe0bd8b5-2b43-45e3-8295-80bee5415716 "更改 Azure 内部负载均衡器的默认 ASCS/SCS 负载均衡规则"
[sap-ha-guide-8.10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e69e9a34-4601-47a3-a41c-d2e11c626c0c "将 Windows 虚拟机添加到域"
[sap-ha-guide-8.11]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#661035b2-4d0f-4d31-86f8-dc0a50d78158 "在 SAP ASCS/SCS 实例的两个群集节点上添加注册表项"
[sap-ha-guide-8.12]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0d67f090-7928-43e0-8772-5ccbf8f59aab "为 SAP ASCS/SCS 实例设置 Windows Server 故障转移群集"
[sap-ha-guide-8.12.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79 "收集群集配置中的群集节点"
[sap-ha-guide-8.12.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e49a4529-50c9-4dcf-bde7-15a0c21d21ca "配置群集文件共享见证"
[sap-ha-guide-8.12.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#06260b30-d697-4c4d-b1c9-d22c0bd64855 "创建文件共享"
[sap-ha-guide-8.12.2.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#4c08c387-78a0-46b1-9d27-b497b08cac3d "在故障转移群集管理器中设置文件共享见证仲裁"
[sap-ha-guide-8.12.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5c8e5482-841e-45e1-a89d-a05c0907c868 "为 SAP ASCS/SCS 群集共享磁盘安装 SIOS DataKeeper Cluster Edition"
[sap-ha-guide-8.12.3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1c2788c3-3648-4e82-9e0d-e058e475e2a3 "添加 .NET Framework 3.5"
[sap-ha-guide-8.12.3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#dd41d5a2-8083-415b-9878-839652812102 "安装 SIOS DataKeeper"
[sap-ha-guide-8.12.3.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#d9c1fc8e-8710-4dff-bec2-1f535db7b006 "设置 SIOS DataKeeper"
[sap-ha-guide-9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a06f0b49-8a7a-42bf-8b0d-c12026c5746b "安装 SAP NetWeaver 系统"
[sap-ha-guide-9.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#31c6bd4f-51df-4057-9fdf-3fcbc619c170 "安装包含高可用性 ASCS/SCS 实例的 SAP 系统"
[sap-ha-guide-9.1.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a97ad604-9094-44fe-a364-f89cb39bf097 "为 SAP ASCS/SCS 群集实例创建虚拟主机名"
[sap-ha-guide-9.1.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#eb5af918-b42f-4803-bb50-eff41f84b0b0 "安装 SAP 的第一个群集节点"
[sap-ha-guide-9.1.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 "修改 ASCS/SCS 实例的 SAP 配置文件"
[sap-ha-guide-9.1.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#10822f4f-32e7-4871-b63a-9b86c76ce761 "添加探测端口"
[sap-ha-guide-9.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#85d78414-b21d-4097-92b6-34d8bcb724b7 "安装数据库实例"
[sap-ha-guide-9.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8a276e16-f507-4071-b829-cdc0a4d36748 "安装第二个群集节点"
[sap-ha-guide-9.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#094bc895-31d4-4471-91cc-1513b64e406a "更改 SAP ERS Windows 服务实例的启动类型"
[sap-ha-guide-9.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 "安装 SAP 主应用程序服务器"
[sap-ha-guide-9.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 "安装 SAP 附加应用程序服务器"
[sap-ha-guide-10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9 "测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制"
[sap-ha-guide-10.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#65fdef0f-9f94-41f9-b314-ea45bbfea445 "SAP ASCS/SCS 实例在群集节点 A 上运行"
[sap-ha-guide-10.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5e959fa9-8fcd-49e5-a12c-37f6ba07b916 "从节点 A 故障转移到节点 B"
[sap-ha-guide-10.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#755a6b93-0099-4533-9f6d-5c9a613878b5 "SAP ASCS/SCS 实例在群集节点 B 上运行"


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

[sap-ha-guide-figure-6001]: ./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]: ./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]: ./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]: ./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]: ./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]: ./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[resource-group-authoring-templates]: /documentation/articles/resource-group-authoring-templates/
[resource-group-overview]: /documentation/articles/resource-group-overview/
[resource-groups-networking]: /documentation/articles/resource-groups-networking/
[networking-limits-azure-resource-manager]: /documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits
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
[virtual-machines-azure-resource-manager-architecture]: /documentation/articles/resource-group-overview/
[virtual-machines-azure-resource-manager-architecture-benefits-arm]: /documentation/articles/resource-group-overview/#the-benefits-of-using-resource-manager
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

[load-balancer-multivip-overview]: /documentation/articles/load-balancer-multivip-overview/

[vpn-gateway-about-vpn-devices]: /documentation/articles/vpn-gateway-about-vpn-devices/
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/
[vpn-gateway-cross-premises-options]: /documentation/articles/vpn-gateway-plan-design/
[vpn-gateway-site-to-site-create]: /documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/
[vpn-gateway-vpn-faq]: /documentation/articles/vpn-gateway-vpn-faq/
[xplat-cli]: /documentation/articles/xplat-cli-install/
[xplat-cli-azure-resource-manager]: /documentation/articles/xplat-cli-azure-resource-manager/


在 9 月份，Microsoft 推出了一项可用于管理 [Azure 内部负载均衡器的多个虚拟 IP 地址][load-balancer-multivip-overview]的功能。Azure 外部负载均衡器已有类似的功能。

在 SAP 部署中，Azure ILB 用于创建 SAP ASCS/SCS 的 Windows 群集配置，如主要文档 [High-availability SAP NetWeaver on Windows virtual machines guide][sap-ha-guide]（Windows 虚拟机上的高可用性 SAP NetWeaver 指南）中所述。

本文重点介绍如何根据该文档中的现有内容，通过在现有 Windows 故障转移服务器群集 (WSFC) 中安装附加的 SAP ASCS/SCS 群集实例，从此类单一 ASCS/SCS 安装转移到 SAP 多 SID 配置。这样，便可以配置所谓的 SAP 多 SID 群集。

> [AZURE.NOTE]
此功能仅在 Azure Resource Manager 部署模型中可用。
>
>

## 先决条件
已根据以下文档配置了用于一个 SAP ASCS/SCS 实例的 WSFC：[High-availability SAP NetWeaver on Windows virtual machines guide][sap-ha-guide]（Windows 虚拟机上的高可用性 SAP NetWeaver 指南）。

![图 1：高可用性 SAP ASCS/SCS 实例][sap-ha-guide-figure-6001]  


_**图 1：**Azure 中有一个 SAP ASCS/SCS 群集实例_


## 目标体系结构

目标是能够在同一个 WSAFC 群集中安装多个 **SAP ABAP ASCS** 或 **SAP Java SCS** 群集实例。

![图 2：Azure 中有多个 SAP ASCS/SCS 群集实例][sap-ha-guide-figure-6002]  


_**图 2：**Azure 中有多个 SAP ASCS/SCS 群集实例_

> [AZURE.NOTE]
**每个 Azure 内部负载均衡器的专用前端 IP** 的最大数目有限制。
>
>这意味着，**一个 WSFC 群集中的最大 SAP ASCS/SCS 实例数**等于**每个 Azure 内部负载均衡器的最大专用前端 IP 数**。
>

有关负载均衡器限制的信息，请参阅[网络限制 - Azure Resource Manager][networking-limits-azure-resource-manager] 下面的**每个负载均衡器的专用前端 IP**

包含两个高可用性 SAP 系统的完整布局的大体形势如下所示：

![图 3：包含两个 SAP 系统的 SAP 高可用性多 SID 设置][sap-ha-guide-figure-6003]  


_**图 3：**包含两个 SAP 系统的 SAP 高可用性多 SID 设置_

> [AZURE.IMPORTANT]
要点如下：
> - **SAP ASCS/SCS** 实例**共享同一个 WSFC 群集**。
> - 每个 **DBMS SID** 具有**自身的专用 WSFC 群集**。
>- 属于一个 SAP 系统 SID 的 **SAP 应用程序服务器**具有**自身的专用 VM**。
>

## 准备基础结构
假设要使用以下参数安装**附加的** SAP ASCS/SCS 实例：

| 参数名称 | 值 |
| --- | --- |
|SAP ASC/SCS SID |pr1-lb-ascs |
| SAP DBMS 内部负载均衡器 | PR5 |
| SAP 虚拟主机名 | pr5-sap-cl |
| SAP ASCS/SCS 虚拟主机 IP 地址（附加的 Azure 负载均衡 IP 地址） | 10\.0.0.50 |
| SAP ASCS/SCS 实例编号 | 50 |
| 附加 SAP ASCS/SCS 实例的 ILB 探测端口 | 62350 |

> [AZURE.NOTE]
对于 SAP ASCS/SCS 群集实例，每个 IP 地址需要特定的探测端口。例如，如果 Azure 内部负载均衡器上有一个 IP 地址使用探测端口 62300，该负载均衡器上的其他任何 IP 地址就不能使用探测端口 62300。
>
>在本例中，探测端口 62300 已被保留，因此我们使用另一个探测端口 62350。
>

我们要在**现有**的 WSFC 群集中安装包含两个节点的附加 SAP ASCS/SCS 实例：

| 虚拟机角色 | 虚拟机主机名 | 静态 IP 地址 |
| --- | --- | --- |
| ASCS/SCS 实例的第 1 个群集节点 |pr1-ascs-0 |10\.0.0.10 |
| ASCS/SCS 实例的第 2 个群集节点 |pr1-ascs-1 |10\.0.0.9 |

### 在 DNS 服务器上创建 SAP ASCS/SCS 群集实例的虚拟主机名

使用以下参数创建 ASCS/SCS 实例虚拟主机名的 DNS 项：

| 新的 SAP ASCS/SCS 虚拟主机名 | 关联的 IP 地址 |
| --- | --- | --- |
|pr5-sap-cl |10\.0.0.50 |

![图 4：定义附加的新 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 项][sap-ha-guide-figure-6004]  


_**图 4：**定义附加的新 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的 DNS 项_

主要文档 [SAP NetWeaver on Windows virtual machines (VMs) - High-Availability Guide][sap-ha-guide-9.1.1]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南）中也详细介绍了如何创建 DNS 项。

> [AZURE.NOTE]
请记住，分配给附加 ASCS/SCS 实例虚拟主机名的新 IP 地址必须与分配给 SAP Azure Load Balancer 的新 IP 地址相同。
>
>在本例中，该 IP 地址为 10.0.0.50。

### 使用 PowerShell 将 IP 地址添加到现有 Azure 内部负载均衡器

若要在同一个 WSFC 群集中使用多个 SAP ASCS/SCS 实例，请使用 PowerShell 将附加的 IP 地址添加到现有的 Azure 内部负载均衡器。每个 IP 地址需要有自身的负载均衡规则、探测端口、前端 IP 池和后端池。

以下脚本将新的 IP 地址添加到现有负载均衡器。更新环境的 PowerShell 变量。该脚本将为所有 SAP ASCS/SCS 端口创建全部所需的负载均衡规则。

    # Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    # Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
    Clear-Host
    $ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing Resource group name
    $VNetName = "pr2-vnet"                        # Existing Virtual network name
    $SubnetName = "Subnet"                        # Existing Subnet name
    $ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
    $ILBIP = "10.0.0.50"                          # New IP address
    $VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster Virtual machine names
    $SAPInstanceNumber = 50                       # SAP ASCS/SCS Instance Number - must be unique value per cluster
    [int]$ProbePort = "623$SAPInstanceNumber"     # Probe port - MUST be unique value per IP and load balancer

    $ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

    $count = $ILB.FrontendIpConfigurations.Count + 1
    $FrontEndConfigurationName ="lbFrontendASCS$count"
    $LBProbeName = "lbProbeASCS$count"

    # Get the Azure VNet and Subnet
    $VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
    $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

    # Add Second Frontend and Probe config
    Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
    $ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
    $ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzureRmLoadBalancer

    #Get new updated config
    $ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
    # Get new updated LP FrontendIP COnfig
    $FEConfig = Get-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
    $HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

    # Add new backend config into existign ILB
    $BackEndConfigurationName  = "backendPoolASCS$count"
    Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
    $BEConfig = Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzureRmLoadBalancer



    #Get new updated config
    $ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

    # Assign VM NICs to backend pool
    $BEPool = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
    foreach($VMName in $VMNames){
            $VM = Get-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VMName
            $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
            $NIC = Get-AzureRmNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
            $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
            Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
            Set-AzureRmNetworkInterface -NetworkInterface $NIC
            #start-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
    }


    # Create Load Balancing Rules
    $Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
    $ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
    $FEConfig = get-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
    $BEConfig = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
    $HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

    Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

    foreach ($Port in $Ports) {

            $LBConfigrulename = "lbrule$Port" + "_$count"
            Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

            $ILB | Add-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
    }

    $ILB | Set-AzureRmLoadBalancer

    Write-Host "Succesfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green



运行完脚本后，可在 Azure 门户预览中看到结果。

![图 5：Azure 端口中的新前端 IP 池][sap-ha-guide-figure-6005]  


_**图 5：**Azure 端口中的新前端 IP 池_

### 将附加磁盘添加到群集计算机并配置 SIOS 群集共享磁盘

需要使用一个新的群集共享磁盘来存储新的附加 SAP ASCS/SCS 实例。在 Windows Server 2012 R2 中，WSFC 群集共享磁盘目前由 SIOS DataKeeper 软件解决方案使用。

因此：
-	请将相同大小的附加磁盘（或其他要条带化的磁盘）添加到每个群集节点，然后将其格式化。
-	使用 SIOS DataKeeper 配置存储复制。

假设已在 WSFC 群集计算机上安装 SIOS DataKeeper，因此只需在两者之间配置复制即可。

主要文档 [SAP NetWeaver on Windows virtual machines (VMs) - High-Availability Guide][sap-ha-guide-8.12.3.3]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南）中对此做了详细介绍。

![图 6：新的 SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态][sap-ha-guide-figure-6006]  


_**图 6：**新的 SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像处于活动状态_

### 为 SAP 应用程序服务器和 DBMS 群集部署 VM

若要完成第二个 SAP 系统的基础结构准备，需要：

- 为 SAP 应用程序服务器部署专用 VM，并将这些 VM 放在其自身的专用可用性组中
- 为 DBMS 群集部署专用 VM，并将这些 VM 放在其自身的专用可用性组中


## 安装第二个 SAP SID2 NetWeaver 系统

现在，可以安装第二个 SAP **SID2** 系统。

主要文档 [SAP NetWeaver on Windows virtual machines (VMs) - High-Availability Guide][sap-ha-guide-9]（Windows 虚拟机 (VM) 上的 SAP NetWeaver - 高可用性指南）中介绍了整个过程。

下面说明了概要过程：

- [安装 SAP 的第一个群集节点][sap-ha-guide-9.1.2]

    在**现有 WSFC 群集节点 1** 上安装包含高可用性 ASCS/SCS 实例的 SAP 系统。

- [修改 ASCS/SCS 实例的 SAP 配置文件][sap-ha-guide-9.1.3]

- [配置探测端口][sap-ha-guide-9.1.4]

    使用 PowerShell 配置 SAP 群集资源 **SAP-SID2-IP** 探测端口。在其中一个 SAP ASCS/SCS 群集节点上执行此操作。

- [安装数据库实例][sap-ha-guide-9.2]

    在**专用 WSFC 群集**上安装 DBMS。

- [安装第二个群集节点][sap-ha-guide-9.3]

    在**现有 WSFC 群集节点 2** 上安装包含高可用性 ASCS/SCS 实例的 SAP 系统。

- **打开 SAP ASCS/SCS 实例的 Windows 防火墙端口和探测端口**

    在用于 SAP ASCS/SCS 实例的两个群集节点上，打开 SAP ASCS/SCS 端口使用的所有 Windows 防火墙端口。[此处][sap-ha-guide-8.8]列出了这些端口。

    此外，打开 Azure 内部负载均衡器探测端口，在本例中为 62350。

- [更改 SAP ERS Windows 服务实例的启动类型][sap-ha-guide-9.4]

- [安装 SAP 主应用程序服务器][sap-ha-guide-9.5]

    在**新的专用 VM** 上安装 SAP 主应用程序服务器。

- [安装 SAP 附加应用程序服务器][sap-ha-guide-9.6]

    在**新的专用 VM** 上安装 SAP SID2 附加应用程序服务器。

- [测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制][sap-ha-guide-10]

## 后续步骤

- [网络限制 - Azure 资源管理器][networking-limits-azure-resource-manager]
- [Azure Load Balancer 的多个 VIP][load-balancer-multivip-overview]
- [Windows 虚拟机上的高可用性 SAP NetWeaver 主要指南][sap-ha-guide]

<!---HONumber=Mooncake_0109_2017-->