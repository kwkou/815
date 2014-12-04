<properties pageTitle="Azure API Management Policy Reference" metaKeywords="" description="Learn about the policies available to configure API Management." metaCanonical="" services="" documentationCenter="API Management" title="Azure API Management Policy Reference" authors="sdanie" solutions="" manager="" editor="" />

# Azure API 管理策略参考

本主题提供以下 API 管理（预览版）策略的参考信息。有关添加和配置策略的信息，请参阅 [API 管理中的策略][API 管理中的策略]。

-   [访问限制策略][访问限制策略]

    -   [用量配额][用量配额] - 可让你强制实施可续订或生存期调用量和/或带宽配额。
    -   [速率限制][速率限制] - 通过限制调用和/或带宽占用率来防止 API 用量尖峰。
    -   [调用方 IP 限制][调用方 IP 限制] - 筛选（允许/拒绝）来自特定 IP 地址和/或地址范围的调用。
-   [内容转换策略][内容转换策略]

    -   [设置 HTTP 标头][设置 HTTP 标头] - 向现有的响应和/或请求标头赋值，或者添加新的响应和/或请求标头。
    -   [将 XML 转换为 JSON][将 XML 转换为 JSON] - 将请求或响应正文从 XML 转换为“JSON”或“遵从 XML”格式的 JSON。
    -   [替换正文中的字符串][替换正文中的字符串] - 查找请求或响应子字符串并将其替换为不同的子字符串。
    -   [设置查询字符串参数][设置查询字符串参数] - 添加、删除请求查询字符串参数或替换其值。
-   [缓存策略][缓存策略]

    -   [存储到缓存][存储到缓存] - 根据指定的缓存配置来缓存响应。
    -   [从缓存中获取][从缓存中获取] - 执行缓存查找，并返回有效的缓存响应（如果有）。
-   [其他策略][其他策略]

    -   [重写 URI][重写 URI] - 将请求 URL 从其公用格式转换为 Web 服务所需的格式。
    -   [在内容中屏蔽 URLS][在内容中屏蔽 URLS] - 重写（屏蔽）响应正文和位置标头中的链接，使其通过代理指向等效的链接。
    -   [允许跨域调用][允许跨域调用] - 使 API 能够通过 Adobe Flash 和基于 Microsoft Silverlight 浏览器的客户端进行访问。
    -   [JSONP][JSONP] - 向操作或 API 添加填充型 JSON (JSONP) 支持，以便从基于 JavaScript 浏览器的客户端执行跨域调用。
    -   [CORS][CORS] - 向操作或 API 添加跨源资源共享 (CORS) 支持，以便从基于浏览器的客户端执行跨域调用。

## <a name="access-restriction-policies"> </a>访问限制策略

-   [用量配额][用量配额] - 可让你强制实施可续订或生存期调用量和/或带宽配额。
-   [速率限制][速率限制] - 通过限制调用和/或带宽占用率来防止 API 用量尖峰。
-   [调用方 IP 限制][调用方 IP 限制] - 筛选（允许/拒绝）来自特定 IP 地址和/或地址范围的调用。

### <a name="usage-quota"> </a>用量配额

**说明：**
强制实施可续订或生存期调用量和/或带宽配额。

**策略语句：**

    <quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
        <api name="name" calls="number" bandwidth="kilobytes" renewal-period="seconds">
            <operation name="name" calls="number" bandwidth="kilobytes" renewal-period="seconds" />
        </api>
    </quota>

**示例：**

    <policies>
        <inbound>
            <base />
            <quota calls="10000" bandwidth="40000" renewal-period="3600" />
        </inbound>
        <outbound>
            <base />
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
在 Product 范围的入站部分中使用。

**可在哪个位置应用该策略：**
当要限制产品的操作和/或 API 用量时。

**为何要或不要应用该策略：**
该策略可以基于时间和带宽定义产品的用量限制。当 API 达到定义的配额限制时，后续调用将被拒绝并导致出现错误消息。配额通常定义为在预定义的整个时间间隔内允许的请求消息数和带宽限制。

