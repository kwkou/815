<!-- ARM -->

<properties
   pageTitle="在 Linux 虚拟机 (VM) 上使用 SAP | Azure"
   description="了解如何在 Azure 中的 Linux 虚拟机 (VM) 上使用 SAP"
   services="virtual-machines-linux,virtual-network,storage"
   documentationCenter="saponazure"
   authors="MSSedusch"
   manager="juergent"
   editor=""
   tags="azure-resource-manager"
   keywords=""/>
<tags
	ms.service="virtual-machines-linux"
	ms.date="05/17/2016"
	wacn.date=""/>

# 在 Linux 虚拟机 (VM) 上使用 SAP

[767598]: https://service.sap.com/sap/support/notes/767598
[773830]: https://service.sap.com/sap/support/notes/773830
[826037]: https://service.sap.com/sap/support/notes/826037
[965908]: https://service.sap.com/sap/support/notes/965908
[1139904]: https://service.sap.com/sap/support/notes/1139904
[1173395]: https://service.sap.com/sap/support/notes/1173395
[1245200]: https://service.sap.com/sap/support/notes/1245200
[1409604]: https://service.sap.com/sap/support/notes/1409604
[1558958]: https://service.sap.com/sap/support/notes/1558958
[1585981]: https://service.sap.com/sap/support/notes/1585981
[1588316]: https://service.sap.com/sap/support/notes/1588316
[1590719]: https://service.sap.com/sap/support/notes/1590719
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
[1999351]: https://service.sap.com/sap/support/notes/1999351
[2015553]: https://service.sap.com/sap/support/notes/2015553
[2039619]: https://service.sap.com/sap/support/notes/2039619
[2121797]: https://service.sap.com/sap/support/notes/2121797
[2134316]: https://service.sap.com/sap/support/notes/2134316
[2178632]: https://service.sap.com/sap/support/notes/2178632
[2191498]: https://service.sap.com/sap/support/notes/2191498
[2233094]: https://service.sap.com/sap/support/notes/2233094

[azure-portal]: https://portal.azure.cn
[azure-script-ps]: https://go.microsoft.com/fwlink/p/?LinkID=395017

[planning-guide]: /documentation/articles/virtual-machines-linux-sap-planning-guide
[planning-guide-classic]: /documentation/articles/virtual-machines-windows-classic-sap-planning-guide
[deployment-guide]: /documentation/articles/virtual-machines-linux-sap-deployment-guide
[deployment-guide-classic]: /documentation/articles/virtual-machines-windows-classic-sap-deployment-guide
[dbms-guide]: /documentation/articles/virtual-machines-linux-sap-dbms-guide.md
[dbms-guide-classic]: /documentation/articles/virtual-machines-windows-classic-sap-dbms-guide
[dr-guide-classic]: /documentation/articles/virtual-machines-windows-classic-sap-dr-guide
[ha-guide-classic]: /documentation/articles/virtual-machines-windows-classic-sap-ha-guide

[getting-started]: /documentation/articles/virtual-machines-linux-sap-getting-started-arm
[getting-started-windows-classic]: /documentation/articles/virtual-machines-windows-classic-sap-getting-started

[getting-started-windows-classic-dbms]: /documentation/articles/virtual-machines-windows-classic-sap-getting-started#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-planning]: virtual-machines-windows-classic-sap-getting-started.md#f2a5e9d8-49e4-419e-9900-af783173481c
[getting-started-windows-classic-deployment]: virtual-machines-windows-classic-sap-getting-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]: virtual-machines-windows-classic-sap-getting-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]: virtual-machines-windows-classic-sap-getting-started.md#4bb7512c-0fa0-4227-9853-4004281b1037

[getting-started-planning]: virtual-machines-linux-sap-getting-started-arm.md#3da0389e-708b-4e82-b2a2-e92f132df89c
[getting-started-deployment]: virtual-machines-linux-sap-getting-started-arm.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-dbms]: virtual-machines-linux-sap-getting-started-arm.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82

