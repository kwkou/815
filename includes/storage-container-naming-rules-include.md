Azure 存储中的每个 Blob 必须驻留在一个容器中。该容器构成 Blob 名称的一部分。例如，在这些示例 Blob URI 中，`mycontainer` 是容器的名称：

	https://storagesample.blob.core.chinacloudapi.cn/mycontainer/blob1.txt
	https://storagesample.blob.core.chinacloudapi.cn/mycontainer/photos/myphoto.jpg
 
容器名称必须是有效的 DNS 名称，并符合以下命名规则：

1. 容器名称必须以字母或数字开头，并且只能包含字母、数字和短划线 (-) 字符。
1. 每个短划线 (-) 字符的前面和后面都必须是一个字母或数字；在容器名称中不允许连续的短划线 (-)。
1. 容器名称中的所有字母都必须为小写。
1. 容器名称必须介于 3 到 63 个字符。

> [AZURE.IMPORTANT] 请注意，容器的名称必须始终为小写。如果你在容器名称中包括大写字母或以其他方式违反了容器命名规则，则可能会收到 400 错误（错误请求）。

<!---HONumber=Mooncake_0104_2016-->