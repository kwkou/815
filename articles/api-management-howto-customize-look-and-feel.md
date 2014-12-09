<properties pageTitle="如何在 Azure API 管理中自定义开发人员门户的外观" metaKeywords="" description="如何在 Azure API 管理中自定义开发人员门户的外观。" metaCanonical="" services="" documentationCenter="API Management" title="如何在 Azure API 管理中自定义开发人员门户的外观" authors="sdanie" solutions="" manager="" editor="" />

# 如何在 Azure API 管理中自定义开发人员门户的外观

由样式规则定义开发人员门户的外观和感觉的颜色、字体、大小、间距和其他方面。这些规则集存在页面的每个结构元素 - 标头、菜单、内容正文、页面标题等等。在本操作指南中，您将学习如何修改样式规则。

要编辑样式规则，请单击发布者门户中的**开发人员门户**菜单的**外观**。然后单击**开始自定义**以启用样式编辑器。

您的浏览器将切换到包含内容示例的开发人员门户中的隐藏页面，以及站点上任何位置使用的所有样式规则的示例。要打开样式编辑器，请将光标移到页面最左侧部分的灰色细竖线上。应显示编辑器工具栏。

![自定义工具栏][自定义工具栏]

有两种主要模式的编辑样式规则 - **编辑所有规则**显示任何位置使用的所有样式规则列表；同时 **Pick 元素**允许您从所在的页上选择一个元素，并将显示该元素样式。

本部分中我们只想更改特定元素的样式 - 例如，网页标头。单击样式编辑器工具栏的 **Pick 元素**选项，然后单击**选择要自定义的元素**。您将鼠标悬停在其上时元素变得突出显示，以表示您单击将看到的元素样式。

将鼠标移到标头中表示页面标题的文本，然后单击它。样式编辑器中将显示一组分类并命名的样式规则。

每个规则表示所选元素的样式属性。例如，对于上面选择的标头文本，文本的大小是 @font-size-h1，同时带替代项的字体名称在 @headings-font-family 中。

> 如果您熟悉 [bootstrap][http://getbootstrap.com/]，这些规则实际上是开发人员门户中使用的 bootstrap 主题中的 [LESS variables][http://getbootstrap.com/css/]。

让我们来更改标题文本的颜色。选择 **@headings-color** 字段中中的条目，键入 \#000000。这是黑色的十六进制代码。执行此操作，您将看到一个正方形颜色指示符出现在文本框中的末尾。如单击该指示符，颜色选取器将允许您选择一种颜色。

![颜色选取器][颜色选取器]

完成对所选元素的样式更改时，单击**预览更改**查看屏幕上的结果。这次它们只对管理员可见。要使这些更改对每个人都可见，请单击样式编辑器中的**发布**按钮，然后确认所做的更改。

![发布窗体][发布窗体]

> 要更改适用于页面上的任何其他元素的样式规则，请遵循对标头所做的相同程序 - 单击样式编辑器的**选取元素**，选择您感兴趣的元素，然后开始修改屏幕上显示的样式规则的值。

  [自定义工具栏]: ./media/api-management-howto-customize-look-and-feel/api-management-customization-toolbar.png
  [颜色选取器]: ./media/api-management-howto-customize-look-and-feel/api-management-customization-toolbar-color-picker.png
  [发布窗体]: ./media/api-management-howto-customize-look-and-feel/api-management-customization-toolbar-publish-form.png
