<properties pageTitle="Azure API 管理中的策略" metaKeywords="" description="了解如何在 API 管理创建、编辑和配置策略。" metaCanonical="" services="" documentationCenter="API Management" title="Azure API 管理中的策略" authors="sdanie" solutions="" manager="" editor="" />

# Azure API 管理中的策略

在 Azure API 管理（预览版）中，策略是一项强大的系统功能，允许发布者通过配置更改 API 的行为。策略是一组语句，在请求或 API 的响应时按顺序执行。流行的语句包括从 XML 到 JSON 的格式转换，并调用速率限制来限制从一名开发人员的传入调用。许多策略开箱即用。

请参阅[策略参考][策略参考]了解完整政策说明和他们的设置。

代理内部应用的策略介于 API 消费者和管理的 API 之间的代理。该代理接收所有请求，并通常将其原封不动地转发到基础 API 。但是策略可以将更改应用于入站的请求和出站响应。

## 如何配置策略

可全局范围内配置策略，或[产品][产品]的范围、[API][API]或[操作][操作]。要配置策略，导航到发布服务器门户中的策略编辑器。

![策略菜单][策略菜单]

该策略编辑器由三个主要部分组成：策略范围（上）、编辑策略的策略定义（左）和说明列表（右）：

![策略编辑器][策略编辑器]

要开始配置策略，必须首先选择该策略将应用的范围。在屏幕快照中选择 15 天免费试用版产品。请注意，策略名称旁的正方形符号表示某一策略已应用于此级别。

![范围][范围]

由于已应用策略，配置如定义视图中所示。

![配置][配置]

第一个显示只读的策略。为了编辑定义，请单击“配置策略”操作。

![编辑][编辑]

策略定义是一个简单的 XML 文档，用于描述一个入站和出站语句序列。可以直接在定义窗口中编辑 XML。右侧提供语句的列表，同时启用适用于当前范围的语句并突出显示；如上面的屏幕快照中的“限制调用速率”语句所示。

单击启用的语句将在定义视图中的光标位置添加相应的 XML。

策略语句的完整列表以及它们的设置在[策略参考][策略参考]中提供。

例如，要添加新的语句以限制到指定 IP 地址的入站请求，请将光标置于“入站”XML 元素的内容中，然后单击限制调用方 IP 语句。

![限制策略][限制策略]

这将 XML 代码段添加到“入站”元素，提供如何配置该语句的指导。

    <ip-filter action="allow | forbid">
        <address>address</address>
        <address-range from="address" to="address"/>
    </ip-filter>

要限制入站请求并接受来自 IP 地址 1.2.3.4 的那些，请修改 XML，如下所示：

    <ip-filter action="allow">
        <address>1.2.3.4</address>
    </ip-filter>

![保存][保存]

完成策略语句的配置后，单击“保存”，更改将立即传播到 API 管理代理。

# 了解策略配置

策略是一系列按请求和响应顺序执行的语句。配置相应地划分为一个入站（请求）和出站（策略），如配置所示。

    <policies>
        <inbound>
            <!-- statements to be applied to the request go here -->
        </inbound>
        <outboud>
            <!-- statements to be applied to the response go here -->
        <outbound>
    </policies>

由于策略可以在不同级别（全局、产品、api 和操作）指定，配置提供方法，让您指定此定义的语句和父策略执行的顺序。

例如，如果您在全局级别有一个策略并且为 API 配置一个策略，则只要使用该特定 API - 这两种策略都将被应用。API 管理允许通过基础元素实现组合策略声明的确定性排序。

    <policies>
        <inbound>
            <cross-domain />
            <base />
            <find-and-replace from="xyz" to="abc" />
        </inbound>
    </policies>

在上述示例策略定义中，跨域语句将在执行任何更高版本的策略前执行，之后是查找和替换策略。

注意：全局策略没有更高版本策略，*基础*元素始终是*不执行操作*，或不起作用。

  [策略参考]: /api-management-policy-reference
  [产品]: ../apimanagement-howto-add-products
  [API]: ../apimanagement-howto-add-apis
  [操作]: ../apimanagement-howto-add-operations
  [策略菜单]: ./media/api-management-howto-policies/api-management-policies-menu.png
  [策略编辑器]: ./media/api-management-howto-policies/api-management-policies-editor.png
  [范围]: ./media/api-management-howto-policies/api-management-policies-scope.png
  [配置]: ./media/api-management-howto-policies/api-management-policies-configure.png
  [编辑]: ./media/api-management-howto-policies/api-management-policies-edit.png
  [限制策略]: ./media/api-management-howto-policies/api-management-policies-restrict.png
  [保存]: ./media/api-management-howto-policies/api-management-policies-save.png