| 属性                     | 说明                                                                     |
|--------------------------|--------------------------------------------------------------------------|
| quota calls="number"     | 在整个续订期内允许的最大订阅调用次数（例如 10000)                        |
| bandwidth="kilobytes"    | 在指定的整个续订期内允许此产品、API 或操作使用的最大字节数（例如 40000). |
| renewal-period="seconds" | 应用配额的时间段，以秒为单位（例如 3600）。                              |
| api name="name"          | 要向其应用配额的 API 调用名称。                                          |
| calls="number"           | 在指定的续订期内可对 API 或操作执行的最大调用次数（例如 5000).           |
| operation name="name"    | 要向其应用配额的操作名称。                                               |

### <a name="rate-limit"> </a>速率限制

**说明：**
通过限制流向 API（在某些情况下还包括特定的 API 操作）的请求速率来遏制 API 用量尖峰。触发策略时，使用者将会收到`429 Too Many Requests` 响应状态代码。

**策略语句：**

    <rate-limit calls="number" renewal-period="seconds">
        <api name="name" calls="number" renewal-period="seconds">
            <operation name="name" calls="number" renewal-period="seconds" />
        </api>
    </rate-limit>

**示例：**

    <policies>
        <inbound>
            <base />
            <rate-limit calls="20" renewal-period="90" />
        </inbound>
        <outbound>
            <base />        
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
在 product 范围的入站部分中使用。

| 元素/属性                | 说明                                     |
|--------------------------|------------------------------------------|
| calls="number"           | 每个续订期允许的调用次数                 |
| renewal-period="seconds" | 续订速率限制调用配额之前所要经过的时间段 |
| api name="name"          | 速率限制应用到的 API 名称                |
| operation name="name"    | 速率限制应用到的操作名称                 |

### <a name="caller-ip-restriction"> </a>调用方 IP 限制

**说明：**
筛选（允许/拒绝）来自特定 IP 地址和/或地址范围的调用。

**策略语句：**

    <ip-filter action="allow | forbid">
        <address>address</address>
        <address-range from="address" to="address" />
    </ip-filter>

**示例：**

    <ip-filter action="allow | forbid">
        <address>address</address>
        <address-range from="address" to="address" />
    </ip-filter>

**可在哪个位置应用该策略：**
在任何范围的入站部分中使用。

**何时应该应用该策略：**
当需要对谁可以访问你的服务进行具体控制时，可使用此策略。

**为何要或不要应用该策略：**
仅当需要根据每台主机或一系列主机严格控制访问权限时（例如，对于安全要求较高的服务），才需要应用此策略。


<table>
<thead>
<tr>
  <th>属性</th>
  <th>说明</th>
</tr>
</thead>
<tbody>
<tr>
  <td>ip-filter action="allow | forbid"</td>
  <td>指定是否应允许指定的地址执行调用。</td>
</tr>
<tr>
  <td>address="address"</td>
  <td>允许或拒绝其访问的一个或多个 IP 地址。</td>
</tr>
<tr>
  <td>address-range from="address" to="address"</td>
  <td>允许或拒绝其访问的某个 IP 地址范围。</td>
</tr>
</tbody>
</table>

## <a name="content-transformation-policies"> </a>内容转换策略

-   [设置 HTTP 标头][设置 HTTP 标头] - 向现有的响应和/或请求标头赋值，或者添加新的响应和/或请求标头。
-   [将 XML 转换为 JSON][将 XML 转换为 JSON] - 将请求或响应正文从 XML 转换为“JSON”或“遵从 XML”格式的 JSON。
-   [替换正文中的字符串][替换正文中的字符串] - 查找请求或响应子字符串并将其替换为不同的子字符串。
-   [设置查询字符串参数][设置查询字符串参数] - 添加、删除请求查询字符串参数或替换其值。

### <a name="set-http-headers"> </a>设置 HTTP 标头

**说明：**
向现有的响应和/或请求标头赋值，或者添加新的响应和/或请求标头。

在 HTTP 消息中插入 HTTP 标头列表。将此策略放到入站管道中后，它将为传递给目标服务的请求设置 HTTP 标头。将此策略放到出站管道中后，它将为发送到代理客户端的响应设置 HTTP 标头。

