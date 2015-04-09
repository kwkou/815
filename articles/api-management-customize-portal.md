<properties pageTitle="在 Azure API 管理中自定义开发人员门户" metaKeywords="" description="在 Azure API 管理中自定义开发人员门户。" metaCanonical="" services="" documentationCenter="API Management" title="在 Azure API 管理中自定义开发人员门户" authors="sdanie" solutions="" manager="" editor="" />

# 在 Azure API 管理中自定义开发人员门户

本指南演示如何修改 API 管理（预览版）中开发人员门户的外观，与自己的品牌保持一致。

## 本主题内容

-   [更改页面标头中的文本/徽标][更改页面标头中的文本/徽标]
-   [更改标头的样式][更改标头的样式]
-   [编辑页面的内容][编辑页面的内容]
-   [后续步骤][后续步骤]

## <a name="change-page-headers"> </a>更改页面标头中的文本/徽标

门户自定义的关键之一是用您公司名称或徽标来替换所有页面顶部的文本。

通过发布者门户修改开发人员门户中的内容，可通过 Azure 管理门户进行访问。要访问 API 管理控制台，请为 API 管理服务单击 Azure 门户中的**管理控制台**。

![管理控制台][管理控制台]

开发人员门户基于内容管理系统或 CMS。每一页显示的标题是内容的一种特殊类型，成为小组件。要编辑小组件的内容，请单击左侧**开发人员门户**菜单上的**小组件**，然后从列表选择**标头**小组件。

![小组件标头][小组件标头]

标头的内容可从**正文**字段编辑。将文本转换为“Fabrikam 开发人员门户”，然后单击页面底部的**保存**。

现在，您应能够在开发人员门户中的每一页上查看新的标头！

> 要打开管理控制台中的开发人员门户，请单击顶栏中的**开发人员门户**。

## <a name="change-headers-styling"> </a>更改标头的样式

由样式规则定义的门户上的任何页面的颜色、字体、大小、间距和其他样式相关的元素。要编辑样式，请单击发布者门户中的**开发人员门户**菜单的**外观**。然后单击**开始自定义**以启用样式编辑器。

您的浏览器将切换到包含内容示例的开发人员门户中的隐藏页面，以及站点上任何位置使用的所有样式规则的示例。要打开样式编辑器，请将光标移到页面最左侧部分的灰色细竖线上。应显示编辑器工具栏。

![自定义工具栏][自定义工具栏]

有两种主要模式的编辑样式规则 - **编辑所有规则**显示任何位置使用的所有样式规则列表；同时 **Pick 元素**允许您从所在的页上选择一个元素，并仅显示该元素样式。

本部分中我们想要仅更改标头的样式。单击样式编辑器工具栏的 **Pick 元素**选项，然后单击**选择要自定义的元素**。您将鼠标悬停在其上时元素变得突出显示，以表示您单击将开始编辑的元素样式。将鼠标移到标头中表示公司名称的文本（为“Fabrikam 开发人员门户”，如果您遵循上一节中的说明），然后单击它。样式编辑器中将显示一组分类并命名的样式规则。

每个规则表示所选元素的样式属性。例如，对于上面选择的标头文本，文本的大小是 @font-size-h1，同时带替代项的字体名称在 @headings-font-family 中。

> 如果您熟悉 [bootstrap][http://getbootstrap.com/]，这些规则实际上是开发人员门户中使用的 bootstrap 主题中的 [LESS variables][http://getbootstrap.com/css/]。

让我们来更改标题文本的颜色。选择 **@headings-color** 字段中中的条目，键入 \#000000。这是黑色的十六进制代码。执行此操作，您将看到一个正方形颜色指示符出现在文本框中的末尾。如单击该指示符，颜色选取器将允许您选择一种颜色。

![颜色选取器][颜色选取器]

完成对所选元素的样式更改时，单击**预览更改**查看屏幕上的结果。这次它们只对管理员可见。要使这些更改对每个人都可见，请单击样式编辑器中的**发布**按钮，然后确认所做的更改。

[Publish menu][api-management-customization-toolbar-publish-menu]

> 要更改适用于页面上的任何其他元素的样式规则，请遵循对标头所做的相同程序 - 单击样式编辑器的**选取元素**，选择您感兴趣的元素，然后开始修改屏幕上显示的样式规则的值。

## <a name="edit-page-contents"> </a>编辑页面的内容

开发人员门户由自动生成的页面组成，如 API、产品、应用程序、问题和手动编写的内容。由于它基于内容管理系统，您可根据需要创建此类的内容。

要查看所有现有内容页面的列表，请单击管理控制台中**开发人员门户**菜单的**内容**。

![管理内容][管理内容]

单击“欢迎”页面以编辑开发人员门户主页上要显示的内容。按所需进行更改，如有必要预览它们，然后单击**立即发布**使它们对每个人可见。

> 主页使用特殊的布局，允许它在顶部显示横幅。从“内容”部分不能编辑此横幅。要编辑此横幅，请单击**开发人员门户**菜单的**小组件**，从**当前层**下拉列表选择**主页**，然后打开“特色”部分下的**横幅**项。此小组件的内容就像任何其他页一样都可编辑。

## <a name="next-steps"> </a>后续步骤

-   查看[开始使用高级 API 配置][开始使用高级 API 配置]教程中的其他主题。

  [更改页面标头中的文本/徽标]: #change-page-headers
  [更改标头的样式]: #change-headers-styling
  [编辑页面的内容]: #edit-page-contents
  [后续步骤]: #next-steps
  [管理控制台]: ./media/api-management-customize-portal/api-management-management-console.png
  [小组件标头]: ./media/api-management-customize-portal/api-management-widgets-header.png
  [自定义工具栏]: ./media/api-management-customize-portal/api-management-customization-toolbar.png
  [颜色选取器]: ./media/api-management-customize-portal/api-management-customization-toolbar-color-picker.png
  [管理内容]: ./media/api-management-customize-portal/api-management-customization-manage-content.png
  [开始使用高级 API 配置]: ../api-management-get-started-advanced
