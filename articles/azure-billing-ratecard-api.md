<properties
   pageTitle="获取 Azure 订阅中所用资源的价格和元数据信息 | Azure"
   description="了解如何获取 Azure 订阅中所用资源的价格和元数据信息"
   services=""
   documentationCenter=""
   authors=""
   manager=""
   editor=""
   tags="billing"/>  


<tags
   ms.service="billing"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/16/2017"
   wacn.date="01/16/2017"/>  

# 获取 Azure 订阅中所用资源的价格和元数据信息

您可以查询订阅中所用的资源/仪表元数据以及相关的价格，按照以下参数查询：

• 提供项 ID（标识）
• 货币
• 地域文化
• 地区

>[AZURE.IMPORTANT]与计费表有关的元数据，包括但不限于服务名称、类型、资源、计量单位和地区等，可能随时在未经通知的情况下变更。如果您打算以自动方式使用这些计费数据，请使用计费表 GUID（全局唯一标识符）识别每项可计费项目。如果由于采用新的计费模式而计划变更计费表 GUID，会提前通知您。

##请求
请见“[Resource RateCard（资源价格表）（预览）](https://msdn.microsoft.com/zh-cn/library/azure/mt219005.aspx)”中的“**通用参数和头信息**”小节，了解与 Resource RateCard API（应用程序编程接口）有关的所有请求使用的头信息和参数。

**方法** |**请求 URI（统一资源标识符）**
---|---
GET | `https://https://management.chinacloudapi.cn/subscriptions/{subscription-Id}/providers/Microsoft.Commerce/RateCard?api-version={api-version}&$filter=OfferDurableId eq '{OfferDurableId}' and Currency eq '{Currency}' and Locale eq '{Locale}' and RegionInfoeq '{RegionInfo}'`

• 把 {OfferDurableId}（提供项持久 ID）设定为一个有效的提供项 ID 代码（例如 MS-MC-AZR-33P）。请见 “[Azure优惠项目详情](/support/legal/offer-rate-plans/)”了解关于可用提供项 ID、国家/地区可用性和计费货币的更多信息。提供项 ID 参数包括 “MS-AZR-” 前缀再加上提供项 ID 编号。
• 把 {Currency}（货币）设定为用于表示资源价格的货币。
• 把 {Locale}（地域文化）设定为资源元数据需要面向的地域文化。
• 把 {RegionInfo}（地区信息）设定为提供项购买地区的双字母 ISO（国际标准化组织）代码。

>[AZURE.NOTE]请注意，需要全部 4 个查询参数。另外，$filter（过滤器）查询选项此时仅支持‘等于’和‘与’逻辑运算符。没有请求体。

## 响应

### 响应示例

下面是多种提供项 ID 和地区的 HTTP（超文本传输协议）响应的示例，显示了请求 URL 和查询参数的替代值，以及对应的 JSON（JavaScript 对象表示法）编码响应体。

下面的例子显示了“随收随付”提供项查询，其中**提供项持久ID = MS-MC-AZR-0033P，用于中国地区，价格用人民币表示，元数据面向美国英语（en-US）地域文化。**

    https://management.chinacloudapi.cn/subscriptions/{subscription-Id}/providers/Microsoft.Commerce/RateCard?api-version=2015-06-01-preview&$filter=OfferDurableId eq ’MS-MC-AZR-0033P’ and Currency eq ’CNY’ and Locale eq ’en-US’ and RegionInfoeq ’CN’

JSON

    {
    "OfferTerms": [],
    "Meters": [
        {
            "MeterId": "1822fcc4-6059-4cbb-a132-54a187aaac46",
            "MeterName": "Compute Hours",
            "MeterCategory": "Virtual Machines",
            "MeterSubCategory": "Basic_D6 VM (Non-Windows)",
            "Unit": "Hours",
            "MeterTags": [],
            "MeterRates": {
                "0": 3.136
            },
            "EffectiveDate": "2015-02-01T00:00:00Z",
            "IncludedQuantity": 0.0
        },
        {
            "MeterId": "3c5324ad-eb8c-44c6-af9a-6741ae75fc90",
            "MeterName": "Data Transfer Out at 500 Mbps (GB)",
            "MeterCategory": "Networking",
            "MeterSubCategory": "ExpressRoute (IXP)",
            "Unit": "GB",
            "MeterTags": [],
            "MeterRates": {
                "0": 0.1
            },
            "EffectiveDate": "2014-08-01T00:00:00Z",
            "IncludedQuantity": 2048.0
        },

        {
            "MeterId": "9ee077eb-c902-46ef-b7f9-2caeade852e0",
            "MeterName": "Compute Hours",
            "MeterCategory": "Cloud Services",
            "MeterSubCategory": "A6 Cloud Services",
            "Unit": "Hours",
            "MeterTags": [],
            "MeterRates": {
            "0": 0.71
            },
            "EffectiveDate": "2013-12-01T00:00:00Z",
            "IncludedQuantity": 0.0
        },
    …   
    ]
    "Currency": "CNY",
    "Locale": "en-US",
    "IsTaxIncluded": false,
        }

## JSON元素定义

下面列出了 HTTP 响应体中可能出现的 JSON 数据元素。

**元素名称** | **说明**
---|---
<p>Credit</p><p>信用</p> | 依据提供项条款提供的信用金额。此域仅供“货币承诺”类提供项条款使用。
<p>Currency</p><p>货币</p> | 用于表示价格的货币。
<p>EffectiveDate</p><p>生效日期</p> | 仪表价格或提供项条款从此日起生效。
<p>ExcludedMeterIds</p><p>排除的仪表ID</p> | 从提供项条款中排除的仪表 ID。
<p>IncludedQuantity</p><p>包含的数量</p> | 提供项中包含的免费资源数量。超出此数量会收费。
<p>IsTaxIncluded</p><p>是否含税</p> | 所有价格均为税前价格，因此返回的数值始终为“假”。
<p>Locale</p><p>地域文化</p> | 资源信息面向的地域文化。
<p>MeterCategory</p><p>仪表类别</p> | 仪表的类别，例如“云服务”、“联网”等。
<p>MeterId</p><p>仪表 ID</p> | 资源的唯一标识符。
<p>MeterName</p><p>仪表名称</p> | 仪表的名称，在给定的仪表类别内。
<p>MeterRates</p><p>仪表价格</p> | 仪表价格的键值对列表，采用“键”:“值”格式，其中键 = 仪表数量，值 = 对应的价格。
<p>MeterRegion</p><p>仪表地区</p> |	Azure 服务的可用地区。
<p>MeterSubCategory</p><p>仪表次级类别</p> | 仪表的次级类别，例如 “A6 云服务”、“ExpressRoute（特快路由）（IXP）”等。
<p>MeterStatus</p><p>仪表状态</p> |	仪表 ID 是“在用”还是“降级”。
<p>Name</p><p>名称</p> | Azure 提供项条款的名称，例如“货币信用”。
<p>ServiceRegion</p><p>服务地区</p> | Azure 服务的可用地区。
<p>Tags</p><p>标签</p> | <p>提供更多的仪表数据。</p><p>“第三方”表示无折扣的仪表。空白表示第一方。</p>
<p>TieredDiscount</p><p>分级折扣</p> | 分级仪表价格的键值对列表，采用“键”:“值”格式，其中键 = 价格，值 = 对应的折扣比例。此域仅供“货币承诺”类提供项条款使用。
<p>Unit</p><p>单位</p> | 仪表消费的计量单位，例如“小时”、“吉字节”等。

## 响应代码
**HTTP 状态代码** |	**错误代码** | **说明**
---|---|---
<p>200/OK</p><p>200/成功</p> |<p>n/a</p><p>无</p> | 成功查询的正常响应。响应体包含与指定过滤器匹配的数据。
<p>400/Bad Request</p><p>400/无效请求</p> |	<p>InvalidApiVersion</p><p>无效的 API 版本</p> |	不支持 API 版本查询参数或者格式不正确。
<p>400/Bad Request</p><p>400/无效请求</p> |	<p>InvalidProperty</p><p>无效的属性</p> |属性缺失或者含有不合格的数值。响应体中的错误“代码”伴有对应的“消息”，解释有问题的属性。
<p>400/Bad Request</p><p>400/无效请求</p> |	<p>NoApiVersion</p><p>没有 API 版本</p> | API 版本查询参数缺失。
<p>401/Unauthorized</p><p>401/未授权</p> |	<p>AuthenticationFailed</p><p>验证失败</p> | 通常由于使用无效或过期的访问令牌导致此错误。响应体中的错误“代码”伴有对应的“消息”，对错误进行解释。请见“[验证 Azure 资源管理器请求](https://msdn.microsoft.com/zh-cn/library/azure/dn790557.aspx)”，了解关于获取安全的访问令牌的详情。
<p>401/Unauthorized</p><p>401/未授权</p> |	<p>MissingAuthorization</p><p>验证缺失</p> | 授权头信息缺失或不完整。请见“[验证 Azure 资源管理器请求](https://msdn.microsoft.com/zh-cn/library/azure/dn790557.aspx)”，了解关于获取安全的访问令牌的详情。
<p>404/Not Found</p><p>404/未发现</p> | <p>InvalidResourceType</p><p>无效的资源类型</p> |	API 版本参数中的版本不正确。响应中的错误“代码”伴有对应的“消息”，对错误进行解释。
<p>404/Not Found</p><p>404/未发现</p> | <p>ObjectNotFound</p><p>未发现对象</p> | 客户没有授权。
<p>404/Not Found</p><p>404/未发现</p> | <p>SubscriptionNotFound</p><p>未发现订阅</p> | 请求 URL 中的订阅 ID 值无效或者未发现订阅 ID 值。


想了解在境外其它国家由微软运营的Azure 服务，请访问[微软 Azure 提供的项目详情](http://azure.microsoft.com/en-us/support/legal/offer-details/)。