**策略语句：**

    <set-headers reconcile-action="override | skip |append">
        <header name="header name" exists-action="override | skip | append">
        <value>value</value> <!--for multiple headers with the same name add additional value elements-->
        </header> 
    </set-headers>

**示例：**

    <set-headers exists-action="override">
        <header name="some value to set" exists-action="override">
        <value>20</value> 
        </header>
    </set-headers>

**可在哪个位置应用该策略：**
在任何范围（*Publisher* 除外）的入站和出站部分中使用。

**何时应该应用该策略：**
用于指定操作参数和/或 HTTP 事务的返回值。

**为何要或不要应用该策略：**
大多数 HTTP 请求和许多响应都需要使用标头来指定输入/响应参数。因此，很有可能此策略几乎适用于你的所有服务。

<table>
<thead>
<tr>
  <th>属性</th>
  <th>说明</th>
</tr>
</thead>
<tbody>
<tr>
  <td>reconcile-action="override|skip|append"</td>
  <td>指定当标头已指定时要执行的操作。注意：如果设置为“override”，则登记多个同名的条目会导致根据所有条目（将多次列出）设置标头；结果中只会设置列出的值。</td>
</tr>
<tr>
  <td>header name="header name"</td>
  <td>指定要设置的标头的名称。必需。</td>
</tr>
<tr>
  <td>exists-action="override | skip | append"</td>
  <td>指定当标头已指定时要执行的操作</td>
</tr>
<tr>
  <td>value="value"</td>
  <td>指定要设置的标头的值。</td>
</tr>
</tbody>
</table>
### <a name="convert-xml-to-json"> </a>将 XML 转换为 JSON

**说明：**
将请求或响应正文从 XML 转换为 JSON。

**策略语句：**

    <xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>

**示例：**

    <policies>
        <inbound>
            <base />
        </inbound>
        <outbound>
            <base />
            <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
在 API 或 operation 范围的入站或出站部分中使用。

**为何要或不要应用该策略：**
该策略可以根据仅用 XML 的后端 Web 服务来提升 API。



<table>
<thead>
<tr>
  <th>属性</th>
  <th>说明</th>
</tr>
</thead>
<tbody>
<tr>
  <td>kind="javascript-friendly | direct</td>
  <td>转换后的 JSON 采用 JavaScript 开发人员所熟知的格式 | 转换后的 JSON 将反映原始 XML 文档的结构</td>
</tr>
<tr>
  <td>apply= always | content-type-xml</td>
  <td>始终转换 | 仅当 <code>Content-Type</code> 标头指示使用了 XML 时才转换</td>
</tr>
<tr>
  <td>consider-accept-header="true | false"</td>
  <td>基于 <code>Accept</code> 标头的值应用转换 | 始终转换</td>
</tr>
</tbody>
</table>

### <a name="replace-string-in-body"> </a>替换正文中的字符串

**说明：**
查找请求或响应中的子字符串并将其替换为不同的字符串。

**策略语句：**

    <find-and-replace from="what to replace" to="replacement" />

**示例：**

    <find-and-replace from="notebook" to="laptop" />

**可在哪个位置应用该策略：**
在任何范围的入站和出站部分中使用。

| 属性                   | 说明                             |
|------------------------|----------------------------------|
| from="what to replace" | 要搜索的字符串。                 |
| to="replacement"       | 用于取代上面找到的结果的字符串。 |

### <a name="set-query-string-parameters"> </a>设置查询字符串参数

**说明：**
添加、删除请求查询字符串参数或替换其值。

**策略语句：**

    <set-query-parameters>
        <parameter name="param name" exists-action="override | skip | append | delete">
    </set-query-parameters>

**示例：**

    <policies>
        <inbound>
            <base />
            <set-query-parameters>
                <parameter name="api-key" exists-action="skip">
                    <value>12345678901</value>
                </parameter>
                <!-- for multiple parameters with the same name add additional value elements -->
            </set-query-parameters>
        </inbound>
        <outbound>
            <base />
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
可在任何范围的入站部分中使用。

**为何要或不要应用该策略：**
用于传递后端服务所需的查询参数，这些参数是可选的或者永远不能出现在请求中。

