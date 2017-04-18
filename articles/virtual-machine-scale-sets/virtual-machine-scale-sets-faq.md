<properties
    pageTitle="Azure 虚拟机规模集常见问题解答 | Azure"
    description="获取有关虚拟机规模集常见问题的解答"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gatneil"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="76ac7fd7-2e05-4762-88ca-3b499e87906e"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="3/17/2017"
    wacn.date="04/17/2017"
    ms.author="negat"
    ms.custom="na"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="f0c03c21aaa062cf10c0bc5eedf76de8df085e20"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-virtual-machine-scale-sets-faq"></a>Azure 虚拟机规模集常见问题

本文包含有关规模集的常见问题解答。

## <a name="certificates"></a>证书

### <a name="how-do-you-securely-ship-a-certificate-into-the-vm--is-there-an-example-of-provisioning-a-scale-set-to-run-a-website-where-the-ssl-for-the-website-is-shipped-securely-from-a-certificate-configuration--the-common-certificate-rotation-operation-would-amount-to-a-configuration-update-operation"></a>你们如何安全地将证书传送到 VM 中？  是否有示例说明如何预配规模集来运行网站，并且可以通过证书配置安全传送该网站的 SSL？  常见的证书轮转操作相当于配置更新操作。

我们支持直接将客户证书从其 Key Vault 安装到 Windows 证书存储中。

在规模集的上下文中...

https://msdn.microsoft.com/zh-cn/library/mt589035.aspx

            "secrets": [ {
                  "sourceVault": {
                          "id": "/subscriptions/{subscriptionid}/resourceGroups/myrg1/providers/Microsoft.KeyVault/vaults/mykeyvault1"
              },
              "vaultCertificates": [ {
                          "certificateUrl": "https://mykeyvault1.vault.chinacloudapi.cn/secrets/{secretname}/{secret-version}",
                      "certificateStore": "certificateStoreName"
              } ]
            } ]

支持 Windows 和 Linux。

### <a name="self-signed-certificate-example"></a>自签名证书示例：

#### <a name="create-a-self-signed-cert-in-a-keyvault"></a>在 KeyVault 中创建自签名证书

在 KeyVault 中创建自签名证书的一种方法是参考以下 Service Fabric 文章中的说明：https://www.azure.cn/documentation/articles/service-fabric-cluster-security/

PowerShell 命令：

    Import-Module "C:\Users\mikhegn\Downloads\Service-Fabric-master\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

    Invoke-AddCertToKeyVault -SubscriptionId <Your SubID> -ResourceGroupName KeyVault -Location chinanorth -VaultName MikhegnVault -CertificateName VMSSCert -Password VmssCert -CreateSelfSignedCertificate -DnsName vmss.mikhegn.azure.com -OutputPath c:\users\mikhegn\desktop\

上述命令将提供 Resource Manager 模板的输入。

#### <a name="change-resource-manager-template"></a>更改 Resource Manager 模板

将以下属性作为规模集资源的一部分添加到“virtualMachineProfile”：

    "osProfile": {
                "computerNamePrefix": "[variables('namingInfix')]",
                "adminUsername": "[parameters('adminUsername')]",
                "adminPassword": "[parameters('adminPassword')]",
                "secrets": [
                  {
                    "sourceVault": {
                      "id": "[resourceId('KeyVault', 'Microsoft.KeyVault/vaults', 'MikhegnVault')]"
                    },
                    "vaultCertificates": [
                      {
                        "certificateUrl": "https://mikhegnvault.vault.chinacloudapi.cn:443/secrets/VMSSCert/20709ca8faee4abb84bc6f4611b088a4",
                        "certificateStore": "My"
                      }
                    ]
                  }
                ]
              }

### <a name="is-there-a-way-to-specify-an-ssh-key-pair-that-i-want-to-use-for-ssh-authentication-with-a-linux-scale-set-from-a-resource-manager-template"></a>是否可以通过 Resource Manager 模板指定一个 SSH 密钥对，用于对 Linux 规模集进行 SSH 身份验证？  

适用于 osProfile 的 REST API 类似于普通 VM 中的用法：

https://msdn.microsoft.com/zh-cn/library/azure/mt589035.aspx#linuxconfiguration