[deployment-guide-2.2]: virtual-machines-linux-sap-deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 "SAP 资源"
[deployment-guide-3]: virtual-machines-linux-sap-deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e "Azure 上 SAP 的 VM 部署方案"
[deployment-guide-3.1.2]: virtual-machines-linux-sap-deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab "使用自定义映像部署 VM"
[deployment-guide-3.2]: virtual-machines-linux-sap-deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 "方案 1：为 SAP 部署从 Azure 库中取出的 VM"
[deployment-guide-3.3]: virtual-machines-linux-sap-deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 "方案 2：使用自定义映像为 SAP 部署 VM"
[deployment-guide-3.4]: virtual-machines-linux-sap-deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 "方案 3：使用包含 SAP 的非通用化 Azure VHD 从本地移动 VM"
[deployment-guide-4.1]: virtual-machines-linux-sap-deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 "部署 Azure PowerShell cmdlet"
[deployment-guide-4.2]: virtual-machines-linux-sap-deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e "下载并导入 SAP 相关的 PowerShell cmdlet"
[deployment-guide-4.3]: virtual-machines-linux-sap-deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc "将 VM 加入本地域 - 仅限 Windows"
[deployment-guide-4.4]: virtual-machines-linux-sap-deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d "下载、安装并启用 Azure VM 代理"
[deployment-guide-4.4.2]: virtual-machines-linux-sap-deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 "Linux"
[deployment-guide-4.5]: virtual-machines-linux-sap-deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca "配置适用于 SAP 的 Azure 增强型监视扩展"
[deployment-guide-4.5.1]: virtual-machines-linux-sap-deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 "Azure PowerShell"
[deployment-guide-4.5.2]: virtual-machines-linux-sap-deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f "Azure CLI"
[deployment-guide-5.1]: virtual-machines-linux-sap-deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 "适用于 SAP 的 Azure 增强型监视的就绪状态检查"
[deployment-guide-5.2]: virtual-machines-linux-sap-deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 "Azure 监视基础结构配置的运行状况检查"
[deployment-guide-5.3]: virtual-machines-linux-sap-deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 "对适用于 SAP 的 Azure 监视基础结构进一步执行故障排除"
[deployment-guide-install-vm-agent-windows]: virtual-machines-linux-sap-deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-configure-proxy]: virtual-machines-linux-sap-deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d "配置代理"
[deployment-guide-configure-monitoring-scenario-1]: virtual-machines-linux-sap-deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b "配置监视"
[deployment-guide-troubleshooting-chapter]: virtual-machines-linux-sap-deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b "Azure 上 SAP 的端到端监视设置的检查和故障排除"
[deployment-guide-figure-100]: ./media/virtual-machines-linux-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-300]: ./media/virtual-machines-linux-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]: ./media/virtual-machines-linux-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-500]: ./media/virtual-machines-linux-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-600]: ./media/virtual-machines-linux-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-700]: ./media/virtual-machines-linux-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]: ./media/virtual-machines-linux-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-50]: ./media/virtual-machines-linux-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-900]: ./media/virtual-machines-linux-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-1000]: ./media/virtual-machines-linux-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-1100]: ./media/virtual-machines-linux-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]: ./media/virtual-machines-linux-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]: ./media/virtual-machines-linux-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-1400]: ./media/virtual-machines-linux-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-5]: /documentation/articles/virtual-machines-linux-sap-deployment-guide#figure-5
[deployment-guide-figure-6]: virtual-machines-linux-sap-deployment-guide.md#figure-6
[deployment-guide-figure-7]: virtual-machines-linux-sap-deployment-guide.md#figure-7
[deployment-guide-figure-11]: virtual-machines-linux-sap-deployment-guide.md#figure-11
[deployment-guide-figure-14]: virtual-machines-linux-sap-deployment-guide.md#figure-14
[deployment-guide-figure-azure-cli-installed]: virtual-machines-linux-sap-deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]: virtual-machines-linux-sap-deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda

