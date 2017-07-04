<properties
    pageTitle="使用 PowerShell 调用 REST API 完成经典部署资源的跨订阅迁移"
    description="使用 PowerShell 调用 REST API 完成经典部署资源的跨订阅迁移"
    service=""
    resource=""
    authors="Steve Shi"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Subscription, ARM, PowerShell, REST API"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sample-code-aog"
    ms.date=""
    wacn.date="05/25/2017" />

# 使用 PowerShell 调用 REST API 完成经典部署资源的跨订阅迁移

资源管理器 (ARM) 模式部署的资源可以通过新管理门户中的 Move 功能轻易地进行跨资源组或跨订阅迁移。
然而在跨订阅迁移经典部署资源时，除了通用的限制以外，还额外存在以下限制：

- 必须在同一操作中移动订阅中的所有经典资源。
- 目标订阅不得包含任何其他经典资源。
- 只能通过独立的适用于经典移动的 REST API 来请求移动。 将经典资源移到新订阅时，不能使用标准的 Resource Manager 移动命令。

具体限制请参阅文档[将资源移到新资源组或订阅中](/documentation/articles/resource-group-move-resources)的“移动资源前需查看的清单”和“经典部署限制”。

在文档[将资源移到新资源组或订阅](/documentation/articles/resource-group-move-resources)中，列出了如何调用 REST API 执行迁移操作。对不熟悉 REST API 的用户来说，可能对文档里的步骤无从下手。本文列举了使用 PowerShell 命令 `Invode-RestMethod` 调用 REST API 的示例，以供参考。

1. 定义 Azure AD 租户和迁移操作的源、目标订阅。

    Azure AD 的租户就是登录 Azure 门户所用的账号后缀名，还需要用户提供源订阅与目标订阅的订阅号。

        $Tenant = "xxx.partner.onmschina.cn"
        $sourcesubid = "source-subscription-id"
        $targetsubid = "target-subscription-id"

2. 定义 `function GetAuthToken`，以抓取登录时产生的验证令牌。

        function GetAuthToken
        {
            param
            (
                [Parameter(Mandatory=$true)]
                $ApiEndpointUri,

                [Parameter(Mandatory=$true)]
                $AADTenant
            )

            $adal = "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Services\" + `
                    "Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
            $adalforms = "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Services\" + `
                        "Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll"

            [System.Reflection.Assembly]::LoadFrom($adal) | Out-Null
            [System.Reflection.Assembly]::LoadFrom($adalforms) | Out-Null

            $clientId = "1950a258-227b-4e31-a9cf-717495945fc2"
            $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
            $authorityUri = “https://login.partner.microsoftonline.cn/$aadTenant”

            $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authorityUri

            $authResult = $authContext.AcquireToken($ApiEndpointUri, $clientId,$redirectUri, "Auto")

            return $authResult
        }

        $token = GetAuthToken -ApiEndPointUri "https://management.core.chinacloudapi.cn/" -AADTenant $Tenant

3. 生成 REST API 请求的 header。

        $header = @{
            "Accept"="application/json"
            ‘Content-Type’="application/json"
            "Authorization"=$token.CreateAuthorizationHeader()
        }

4. 针对迁移的源订阅，生成 REST API 请求的 body，并调用 API 检查源订阅是否支持迁移。

        $sourcebody = ConvertTo-Json @{
            "role"="source"
        }

        $sourceuri = "https://management.chinacloudapi.cn/subscriptions/$sourcesubid/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01"
        $sourceresponse = Invoke-RestMethod -Uri $sourceuri -Headers $header -Body $sourcebody -Method POST

5. 针对迁移的目标订阅，生成 REST API 请求的 body，并调用 API 检查目标订阅是否支持迁移。

        $targetbody = ConvertTo-Json @{
            "role"="target"
        }

        $targeturi = "https://management.chinacloudapi.cn/subscriptions/$targetsubid/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01"
        $targetresponse = Invoke-RestMethod -Uri $targeturi -Headers $header -Body $targetbody -Method POST

6. 抓取 REST API 的响应，查看是否源订阅与目标订阅是否支持迁移。

        Write-Host "----------Source subscription error------------"
        $sourceresponse.reasons

        Write-Host "----------Target subscription error------------"
        $targetresponse.reasons

7. 如果源订阅与目标订阅都未报错，那说明迁移可以正常执行，继续生成迁移的 REST API 请求 body，调用 REST API 执行迁移动作。

        $executebody = ConvertTo-Json @{
            "target"="/subscriptions/$targetsubid"
        }

        $executeuri = "https://management.chinacloudapi.cn/subscriptions/$sourcesubid/providers/Microsoft.ClassicCompute/moveSubscriptionResources?api-version=2016-04-01"
        $executeresponse = Invoke-RestMethod -Uri $executeuri -Headers $header -body $executebody -Method POST

8. 如果源订阅与目标订阅有报错，请先仔细查看报错信息，将阻碍迁移的资源事先迁移或移除之后再进行操作。

## 后续步骤

关于资源管理器部署的资源迁移，请参考文档[将资源移到新资源组或订阅中](/documentation/articles/resource-group-move-resources)。

