<properties
	pageTitle="GUID 迁移 - Microsoft Azure"
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>
	
<tags ms.service="announcement" ms.date="" wacn.date="" wacn.lang="cn"/>

# GUID 迁移
随着各区域 A 系列虚拟机的陆续更新，虚拟机的 GUID 将产生变化。下表列出了不同类型资源变化前后的 GUID。

## SQL Server 虚拟机
下列各系列虚拟机计量指标的资源 GUID 将于 2016 年 6 月 20 日发生变化。更新后的资源名称、使用量乘数和资源 GUID 信息，请见下表：

|	服务	|	资源名称	|	服务类型	|	内核数	|	原使用量乘数	|	原 GUIID	|	新资源名称	|	新使用量乘数	|	新 GUIID	|
|---|----|-----|---|---|-------|----|---|-------|
|虚拟机	|SQL Server Web	|	A0, A1, A2, A3, D1, D1v2	|	1	|	1	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Web	|	A5, D2, D11, D2v2, D11v2	|	2	|	1	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Web	|	A6, D3, D12, D3v2, D4v2, D12v2	|	4	|	1	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Web	|	A4, A7, D4, D13, D13v2	|	8	|	2	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (8 core)	|	1	|	f0185c96-1902-44c9-b4b5-9bf534862606	|
|虚拟机	|	SQL Server Web	|	D14, D5v2, D14v2	|	16	|	4	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (16 core)	|	1	|	d8f0abe0-ebe9-4181-9239-6b6f9d516e8d	|
|虚拟机	|	SQL Server Web	|	D15v2	|	20	|	5	|	8e9af6db-7104-4f1a-830e-93eabb955444	|	SQL Server Web (20 core)	|	1	|	c44e509d-63cf-4ee5-8ba2-ad5a6bd6061d	|
|虚拟机	|	SQL Server Standard (标准版)	|	A0, A1, A2, A3, D1, D1v2	|	1	|	1	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Standard (标准版)	|	A5, D2, D11, D2v2, D11v2	|	2	|	1	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Standard (标准版)	|	A6, D3, D12, D3v2, D4v2, D12v2	|	4	|	1	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Standard (标准版)	|	A4, A7, D4, D13, D13v2	|	8	|	2	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (8 core)	|	1	|	734b802a-0a4e-490c-98dd-a4ca285d835b	|
|虚拟机	|	SQL Server Standard (标准版)	|	D14, D5v2, D14v2	|	16	|	4	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (16 core)	|	1	|	339fc49d-f3cc-4c6d-83f9-44cf9c0c9d38	|
|虚拟机	|	SQL Server Standard（标注版）	|	D15v2	|	20	|	5	|	6b44a2ed-1103-41f3-90fe-8aaf147c3e41	|	SQL Server Standard (20 core)	|	1	|	8f89de2d-5e65-4560-8d94-208ad84b4881	|
|虚拟机	|	SQL Server Enterprise (企业版)	|	A0, A1, A2, A3, D1, D1v2	|	1	|	1	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Enterprise (企业版)	|	A5, D2, D11, D2v2, D11v2	|	2	|	1	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Enterprise (企业版)	|	A6, D3, D12, D3v2, D4v2, D12v2	|	4	|	1	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|虚拟机	|	SQL Server Enterprise (企业版)	|	A4, A7, D4, D13, D13v2	|	8	|	2	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (8 core)	|	1	|	7f3e1d2c-74d6-4f47-b1f5-f929290ece86	|
|虚拟机	|	SQL Server Enterprise (企业版)	|	D14, D5v2, D14v2	|	16	|	4	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (16 core)	|	1	|	2f6d02be-a7f9-4918-96d1-c291793047e6	|
|虚拟机	|	SQL Server Enterprise（企业版）	|	D15v2	|	20	|	5	|	12699064-6912-437d-8cf7-9a50364cfb1d	|	SQL Server Enterprise (20 core)	|	1	|	f438a8b0-a0a8-4633-8cb1-21daba7270a0	|

## HDInsight
下列 A 系列计量指标的资源 GUID 将于 2016 年 2 月 15 日发生变化。

**HDINSIGHT 的新老计量指标**

|服务	|	服务类型	|	资源	|	原 GUID	|	新 GUID	|
|---|-----|---|------|------|
|HDInsight	|	A1 HDInsight	|	计算小时数	|	b0b01749-c39b-4275-b283-6a92aabc3347	|	b0b01749-c39b-4275-b283-6a92aabc3347	|
|HDInsight	|	A2 HDInsight	|	计算小时数	|	b0b01749-c39b-4275-b283-6a92aabc3347	|	a8fccba6-1070-483d-a82b-88bbabcb409f	|
|HDInsight	|	A3 HDInsight	|	计算小时数	|	b0b01749-c39b-4275-b283-6a92aabc3347	|	945b8bdc-4942-45f4-bb2d-65b849304aff	|
|HDInsight	|	A4 HDInsight	|	计算小时数	|	b0b01749-c39b-4275-b283-6a92aabc3347	|	06fb8bad-0653-4cd7-bfb0-8fcd3ac50371	|

