<properties 
	pageTitle="Azure Multi-Factor Authentication - 工作原理" 
	description="Azure Multi-Factor Authentication 可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。它通过要求第二种形式的身份验证提供额外的安全性，并通过一系列简单的身份验证选项提供增强式身份验证。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtland"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.date="05/16/2016"
	wacn.date="06/06/2016"/>

#Azure 多重身份验证的工作原理

多重身份验证的安全性在于它的分层方法。破坏多重身份验证系统对于攻击者来说是巨大的挑战。即使攻击者设法得到用户的密码，如果没有同时占有可信设备也没有用处。如果用户丢失了设备，捡到该设备的人也无法使用它，除非他（她）也知道该用户的密码。

![验证](./media/multi-factor-authentication-how-it-works/howitworks.png)


Azure 多重身份验证可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。它通过要求第二种形式的身份验证提供额外的安全性，并通过一系列简单的身份验证选项提供增强式身份验证：

- 电话呼叫 
- 短信
- 移动应用通知 - 允许用户选择偏好的方法
- 移动应用验证码

<!--For additional information oh how it works see the following video.

[AZURE.VIDEO multi-factor-authentication-deep-dive-securing-access-on-premises]-->

##可用于多重身份验证的方法
当用户登录时，系统会将额外的身份验证发送给该用户。以下是可用于这种二次身份验证的方法列表。
<table class="table table-bordered table-striped table-condensed">
   <tr>
      <th>验证方法</th>
      <th>说明</th>
    
   </tr>
   <tr>
      <td>电话呼叫</td>
      <td>向用户的智能手机拨打电话，要求他们按 # 号来验证其登录。这样就会完成验证过程。此选项可配置，并可以更改为你指定的代码</td>
      
   </tr>
   <tr>
      <td>短信</td>
      <td>将包含 6 位数代码的短信发送到用户的智能手机。输入此代码即可完成验证过程</td>
   </tr>
   <tr>
      <td>移动应用通知</td>
      <td>将验证请求发送到用户的智能手机，要求他们通过在移动应用中选择“验证”来完成验证。如果你将应用通知选作主要验证方法，则会发生这种情况。如果用户在未登录时收到通知，他们可以选择将通知举报为欺诈</td>
   </tr>
   <tr>
      <td>移动应用的验证码</td>
      <td>将验证码发送到用户智能手机中运行的移动应用。如果将验证码选作主要验证方法，则会发生这种情况</td>
   </tr>
</table>

##可用的多重身份验证版本
Azure 多重身份验证有两个不同的版本。下表较详细地描述了每个版本。

<table class="table table-bordered table-striped table-condensed">
   <tr>
      <th>版本 </th>
      <th>说明</th>
    
   </tr>
   <tr>
      <td>适用于 Office 365 的多重身份验证</td>
      <td>此版本专门与 Office 365 应用程序配合使用，可以从 Office 365 门户进行管理。因此，管理员现在可以借助多重身份验证来保护其 Office 365 资源。Office 365 订阅随附了此版本</td>
      
   </tr>
   <tr>
      <td>面向 Azure 管理员的多重身份验证</td>
      <td>适用于 Office 365 的多重身份验证功能的相同子集将免费提供给所有 Azure 管理员使用。Azure 订阅的每个管理员帐户现在可以通过启用这项核心多重身份验证功能来获得更多的保护。因此，如果某个管理员想要访问 Azure 门户以创建 VM 和网站以及管理存储、移动服务或任何其他 Azure 服务，则可在其管理员帐户中添加多因素身份验证</td>
         </tr>
</table>
##版本功能比较
下表提供了 Azure 多重身份验证各版本中可用的功能列表。

<table class="table table-bordered table-striped table-condensed">
   <tr>
      <th>功能</th>
      <th>适用于 Office 365 的多重身份验证（随附在 Office 365 SKU 中）</th>
      <th>面向 Azure 管理员的多重身份验证（随附在 Azure 中）</th>   
      <th> Azure 多重身份验证（随附在 Azure AD Premium 和企业移动套件中）</th>
   </tr>
   <tr>
      <td>管理员可以使用 MFA 保护帐户</td>
      <td>*</td>
      <td>*（仅适用于 Azure 管理员帐户）</td>
      <td>*</td>
  </tr>
  <tr>
      <td>将移动应用用作第二个因素</td>
      <td>*</td>
      <td>*</td>
      <td>*</td>
   </tr>
  <tr>
      <td>将电话呼叫用作第二个因素</td>
      <td>*</td>
      <td>*</td>
      <td>*</td>
  </tr>
 <tr>
      <td>不支持 MFA 的客户端的应用密码</td>
      <td>*</td>
      <td>*</td>
      <td>*</td>
  </tr>
 <tr>
      <td>欺诈警报</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
 <tr>
      <td>一次性跳过</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
 <tr>
      <td>可自定义来电者</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
 <tr>
      <td>受信任的 IP</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
 <tr>
      <td>暂停对已记住的设备进行 MFA（公共预览版）</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
 <tr>
      <td>MFA SDK</td>
      <td></td>
      <td></td>
      <td>*</td>
  </tr>
</table>

##如何获取多重身份验证



如果你是 Office 365 用户或 Azure 订户，并想要充分利用多重身份验证提供的其他功能，请继续阅读其他部分。

如果你没有上述任何产品并想要使用多重身份验证，首先你需要一个 Azure 订阅或 [Azure 试用版订阅](/pricing/1rmb-trial/)。


选择最适合你的组织的按用户或按使用量模式。接下来，若要开始使用，请参阅[入门](/documentation/articles/multi-factor-authentication-get-started-cloud/)



 
<!---HONumber=Mooncake_0530_2016-->