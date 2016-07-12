#如何在 Azure 中创建 MySQL 数据库

本指南将说明如何通过 [ClearDB] 从 [Azure 应用商店]创建 MySQL 数据库，以及如何在您创建 [Azure Web 应用 ][waws] 时创建作为链接资源的 MySQL 数据库。[ClearDB] 是可容错的数据库即服务提供商，允许您在 Azure 数据中心中运行和管理 MySQL 数据库，并使您能够从任何应用程序连接到这些数据库。  

##目录
* [如何：从 Azure 应用商店创建 MySQL 数据库](#CreateFromStore)
* [如何：创建作为 Azure Web 应用的链接资源的 MySQL 数据库](#CreateForWebSite)

<div class="dev-callout"> 
<b>注意</b>
<p>在创建 Web 应用期间创建 MySQL 数据库时，您只能创建免费数据库。从 Azure 应用商店创建 MySQL 数据库将允许您创建免费数据库，但您也可以从付费选项中进行选择。</p> 
</div>

<h2><a id="CreateFromStore"></a>如何：从 Azure 应用商店创建 MySQL 数据库</h2>

若要从 [Azure 应用商店]创建 MySQL 数据库，请执行以下操作：

1. 登录到 [Azure 管理门户][门户]。
2. 单击页面底部的"+新建"，然后选择"应用商店"。

	![从应用商店选择外接程序](./media/create-mysql-db/select-store.png)

3. 选择"ClearDB MySQL 数据库"，然后单击页面底部的箭头。

	![选择 ClearDB MySQL 数据库](./media/create-mysql-db/select-cleardb-mysql.png)

4. 选择相应计划，输入数据库名称并选择区域，然后单击页面底部的箭头。

	![从应用商店购买 MySQL 数据库](./media/create-mysql-db/purchase-mysql.png)

5. 单击复选标记完成购买。

	![查看和完成购买](./media/create-mysql-db/complete-mysql-purchase.png)

6. 在创建数据库后，可以从管理门户中的"外接程序"选项卡管理该数据库。

	![在 Azure 门户中管理 MySQL 数据库](./media/create-mysql-db/manage-mysql-add-on.png)

7. 通过单击页面底部的"连接信息"（如上图中所示），您可以获得数据库连接信息。

	![MySql 连接信息](./media/create-mysql-db/mysql-conn-info.png) 


<h2><a id="CreateForWebSite"></a>如何：创建作为 Azure Web 应用的链接资源的 MySQL 数据库</h2>

若要在创建 [Azure Web 应用][waws]时创建作为链接资源的 MySQL 数据库，请执行以下操作：

1. 登录到 [Azure 管理门户][门户]。
2. 单击页面底部的"+新建"，然后依次选择"计算"、" Web 应用"和"与数据库一起创建"。

	![创建 Web 应用和数据库](./media/create-mysql-db/custom_create.png)

3. 提供您的 Web 应用的 **URL**，为 Web 应用选择"区域"，并从"数据库"下拉列表中选择"新建 MySQL 数据库"。还可以选择替换连接字符串的默认名称。单击页面底部的箭头。

	![提供 Web 应用详细信息](./media/create-mysql-db/provide-website-details.png) 

4. 提供数据库"名称"，为数据库选择"区域"（这应该与 Web 应用区域相同），同意 ClearDB 的法律条款，然后单击页面底部的复选标记。

	![提供 MySQL 详细信息](./media/create-mysql-db/provide-mysql-details.png)

5. 在创建 Web 应用后，单击 Web 应用名称以转至 Web 应用仪表板。

	![转至 Web 应用仪表板](./media/create-mysql-db/go-to-website-dashboard.png)

6. 单击"配置"。

	![转至"配置"选项卡](./media/create-mysql-db/go-to-configure-tab.png)

7. 向下滚动到"连接字符串"部分，单击"显示连接字符串"。 

	![显示连接字符串](./media/create-mysql-db/show-conn-string.png)

8. 复制要用于您的应用程序的连接字符串。

	![显示连接字符串](./media/create-mysql-db/shown-conn-string.png)

> [WACOM.NOTE] 您的 Web 应用应用程序可通过连接字符串名称来获取连接字符串。在 .NET 应用程序中，连接字符串在 **connectionStrings** 对象中提供。在其他编程语言中，连接字符串作为环境变量提供。有关更多信息，请参见[如何配置 Web 应用][配置]。

[ClearDB]: http://www.cleardb.com/
[waws]: /zh-cn/documentation/services/web-sites/
[Azure 应用商店]: /zh-cn/gallery/store/
[门户]: http://manage.windowsazure.cn
[配置]: /documentation/articles/web-sites-configure/
<!--HONumber=41-->