## 云服务
下列 A 系列计量指标的资源 GUID 将于 2016 年 1 月 25 日发生变化。

**云服务的新老计量指标**

|服务	|	服务类型	|	资源	|	原 GUID	|	新 GUID	|
|---|-----|---|------|------|
|云服务	|	A0 云服务	|	计算小时数	|	8b8c895b-bca0-4f0e-9fbb-16c13512b8d0	|	acb6aa52-e8df-46eb-aa55-bb5433a00798	|
|云服务	|	A2 云服务	|	计算小时数	|	8b8c895b-bca0-4f0e-9fbb-16c13512b8d0	|	f13c5c53-769e-4e4f-b4dd-c0798c60bce7	|
|云服务	|	A3 云服务	|	计算小时数	|	8b8c895b-bca0-4f0e-9fbb-16c13512b8d0	|	36baac80-9e06-4b23-ab28-94c865225fde	|
|云服务	|	A4 云服务	|	计算小时数	|	8b8c895b-bca0-4f0e-9fbb-16c13512b8d0	|	5c583d6e-1538-44e3-9dac-b699951a8084	|

## 虚拟机
下列 A 系列计量指标的资源 GUID 将于 2016 年 1 月 25 日发生变化。

**虚拟机的新老计量指标**
 
|服务	|	服务类型	|	资源	|	原 GUID	|	新 GUID	|
|---|-----|---|------|------|
|虚拟机	|	A0 虚拟机（非 Windows）	|	计算小时数	|	953fe2ae-f932-4f3d-b34d-b5d1f950fa25	|	516b993b-ba97-4670-9ed4-efdb0e1d42e8	|
|虚拟机	|	A0 虚拟机（Windows）	|	计算小时数	|	19c6b5e3-4162-4be1-9911-fba6f27991c8	|	15998f24-ce11-4379-85f7-0ac898afb4ac	|
|虚拟机	|	A2 虚拟机（非 Windows）	|	计算小时数	|	953fe2ae-f932-4f3d-b34d-b5d1f950fa25	|	7890f5dc-e715-4216-88f9-2c80da9954fb	|
|虚拟机	|	A2 虚拟机（Windows）	|	计算小时数	|	19c6b5e3-4162-4be1-9911-fba6f27991c8	|	bd25a007-f0b4-4fd3-8b5e-bfd2cc0772f7	|
|虚拟机	|	A3 虚拟机（非 Windows）	|	计算小时数	|	953fe2ae-f932-4f3d-b34d-b5d1f950fa25	|	bffbce65-8a40-4ab1-a329-fc3fd0e29131	|
|虚拟机	|	A3 虚拟机（Windows）	|	计算小时数	|	19c6b5e3-4162-4be1-9911-fba6f27991c8	|	2bd3935c-26d3-4700-881f-967ea10e99c7	|
|虚拟机	|	A4 虚拟机（非 Windows）	|	计算小时数	|	953fe2ae-f932-4f3d-b34d-b5d1f950fa25	|	4d6d4916-214b-4f65-9530-0d2e5d39a427	|
|虚拟机	|	A4 虚拟机（Windows）	|	计算小时数	|	19c6b5e3-4162-4be1-9911-fba6f27991c8	|	2ddcf5d0a-f81a-4e29-8fb6-feab48089b35	|
|虚拟机	|	BASIC.A0 虚拟机（非 Windows）	|	计算小时数	|	b8b227a6-d291-423a-8dbf-423c9dd94a94	|	d4d98456-7fe3-45dc-aac3-05e2af2bbba4	|
|虚拟机	|	BASIC.A0 虚拟机（Windows）	|	计算小时数	|	a51ac8a1-8bfe-4979-9488-658ede49f108	|	c418e9f8-66cd-4b69-a59f-b927c5cef57a	|
|虚拟机	|	BASIC.A2 虚拟机（非 Windows）	|	计算小时数	|	b8b227a6-d291-423a-8dbf-423c9dd94a94	|	ea3fdc40-2029-44fd-959a-60f04f73c58a	|
|虚拟机	|	BASIC.A2 虚拟机（Windows）	|	计算小时数	|	a51ac8a1-8bfe-4979-9488-658ede49f108	|	7d46cdc5-51bb-4fec-8721-a73b923d6132	|
|虚拟机	|	BASIC.A3 虚拟机（非 Windows）	|	计算小时数	|	b8b227a6-d291-423a-8dbf-423c9dd94a94	|	9267dd5e-4917-47bb-9652-13d42abec6fe	|
|虚拟机	|	BASIC.A3 虚拟机（Windows）	|	计算小时数	|	a51ac8a1-8bfe-4979-9488-658ede49f108	|	5911f30f-0878-4405-b878-d205e2fbe684	|
|虚拟机	|	BASIC.A4 虚拟机（非 Windows）	|	计算小时数	|	b8b227a6-d291-423a-8dbf-423c9dd94a94	|	e043378d-3200-4647-a9e4-efafe17e98e1	|
|虚拟机	|	BASIC.A4 虚拟机（Windows）	|	计算小时数	|	a51ac8a1-8bfe-4979-9488-658ede49f108	|	feea5525-a1b9-4f00-bcc9-a5db4e7927cf	|

