<properties pageTitle="如何创建和使用组来管理 Azure API 管理中的开发人员帐户" metaKeywords="" description="了解如何使用 Azure API 管理中的组管理开发人员帐户" metaCanonical="" services="" documentationCenter="API Management" title="如何创建和使用组来管理 Azure API 管理中的开发人员帐户" authors="sdanie" solutions="" manager="" editor="" />

# 如何创建和使用组来管理 Azure API 管理中的开发人员帐户

在 API 管理（预览版）中，使用组来管理产品对开发人员的可见性。产品首次对组可见，然后这些组中的开发人员可以查看和订阅与组关联的产品。

API 管理具有下列内置组。

-   **管理员** - 管理员管理 API 管理服务实例、创建 API、操作和开发人员所使用的产品。
-   **开发人员** - 开发人员是使用您的 API 构建应用程序的客户。开发人员有权访问开发人员门户，并构建调用 API 操作的应用程序。
-   **来宾** - 未经身份验证的用户，如访问此组中 API 管理实例的开发人员门户的潜在客户。它们可以被授予某些只读访问权限，如能够查看 API，但不能调用它们。

除了这些内置组，管理员可以创建自定义组。自定义组具有和内置开发人员组相同的权限，并可以用于管理开发人员的多个组。例如，可以为使用来自一个产品的 API 的开发人员创建一个自定义组，为将使用不同产品的 API 的开发人员创建另一个组。

本指南演示 API 管理实例的管理员如何添加新组并将它们关联产品和开发人员。

## 本主题内容

-   [创建组][创建组]
-   [将组与产品关联][将组与产品关联]
-   [将组与开发人员关联][将组与开发人员关联]
-   [后续步骤][后续步骤]

## <a name="create-group"> </a>创建组

要创建新组，请单击 API 管理服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

单击左侧 **API 管理**菜单的**组**，然后单击**添加组**。

![添加新组][添加新组]

输入组和可选说明的唯一名称，然后单击**保存**。

![添加新组][1]

新组显示在组选项卡中。要编辑组的**名称**或**说明**，请单击列表中组的名称。要删除组，请单击**删除**。

![添加的组][添加的组]

现在已创建组，它可以与产品和开发人员相关联。

## <a name="associate-group-product"> </a>将组与产品关联

要将某一组与产品相关联，请单击左侧 **API 管理**菜单的**产品**，然后单击所需产品的名称。

![设置可见性][设置可见性]

选择**可见性**选项卡添加和删除组，并查看有关该产品的当前组。要添加或删除组，选中或取消所需组的复选框，然后单击**保存**。

![设置可见性][2]

> 要从产品的**可见性**选项卡配置组，请单击**管理组**。

一种产品与组相关联后，该组中的开发人员可以查看和订阅该产品。

## <a name="associate-group-developer"> </a>将组与开发人员关联

要将组与开发人员相关联，请单击左侧 **API 管理**菜单的**开发人员**，然后选中您想和一组相关联的开发人员旁边的框。

![将开发人员添加到组][将开发人员添加到组]

选中所需的开发人员后，请单击**添加到组**下拉列表中所需的组。使用**从组中删除**下拉列表可以从组中移除开发人员。

![开发人员][开发人员]

一旦开发人员和组之间添加关联，您可以在**开发人员**选项卡中查看它。

## <a name="next-steps"> </a>后续步骤

一旦开发人员添加到组，他们可以查看和订阅与该组关联的产品。有关详细信息，请参阅[如何在 Azure API 管理中创建和发布产品][如何在 Azure API 管理中创建和发布产品]。

  [创建组]: #create-group
  [将组与产品关联]: #associate-group-product
  [将组与开发人员关联]: #associate-group-developer
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-create-groups/api-management-management-console.png
  [添加新组]: ./media/api-management-howto-create-groups/api-management-add-group.png
  [1]: ./media/api-management-howto-create-groups/api-management-add-group-window.png
  [添加的组]: ./media/api-management-howto-create-groups/api-management-new-group.png
  [设置可见性]: ./media/api-management-howto-create-groups/api-management-add-group-to-product.png
  [2]: ./media/api-management-howto-create-groups/api-management-add-group-to-product-visibility.png
  [将开发人员添加到组]: ./media/api-management-howto-create-groups/api-management-add-group-to-developer.png
  [开发人员]: ./media/api-management-howto-create-groups/api-management-add-group-to-developer-saved.png
  [如何在 Azure API 管理中创建和发布产品]: ../api-management-howto-add-products
