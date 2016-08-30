<properties
	pageTitle="Azure 存储的指标更新"
    description=""
    services=""
    documentationCenter=""
    authors=""
    manager=""
    editor=""
    tags=""/>
	
<tags ms.service="announcement" ms.date="" wacn.date="" wacn.lang="cn"/>

# Azure 存储的指标更新
为明确 Azure 存储相关的计费，我们将对存储资源 GUID 进行更改，生效日期为 2016 年 10 月 1 日。我们建议你在此之前在自定义应用程序中对存储资源 GUID 进行适当更改。下列资源名称也会发生改变：标准 IO - 表/队列 (GB)。下表列出了各种资源类型中 GUID 的前后变化。    

|	服务名称	|	  服务类型	|	原资源名称	|	原 GUID	|	新资源名称	|	新 GUID	|
|---|----|-----|---|---|-------|
|数据管理	|	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 块 Blob 写入操作单元（以 10,000 计）	|	211e620c-ebcf-4db5-a7fd-996abebe9546	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 文件写入操作单元（以 10,000 计）	|	f90f81d2-238a-49d0-9f1b-9df13a859843	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 页 Blob 写入操作单元（以 10,000 计）	|	c01a1eed-b19a-4aad-bb83-8d62cdc29778	|	SQL Server Web (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表写入批处理操作单元（以 10,000 计）	|	b9e5e77c-a0b3-4a2c-9b8b-57fa54f31c52	|	SQL Server Web (8 core)	|	1	|	f0185c96-1902-44c9-b4b5-9bf534862606	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表写入操作单元（以 10,000 计）	|	9cb0bde8-bc0d-468c-8423-a25fe06779d3	|	SQL Server Web (16 core)	|	1	|	d8f0abe0-ebe9-4181-9239-6b6f9d516e8d	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 块 Blob 删除操作单元（以 10,000 计）	|	923978e1-fd3f-4bd5-a798-f4b533057e46	|	SQL Server Web (20 core)	|	1	|	c44e509d-63cf-4ee5-8ba2-ad5a6bd6061d	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 块 Blob 列表操作单元（以 10,000 计）	|	f8c187bb-5a47-46ae-b874-f186d207fff4	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 块 Blob 读取操作单元（以 10,000 计）	|	40551b4c-e8be-48ed-b70b-f8d25c7de724	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|数据管理	|	异地冗余	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 文件写入操作单元（以 10,000 计）	|	b2048190-2afb-43a5-b5f4-af7a885396c1	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 页 Blob 删除操作单元 - 免费（以 10,000 计）	|	f14382b0-1838-48e9-9314-c7b6eababc81
|数据管理	|		|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 页 Blob 列表操作单元（以 10,000 计）	|	6417d428-fe3b-4270-951d-5a67e6411a8f	|	SQL Server Standard (16 core)	|	1	|	339fc49d-f3cc-4c6d-83f9-44cf9c0c9d38	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 页 Blob 读取操作单元（以 10,000 计）	|	d3824379-dc7e-472b-9e67-3f4a7eadc05b	|	SQL Server Standard (20 core)	|	1	|	8f89de2d-5e65-4560-8d94-208ad84b4881	|
|数据管理	|	异地冗余	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 页 Blob 写入操作单元（以 10,000 计）	|	11278850-f161-4a6e-86ef-d650a29fb62f	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 队列操作单元（以 10,000 计）	|	5e946cfc-acc9-4a96-895b-92386fa63c9c	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|数据管理	|	异地冗余	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表批处理写入操作单元（以 10,000 计）	|	bb20dfda-6d18-4bb1-b7fc-86a05d9cfb8d	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表删除操作单元（以 10,000 计）	|	e3a600a6-5a85-4679-9721-1c0d192ed142	|	SQL Server Enterprise (8 core)	|	1	|	7f3e1d2c-74d6-4f47-b1f5-f929290ece86	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表列表操作单元（以 10,000 计）	|	66ce8e33-adda-4e04-a0cb-0f1ce31849b6	|	SQL Server Enterprise (16 core)	|	1	|	2f6d02be-a7f9-4918-96d1-c291793047e6	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表读取操作单元（以 10,000 计）	|	12da282f-7e96-49e2-983a-9a65da2a4866	|	SQL Server Enterprise (20 core)	|	1	|	f438a8b0-a0a8-4633-8cb1-21daba7270a0	|
|数据管理	|	        	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表扫描操作单元（以 10,000 计）	|	9a56b4f4-1d78-416e-a19b-815490ad7910	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|数据管理	|	异地冗余	|	存储事务数（以 10,000 计）	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表写入操作单元（以 10,000 计）	|	ad22fac8-9da5-4577-8683-56ae94d39e42	|	SQL Server Standard (up to 4 cores)	|	1	|	不变	|
|存储	|	异地冗余	|	标准 IO – 页 Blob 和磁盘 (GB)	|		|	标准 IO – 页 Blob 和磁盘 (GB)	|	0e9d0c9b-ab6d-4312-9c7e-3794e22af9c4
|存储	|	本地冗余	|	标准 IO – 页 Blob 和磁盘 (GB)	|		|	标准 IO – 页 Blob 和磁盘 (GB)	|	d23a5753-ff85-4ddf-af28-8cc5cf2d3882	|	SQL Server Standard (16 core)	|	1	|	339fc49d-f3cc-4c6d-83f9-44cf9c0c9d38	|
|存储	|	只读访问跨地域冗余	|	标准 IO – 页 Blob 和磁盘 (GB)	|		|	标准 IO – 页 Blob 和磁盘 (GB)	|	5e7a80e5-0273-44b8-a179-e4f62ea6ec9f	|	SQL Server Standard (20 core)	|	1	|	8f89de2d-5e65-4560-8d94-208ad84b4881	|
|存储	|	本地冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表 (GB)	|	3f2b1e1c-c886-4ec6-ad6f-dd0ef38819c9	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|存储	|	异地冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表 (GB)	|	92567832-8613-47c4-9e8a-b2b9461a4875	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|存储	|	只读访问跨地域冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 表 (GB)	|	6c76daf0-aeb1-4238-b8b4-18820c91eeef	|	SQL Server Enterprise (up to 4 cores)	|	1	|	不变	|
|存储	|	本地冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 队列 (GB)	|	a3dd3839-f638-493d-a529-e1e1427a5b78	|	SQL Server Enterprise (8 core)	|	1	|	7f3e1d2c-74d6-4f47-b1f5-f929290ece86	|
|存储	|	异地冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 队列 (GB)	|	d6eb6e09-ee00-4672-aa66-ed1219c1113d	|	SQL Server Enterprise (16 core)	|	1	|	2f6d02be-a7f9-4918-96d1-c291793047e6	|
|存储	|	只读访问跨地域冗余	|	标准 IO – 表和队列 (GB)	|	964c283a-83a3-4dd4-8baf-59511998fe8b	|	标准 IO – 队列 (GB)	|	59d1ce45-9e3e-44b2-85c5-4677230ecc88|	SQL Server Enterprise (20 core)	|	1	|	f438a8b0-a0a8-4633-8cb1-21daba7270a0	|


