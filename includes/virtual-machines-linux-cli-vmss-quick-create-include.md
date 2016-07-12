如果尚未这样做，你可以获取 [Azure 订阅试用版](/pricing/1rmb-trial/)和 [Azure CLI](/documentation/articles/xplat-cli-install/) 来[连接到你的 Azure 帐户](/documentation/articles/xplat-cli-connect/)。然后，可以运行以下命令来快速创建缩放集：

	# make sure we are in Resource Manager mode (/documentation/articles/resource-manager-deployment-model/)
	azure config mode arm
	
	# quick-create a scale set
	#
	# generic syntax:
	# azure vmss quick-create -n SCALE-SET-NAME -g RESOURCE-GROUP-NAME -l LOCATION -u USERNAME -p PASSWORD -C INSTANCE-COUNT -Q IMAGE-URN
	#
	# example:
	azure vmss quick-create -n negatvmss -g negatvmssrg -l chinanorth -u negat -p P4ssw0rd -C 5 -Q Canonical:UbuntuServer:14.04.3-LTS:latest

如果想要自定义位置或 image-urn，请查看命令 `azure location list` 和 `azure vm image {list-publishers|list-offers|list-skus|list|show}`。

此命令返回后，将创建缩放集。此缩放集将包含一个负载平衡器，其 NAT 规则会将负载平衡器上的端口 50,000 + 映射到 VM i 上的端口 22。因此，在确定负载平衡器的 FQDN 后，我们就可以通过 ssh 连接到 VM：

	# (if you decide to run this as a script, please invoke using bash)
	
	# list load balancers in the resource group we created
	#
	# generic syntax:
	# azure network lb list -g RESOURCE-GROUP-NAME
	#
	# example with some quick-and-dirty grep-fu to store the result in a variable:
	line=$(azure network lb list -g negatvmssrg | grep negatvmssrg)
	split_line=( $line )
	lb_name=${split_line[1]}
	
	# now that we have the name of the load balancer, we can show the details to find which Public IP (PIP) is associated to it
	#
	# generic syntax:
	# azure network lb show -g RESOURCE-GROUP-NAME -n LOAD-BALANCER-NAME
	#
	# example with some quick-and-dirty grep-fu to store the result in a variable:
	line=$(azure network lb show -g negatvmssrg -n $lb_name | grep loadBalancerFrontEnd)
	split_line=( $line )
	pip_name=${split_line[4]}
	
	# now that we have the name of the public IP address, we can show the details to find the FQDN
	#
	# generic syntax:
	# azure network public-ip show -g RESOURCE-GROUP-NAME -n PIP-NAME
	#
	# example with some quick-and-dirty grep-fu to store the result in a variable:
	line=$(azure network public-ip show -g negatvmssrg -n $pip_name | grep FQDN)
	split_line=( $line )
	FQDN=${split_line[3]}
	
	# now that we have the FQDN, we can use ssh on port 50,000+i to connect to VM i (where i is 0-indexed)
	#
	# example to connct via ssh into VM "0":
	ssh -p 50000 negat@$FQDN

<!---HONumber=Mooncake_0425_2016-->