请按以下示例所示在模板中包含 `osProfile`：

    "osProfile": {
              "computerName": "[variables('vmName')]",
              "adminUsername": "[parameters('adminUserName')]",
              "linuxConfiguration": {
                "disablePasswordAuthentication": "true",
                "ssh": {
                  "publicKeys": [
                    {
                      "path": "[variables('sshKeyPath')]",
                      "keyData": "[parameters('sshKeyData')]"
                    }
                  ]
                }
              }
            }

此 JSON 块将在以下快速启动模板中使用：

https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json

另请查看此模板中的 OS 配置文件：

https://github.com/ExchMaster/gadgetron/blob/master/Gadgetron/Templates/grelayhost.json

### <a name="how-do-i-remove-deprecated-certificates"></a>如何删除已弃用的证书？ 

必须从保管库证书列表中删除旧证书，但要保留你想要在计算机上继续使用的所有证书。 这样做不会从所有 VM 中删除该证书，但也不会将证书添加到在规模集中创建的新 VM。 若要从现有 VM 中删除证书，必须手动编写一个自定义脚本扩展用于从证书存储中删除证书。

### <a name="how-do-i-take-an-existing-ssh-public-key-and-inject-it-into-the-scale-set-ssh-layer-during-provisioning--i-would-like-to-store-the-ssh-public-key-values-in-azure-key-vault-and-then-utilize-them-in-my-resource-manager-template"></a>如何捕获现有的 SSH 公钥并在预配期间将其注入到规模集 SSH 层？  我偏向于将 SSH 公钥值存储在 Azure Key Vault 中，然后在 Resource Manager 模板中利用这些值。

如果你只是想要为 VM 提供一个公共 SSH 密钥，则没有理由将公钥放在 Key Vault 中，因为公钥并不是机密。

可以在创建 Linux VM 时以明文形式提供 SSH 公钥。
以下网页提供了一个示例：

https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json

