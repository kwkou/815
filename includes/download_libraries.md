#下载 Azure SDK for Java

##针对 Java 的 Azure 客户端库 - 手动下载

将根据 [Apache 许可证 2.0 版][license]分发 Azure Libraries for Java。有关库和所有依赖项的 ZIP 文件，请单击[此处][zip-download]。该文件由 Microsoft Open Technologies, Inc. 提供。有关许可证和其他信息，请参见 ZIP 文件中的 license.txt 和 ThirdPartyNotices.txt 文件。

##Azure Libraries for Java - Maven

如果您的项目已设置为使用 Maven 进行构建，请将以下依赖项添加到您的 pom.xml 文件。

	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-compute</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-network</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-sql</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-storage</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-management-websites</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-media</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-servicebus</artifactId>
	    <version>n.n.n</version>
	</dependency>
	<dependency>
	    <groupId>com.microsoft.azure</groupId>
	    <artifactId>azure-serviceruntime</artifactId>
	    <version>n.n.n</version>
	</dependency>



在 `<version>` 元素中，将  *n.n.n* 替换为有效版本号，可从 [Maven 上的 Azure 库存储库](http://go.microsoft.com/fwlink/?LinkID=286274)中获取此版本号。

[license]: http://www.apache.org/licenses/LICENSE-2.0.html
[zip-download]:  http://go.microsoft.com/fwlink/?LinkId=253887
<!--HONumber=41-->
