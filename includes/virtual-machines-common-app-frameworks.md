<!-- Ibiza portal: tested -->

## 应用程序

在此表中，你可以找到在模板中使用的参数的详细信息，可以在部署模板之前对其进行检查。

| 应用程序 | 查看模板 |
|:---|:---:|
| Active Directory | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain-ha-2-dc) |
| Apache | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/apache2-on-ubuntu-vm) |
| Couchbase | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/couchbase-on-ubuntu) |
| DataStax | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/datastax) |
| Django | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/django-app) |
| Docker | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu) |
| Elasticsearch | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/elasticsearch) |
| Jenkins | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/jenkins-on-ubuntu) |
| Kafka | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/kafka-on-ubuntu) |
| LAMP | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/lamp-app) |
| MongoDB | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-ubuntu) |
| Redis | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/redis-high-availability) |
| SharePoint | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/sharepoint-three-vm) |
| Spark | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/spark-ubuntu-multidisks) |
| Tomcat | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/openjdk-tomcat-ubuntu-vm) |
| WordPress | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/wordpress-single-vm-ubuntu) |
| ZooKeeper | [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/zookeeper-cluster-ubuntu-vm) |

除了这些模板，你也可以在 [GitHub 仓库](https://github.com/Azure/azure-quickstart-templates/)中搜索模板。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

## Azure PowerShell

下载 https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/{template-name}/azuredeploy.json，作必要的修改。

在将括号中的文本替换为资源组名称、位置、部署名称和模板名称以后，运行以下命令即可创建资源组和部署：

	New-AzureRmResourceGroup -Name {resource-group-name} -Location {location}
	New-AzureRmResourceGroupDeployment -Name {deployment-name} -ResourceGroupName {resource-group-name} -TemplateFile "/path/to/azuredeploy.json"

运行 **New-AzureRmResourceGroupDeployment** 命令时，系统会提示你输入模板中参数的值。根据具体模板，Azure 可能需花费一些时间部署资源。

## Azure CLI

[安装 Azure CLI](/documentation/articles/xplat-cli-install/)，登录，确保启用 Resource Manager 命令。

下载 https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/{template-name}/azuredeploy.json，作必要的修改。

在将括号中的文本替换为资源组名称、位置、部署名称和模板名称以后，运行以下命令即可创建资源组和部署：

	azure group create {resource-group-name} {location}
	azure group deployment create --template-file /path/to/azuredeploy.json {resource-group-name} {deployment-name}

运行 **azure group deployment create** 命令时，系统会提示你输入模板中参数的值。根据具体模板，Azure 可能需花费一些时间部署资源。

## 后续步骤

发现 [GitHub](https://github.com/Azure/azure-quickstart-templates) 上可自由应用的所有模板。

了解有关 [Azure 资源管理器](/documentation/articles/resource-group-template-deploy/)的详细信息。