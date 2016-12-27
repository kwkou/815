<!-- need to be verified -->


如果尚未这样做，则你可以获取 [Azure 订阅试用版](/pricing/1rmb-trial/)和[连接到你的 Azure 帐户](/documentation/articles/xplat-cli-connect/)的 [Azure CLI](/documentation/articles/xplat-cli-install/)。请确保 Azure CLI 处于 Resource Manager 模式下，如下所示：

    azure config mode arm

现在，使用 `azure vmss quick-create` 命令创建规模集。以下示例在名为 `myResourceGroup` 的资源组中创建具有 5 个 VM 实例的名为 `myVMSS` 的规模集：

    azure vmss quick-create -n myVMSS -g myResourceGroup -l chinanorth \
        -u ops -p P@ssw0rd! \
        -C 5 -Q Canonical:UbuntuServer:14.04.3-LTS:latest

如果想要自定义位置或 image-urn，请查看命令 `azure location list` 和 `azure vm image {list-publishers|list-offers|list-skus|list|show}`。

此命令返回后，将创建规模集。此规模集将包含一个负载均衡器，其 NAT 规则会将负载均衡器上的端口 50,000 + 映射到 VM i 上的端口 22。因此，在确定负载均衡器的 FQDN 后，我们就可以通过 ssh 连接到 VM：

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

    # now that we have the name of the load balancer, we can show the details to find which Public IP (PIP) is 
    # associated to it
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

<!---HONumber=Mooncake_1212_2016-->