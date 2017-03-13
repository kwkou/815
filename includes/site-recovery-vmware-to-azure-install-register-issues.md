### 安装失败
| **示例错误消息** | **建议的操作** |
|--------------------------|------------------------|
|错误: 未能加载帐户。错误: System.IO.IOException: 安装和注册 CS 服务器时无法从传输连接读取数据。| 请确保在计算机上启用 TLS 1.0。 |

### 注册失败
可通过查看 **%ProgramData%\\ASRLogs** 文件夹中的日志来调试注册失败。

| **示例错误消息** | **建议的操作** |
|--------------------------|------------------------|
|**09:20:06**:InnerException.Type: SrsRestApiClientLib.AcsException，InnerException。<br>消息: ACS50008: SAML 令牌无效。<br>跟踪 ID: 1921ea5b-4723-4be7-8087-a75d3f9e1072<br>相关性 ID: 62fea7e6-2197-4be4-a2c0-71ceb7aa2d97><br>时间戳: **2016-12-12 14:50:08Z<br>** | 确保系统时钟的时间与本地时间之间的偏差不超过 15 分钟。重新运行安装程序以完成注册。|
|**09:35:27** : 尝试获取选定证书的所有灾难恢复保管库时发生 DRRegistrationException: : 引发了 Exception.Type:Microsoft.DisasterRecovery.Registration.DRRegistrationException，Exception.Message: ACS50008: SAML 令牌无效。<br>跟踪 ID: e5ad1af1-2d39-4970-8eef-096e325c9950<br>相关性 ID: abe9deb8-3e64-464d-8375-36db9816427a<br>时间戳: **2016-05-19 01:35:39Z**<br> | 确保系统时钟的时间与本地时间之间的偏差不超过 15 分钟。重新运行安装程序以完成注册。|
|06:28:45: 未能创建证书<br>06:28:45: 安装无法继续。无法创建用于在 Site Recovery 中进行身份验证的证书。重新运行安装程序 | 确保以本地管理员的身份运行安装程序。 |

<!---HONumber=Mooncake_0306_2017-->