具体而言：

    "linuxConfiguration": {  
              "ssh": {  
                "publicKeys": [ {  
                  "path": "path",
                  "keyData": "publickey"
                } ]
              }

linuxConfiguration 元素名称 | 必选 | 类型 | 说明
--- | --- | --- | --- |  ---
ssh | 否 | 集合 | 指定 Linux OS 的 SSH 密钥配置。
path | 是 | String | 指定 SSH 密钥或证书应放置到的 Linux 文件路径。
keyData | 是 | String | 指定 base64 编码的 SSH 公钥。

### <a name="when-i-run-update-azurermvmss-after-more-than-one-certificate-from-the-same-keyvault-i-get-the-following-error"></a>针对同一 KeyVault 中的多个证书运行 Update-AzureRmVmss 时，出现以下错误：

Update-AzureRmVmss: 列表机密包含 /subscriptions/<my-subscription-id>/resourceGroups/internal-rg-dev/providers/Microsoft.KeyVault/vaults/internal-keyvault-dev 的重复实例，这是不允许的。 为什么不能添加同一 KeyVault 中的两个证书？

如果尝试添加同一个保管库两次，而不是添加现有 sourceVault 的新 vaultCertificate，则可能会发生此行为。 Add-AzureRmVmssSecret 无法正常用于添加其他机密。

如果想要添加同一 Key Vault 中的其他机密，应该更新列表 $vmss.properties.osProfile.secrets[0].vaultCertificates

以下网页提供了预期的输入结构：https://msdn.microsoft.com/zh-cn/library/azure/mt589035.aspx

需要在使用相同包含 Key Vault 的规模集对象中查找机密。 然后，必须将证书引用（URL 以及机密存储名称）添加到与保管库关联的列表中。

注意：目前不支持通过规模集 API 从 VM 中删除证书。

新 VM 不会包含旧证书，但使用已部署证书的 VM 仍会包含旧证书。

### <a name="is-there-a-way-to-get-certificates-pushed-to-the-scale-set-without-providing-the-password-when-the-certificate-is-in-secretstore-currently"></a>如果证书当前位于 SecretStore 中，是否可以在不提供密码的情况下获取推送到规模集的证书？

无需在脚本中对密码进行硬编码；可以使用运行部署脚本时所用的任何权限来动态检索密码。 如果某个脚本可将证书从机密存储移到 Key Vault，对机密存储运行获取证书命令也会输出 pfx 文件的密码。

### <a name="how-does-the-secrets-property-of-virtualmachineprofileosprofile-of-a-scale-set-work-why-do-you-need-sourcevault-when-you-have-to-specify-the-absolute-uri-to-a-certificate-with-certificateurl"></a>规模集的 virtualMachineProfile.osProfile 机密属性的工作原理是什么？ 如果必须结合 certificateUrl 指定证书的绝对 URI，为何需要指定 sourceVault？ 

Win RM 证书引用必须在 OS 配置文件的机密属性中存在。 

指定源保管库的目的是为了实施 CSM 中存在的 ACL 策略。 如果不指定源保管库，无权在 Key Vault 中部署/访问机密的用户可以通过 CRP 实现此目的。 即使是不存在的资源，它们也有 ACL。

如果提供错误的 sourceVault ID 但提供有效的 Key Vault URL，轮询操作时系统会报告错误

### <a name="if-i-add-secrets-to-an-existing-scale-set-does-it-inject-them-in-existing-instances-or-only-new-ones"></a>如果将机密添加到现有规模集，这些机密是注入到现有实例还是只注入到新实例？ 

证书将添加到所有 VM，包括现有的 VM。 如果规模集的 upgradePolicy 属性设置为“Manual”，则当你在 VM 上执行手动更新时，证书将添加到该 VM。

### <a name="where-do-certificates-go-for-linux-vms"></a>在 Linux VM 上，证书位于哪个位置？

请参阅 https://blogs.technet.microsoft.com/kv/2015/07/14/deploy-certificates-to-vms-from-customer-managed-key-vault/

### <a name="how-do-you-add-a-new-vault-certificate-to-a-new-certificate-object"></a>如何将新的保管库证书添加到新的证书对象？

如果想要将保管库证书添加到现有密钥（应是唯一一个机密对象），可按以下 PowerShell 示例中所示执行此操作：

    $newVaultCertificate = New-AzureRmVmssVaultCertificateConfig -CertificateStore MY -CertificateUrl https://sansunallapps1.vault.chinacloudapi.cn:443/secrets/dg-private-enc/55fa0332edc44a84ad655298905f1809

    $vmss.VirtualMachineProfile.OsProfile.Secrets[0].VaultCertificates.Add($newVaultCertificate)

    Update-AzureRmVmss -VirtualMachineScaleSet $vmss -ResourceGroup $rg -Name $vmssName

### <a name="what-happens-to-certificates-if-you-reimage-a-vm"></a>如果重置 VM 的映像，证书会发生什么情况？

如果重置 VM 的映像，证书将会消失，因为重置映像会删除整个 OS 磁盘。 

### <a name="what-happens-if-you-delete-a-certificate-from-the-key-vault"></a>从 Key Vault 中删除证书会发生什么情况？

如果在 Key Vault 中删除机密并停止解除分配所有 VM，则会将这些 VM 启动，从而会发生失败。 发生这种失败的原因是 CRP 需要从 Key Vault 中检索机密，但却无法检索。 在这种情况下，可以从规模集模型中删除证书。 

CRP 组件不会持久保留任何客户机密。 如果停止解除分配规模集中的所有 VM，则会删除缓存。 在这种情况下，将从 Key Vault 中检索机密。

横向扩展时不会发生此问题，因为结构中包含机密的缓存副本（至少在单结构租户模型中是这样）。

### <a name="why-do-we-have-to-specify-the-exact-location-for-the-certificate-url-as-referenced-here-per-httpswwwazurecndocumentationarticlesservice-fabric-cluster-security"></a>为何需要按以下网页中所述，为证书 URL 指定确切的位置：https://www.azure.cn/documentation/articles/service-fabric-cluster-security/、 
https://<name of the vault>.vault.chinacloudapi.cn:443/secrets/<exact location>

根据 KeyVault 文档，在未指定版本的情况下，get-secret REST API 应返回机密的最新版本：

方法 | 代码
--- | ---
GET | https://mykeyvault.vault.chinacloudapi.cn/secrets/{secret-name}/{secret-version}?api-version={api-version}

请将 {secret-name} 替换为要检索的机密的名称，将 {secret-version} 替换为该机密的版本。 可以不包含机密版本，在这种情况下，将检索当前版本。

### <a name="why-does-certificate-version-have-to-be-specified-when-using-key-vault"></a>使用 Key Vault 时，为何需要指定证书版本？

提出此项要求的原因是为了让用户更清楚地了解其 VM 上部署了哪个证书。

如果创建了 VM，然后更新 Key Vault 中的机密，新证书不会下载到 VM。 但是，VM 看上去像是引用了该证书，并且新 VM 将获取新机密。 为了避免这种混淆，需要引用明确的机密版本。

### <a name="my-team-works-with-several-certificates-that-are-distributed-to-us-as-cer-public-keys-what-is-the-recommended-approach-is-for-deployment-of-these-certs-to-a-scale-set"></a>本团队正在开发几个证书，到时将以 .cer 公钥的形式分发给大家使用。 在规模集中部署这些证书的建议方法是什么？

可以生成仅包含 .cer 文件的 pfx 文件，并在其中设置 X509ContentType = Pfx。 例如，在 C# 或 PowerShell 中以 x509Certificate2 对象的形式加载该 .cer 文件，然后调用此方法：https://msdn.microsoft.com/zh-cn/library/24ww6yzk(v=vs.110).aspx

### <a name="i-do-not-see-an-option-for-users-to-pass-in-certificates-as-base64-strings-that-most-other-resource-providers-provide"></a>我没有看到可让用户以 base64 字符串形式传入证书的选项，而其他大多数资源提供程序却提供该选项。

可以在 Resource Manager 模板中提取最新版本的 URL 来模拟所述的行为。 可以在 Resource Manager 模板中包含以下 JSON 属性：

    "certificateUrl": "[reference(resourceId(parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults/secrets', parameters('vaultName'), parameters('secretName')), '2015-06-01').secretUriWithVersion]"

### <a name="do-we-have-to-wrap-certs-in-json-objects-in-keyvaults"></a>是否需要在 Key Vault 中将证书包装在 JSON 对象中？

这是规模集/VM 的一项要求。 我们还支持 application/x-pkcs12 内容类型。 以下网页提供了说明：http://www.rahulpnath.com/blog/pfx-certificate-in-azure-key-vault/

目前不支持 .cer 文件，必须将 .cer 文件导出到 pfx 容器中。

## <a name="compliance"></a>合规性

### <a name="are-scale-sets-pci-compliant"></a>规模集是否符合 PCI 规范？

规模集是位于计算资源提供程序顶层的一个精简 API 层，它整体构成 Azure 服务树中“计算平台”区域的一部分。

因此，从符合性的角度看，规模集是 Azure 计算平台的基础部分。 因此，它们与计算资源提供程序 (CRP) 本身共享相同的团队、工具、流程、部署方法、安全控制、JIT、监视、警报等。  规模集符合 PCI 规范，因为计算资源提供程序已通过最新的 PCI DSS 认证：

有关详细信息，请参阅：[https://www.trustcenter.cn/](https://www.trustcenter.cn/)。

## <a name="extensions"></a>扩展

### <a name="how-do-you-delete-a-scale-set-extension"></a>如何删除规模集扩展？

下面是使用 PowerShell 执行此操作的示例：

    $vmss = Get-AzureRmVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName" 

    $vmss=Remove-AzureRmVmssExtension -VirtualMachineScaleSet $vmss -Name "extensionName"

    Update-AzureRmVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName" -VirtualMacineScaleSet $vmss

可在 `$vmss` 中找到 extensionName。

### <a name="extensions-seem-to-run-in-parallel-on-scale-sets-causing-my-custom-script-extension-to-fail-what-can-i-do-to-fix-this-behavior"></a>扩展似乎在规模集上并行运行，从而导致自定义脚本扩展失败。 如何修复此行为？

请参阅 https://msftstack.wordpress.com/2016/05/12/extension-sequencing-in-azure-vm-scale-sets/ 

### <a name="how-do-i-reset-the-password-for-scale-set-vms"></a>如何重置规模集 VM 的密码？

使用 VM 访问扩展

下面是使用 PowerShell 执行此操作的示例：

    $vmssName = "myvmss"
    $vmssResourceGroup = "myvmssrg"
    $publicConfig = @{"UserName" = "newuser"}
    $privateConfig = @{"Password" = "********"}

    $extName = "VMAccessAgent"
    $publisher = "Microsoft.Compute"
    $vmss = Get-AzureRmVmss -ResourceGroupName $vmssResourceGroup -VMScaleSetName $vmssName
    $vmss = Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmss -Name $extName -Publisher $publisher -Setting $publicConfig -ProtectedSetting $privateConfig -Type $extName -TypeHandlerVersion "2.0" -AutoUpgradeMinorVersion $true
    Update-AzureRmVmss -ResourceGroupName $vmssResourceGroup -Name $vmssName VirtualMachineScaleSet $vmss

### <a name="how-do-i-add-an-extension-to-all-vms-in-my-scale-set"></a>如何将扩展添加到规模集中的所有 VM？

- 如果更新策略设置为自动，使用新扩展属性重新部署模板可更新每个 VM。
- 如果更新策略设置为手动，则必须更新扩展，然后在所有实例上执行 manualUpdate。

### <a name="if-the-extensions-associated-with-an-existing-scale-set-are-updated-would-they-affect-already-existing-vms-that-is-would-the-vms-show-up-as-not-matching-the-scale-set-model-or-would-they-be-ignored-when-an-existing-machine-is-service-healed--reimaged--etc-would-the-scripts-that-are-currently-configured-on-the-scale-set-be-executed-or-would-the-ones-that-were-configured-when-the-machine-was-first-created-be-used"></a>如果更新与现有规模集关联的扩展，是否会影响现有的 VM？ （也就是说，VM 是否显示为与规模集模型不匹配）？ 或者，是否可以忽略这种情况？ 如果对现有计算机执行服务修复/重置映像等操作，是否会执行当前在规模集上配置的脚本，或者，是否会使用最初创建计算机时配置的脚本？

- 如果规模集模型中的扩展定义已更新，则在 upgradePolicy 设置为自动的情况下，将会更新 VM；在 upgradePolicy 设置为手动的情况下，这些 VM 将标记为与模型不匹配。 

- 如果对现有 VM 执行服务修复，这种行为类似于重新启动，因此不会重新运行扩展。 如果对 VM 执行重置映像，这种行为类似于将 OS 驱动器替换为源映像以及最新模型中的任何专用设置，在这种情况下，扩展将会运行。

### <a name="how-do-i-get-a-scale-set-to-join-an-ad-domain"></a>如何获取一个用于加入 AD 域的规模集？

可以使用 JsonADDomainExtension 来定义此类扩展，例如：

                        "extensionProfile": {
                            "extensions": [
                                {
                                    "name": "joindomain",
                                    "properties": {
                                        "publisher": "Microsoft.Compute",
                                        "type": "JsonADDomainExtension",
                                        "typeHandlerVersion": "1.0",
                                        "settings": {
                                            "Name": "[parameters('domainName')]",
                                            "OUPath": "[variables('ouPath')]",
                                            "User": "[variables('domainAndUsername')]",
                                            "Restart": "true",
                                            "Options": "[variables('domainJoinOptions')]"
                                        },
                                        "protectedsettings": {
                                            "Password": "[parameters('domainJoinPassword')]"
                                        }
                                    }
                                }
                            ]
                        }

### <a name="my-scale-set-extension-is-trying-to-install-something-that-requires-a-reboot-for-instance-commandtoexecute-powershellexe--executionpolicy-unrestricted-install-windowsfeature--name-fs-resource-manager--includemanagementtools"></a>我的规模集扩展正在尝试安装某个需要重新启动的组件，例如："commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools"

可以使用 DSC 扩展。 如果 OS 是 2012 R2，则 Azure 会提取 WMF5.0 安装程序，重新启动，然后继续配置。 

### <a name="how-can-i-enable-antimalware-on-my-scale-set"></a>如何在规模集上启用反恶意软件？

下面是一个 PowerShell 示例：

    $rgname = 'autolap'
    $vmssname = 'autolapbr'
    $location = 'chinaeast'

    # retrieve the most recent version number of the extension
    $allVersions= (Get-AzureRmVMExtensionImage -Location $location -PublisherName "Microsoft.Azure.Security" -Type "IaaSAntimalware").Version
    $versionString = $allVersions[($allVersions.count)-1].Split(".")[0] + "." + $allVersions[($allVersions.count)-1].Split(".")[1]

    $VMSS = Get-AzureRmVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname
    echo $VMSS
    Add-AzureRmVmssExtension -VirtualMachineScaleSet $VMSS -Name "IaaSAntimalware" -Publisher "Microsoft.Azure.Security" -Type "IaaSAntimalware" -TypeHandlerVersion $versionString
    Update-AzureRmVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $VMSS 

### <a name="i-need-to-execute-a-custom-script-hosted-on-a-private-storage-account-i-have-no-problems-when-the-storage-is-public-but-when-i-try-to-use-a-shared-access-signaturesas-it-fails-with-the-error-missing-mandatory-parameters-for-valid-shared-access-signature-i-know-that-linksas-works-fine-from-my-local-browser"></a>我需要执行一个在专用存储帐户中托管的自定义脚本。 如果存储是公用的，则不会出现问题，但尝试使用共享访问签名 (SAS) 时，该脚本失败并出现错误：“缺少有效共享访问签名的必需参数”。 我知道，可以通过本地浏览器正常使用“链接+SAS”。

要正常运行此方案，必须使用存储帐户密钥和名称指定保护设置。 请参阅 https://www.azure.cn/documentation/articles/virtual-machines-windows-extensions-customscript/#template-example-for-a-windows-vm-with-protected-settings

## <a name="networking"></a>网络

### <a name="how-do-i-do-vip-swap-for-scale-sets-in-the-same-subscription-and-same-region"></a>如何针对同一订阅和同一区域中的规模集执行 VIP 交换？

请参阅：https://msftstack.wordpress.com/2017/02/24/vip-swap-blue-green-deployment-in-azure-resource-manager/ 

### <a name="what-is-the-resourceguid-property-on-a-nic-for"></a>NIC 上的 resourceGuid 属性有什么作用？

它是一个唯一 ID。 在将来的某个时间，较低的层将记录此 ID。 

### <a name="how-do-i-specify-a-range-of-private-ip-addresses-for-static-private-ip-address-allocation"></a>如何为静态专用 IP 地址分配指定专用 IP 地址范围？

IP 是从指定的子网中选择的。 

规模集 IP 分配方法始终是“动态的”。 但这并不意味着这些 IP 可以改变， 而只是意味着不要在 PUT 请求中指定 IP。 换而言之，应该使用子网指定静态设置。 

### <a name="how-do-i-deploy-a-scale-set-into-an-existing-vnet"></a>如何在现有的 VNET 中部署规模集？ 

请参阅 https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-existing-vnet 

### <a name="how-do-i-add-a-scale-sets-first-vms-ip-address-to-the-output-of-a-template"></a>如何将规模集的第一个 VM 的 IP 地址添加到模板的输出？

请参阅：http://stackoverflow.com/questions/42790392/arm-get-vmsss-private-ips

## <a name="scale"></a>缩放

### <a name="why-would-you-ever-create-a-scale-set-with-fewer-than-2-vms"></a>创建包含 2 个以下 VM 的规模集的原因是什么？

原因之一是利用规模集的弹性属性。 例如，可以部署不包含任何 VM 的规模集来定义基础结构，这样就无需支付 VM 运行费。 然后，在准备好部署 VM 时，可将规模集的“容量”提高到生产实例计数。

另一个原因是要对规模集执行某项操作，同时不关心可用性，就如同在离散的 VM 中使用可用性集的情况一样。 此外，可以借助规模集来使用可替代的无差别计算单位。 这种一致性是规模集相比可用性集存在的一项重要优势。 许多无状态工作负载不关心单个单位，可以在工作负荷下降时缩减为一个计算单位，在工作负荷提高时扩展回到多个计算单位。

### <a name="how-do-you-change-the-number-of-vms-in-a-scale-set"></a>如何更改规模集中的 VM 数目？

请参阅：https://msftstack.wordpress.com/2016/05/13/change-the-instance-count-of-an-azure-vm-scale-set/

### <a name="how-can-you-define-custom-alerts-for-when-certain-thresholds-are-reached"></a>如何定义达到特定阈值时触发的自定义警报？

可以灵活处理警报。例如，可按以下示例所示，通过 Resource Manager 模板定义自定义的 Webhook：

       {
             "type": "Microsoft.Insights/autoscaleSettings",
               "apiVersion": "[variables('insightsApi')]",
                     "name": "autoscale",
                       "location": "[parameters('resourceLocation')]",
                         "dependsOn": [
                             "[concat('Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]"
                     ],
                     "properties": {
                             "name": "autoscale",
                         "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/',  resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]",
                         "enabled": true,
                         "notifications": [{
                                   "operation": "Scale",
                                  "email": {
                                         "sendToSubscriptionAdministrator": true,
                                         "sendToSubscriptionCoAdministrators": true,
                                         "customEmails": [
                                            "youremail@address.com"
                                         ]},
                                  "webhooks": [{
                                        "serviceUri": "https://events.pagerduty.com/integration/0b75b57246814149b4d87fa6e1273687/enqueue",
                                        "properties": {
                                            "key1": "custommetric",
                                            "key2": "scalevmss"
                                        }
                                        }
                                  ]}],

在此示例中，达到阈值时，警报将转到 Pagerduty。

## <a name="troubleshooting"></a>故障排除

### <a name="how-do-i-enable-boot-diagnostics"></a>如何启用启动诊断？

创建一个存储帐户并将此 JSON 块放在规模集的 virtualMachineProfile 中，然后更新规模集：

          "diagnosticsProfile": {
            "bootDiagnostics": {
              "enabled": true,
              "storageUri": "http://yourstorageaccount.blob.core.chinacloudapi.cn"
            }
          }

然后，在创建新 VM 时，VM 的 InstanceView 将显示屏幕截图等的详细信息。例如：

    "bootDiagnostics": {
        "consoleScreenshotBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.chinacloudapi.cn/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.screenshot.bmp",
        "serialConsoleLogBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.chinacloudapi.cn/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.serialconsole.log"
      }

## <a name="updates"></a>更新

### <a name="how-to-i-update-my-scale-set-to-a-new-image-and-manage-patching"></a>如何将规模集更新为新映像以及管理修补？

请参阅：https://www.azure.cn/documentation/articles/virtual-machine-scale-sets-upgrade-scale-set/

### <a name="can-you-use-the-reimage-operation-to-reset-a-vm-without-changing-the-image-that-is-reset-a-vm-to-factory-settings-rather-than-to-a-new-image"></a>是否可以在不更改映像的情况下，使用重置映像操作来重置 VM？ （也就是说，将 VM 重置为出厂设置而不是重置为新映像）？

是的。 请参阅：https://docs.microsoft.com/zh-cn/rest/api/virtualmachinescalesets/manage-all-vms-in-a-set

但是，如果规模集引用了版本为“latest”的平台映像，则调用重置映像时，VM 可以更新为更高版本的 OS 映像。

## <a name="vm-properties"></a>VM 属性

### <a name="how-do-i-get-property-information-for-each-vm-without-having-to-make-multiple-calls-for-example-getting-the-fault-domain-for-each-vm-in-my-100-scale-set"></a>如何在不发出多个调用的情况下获取每个 VM 的属性信息？ 例如：如何获取 100 个规模集中每个 VM 的容错域？

可以通过针对以下资源 URI 执行 REST API `GET` 来调用 ListVMInstanceViews：

`/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scaleset_name>/virtualMachines?$expand=instanceView&$select=instanceView`

### <a name="are-there-ways-to-pass-different-extension-arguments-to-different-vms-in-a-scale-set"></a>是否可将不同的扩展参数传递给规模集中的不同 VM？

不可以，但扩展可以根据它们运行所在的 VM 的唯一属性（例如计算机名）运行。 此外，扩展可以在 http://169.254.169.254 上查询实例元数据来获取更多信息。

### <a name="why-are-there-gaps-between-my-scale-set-vm-machine-names-and-vm-ids-for-example-0-1-3"></a>规模集 VM 计算机名和 VM ID 为什么不是连续的？ 例如：0，1，3...

之所以不连续，是因为规模集的 overprovision 属性设置为默认值 true。 在 overprovision 属性设置为 true 的情况下，创建的 VM 数比请求的数目要多，随后会删除多余的 VM。 结果是部署可靠性得到提高，但代价是无法遵守连续命名和连续的 NAT 规则。 可将此属性设置为 false。对于小型规模集而言，这样做不会给部署可靠性造成很大差别。

### <a name="what-is-the-difference-between-deleting-a-vm-in-a-scale-set-vs-deallocating-the-vm-when-should-i-choose-one-over-the-other"></a>删除规模集中的 VM 与解除分配 VM 有什么区别？ 如何在这两种做法之间做出选择？

主要区别是，解除分配不会删除 VHD，因此会产生与停止解除分配相关的存储费用。 选择其中一种做法的理由包括：

- 不再想要支付计算费用，但要保留 VM 的磁盘状态。
- 希望启动一组 VM 的速度比横向扩展规模集更快。
  - 出于这种方案，你创建了自己的缩放引擎，并希望以更快的速度完成端到端缩放。
  - 规模集在 FD/UD 之间不均匀分布（由于有选择性地删除了 VM，或者过度预配后删除了 VM）。 停止解除分配，然后在规模集上启动 VM 可在 FD/UD 之间均匀分布 VM。