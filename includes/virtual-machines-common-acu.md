我们已创建 Azure 计算单元 (ACU)，提供一种比较 Azure SKU 的计算 (CPU) 性能的方法。 这有助于轻松确定最有可能满足性能需求的 SKU。  ACU 目前在小型 (Standard_A1) VM 上标准为 100，而所有其他 SKU 表示 SKU 在运行标准基准测试时大约可以有多快。 

> [AZURE.IMPORTANT]
> ACU 只是一种规则。  工作负荷的结果可能会有所不同。 
> 
> 

<br>

| SKU 系列 | ACU/核心 |
| --- | --- |
| [A0](/documentation/articles/virtual-machines-windows-sizes-general/) |50 |
| [A1-A4](/documentation/articles/virtual-machines-windows-sizes-general/) |100 个 |
| [A5-A7](/documentation/articles/virtual-machines-windows-sizes-general/) |100 |
| [A1_v2-A8_v2](/documentation/articles/virtual-machines-windows-sizes-general/) |100 |
| [A2m_v2-A8m_v2](/documentation/articles/virtual-machines-windows-sizes-general/) |100 |
| [D1-D14](/documentation/articles/virtual-machines-windows-sizes-general/) |160 |
| [D1_v2-D15_v2](/documentation/articles/virtual-machines-windows-sizes-general/) |210 - 250* |
| [DS1-DS14](/documentation/articles/virtual-machines-windows-sizes-memory/) |160 |
| [DS1_v2-DS15_v2](/documentation/articles/virtual-machines-windows-sizes-memory/) |210-250* |
| [F1-F16](/documentation/articles/virtual-machines-windows-sizes-compute/) |210-250* |
| [F1s-F16s](/documentation/articles/virtual-machines-windows-sizes-compute/) |210-250* |

ACU 标有 *使用 Intel® Turbo 技术来增加 CPU 频率，并提升性能。  提升量可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。