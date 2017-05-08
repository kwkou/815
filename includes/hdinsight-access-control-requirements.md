你可能会使用你不是管理员或所有者的 Azure 订阅，例如公司拥有的订阅。 如果是这样，必须确保已获得以下项，才能按照本文中的步骤进行操作：

* “参与者”访问权限。 若要登录到 Azure，至少需要有对 Azure 资源组的参与者访问权限。 此资源组用于创建 HDInsight 群集和其他 Azure 资源。
* 提供程序注册。 至少具有对 Azure 订阅的参与者访问权限的用户必须之前已注册所用资源的提供程序。 对订阅具有参与者访问权限的用户首次在订阅上创建资源时，会进行提供程序注册。 不创建资源也可[通过 REST 注册提供程序](https://msdn.microsoft.com/zh-cn/library/azure/dn790548.aspx)来完成该操作。

有关使用访问管理的详细信息，请参阅以下文章：

* [Azure 门户预览中的访问管理入门](/documentation/articles/role-based-access-control-what-is/)
* [使用角色分配管理对 Azure 订阅资源的访问权限](/documentation/articles/role-based-access-control-configure/)