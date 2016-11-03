<properties
	pageTitle="如何为 Azure Java web 应用配置更大的内存来解决内存不足的问题"
	description="如何为 Azure Java web 应用配置更大的内存来解决内存不足的问题。"
	services="app-service-web"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags=""/>

<tags
	ms.service="app-service-web-aog"
	ms.date="10/27/2016"
	wacn.date="11/03/2016"/>


#如何为 Azure Java web 应用配置更大的内存来解决内存不足的问题

当将 Java 网站部署在 Azure Web 应用上后，如果运行过程中发现内存不足错误或者性能问题后，开发人员首先需要排查是否是 Java 代码的缺陷导致了内存泄漏，在确认代码没有问题后，并且确认网站的确需要更大的内存后，可以参照以下步骤修改 Java 虚拟机配置，来缓解网站内存压力问题。


1.	登录网站 FTP，打开 web.config 文件，默认情况下 web.config 文件位于 `site\wwwroot` 下.
2.	在 web.config 文件中，修改 JAVA_OPTS 参数的值，比如: 


		<environmentVariable name="JAVA_OPTS" value="-Djava.net.preferIPv4Stack=true -Xms256m -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=256m"/>


	各参数的含义如下：

            -Xms<size>        			设置初始Java堆大小
            -Xmx<size>         			设置最大Java堆大小
            -XX:PermSize<size>  		设置初始PermGen大小
            -XX:MaxPermSize<size> 		设置最大PermGen大小

	另外当日志中发现如下错误时，才需要考虑增加 MaxPermSize 的值
               java.lang.OutOfMemoryError: PermGen space

3.	当网站所需要的总内存超过2G的时候，需要登录 Azure Portal，在配置页面下，将网站运行模式改成64位, 如图:

 	![](./media/aog-web-app-java-memory-out/bit-change.png)


4.	在 Azure Portal 上重启网站，使配置生效。
