<properties
    pageTitle="在 Azure 中创建 SAP 多 SID 配置 | Azure"
    description="Windows 虚拟机上的高可用性 SAP NetWeaver 多 SID 配置指南"
    services="virtual-machines-windows, virtual-network, storage"
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
    wacn.date="05/22/2017"
    ms.author="goraco"
    ms.custom="H1Hack27Feb2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ddc307ebc9d952da2f964e0536877f55698e21ce"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-an-sap-netweaver-multi-sid-configuration"></a>创建 SAP NetWeaver 多 SID 配置

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-cli]: /documentation/articles/cli-install-nodejs/
[azure-portal]:https://portal.azure.cn
[azure-ps]:https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]: /documentation/articles/azure-subscription-service-limits/
[azure-subscription-service-limits-subscription]: /documentation/articles/azure-subscription-service-limits/

[dbms-guide]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/
[dbms-guide-2.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f
[dbms-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#c8e566f9-21b7-4457-9f7f-126036971a91
[dbms-guide-2.3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#10b041ef-c177-498a-93ed-44b3441ab152
[dbms-guide-2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#65fa79d6-a85f-47ee-890b-22e794f51a64
[dbms-guide-3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#871dfc27-e509-4222-9370-ab1de77021c3
[dbms-guide-5.5.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#0fef0e79-d3fe-4ae2-85af-73666a6f7268
[dbms-guide-5.5.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#f9071eff-9d72-4f47-9da4-1852d782087b
[dbms-guide-5.6]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#1b353e38-21b3-4310-aeb6-a77e7c8e81c8
[dbms-guide-5.8]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#9053f720-6f3b-4483-904d-15dc54141e30
[dbms-guide-5]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#3264829e-075e-4d25-966e-a49dad878737
[dbms-guide-8.4.1]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#b48cfe3b-48e9-4f5b-a783-1d29155bd573
[dbms-guide-8.4.2]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#23c78d3b-ca5a-4e72-8a24-645d141a3f5d
[dbms-guide-8.4.3]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#77cd2fbb-307e-4cbf-a65f-745553f72d2c
[dbms-guide-8.4.4]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#f77c1436-9ad8-44fb-a331-8671342de818
[dbms-guide-900-sap-cache-server-on-premises]: /documentation/articles/virtual-machines-windows-sap-dbms-guide/#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]:./media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:./media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:./media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:./media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:./media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:./media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:./media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:./media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:./media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]: /documentation/articles/virtual-machines-sap-deployment-guide/
[deployment-guide-2.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94
[deployment-guide-3.1.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#3688666f-281f-425b-a312-a77e7db2dfab
[deployment-guide-3.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#db477013-9060-4602-9ad4-b0316f8bb281
[deployment-guide-3.3]: /documentation/articles/virtual-machines-sap-deployment-guide/#54a1fc6d-24fd-4feb-9c57-ac588a55dff2
[deployment-guide-3.4]: /documentation/articles/virtual-machines-sap-deployment-guide/#a9a60133-a763-4de8-8986-ac0fa33aa8c1
[deployment-guide-3]: /documentation/articles/virtual-machines-sap-deployment-guide/#b3253ee3-d63b-4d74-a49b-185e76c4088e
[deployment-guide-4.1]: /documentation/articles/virtual-machines-sap-deployment-guide/#604bcec2-8b6e-48d2-a944-61b0f5dee2f7
[deployment-guide-4.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e
[deployment-guide-4.3]: /documentation/articles/virtual-machines-sap-deployment-guide/#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc
[deployment-guide-4.4.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542
[deployment-guide-4.4]: /documentation/articles/virtual-machines-sap-deployment-guide/#c7cbb0dc-52a4-49db-8e03-83e7edc2927d
[deployment-guide-4.5.1]: /documentation/articles/virtual-machines-sap-deployment-guide/#987cf279-d713-4b4c-8143-6b11589bb9d4
[deployment-guide-4.5.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#408f3779-f422-4413-82f8-c57a23b4fc2f
[deployment-guide-4.5]: /documentation/articles/virtual-machines-sap-deployment-guide/#d98edcd3-f2a1-49f7-b26a-07448ceb60ca
[deployment-guide-5.1]: /documentation/articles/virtual-machines-sap-deployment-guide/#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2
[deployment-guide-5.2]: /documentation/articles/virtual-machines-sap-deployment-guide/#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]: /documentation/articles/virtual-machines-sap-deployment-guide/#fe25a7da-4e4e-4388-8907-8abc2d33cfd8

[deployment-guide-configure-monitoring-scenario-1]: /documentation/articles/virtual-machines-sap-deployment-guide/#ec323ac3-1de9-4c3a-b770-4ff701def65b
[deployment-guide-configure-proxy]: /documentation/articles/virtual-machines-sap-deployment-guide/#baccae00-6f79-4307-ade4-40292ce4e02d
[deployment-guide-figure-100]:./media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:./media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]: /documentation/articles/virtual-machines-sap-deployment-guide/#figure-11
[deployment-guide-figure-1100]:./media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:./media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:./media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]: /documentation/articles/virtual-machines-sap-deployment-guide/#figure-14
[deployment-guide-figure-1400]:./media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:./media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:./media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]: /documentation/articles/virtual-machines-sap-deployment-guide/#figure-5
[deployment-guide-figure-50]:./media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:./media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]: /documentation/articles/virtual-machines-sap-deployment-guide/#figure-6
[deployment-guide-figure-600]:./media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]: /documentation/articles/virtual-machines-sap-deployment-guide/#figure-7
[deployment-guide-figure-700]:./media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:./media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:./media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]: /documentation/articles/virtual-machines-sap-deployment-guide/#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]: /documentation/articles/virtual-machines-sap-deployment-guide/#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]: /documentation/articles/virtual-machines-sap-deployment-guide/#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]: /documentation/articles/virtual-machines-sap-deployment-guide/#564adb4f-5c95-4041-9616-6635e83a810b

[deploy-template-cli]: /documentation/articles/resource-group-template-deploy/#deploy-with-azure-cli-for-mac-linux-and-windows
[deploy-template-portal]: /documentation/articles/resource-group-template-deploy/#deploy-with-the-preview-portal
[deploy-template-powershell]: /documentation/articles/resource-group-template-deploy/#deploy-with-powershell

[dr-guide-classic]:http://go.microsoft.com/fwlink/?LinkID=521971

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

[ha-guide-classic]:http://go.microsoft.com/fwlink/?LinkId=613056

[ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/

[install-extension-cli]: /documentation/articles/virtual-machines-linux-enable-aem/

[Logo_Linux]:./media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:./media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/zh-cn/library/azure/mt670598.aspx

[planning-guide]: /documentation/articles/virtual-machines-windows-sap-planning-guide/
[planning-guide-1.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4e165b58-74ca-474f-a7f4-5e695a93204f
[planning-guide-2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10
[planning-guide-3.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#be80d1b9-a463-4845-bd35-f4cebdb5424a
[planning-guide-3.2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#df49dc09-141b-4f34-a4a2-990913b30358
[planning-guide-3.2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#fc1ac8b2-e54a-487c-8581-d3cc6625e560
[planning-guide-3.2.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#18810088-f9be-4c97-958a-27996255c665
[planning-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77
[planning-guide-3.3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#ff5ad0f9-f7f4-4022-9102-af07aef3bc92
[planning-guide-5.1.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4d175f1b-7353-4137-9d2f-817683c26e53
[planning-guide-5.1.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#e18f7839-c0e2-4385-b1e6-4538453a285c
[planning-guide-5.2.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#1b287330-944b-495d-9ea7-94b83aff73ef
[planning-guide-5.2.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3
[planning-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6ffb9f41-a292-40bf-9e70-8204448559e7
[planning-guide-5.3.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6e835de8-40b1-4b71-9f18-d45b20959b79
[planning-guide-5.3.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#a43e40e6-1acc-4633-9816-8f095d5a7b6a
[planning-guide-5.4.2]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#9789b076-2011-4afa-b2fe-b07a8aba58a1
[planning-guide-5.5.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#4efec401-91e0-40c0-8e64-f2dceadff646
[planning-guide-5.5.3]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#17e0d543-7e8c-4160-a7da-dd7117a1ad9d
[planning-guide-7.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#3e9c3690-da67-421a-bc3f-12c520d99a30
[planning-guide-7]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#96a77628-a05e-475d-9df3-fb82217e8f14
[planning-guide-9.1]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#6f0a47f3-a289-4090-a053-2521618a28c3
[planning-guide-azure-premium-storage]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#ff5ad0f9-f7f4-4022-9102-af07aef3bc92

[planning-guide-figure-100]:./media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:./media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:./media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:./media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:./media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:./media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:./media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:./media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:./media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:./media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:./media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:./media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]:./media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:./media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:./media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:./media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]:./media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:./media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:./media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]:./media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:./media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:./media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:./media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]: /documentation/articles/virtual-machines-windows-sap-planning-guide/#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/
[sap-ha-guide-1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#217c5479-5595-4cd8-870d-15ab00d4f84c
[sap-ha-guide-2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#42156640c6-01cf-45a9-b225-4baa678b24f1
[sap-ha-guide-3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f76af273-1993-4d83-b12d-65deeae23686
[sap-ha-guide-3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#3e85fbe0-84b1-4892-87af-d9b65ff91860
[sap-ha-guide-4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-4.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a3c5408-b168-46d6-99f5-4219ad1b1ff2
[sap-ha-guide-5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fdfee875-6e66-483a-a343-14bbaee33275
[sap-ha-guide-5.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#be21cf3e-fb01-402b-9955-54fbecf66592
[sap-ha-guide-5.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ff7a9a06-2bc5-4b20-860a-46cdb44669cd
[sap-ha-guide-6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2ddba413-a7f5-4e4e-9a51-87908879c10a
[sap-ha-guide-6.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1a464091-922b-48d7-9d08-7cecf757f341
[sap-ha-guide-6.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#44641e18-a94e-431f-95ff-303ab65e0bcb
[sap-ha-guide-7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2e3fec50-241e-441b-8708-0b1864f66dfa
[sap-ha-guide-7.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#93faa747-907e-440a-b00a-1ae0a89b1c0e
[sap-ha-guide-7.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f559c285-ee68-4eec-add1-f60fe7b978db
[sap-ha-guide-7.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b5b1fd0b-1db4-4d49-9162-de07a0132a51
[sap-ha-guide-7.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#ddd878a0-9c2f-4b8e-8968-26ce60be1027
[sap-ha-guide-7.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#045252ed-0277-4fc8-8f46-c5a29694a816
[sap-ha-guide-8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310
[sap-ha-guide-8.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#47d5300a-a830-41d4-83dd-1a0d1ffdbe6a
[sap-ha-guide-8.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#b22d7b3b-4343-40ff-a319-097e13f62f9e
[sap-ha-guide-8.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#9fbd43c0-5850-4965-9726-2a921d85d73f
[sap-ha-guide-8.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#84c019fe-8c58-4dac-9e54-173efd4b2c30
[sap-ha-guide-8.7]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#7a8f3e9b-0624-4051-9e41-b73fff816a9e
[sap-ha-guide-8.8]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#f19bd997-154d-4583-a46e-7f5a69d0153c
[sap-ha-guide-8.9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e69e9a34-4601-47a3-a41c-d2e11c626c0c
[sap-ha-guide-8.11]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e49a4529-50c9-4dcf-bde7-15a0c21d21ca
[sap-ha-guide-8.12.2.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#06260b30-d697-4c4d-b1c9-d22c0bd64855
[sap-ha-guide-8.12.2.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#4c08c387-78a0-46b1-9d27-b497b08cac3d
[sap-ha-guide-8.12.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-ha-guide-9.1.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#eb5af918-b42f-4803-bb50-eff41f84b0b0
[sap-ha-guide-9.1.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556
[sap-ha-guide-9.1.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#10822f4f-32e7-4871-b63a-9b86c76ce761
[sap-ha-guide-9.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#85d78414-b21d-4097-92b6-34d8bcb724b7
[sap-ha-guide-9.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#8a276e16-f507-4071-b829-cdc0a4d36748
[sap-ha-guide-9.4]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#094bc895-31d4-4471-91cc-1513b64e406a
[sap-ha-guide-9.5]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5
[sap-ha-guide-9.6]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#0ba4a6c1-cc37-4bcf-a8dc-025de4263772
[sap-ha-guide-10]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9
[sap-ha-guide-10.1]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#65fdef0f-9f94-41f9-b314-ea45bbfea445
[sap-ha-guide-10.2]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#5e959fa9-8fcd-49e5-a12c-37f6ba07b916
[sap-ha-guide-10.3]: /documentation/articles/virtual-machines-windows-sap-high-availability-guide/#755a6b93-0099-4533-9f6d-5c9a613878b5

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6001]:./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

[powershell-install-configure]:https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[resource-group-authoring-templates]: /documentation/articles/resource-group-authoring-templates/
[resource-group-overview]: /documentation/articles/resource-group-overview/
[resource-groups-networking]: /documentation/articles/resource-groups-networking/
[networking-limits-azure-resource-manager]: /documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits
[sap-pam]:https://support.sap.com/pam 
[sap-templates-2-tier-marketplace-image]:https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]: /documentation/articles/storage-azure-cli/
[storage-azure-cli-copy-blobs]: /documentation/articles/storage-azure-cli/#copy-blobs
[storage-introduction]: /documentation/articles/storage-introduction/
[storage-powershell-guide-full-copy-vhd]: /documentation/articles/storage-powershell-guide-full/#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]: /documentation/articles/storage-premium-storage/
[storage-redundancy]: /documentation/articles/storage-redundancy/
[storage-scalability-targets]: /documentation/articles/storage-scalability-targets/
[storage-use-azcopy]: /documentation/articles/storage-use-azcopy/
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]: /documentation/articles/virtual-machines-linux-attach-disk-portal/
[virtual-machines-windows-attach-disk-portal]: /documentation/articles/virtual-machines-windows-attach-disk-portal/
[virtual-machines-azure-resource-manager-architecture]: /documentation/articles/resource-group-overview/
[virtual-machines-azure-resource-manager-architecture-benefits-arm]: /documentation/articles/resource-group-overview/#the-benefits-of-using-resource-manager
[virtual-machines-azurerm-versus-azuresm]: /documentation/articles/virtual-machines-windows-compare-deployment-models/
[virtual-machines-windows-classic-configure-oracle-data-guard]: /documentation/articles/virtual-machines-windows-classic-configure-oracle-data-guard/
[virtual-machines-linux-cli-deploy-templates]: /documentation/articles/virtual-machines-linux-cli-deploy-templates/
[virtual-machines-deploy-rmtemplates-powershell]: /documentation/articles/virtual-machines-windows-ps-manage/
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
[virtual-machines-workload-template-sql-alwayson]:https://github.com/Azure/azure-quickstart-templates/tree/master/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]: /documentation/articles/virtual-network-deploy-multinic-arm-cli/
[virtual-network-deploy-multinic-arm-ps]: /documentation/articles/virtual-network-deploy-multinic-arm-ps/
[virtual-network-deploy-multinic-arm-template]: /documentation/articles/virtual-network-deploy-multinic-arm-template/
[vpn-gateway-howto-vnet-vnet-portal-classic]: /documentation/articles/vpn-gateway-vnet-vnet-rm-ps/
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
[xplat-cli]: /documentation/articles/cli-install-nodejs/
[xplat-cli-azure-resource-manager]: /documentation/articles/xplat-cli-azure-resource-manager/

2016 年 9 月，Microsoft 发布了一项功能，让用户可通过 [Azure 内部负载均衡器][load-balancer-multivip-overview]管理多个虚拟 IP 地址。 Azure 外部负载均衡器中已存在此功能。

如果具有 SAP 部署，则可使用内部负载均衡器创建 SAP ASCS/SCS 的 Windows 群集配置，如 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide]中所述。

本文重点介绍如何通过在现有 Windows Server 故障转移群集 (WSFC) 中安装附加的 SAP ASCS/SCS 群集实例，从单一 ASCS/SCS 安装转移到 SAP 多 SID 配置。 此过程完成后，就已配置 SAP 多 SID 群集。

> [AZURE.NOTE]
> 此功能仅在 Azure Resource Manager 部署模型中可用。

## <a name="prerequisites"></a>先决条件
已配置用于单个 SAP ASCS/SCS 实例的 WSFC 群集，如 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide] 中所述并如此图中所示。

![高可用性 SAP ASCS/SCS 实例][sap-ha-guide-figure-6001]

## <a name="target-architecture"></a>目标体系结构

目标是在同一个 WSFC 群集中安装多个 SAP ABAP ASCS 或 SAP Java SCS 群集实例，如下图所示：

![Azure 中的多个 SAP ASCS/SCS 群集实例][sap-ha-guide-figure-6002]

> [AZURE.NOTE]
>每个 Azure 内部负载均衡器的专用前端 IP 数目有限。
>
>一个 WSFC 群集中的最大 SAP ASCS/SCS 实例数等于每个 Azure 内部负载均衡器的最大专用前端 IP 数。
>

有关负载均衡器限制的详细信息，请参阅 [网络限制：Azure Resource Manager][networking-limits-azure-resource-manager]中的“每个负载均衡器的专用前端 IP”。

包含两个高可用性 SAP 系统的完整布局如下所示：

![包含两个 SAP 系统 SID 的 SAP 高可用性多 SID 设置][sap-ha-guide-figure-6003]

> [AZURE.IMPORTANT]
> 安装必须满足以下条件：
> - SAP ASCS/SCS 实例必须共享同一个 WSFC 群集。
> - 每个 DBMS SID 必须具有自身的专用 WSFC 群集。
> - 属于同一 SAP 系统 SID 的 SAP 应用程序服务器必须具有自身的专用 VM。

## <a name="prepare-the-infrastructure"></a>准备基础结构
若要准备基础结构，可使用以下参数安装附加 SAP ASCS/SCS 实例：

| 参数名称 | 值 |
| --- | --- |
| SAP ASCS/SCS SID |pr1-lb-ascs |
| SAP DBMS 内部负载均衡器 | PR5 |
| SAP 虚拟主机名 | pr5-sap-cl |
| SAP ASCS/SCS 虚拟主机 IP 地址（附加 Azure 负载均衡器 IP 地址） | 10.0.0.50 |
| SAP ASCS/SCS 实例编号 | 50 |
| 附加 SAP ASCS/SCS 实例的 ILB 探测端口 | 62350 |

> [AZURE.NOTE]
> 对于 SAP ASCS/SCS 群集实例，每个 IP 地址都需要唯一探测端口。 例如，如果 Azure 内部负载均衡器上有一个 IP 地址使用探测端口 62300，则该负载均衡器上的其他 IP 地址均不能使用探测端口 62300。
>
>在本例中，已保留探测端口 62300，因此使用探测端口 62350。

可在现有的 WSFC 群集中安装包含两个节点的附加 SAP ASCS/SCS 实例：

| 虚拟机角色 | 虚拟机主机名 | 静态 IP 地址 |
| --- | --- | --- |
| ASCS/SCS 实例的第 1 个群集节点 |pr1-ascs-0 |10.0.0.10 |
| ASCS/SCS 实例的第 2 个群集节点 |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>在 DNS 服务器上创建 SAP ASCS/SCS 群集实例的虚拟主机名

可使用以下参数创建 ASCS/SCS 实例虚拟主机名的 DNS 项：

| 新的 SAP ASCS/SCS 虚拟主机名 | 关联的 IP 地址 |
| --- | --- | --- |
|pr5-sap-cl |10.0.0.50 |

新的主机名和 IP 地址显示在 DNS 管理器中，如下面的屏幕截图所示：

![DNS 管理器列表，突出显示新 SAP ASCS/SCS 群集虚拟名称和 TCP/IP 地址的已定义 DNS 项][sap-ha-guide-figure-6004]

有关 DNS 项的详细创建过程，请参阅主要文档 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-9.1.1]。

> [AZURE.NOTE]
> 分配给附加 ASCS/SCS 实例虚拟主机名的新 IP 地址必须与分配给 SAP Azure 负载均衡器的新 IP 地址相同。
>
>本例中，IP 地址是 10.0.0.50。

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>使用 PowerShell 将 IP 地址添加到现有 Azure 内部负载均衡器

若要在同一个 WSFC 群集中创建多个 SAP ASCS/SCS 实例，请使用 PowerShell 将 IP 地址添加到现有 Azure 内部负载均衡器。 每个 IP 地址需要有自身的负载均衡规则、探测端口、前端 IP 池和后端池。

以下脚本将新的 IP 地址添加到现有负载均衡器。 更新环境的 PowerShell 变量。 该脚本将为所有 SAP ASCS/SCS 端口创建全部所需的负载均衡规则。

    # Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
    Clear-Host
    $ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
    $VNetName = "pr2-vnet"                        # Existing virtual network name
    $SubnetName = "Subnet"                        # Existing subnet name
    $ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
    $ILBIP = "10.0.0.50"                          # New IP address
    $VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
    $SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
    [int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

    $ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

    $count = $ILB.FrontendIpConfigurations.Count + 1
    $FrontEndConfigurationName ="lbFrontendASCS$count"
    $LBProbeName = "lbProbeASCS$count"

    # Get the Azure VNet and subnet
    $VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
    $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

    # Add second front-end and probe configuration
    Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
    $ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
    $ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzureRmLoadBalancer

    # Get new updated configuration
    $ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
    # Get new updated LP FrontendIP COnfig
    $FEConfig = Get-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
    $HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

    # Add new back-end configuration into existing ILB
    $BackEndConfigurationName  = "backendPoolASCS$count"
    Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
    $BEConfig = Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzureRmLoadBalancer

    # Get new updated config
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

    # Create load-balancing rules
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


运行脚本后，结果将在 Azure 门户中显示，如下面的屏幕截图所示：

![Azure 门户中的新前端 IP 池][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>将磁盘添加到群集计算机并配置 SIOS 群集共享磁盘

必须为每个附加 SAP ASCS/SCS 实例添加新的群集共享磁盘。 在 Windows Server 2012 R2 中，目前使用的 WSFC 群集共享磁盘是 SIOS DataKeeper 软件解决方案。

请执行以下操作：
1. 将相同大小的一个或多个附加磁盘（需要条带化）添加到每个群集节点，然后将其格式化。
2. 使用 SIOS DataKeeper 配置存储复制。

此过程假定已在 WSFC 群集计算机上安装了 SIOS DataKeeper。 如果已安装，则现在必须配置计算机之间的复制。 有关此过程的详细信息，请参阅主要文档 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-8.12.3.3]。  

![新 SAP ASCS/SCS 共享磁盘的 DataKeeper 同步镜像][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-dbms-cluster"></a>为 SAP 应用程序服务器和 DBMS 群集部署 VM

若要完成第二个 SAP 系统的基础结构准备，请执行以下操作：

1. 为 SAP 应用程序服务器部署专用 VM，并将这些 VM 放在其自身的专用可用性组中。
2. 为 DBMS 群集部署专用 VM，并将这些 VM 放在其自身的专用可用性组中。

## <a name="install-the-second-sap-sid2-netweaver-system"></a>安装第二个 SAP SID2 NetWeaver 系统

有关第二个 SAP SID2 的完整安装过程，请参阅主要文档 [Windows Vm 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-9]。

高级过程如下所示：

1. [安装 SAP 的第一个群集节点][sap-ha-guide-9.1.2]。  
 此步骤中，将在 **现有 WSFC 群集节点 1**上安装包含高可用性 ASCS/SCS 实例的 SAP。

2. [修改 ASCS/SCS 实例的 SAP 配置文件][sap-ha-guide-9.1.3]。

3. [配置探测端口][sap-ha-guide-9.1.4]。  
 此步骤中，将使用 PowerShell 配置 SAP 群集资源 SAP-SID2-IP 探测端口。 在其中一个 SAP ASCS/SCS 群集节点上执行此配置。

4. [安装数据库实例][sap-ha-guide-9.2]。  
 此步骤中，将在专用 WSFC 群集上安装 DBMS。

5. [安装第二个群集节点][sap-ha-guide-9.3]。  
 此步骤中，将在现有 WSFC 群集节点 2 上安装包含高可用性 ASCS/SCS 实例的 SAP。

6. 打开 SAP ASCS/SCS 实例的 Windows 防火墙端口和探测端口。  
 在用于 SAP ASCS/SCS 实例的两个群集节点上，打开 SAP ASCS/SCS 使用的所有 Windows 防火墙端口。 有关这些端口，请参阅 [Windows VM 上的高可用性的 SAP NetWeaver 指南][sap-ha-guide-8.8]。  
 此外，打开 Azure 内部负载均衡器探测端口，在本方案中为 62350。

7. [更改 SAP ERS Windows 服务实例的启动类型][sap-ha-guide-9.4]。

8. [安装 SAP 主应用程序服务器][sap-ha-guide-9.5] 。

9. [安装 SAP 附加应用程序服务器][sap-ha-guide-9.6] 。

10. [测试 SAP ASCS/SCS 实例故障转移和 SIOS 复制][sap-ha-guide-10]。

## <a name="next-steps"></a>后续步骤

- [网络限制：Azure Resource Manager][networking-limits-azure-resource-manager]
- [Azure 负载均衡器的多个 VIP][load-balancer-multivip-overview]
- [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide]

<!--Update_Description: wording update-->