[planning-guide-1.2]: virtual-machines-linux-sap-planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff "资源"
[planning-guide-2.1]: virtual-machines-linux-sap-planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 "仅限云 - 在不将依赖项部署到本地客户网络的情况下将虚拟机部署到 Azure 中"
[planning-guide-2.2]: virtual-machines-linux-sap-planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 "跨界 - 将单个或多个 SAP VM 部署到 Azure 中并要求完全集成到本地网络"
[planning-guide-3.1]: virtual-machines-linux-sap-planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a "Azure 区域"
[planning-guide-3.2]: virtual-machines-linux-sap-planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 "Azure 虚拟机的概念"
[planning-guide-3.2.1]: virtual-machines-linux-sap-planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 "容错域"
[planning-guide-3.2.2]: virtual-machines-linux-sap-planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 "升级域"
[planning-guide-3.2.3]: virtual-machines-linux-sap-planning-guide.md#18810088-f9be-4c97-958a-27996255c665 "Azure 可用性集"
[planning-guide-3.3.2]: virtual-machines-linux-sap-planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"
[planning-guide-5.1.1]: virtual-machines-linux-sap-planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 "使用非通用化磁盘将 VM 从本地移至 Azure"
[planning-guide-5.1.2]: virtual-machines-linux-sap-planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c "使用特定于客户的映像部署 VM"
[planning-guide-5.2]: virtual-machines-linux-sap-planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 "为 Azure 准备包含 SAP 的 VM"
[planning-guide-5.2.1]: virtual-machines-linux-sap-planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef "准备使用非通用化磁盘将 VM 从本地移到 Azure"
[planning-guide-5.2.2]: virtual-machines-linux-sap-planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 "准备使用特定于客户的映像为 SAP 部署 VM"
[planning-guide-5.3.1]: virtual-machines-linux-sap-planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 "Azure 磁盘与 Azure 映像之间的差异"
[planning-guide-5.3.2]: virtual-machines-linux-sap-planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a "将 VHD 从本地上载到 Azure"
[planning-guide-5.4.2]: virtual-machines-linux-sap-planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 "在 Azure 存储帐户之间复制磁盘"
[planning-guide-5.5.1]: virtual-machines-linux-sap-planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 "SAP 部署的 VM/VHD 结构"
[planning-guide-5.5.3]: /documentation/articles/virtual-machines-linux-sap-planning-guide#17e0d543-7e8c-4160-a7da-dd7117a1ad9d "为附加的磁盘设置自动装载"
[planning-guide-7]: virtual-machines-linux-sap-planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14 "SAP 实例的仅限云部署的概念"
[planning-guide-7.1]: virtual-machines-linux-sap-planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30 "用于 SAP NetWeaver 演示/培训的单一 VM 方案"
[planning-guide-9.1]: /documentation/articles/virtual-machines-linux-sap-planning-guide#6f0a47f3-a289-4090-a053-2521618a28c3 "适用于 SAP 的 Azure 监视解决方案"
[planning-guide-11.4.1]: virtual-machines-linux-sap-planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 "SAP 应用程序服务器的高可用性"
[planning-guide-11.5]: virtual-machines-linux-sap-planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f "对 SAP 实例使用 Autostart"
[planning-guide-microsoft-azure-networking]: virtual-machines-linux-sap-planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd "Azure 网络"
[planning-guide-storage-microsoft-azure-storage-and-data-disks]: virtual-machines-linux-sap-planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f "存储：Azure 存储空间和数据磁盘"
[planning-guide-azure-premium-storage]: virtual-machines-linux-sap-planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 "Azure 高级存储"
[planning-guide-figure-100]: ./media/virtual-machines-linux-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-200]: ./media/virtual-machines-linux-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-300]: ./media/virtual-machines-linux-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-400]: ./media/virtual-machines-linux-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]: ./media/virtual-machines-linux-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]: ./media/virtual-machines-linux-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]: ./media/virtual-machines-linux-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-figure-1300]: ./media/virtual-machines-linux-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]: ./media/virtual-machines-linux-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]: ./media/virtual-machines-linux-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]: ./media/virtual-machines-linux-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]: ./media/virtual-machines-linux-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-2100]: ./media/virtual-machines-linux-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]: ./media/virtual-machines-linux-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]: ./media/virtual-machines-linux-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]: ./media/virtual-machines-linux-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]: ./media/virtual-machines-linux-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]: ./media/virtual-machines-linux-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]: ./media/virtual-machines-linux-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]: ./media/virtual-machines-linux-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]: ./media/virtual-machines-linux-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-3000]: ./media/virtual-machines-linux-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]: ./media/virtual-machines-linux-sap-planning-guide/3200-sap-ha-with-sql.png

