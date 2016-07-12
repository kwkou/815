资源|免费|共享（预览）|基本|标准
---|---|---|---|---|---
每个 [App Service 计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)的 [Web 应用](/home/features/web-site/)数<sup>1</sup>|10|100|无限制<sup>2</sup>|无限制<sup>2</sup>
[App Service 计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)|每个区域 1 个|每个资源组 10 个|每个资源组 10 个|每个资源组 10 个
计算实例类型|共享|共享|专用<sup>3</sup>|专用<sup>3</sup>
[横向扩展](/documentation/articles/web-sites-scale/)（最大实例数）|1 个共享|1 个共享|3 个专用<sup>3</sup>|10 个专用<sup>3</sup>
存储<sup>5</sup>|1 GB<sup>5</sup>|1 GB<sup>5</sup>|10 GB<sup>5</sup>|50 GB<sup>5</sup>
CPU 时间（天）<sup>6</sup>|60 分钟|240 分钟|无限制，按标准[费率](/home/features/web-site/pricing/)</a>付费|无限制，按标准费率付费
内存（1 小时）|每个 App Service 计划 1024 MB|每个应用 1024 MB|不适用|不适用
带宽|165 MB|无限制，收取[数据传输费率](/pricing/details/data-transfer/)|无限制，收取数据传输费率|无限制，收取数据传输费率
应用程序体系结构|32 位|32 位|32 位/64 位|32 位/64 位
每个实例的 Web 套接字数<sup>7</sup>|5|35|350|不受限制
[带 FTP/S 和 SSL 的 chinacloudsites.cn 子域](/documentation/articles/web-sites-configure-ssl-certificate/)|X|X|X|X
[自定义域](/documentation/articles/web-sites-custom-domain-name/)支持||X|X|X
自定义域 [SSL 支持](/documentation/articles/web-sites-configure-ssl-certificate/)|||不受限制|无限制，包含 5 个 SNI SSL 和 1 个 IP SSL 连接
集成负载平衡器||X|X|X
[始终打开](/documentation/articles/web-sites-configure/)|||X|X
[计划备份](/documentation/articles/web-sites-backup/)||||每天一次
[自动扩展](/documentation/articles/web-sites-scale/)|||X|X
[WebJobs](/documentation/articles/web-sites-create-web-jobs/)<sup>8</sup>|X|X|X|X
[Azure 计划程序](/home/features/scheduler/)支持||X|X|X
[终结点监视](/documentation/articles/web-sites-monitor/)|||X|X
[过渡槽（预览）](/documentation/articles/web-sites-staged-publishing/)||||5
每个应用的自定义域数</a>||500|500|500
SLA||<p>|99\.9%|99\.95%<sup>9</sup>

<sup>1</sup>除非特别说明，否则应用和存储配额依每个 App Service 计划为准。  
<sup>2</sup>你可以在这些计算机上托管的应用的实际数目取决于应用的活动、计算机实例的大小和相应的资源利用率。  
<sup>3</sup>专用实例可有不同的大小。请参阅 [Azure Web 应用定价](/home/features/web-site/pricing/)以了解更多详细信息。可通过提出支持请求获得其他实例。   
<sup>5</sup>存储限制是跨相同 App Service 计划中所有应用的内容总大小。可以通过提出支持请求增加存储限制。  
<sup>6</sup>这些资源受到专用实例上的物理资源（实例大小和实例数）的限制。  
<sup>7</sup>如果你将基本层的某个应用扩展为两个实例，则其中每个实例有 350 个并发连接。  
<sup>8</sup>按需、按计划或作为 Azure 实例内的后台任务连续运行自定义可执行文件和/或脚本。连续执行 WebJob 需要使用“始终打开”。计划的 WebJob 需要使用 Azure 计划程序免费或标准版。可以在 Azure 实例中运行的 WebJob 的数量没有预定义的限制，但是存在取决于应用程序代码正在尝试执行的任务的实际限制。 
<sup>9</sup>向使用多个实例和为故障转移配置的 Azure 流量管理器的部署提供 99.95% 的 SLA。

<!---HONumber=Mooncake_0328_2016-->