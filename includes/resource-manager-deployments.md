## <a name="incremental-and-complete-deployments"></a>增量部署和完整部署
部署资源时，可以指定部署为增量更新还是完整更新。 这两种模式的主要区别是 Resource Manager 如何处理资源组中不在模板中的现有资源：

* 在完整模式中，Resource Manager **删除**资源组中已存在但尚未在模板中指定的资源。 
* 在增量模式中，Resource Manager **保留**资源组中已存在但尚未在模板中指定的资源。

对于这两种模式，Resource Manager 都会尝试预配在模板中指定的所有资源。 如果资源已存在于资源组中且其设置未更改，该操作不会导致任何更改。 如果更改某个资源的设置，则会使用这些新设置预配资源。 如果尝试更新现有资源的位置或类型，则部署会失败并出现错误。 请改用所需的位置或类型部署新资源。

Resource Manager 默认使用增量模式。

为了说明增量模式和完整模式的差异，请考虑以下方案。

**现有资源组**包含：

* 资源 A
* 资源 B
* 资源 C

**模板**定义：

* 资源 A
* 资源 B
* 资源 D

在“增量”模式下部署时，资源组包含：

* 资源 A
* 资源 B
* 资源 C
* 资源 D

在“完整”模式下部署时，会删除资源 C。 该资源组包含：

* 资源 A
* 资源 B
* 资源 D

<!--Update_Description: Add new illustration to understand the difference between incremental and complete deployment-->