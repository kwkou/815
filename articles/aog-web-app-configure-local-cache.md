<properties 
	pageTitle="如何为 Azure Web 应用配置本地缓存" 
	description="如何为 Azure Web 应用配置本地缓存。" 
	services="app-service-web" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="app-service-web-aog" ms.date="" wacn.date="08/31/2016"/>
#如何为 Azure Web 应用配置本地缓存

##Azure Web 应用内容的存储方式

Azure web 应用内容（包括代码文件，资源文件等）默认并不是存放在本地虚机的磁盘上，而是存放在远程的共享目录中。这样的设计有以下几个特点：

- 多个 web 应用的虚拟机实例共享同一个 web 应用目录。
- 内容是持久化存储，运行中的 web 应用可以修改共享目录下的内容。
- Web 应用的日志文件也存放在相同的共享目录下。
- 新内容是直接发布到共享目录中，用户可以立即在 SCM 站点看到新发布的内容。并且基于 ASP.NET 的 web 应用在收到文件更改通知后，会触发重启获取最新内容。

##Web 应用本地缓存介绍

除了使用默认的共享目录来存放 web 应用内容，用户可以为 web 应用开启本地缓存。开启本地缓存后，web 应用内容会从共享目录复制到本地虚拟机实例的磁盘上。启用本地缓存有以下几点好处:

- 可以提高 web 应用的效率，避免访问远程存储所带来的网络延时。
- 如果远程文件服务器发生任何故障，用户的 web 应用将不受此影响。
- 减少由于远程文件服务器迁移而造成的 web 应用重启次数。

开启本地缓存后，web 应用具有以下行为:

- 当 web 应用重启时，在虚拟机实例上创建 /site 和 /siteextensions 文件夹的本地拷贝
- 本地缓存可以读写，但是当 web 应用的虚机发生迁移，或者 web 应用重启后，所有的改动将会丢失。因此不建议将重要的数据写到本地缓存中。
- 如果 web 应用开启了日志，日志会写入本地缓存中，并定期同步到远程的共享目录。
- 发布的新内容还是继续存放到远程共享目录中，用户需要重启 web 应用将新内容同步到本地缓存。
- Data 和 LogFiles 文件夹下面会生成额外的子文件（虚拟机标识 + 时间戳）来存放数据和日志。 

	![](./media/aog-web-app-configure-local-cache/structure.png)

## 如何启用 web 应用本地缓存

1.	登录 Portal，进入 web 应用的配置界面，为应用设置添加如下参数:


 	![](./media/aog-web-app-configure-local-cache/portal.png)

	>[AZURE.NOTE]默认 local cache 大小为 300MB，最大支持 1G(1024MB)

2.	重启 web 应用后配置生效



## 验证 Web 应用是否已经切换到本地缓存

1.	登录 web 应用的 SCM 站点：https://yourwebsitename.scm.chinacloudsites.cn
2.	进入 Process explorer 面板
 
	![](./media/aog-web-app-configure-local-cache/kudu.png)


3.	点击 w3wp.exe 进程的 Properties 按钮，进入 Environment Variables 面板
4.	如果存在环境变量 `WEBSITE_LOCALCACHE_READY = True`，则说明 web 应用已经成功切换到本地缓存模式。

	![](./media/aog-web-app-configure-local-cache/local-ready.png)