| 元素/属性                | 说明                                   |
|--------------------------|----------------------------------------|
| name="name"              | 用来为参数命名的字符串                 |
| exists-action="override" | 替换参数的值（如果该参数在请求中出现） |
| exists-action="skip"     | 当该参数在请求中出现时不执行任何操作   |
| exists-action="append"   | 将值附加到现有的请求参数               |
| exists-action="delete"   | 从请求中删除参数\*                     |
| value="value"            | 在包容元素中设置参数的值               |

## <a name="caching-policies"> </a>缓存策略

-   [存储到缓存][存储到缓存] - 根据指定的缓存配置来缓存响应。
-   [从缓存中获取][从缓存中获取] - 执行缓存查找，并返回有效的缓存响应（如果有）。

### <a name="store-to-cache"> </a>存储到缓存

**说明：**
根据指定的缓存设置来缓存响应。必须已设置相应的[从缓存中获取][从缓存中获取]策略。

**策略语句：**

    cache-store duration="seconds" caching-mode="cache-on | do-not-cache" />

**示例：**

    <policies>
        <inbound>
            <base />
              <cache-lookup vary-by-developer=*"true | false"* vary-by-developer-groups=*"true | false"* downstream-caching-type=*"none | private | public"*>
                    <vary-by-header>Accept</vary-by-header> <!-- should be present in most cases -->
                    <vary-by-header>Accept-Charset</vary-by-header> <!-- should be present in most cases -->
                    <vary-by-header>header name</vary-by-header> <!-- optional, can repeated several times -->
                    <vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->
            </cache-lookup>
        </inbound>
        <outbound>
            <base />
                <cache-store duration="3600" caching-mode="cache-on" />
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
可以在 API 和/或 operation 范围的出站部分中出现。

**何时应该应用该策略：**
当响应内容在某个时间段内保持静态时，就应该应用该策略。

**为何要或不要应用该策略：**
响应缓存可以降低后端 Web 服务器需要满足的带宽和处理能力要求，并可以减小 API 使用者能够察觉到的延迟。


<table>
<thead>
<tr>
  <th>元素/属性</th>
  <th>说明</th>
</tr>
</thead>
<tbody>
<tr>
  <td>seconds</td>
  <td>缓存条目的生存时间（以秒为单位，默认值为 3600）</td>
</tr>
<tr>
  <td>cache-on | do-not-cache</td>
  <td>缓存响应 | 不缓存响应，并将缓存相关的标头设置为禁止下游缓存</td>
</tr>
</tbody>
</table>

### <a name="get-from-cache"> </a>从缓存中获取

**说明：**
执行缓存查找，并返回有效的缓存响应（如果有）。正确响应来自 API 使用者的缓存验证请求。必须已设置相应的[存储到缓存][存储到缓存]策略。

**策略语句：**

    <cache-lookup vary-by-developer=*"true | false"* vary-by-developer-groups=*"true | false"* downstream-caching-type=*"none | private | public"*>
        <vary-by-header>Accept</vary-by-header> <!-- should be present in most cases -->
        <vary-by-header>Accept-Charset</vary-by-header> <!-- should be present in most cases -->
        <vary-by-header>header name</vary-by-header> <!-- optional, can repeated several times -->
        <vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->
    </cache-lookup>

**示例：**

    <policies>
        <inbound>
            <base />
            <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="none">
                <vary-by-query-parameter>version</vary-by-query-parameter>
                <vary-by-header>Accept</vary-by-header>
                <vary-by-header>Accept-Charset</vary-by-header>
            </cache-lookup>         
        </inbound>
        <outbound>
            <cache-store duration="seconds" caching-mode="cache-on | do-not-cache" />
            <base />        
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
可以在 API 和/或 operation 范围的入站部分中出现。

**何时应该应用该策略：**
当响应内容在某个时间段内保持静态时，就应该应用该策略。

**为何要或不要应用该策略：**
响应缓存可以降低后端 Web 服务器需要满足的带宽和处理能力要求，并可以减小 API 使用者能够察觉到的延迟。


<table>
<thead>
<tr>
  <th>元素/属性</th>
  <th>说明</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vary-by-developer="true | false"</td>
  <td>设置为 *true* 会根据开发人员密钥开始缓存响应；默认情况下设置为 *false*</td>
