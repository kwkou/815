#如何在 Azure 中创建 MongoDB 数据库

本文将说明如何通过 [MongoLab] 从 [Azure 应用商店]创建 MongoDB 数据库。[MongoLab] 是"MongoDB 即服务"提供商，允许您在 Azure 数据中心中运行和管理 MongoDB 数据库，并使您能够从任何应用程序连接到这些数据库。  

若要从 [Azure 应用商店]创建 MongoDB 数据库，请执行以下操作：

1. 登录到 [Azure 管理门户][门户]。
2. 单击页面底部的"+新建"，然后选择"应用商店"。

	![从应用商店选择外接程序](./media/create-mongolab-mongodb/select-store.png)

3. 选择 **MongoLab**，然后单击页面底部的箭头。

	![选择 MongoLab](./media/create-mongolab-mongodb/select-mongo-db.png)

4. 输入数据库名称并选择区域，然后单击页面底部的箭头。

	![从应用商店购买 MongoLab 数据库](./media/create-mongolab-mongodb/purchase-mongodb.png)

5. 单击复选标记完成购买。

	![查看和完成购买](./media/create-mongolab-mongodb/complete-mongolab-purchase.png)

6. 在创建数据库后，可以从管理门户中的"外接程序"选项卡管理该数据库。

	![在 Azure 门户中管理 MongoLab 数据库](./media/create-mongolab-mongodb/manage-mongolab-add-on.png)

7. 通过单击页面底部的"连接信息"（如上图中所示），您可以获得数据库连接信息。

	![MongoLab 连接信息](./media/create-mongolab-mongodb/mongolab-conn-info.png) 

[MongoLab]: https://mongolab.com/home
[waws]: /manage/services/web-sites/
[Azure 应用商店]: /zh-cn/store/overview/
[门户]: http://windows.azure.com/
<!--HONumber=41-->
