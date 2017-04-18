## <a name="prerequisites"></a>先决条件

获取 [Azure 订阅试用版](/pricing/1rmb-trial/)和[连接到 Azure 帐户](/documentation/articles/xplat-cli-connect/)的 [Azure CLI 1.0](/documentation/articles/cli-install-nodejs/)（如果尚未这样做）。 请确保 Azure CLI 处于 Resource Manager 模式下，如下所示：

    azure config mode arm

## <a name="create-the-scale-set"></a>创建规模集

现在，使用 `azure vmss quick-create` 命令创建规模集。 以下示例在名为 `myResourceGroup` 的资源组中创建具有 5 个 VM 实例的 Linux 规模集 `myVMSS`：

    azure vmss quick-create -n myVMSS -g myResourceGroup -l chinanorth \
        -u ops -p P@ssw0rd! \
        -C 5 -Q Canonical:UbuntuServer:16.04-LTS:latest

以下示例将创建具有相同配置的 Windows 规模集：

    azure vmss quick-create -n myVMSS -g myResourceGroup -l chinanorth \
        -u ops -p P@ssw0rd! \
        -C 5 -Q MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest

如果想要自定义位置或 image-urn，请查看命令 `azure location list` 和 `azure vm image {list-publishers|list-offers|list-skus|list|show}`。

规模集完成部署后即返回此命令。 此时，此规模集包含一个负载均衡器，其 NAT 规则会将负载均衡器上的端口 50,000+i 映射到 VM i 上的端口 22。 因此，在确定负载均衡器的 FQDN 后，我们可以通过 ssh 连接到 VM：

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