</tr>
<tr>
  <td>vary-by-developer-groups="true | false"</td>
  <td>设置为 *true* 会根据用户角色开始缓存响应；默认情况下设置为 *false*</td>
</tr>
<tr>
  <td>downstream-caching-type="none | private | public"</td>
  <td>*none* - 不允许下游缓存；这是默认设置 | *private* - 允许下游专用缓存 | *public* - 允许专用和共享下游缓存。</td>
</tr>
<tr>
  <td>vary-by-header: "Accept"</td>
  <td>根据 <code>Accept</code> 标头的值开始缓存响应</td>
</tr>
<tr>
  <td>vary-by-header: Accept-Charset"</td>
  <td>根据 <code>Accept-Charset</code> 标头的值开始缓存响应</td>
</tr>
<tr>
  <td>vary-by-header: "header name"</td>
  <td>根据指定标头的值开始缓存响应，例如 <code>Accept | Accept-Charset | Accept-Encoding | Accept-Language | Authorization | Expect | From | Host | If-Match</code> </td>
</tr>
<tr>
  <td>vary-by-query-parameter: "parameter name"</td>
  <td>根据指定查询参数的值开始缓存响应。请输入一个或多个参数。使用分号作为分隔符。如果未指定任何参数，将使用所有查询参数。</td>
</tr>
</tbody>
</table>

## <a name="other-policies"> </a>其他策略

-   [重写 URI][重写 URI] - 将请求 URL 从其公用格式转换为 Web 服务所需的格式。
-   [在内容中屏蔽 URLS][在内容中屏蔽 URLS] - 重写（屏蔽）响应正文和位置标头中的链接，使其通过代理指向等效的链接。
-   [允许跨域调用][允许跨域调用] - 使 API 能够通过 Adobe Flash 和基于 Microsoft Silverlight 浏览器的客户端进行访问。
-   [JSONP][JSONP] - 向操作或 API 添加填充型 JSON (JSONP) 支持，以便从基于 JavaScript 浏览器的客户端执行跨域调用。
-   [CORS][CORS] - 向操作或 API 添加跨源资源共享 (CORS) 支持，以便从基于浏览器的客户端执行跨域调用。

### <a name="rewrite-uri"> </a>重写 URI

**说明：**
将请求 URL 从其公用格式转换为 Web 服务所需的格式。

**公用 URL：** <http://api.example.com/storenumber/ordernumber>

**请求 URL：** <http://api.example.com/v2/US/hardware/storenumber&ordernumber?City&State>。

请不要在重写 URL 中添加额外的模板路径参数。只能添加查询字符串参数。

**策略语句：**

    <rewrite-uri template="uri template" />

**示例：**

    <policies>
        <inbound>
            <base />
            <rewrite-uri template="/v2/US/hardware/{storenumber}&{ordernumber}?City=city&State=state" />
        </inbound>
        <outbound>
            <base />
        </outbound>
    </policies>

**可在哪个位置应用该策略：**
只能在 Operation 范围的入站部分中使用。

**何时应该应用该策略：**
如果要将用户和/或浏览器友好的 URL 转换成 Web 服务所需的 URL 格式，就应该使用该策略。

**为何要或不要应用该策略：**
仅当公开备用 URL 格式时才需要应用该策略。简洁 URL、RESTful URL、用户友好的 URL 或 SEO 友好的 URL 是纯结构化 URL，不包含查询字符串，而只包含资源的路径（在方案和颁发机构的后面）。通常会出于美观、可用性或搜索引擎优化 (SEO) 目的使用这种 URL。

| 属性                    | 说明                                      |
|-------------------------|-------------------------------------------|
| template="uri template" | 包含任何查询字符串参数的实际 Web 服务 URL |

### <a name="mask-urls-in-content"> </a>在内容中屏蔽 URLS

**说明：**
重写（屏蔽）响应正文和位置标头中的链接，使其通过代理指向等效的链接。

**策略语句：**

    <redirect-body-urls />

**示例：**

    <redirect-body-urls />

**可在哪个位置应用该策略：**
在 *API* 和 *Operation* 范围中使用。可以在入站和出站部分中应用。

