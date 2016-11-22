若要将标记添加到资源组中，请使用 **azure 组集**。如果资源组没有任何现有的标记，则传递标记。

    azure group set -n tag-demo-group -t Dept=Finance

标记作为一个整体更新。如果要将标记添加到具有现有标记的资源组，则传递所有标记。

    azure group set -n tag-demo-group -t Dept=Finance;Environment=Production;Project=Upgrade

资源组中的资源不会继承标记。若要将标记添加到资源，请使用 **azure 资源集**。必须传递要将标记添加到的资源类型的 API 版本号。如果需要检索 API 版本，请对你要设置的类型使用以下具有资源提供程序的命令：

    azure provider show -n Microsoft.Storage --json

在结果中查找所需的资源类型。

    "resourceTypes": [
    {
      "resourceType": "storageAccounts",
      ...
      "apiVersions": [
        "2016-01-01",
        "2015-06-15",
        "2015-05-01-preview"
      ]
    }
    ...

现在，提供 API 版本、资源组名称、资源名称、资源类型和标记值作为参数。

    azure resource set -g tag-demo-group -n storagetagdemo -r Microsoft.Storage/storageAccounts -t Dept=Finance -o 2016-01-01

标记直接存在于资源和资源组中。若要查看现有标记，只需使用 **azure group show** 获取资源组及其资源。

    azure group show -n tag-demo-group --json
    
该操作返回有关资源组的元数据，包括任何应用到其中的标记。
    
    {
      "id": "/subscriptions/4705409c-9372-42f0-914c-64a504530837/resourceGroups/tag-demo-group",
      "name": "tag-demo-group",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "location": "southcentralus",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production",
        "Project": "Upgrade"
      },
      ...

使用 **azure resource show** 可以查看特定资源的标记。

    azure resource show -g tag-demo-group -n storagetagdemo -r Microsoft.Storage/storageAccounts -o 2016-01-01 --json
    
若要检索具有标记值的所有资源，请使用：

    azure resource list -t Dept=Finance --json
    
若要检索具有标记值的所有资源组，请使用：

    azure group list -t Dept=Finance
        
可以通过以下命令查看订阅中的现有标记：

    azure tag list

<!---HONumber=Mooncake_1114_2016-->