[dbms-guide-2]: /documentation/articles/virtual-machines-linux-sap-dbms-guide#65fa79d6-a85f-47ee-890b-22e794f51a64 "RDBMS 部署的结构"
[dbms-guide-2.1]: virtual-machines-linux-sap-dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f "VM 和 VHD 的缓存"
[dbms-guide-2.2]: virtual-machines-linux-sap-dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 "软件 RAID"
[dbms-guide-2.3]: virtual-machines-linux-sap-dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 "Azure 存储空间"
[dbms-guide-3]: virtual-machines-linux-sap-dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 "Azure VM 的高可用性和灾难恢复"
[dbms-guide-5]: virtual-machines-linux-sap-dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 "有关 SQL Server RDBMS 的具体信息"
[dbms-guide-5.5.1]: virtual-machines-linux-sap-dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 "SQL Server 2012 SP1 CU4 和更高版本"
[dbms-guide-5.5.2]: virtual-machines-linux-sap-dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b "SQL Server 2012 SP1 CU3 和更低版本"
[dbms-guide-5.6]: virtual-machines-linux-sap-dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 "使用从 Azure 应用商店取出的 SQL Server 映像"
[dbms-guide-5.8]: virtual-machines-linux-sap-dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 "适用于 Azure 上的 SAP 的 SQL Server 总体摘要"
[dbms-guide-8.4.1]: virtual-machines-linux-sap-dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 "存储配置"
[dbms-guide-8.4.2]: virtual-machines-linux-sap-dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d "备份和还原"
[dbms-guide-8.4.3]: virtual-machines-linux-sap-dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c "备份和还原性能注意事项"
[dbms-guide-8.4.4]: virtual-machines-linux-sap-dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 "其他"
[dbms-guide-figure-100]: ./media/virtual-machines-linux-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]: ./media/virtual-machines-linux-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]: ./media/virtual-machines-linux-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]: ./media/virtual-machines-linux-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]: ./media/virtual-machines-linux-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]: ./media/virtual-machines-linux-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]: ./media/virtual-machines-linux-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]: ./media/virtual-machines-linux-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]: ./media/virtual-machines-linux-sap-dbms-guide/900-sap-cache-server-on-premises.png

[dbms-guide-900-sap-cache-server-on-premises]: /documentation/articles/virtual-machines-linux-sap-dbms-guide#642f746c-e4d4-489d-bf63-73e80177a0a8

[Logo_Windows]: ./media/virtual-machines-linux-sap-shared/Windows.png
[Logo_Linux]: ./media/virtual-machines-linux-sap-shared/Linux.png