### <a name="allow-cross-domain-calls"> </a>允许跨域调用

**说明：**
使 API 能够通过 Adobe Flash 和基于 Microsoft Silverlight 浏览器的客户端进行访问。

**策略语句：**

    <cross-domain />

**示例：**

    <cross-domain />

**可在哪个位置应用该策略：**
在 *Publishers* 范围的入站部分中使用。

### <a name="jsonp"> </a>JSONP

**说明：**
向操作或 API 添加填充型 JSON (JSONP) 支持，以便从基于 JavaScript 浏览器的客户端执行跨域调用。JSONP 是 JavaScript 程序中使用的方法，用于从不同域中的服务器请求数据。JSONP 规避了大多数 Web 浏览器强制实施的只能在同一域中访问网页的限制。

**策略语句：**

    <jsonp callback-parameter-name="callback function name" />

**示例：**

    <jsonp callback-parameter-name="cb" />

如果在调用该方法时未指定回调参数`?cb=XXX` ，该方法将返回无格式 JSON（不带函数调用包装）。

如果添加了回调参数`?cb=XXX` ，它将返回 JSONP 结果，并使用原始 JSON 结果包装回调函数，例如`XXX('<json result goes here>')`;

**可在哪个位置应用该策略：**
只能在出站部分中使用。

**何时应该应用该策略：**
从不同域中的服务器请求数据时。

| 属性                    | 说明                                                       |
|-------------------------|------------------------------------------------------------|
| callback-parameter-name | 以函数所在的完全限定域名为前缀的跨域 JavaScript 函数调用。 |

### <a name="cors"> </a>CORS

**说明：**
向操作或 API 添加跨源资源共享 (CORS) 支持，以便从基于浏览器的客户端执行跨域调用。

CORS 允许浏览器与服务器交互，并确定是否允许特定的跨源请求（例如，通过某个网页上的 JavaScript 对其他域执行 XMLHttpRequests 调用）。与只允许同源请求相比，它的灵活性更高，而且比允许所有跨源请求更安全。

**策略语句：**

     <cors>
        <allowed-origins>
            <origin>*</origin> <!-- allow any -->
            <!-- OR a list of one or more specific URIs (case-sensitive) -->
            <origin>URI</origin> <!-- URI must include scheme, host, and port. If port is omitted, 80 is assumed for http and 443 is assumed for https. -->
        </allowed-origins>
     </cors>

**示例：**

    <cors>
        <allowed-origins>
            <origin>*</origin> <!-- allow any -->
            <!-- OR a list of one or more specific URIs (case-sensitive) -->
            <origin>http://contoso.com:81</origin> <!-- URI must include scheme, host, and port. If port is omitted, 80 is assumed for http and 443 is assumed for https. -->
        </allowed-origins>
    </cors>

**可在哪个位置应用该策略：**
在 *API* 和 *Operation* 范围的入站部分中使用。

| 属性                 | 说明                                                                                        |
|----------------------|---------------------------------------------------------------------------------------------|
| <origin>\*</origin>  | 允许任何 URI，或者允许一个或多个特定 URI 的列表                                             |
| <origin>URI</origin> | URI 必须包括方案、主机和端口。如果省略端口，则会假定 http 使用端口 80，https 使用端口 443。 |

  [API 管理中的策略]: ../api-management-howto-policies
  [访问限制策略]: #access-restriction-policies
  [用量配额]: #usage-quota
  [速率限制]: #rate-limit
  [调用方 IP 限制]: #caller-ip-restriction
  [内容转换策略]: #content-transformation-policies
  [设置 HTTP 标头]: #set-http-headers
  [将 XML 转换为 JSON]: #convert-xml-to-json
  [替换正文中的字符串]: #replace-string-in-body
  [设置查询字符串参数]: #set-query-string-parameters
  [缓存策略]: #caching-policies
  [存储到缓存]: #store-to-cache
  [从缓存中获取]: #get-from-cache
  [其他策略]: #other-policies
  [重写 URI]: #rewrite-uri
  [在内容中屏蔽 URLS]: #mask-urls-in-content
  [允许跨域调用]: #allow-cross-domain-calls
  [JSONP]: #jsonp
  [CORS]: #cors
