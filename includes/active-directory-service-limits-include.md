以下是 Azure Active Directory 服务的使用限制和其他服务限制。

| 类别 | 限制 |
|---|---|
| 目录 | 单个用户只能与最多 20 个 Azure Active Directory 目录相关联。<br />可能的组合示例：<ul> 单个用户创建了 20 个目录。<li></li>单个用户以成员身份添加到 20 个目录。<li></li>单个用户创建了 10 个目录，之后其他人又将该用户添加到 10 个不同的目录。<li></li></ul> |  
| 对象 | <ul><li>在使用 Azure Active Directory 免费版的单一目录中最多可以使用 500,000 个对象。</li><li>非管理员用户最多可以创建 250 个用户。</li></ul> |
| 架构扩展 | <ul><li>String 类型扩展最多只能有 256 个字符。</li><li>Binary 类型扩展限制在 256 字节以内。</li><li>100 扩展值（在 ALL 类型和 ALL 应用程序中）可以编写到任何单一对象中。</li><li>只有“User”、“Group”、“TenantDetail”、“Device”、“Application”和“ServicePrincipal”实体可以用“String”类型或“Binary”类型单一值属性进行扩展。</li><li>架构扩展仅在图形 API 1.21 预览版中可用。必须授予应用程序编写访问注册扩展的权限。</li></ul> |
| 应用程序 | 最多有 10 位用户可以是单一应用程序的所有者。 |
| 组 | <ul><li>最多有 10 位用户可以是单一应用程序的所有者。</li><li>Azure Active Directory 中的单个组可以具有任意数量的对象。</li><li>使用 Azure Active Directory 目录同步 (DirSync) 时，一个组中可以从本地 Active Directory 同步到 Azure Active Directory 的成员数限制为 15000 个。</li><li>使用 Azure AD Connect 时，一个组中可以从本地 Active Directory 同步到 Azure Active Directory 的成员数限制为 50000 个。</li></ul> |
| 访问面板 | <ul><li>对应用程序的数量没有限制，应用程序可在每位订阅 Azure AD Premium 或企业移动套件的最终用户的访问面板中查看。</li><li>最多 10 个应用磁贴（例如：Box、Salesforce 或 Dropbox）可在每位使用 Azure Active Directory 免费版或 Azure AD 基本版的最终用户的访问面板中查看。此限制不适用于管理员帐户。</li></ul> |
| 报告 | 在报告中最多可查看或下载 1,000 行。系统会截断其他任何数据。 |

<!---HONumber=71-->