[vm-size-specs]: /documentation/articles/virtual-machines-size-specs
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits#subscription
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell
[vpn-gateway-cross-premises-options]: /documentation/articles/vpn-gateway-cross-premises-options
[vpn-gateway-site-to-site-create]: /documentation/articles/vpn-gateway-site-to-site-create
[virtual-machines-deploy-rmtemplates-azure-cli]: virtual-machines-deploy-rmtemplates-azure-cli.md "使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机"
[virtual-machines-deploy-rmtemplates-powershell]: virtual-machines-deploy-rmtemplates-powershell.md "使用 Azure 资源管理器与 PowerShell 来管理虚拟机"
[virtual-machines-linux-capture-image-resource-manager]: /documentation/articles/virtual-machines-linux-capture-image-resource-manager
[virtual-machines-manage-availability]: /documentation/articles/virtual-machines-manage-availability
[virtual-machines-linux-how-to-attach-disk]: /documentation/articles/virtual-machines-linux-how-to-attach-disk
[virtual-networks-reserved-private-ip]: /documentation/articles/virtual-networks-static-private-ip-arm-ps
[virtual-machines-sql-server-infrastructure-services]: /documentation/articles/virtual-machines-sql-server-infrastructure-services
[storage-redundancy]: /documentation/articles/storage-redundancy
[storage-scalability-targets]: /documentation/articles/storage-scalability-targets
[virtual-networks-manage-dns-in-vnet]: /documentation/articles/virtual-networks-manage-dns-in-vnet
[resource-groups-networking]: /documentation/articles/resource-groups-networking
[virtual-networks-static-private-ip-arm-pportal]: /documentation/articles/virtual-networks-static-private-ip-arm-pportal
[virtual-networks-multiple-nics]: /documentation/articles/virtual-networks-multiple-nics
[virtual-network-deploy-multinic-arm-template]: /documentation/articles/virtual-network-deploy-multinic-arm-template
[virtual-network-deploy-multinic-arm-ps]: /documentation/articles/virtual-network-deploy-multinic-arm-ps
[virtual-network-deploy-multinic-arm-cli]: /documentation/articles/virtual-network-deploy-multinic-arm-cli
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell
[vpn-gateway-about-vpn-devices]: /documentation/articles/vpn-gateway-about-vpn-devices
[vpn-gateway-vpn-faq]: /documentation/articles/vpn-gateway-vpn-faq
[vpn-gateway-create-site-to-site-rm-powershell]: /documentation/articles/vpn-gateway-create-site-to-site-rm-powershell
[powershell-install-configure]: /documentation/articles/powershell-install-configure
[xplat-cli]: /documentation/articles/xplat-cli-install
[virtual-machines-deploy-rmtemplates-azure-cli]: /documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli
[xplat-cli-azure-resource-manager]: /documentation/articles/xplat-cli-azure-resource-manager
[virtual-machines-linux-create-upload-vhd-suse]: /documentation/articles/virtual-machines-linux-create-upload-vhd-suse
[virtual-machines-linux-capture-image-resource-manager]: /documentation/articles/virtual-machines-linux-capture-image-resource-manager
[storage-use-azcopy]: /documentation/articles/storage-use-azcopy
[virtual-machines-linux-capture-image-resource-manager-capture]: /documentation/articles/virtual-machines-linux-capture-image-resource-manager#capture-the-vm
[storage-azure-cli]: /documentation/articles/storage-azure-cli
[storage-powershell-guide-full-copy-vhd]: /documentation/articles/storage-powershell-guide-full#how-to-copy-blobs-from-one-storage-container-to-another
[storage-azure-cli-copy-blobs]: storage-azure-cli.md#copy-blobs
[virtual-machines-linux-agent-user-guide]: /documentation/articles/virtual-machines-linux-agent-user-guide
[virtual-machines-size-specs]: /documentation/articles/virtual-machines-size-specs
[virtual-machines-sql-server-performance-best-practices]: /documentation/articles/virtual-machines-sql-server-performance-best-practices
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]: /documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions
[virtual-machines-sql-server-alwayson-availability-groups-powershell]: /documentation/articles/virtual-machines-sql-server-alwayson-availability-groups-powershell
[virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener]: /documentation/articles/virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener
[virtual-networks-configure-vnet-to-vnet-connection]: /documentation/articles/virtual-networks-configure-vnet-to-vnet-connection
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits
[virtual-machines-configuring-oracle-data-guard]: /documentation/articles/virtual-machines-configuring-oracle-data-guard
[virtual-machines-linux-configure-raid]: /documentation/articles/virtual-machines-linux-configure-raid
[virtual-machines-attach-disk-preview]: /documentation/articles/virtual-machines-attach-disk-preview
[virtual-machines-workload-template-sql-alwayson]: /documentation/articles/virtual-machines-workload-template-sql-alwayson
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]: /documentation/articles/virtual-machines-linux-how-to-attach-disk#how-to-initialize-a-new-data-disk-in-linux
[resource-group-authoring-templates]: /documentation/articles/resource-group-authoring-templates
[virtual-machines-linux-update-agent]: /documentation/articles/virtual-machines-linux-update-agent
[virtual-machines-linux-create-upload-vhd-step-1]: /documentation/articles/virtual-machines-linux-create-upload-vhd#step-1-prepare-the-image-to-be-uploaded
[deploy-template-powershell]: resource-group-template-deploy.md#deploy-with-powershell
[deploy-template-cli]: resource-group-template-deploy.md#deploy-with-azure-cli-for-mac-linux-and-windows
[deploy-template-portal]: resource-group-template-deploy.md#deploy-with-the-preview-portal
[virtual-networks-udr-overview]: /documentation/articles/virtual-networks-udr-overview
[resource-group-overview]: /documentation/articles/resource-group-overview
[virtual-machines-linux-agent-user-guide-command-line-options]: /documentation/articles/virtual-machines-linux-agent-user-guide#command-line-options
[virtual-machines-linux-capture-image]: /documentation/articles/virtual-machines-linux-capture-image-resource-manager
[virtual-networks-udr-overview]: /documentation/articles/virtual-networks-udr-overview
[virtual-networks-nsg]: /documentation/articles/virtual-networks-nsg
[storage-premium-storage-preview-portal]: /documentation/articles/storage-premium-storage-preview-portal
[storage-introduction]: /documentation/articles/storage-introduction
[virtual-machines-upload-image-windows-resource-manager]: /documentation/articles/virtual-machines-upload-image-windows-resource-manager
[virtual-machines-azure-resource-manager-architecture]: /documentation/articles/virtual-machines-azure-resource-manager-architecture
[virtual-machines-windows-tutorial]: /documentation/articles/virtual-machines-windows-classic-tutorial
[virtual-networks-create-vnet-arm-pportal]: /documentation/articles/virtual-networks-create-vnet-arm-pportal
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]: /documentation/articles/virtual-machines-ps-create-preconfigure-windows-resource-manager-vms
[virtual-machines-linux-tutorial]: /documentation/articles/virtual-machines-linux-tutorial

[msdn-set-azurermvmaemextension]: https://msdn.microsoft.com/zh-cn/library/azure/mt670598.aspx

[virtual-machines-azurerm-versus-azuresm]: /documentation/articles/virtual-machines-azurerm-versus-azuresm

[install-extension-cli]: https://github.com/Azure/azure-linux-extensions/blob/master/AzureEnhancedMonitor/README.md

[azure-quickstart-templates-github]: https://github.com/Azure/azure-quickstart-templates
[sap-templates-2-tier-marketplace-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-user-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]: https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[templates-101-simple-windows-vm]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[template-201-vm-from-specialized-vhd]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd

[sap-pam]: https://support.sap.com/pam "SAP 产品可用性对照表"

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。本文介绍如何使用 Resource Manager 部署模型。Microsoft 建议对大多数新的部署使用该模型，而不是经典部署模型。

云计算是一个广泛使用的术语，它在 IT 行业（从小公司到大型跨国企业）中受到越来越多的重视。Azure 是 Microsoft 提供的一个云服务平台，它提供了各种新的可能性。现在客户将能够快速将应用程序预配为云服务并可取消预配，因此他们不会受到技术或预算方面的限制。公司不用在硬件基础结构中投入时间和预算，而是可以重点关注应用程序、业务流程以及它为客户和用户带来的好处。

借助 Azure 虚拟机服务，Microsoft 提供了全面的基础结构即服务 (IaaS) 平台。Azure 虚拟机 (IaaS) 支持基于 SAP NetWeaver 的应用程序。以下白皮书介绍在作为所选平台的 Azure 中如何规划和实现基于 SAP NetWeaver 的应用程序。

[AZURE.INCLUDE [windows-warning](../includes/virtual-machines-linux-sap-warning.md)]

##  <a name="3da0389e-708b-4e82-b2a2-e92f132df89c"></a>规划和实施

标题：Linux 虚拟机 (VM) 上的 SAP NetWeaver - 规划和实施指南

摘要：如果你正在考虑在 Azure 虚拟机中运行 SAP NetWeaver，可从本文开始着手。此规划和实现指南可帮助你评估现有或计划的基于 SAP NetWeaver 的系统是否可以部署到 Azure 虚拟机环境。它介绍了多个 SAP NetWeaver 部署方案，并包括特定于 Azure 的 SAP 配置。该文列出并说明了在 SAP/Azure 端运行混合 SAP 布局产品时所需的所有必要配置信息。此外，还介绍了为确保 IaaS 上基于 SAP NetWeaver 的系统实现高可用性可以采取的措施。

更新时间：2016 年 3 月

[可在此处找到此指南][planning-guide]
## <a name="6aadadd2-76b5-46d8-8713-e8d63630e955"></a>部署

标题：Linux 虚拟机 (VM) 上的 SAP NetWeaver - 部署指南

摘要：本文档提供将 SAP NetWeaver 软件部署到 Azure 中虚拟机的过程指导。此文重点介绍三种特定部署方案，主要侧重于启用 SAP 的 Azure 监视扩展，包括针对 SAP 的 Azure 监视扩展的故障排除建议。本文假设你已阅读规划和实施指南。

更新时间：2016 年 3 月

[可在此处找到此指南][deployment-guide]

## <a name="1343ffe1-8021-4ce6-a08d-3a1553a4db82"></a>DBMS 部署指南

标题：Linux 虚拟机 (VM) 上的 SAP NetWeaver - DBMS 部署指南

摘要：此文介绍了应与 SAP 一起运行的 DBMS 系统的规划和实现注意事项。在第一部分中，列出并提供了一般注意事项。此文的以下部分与在 Azure 中部署 SAP 所支持的不同 DBMS 相关。提供的不同 DBMS 为 SQL Server、SAP ASE 和 Oracle。在这些特定部分中，讨论了在 Azure 上将 SAP 系统与这些 DBMS 一起运行时必须考虑的注意事项。提供了 Azure 上的不同 DBMS 支持的备份和高可用性方法等主题，以便用于 SAP 应用程序。

更新时间：2016 年 3 月

[可在此处找到此指南][dbms-guide]

<!---HONumber=Mooncake_0620_2016-->