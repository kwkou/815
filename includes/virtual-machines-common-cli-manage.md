
* 如果你尚未安装 Azure CLI 并连接到订阅，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 和[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。如果你想要使用资源管理器模式命令，请务必使用登录方法连接。

* 若要开始在 Azure CLI 中使用资源管理器模式，可能需要切换命令模式。默认情况下，CLI 在服务管理模式下启动。若要更改为资源管理器模式，请运行 `azure config mode arm`。若要回到服务管理模式，请运行 `azure config mode asm`。

* 如需联机命令帮助和选项，请键入 `azure <command> <subcommand> --help` 或 `azure help <command> <subcommand>`。

## VM 任务
下表比较了你可以在服务管理和资源管理器中使用 Azure CLI 命令执行的常见 VM 任务。使用许多资源管理器命令时，你需要传递现有资源组的名称。

<table border="1"><thead><tr>
<th>任务 </th><th> 服务管理模式 </th><th> 资源管理器模式</th>
</tr></thead>
<tr>
<td>创建最基本的 VM </td><td> <code>azure vm create [options] &lt;dns-name&gt; &lt;image&gt; [userName] [password]</code> </td><td> <code>azure vm quick-create [options] &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; &lt;os-type&gt; &lt;image-urn&gt; &lt;admin-username&gt; &lt;admin-password&gt;</code><br/><br/>（从 <code>azure vm image list</code> 命令获取 <code>image-urn</code>。有关示例，请参阅 <a href="/documentation/articles/virtual-machines-windows-cli-ps-findimage/\">Windows</a> 或者 <a href="/documentation/articles/virtual-machines-linux-cli-ps-findimage/\">Linux</a>。）</td>
</tr>
<tr>
<td>创建 Linux VM </td><td> <code>azure vm create [options] &lt;dns-name&gt; &lt;image&gt; [userName] [password]</code> </td><td> <code>azure  vm create [options] &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; -y "Linux"</code></td>
</tr>
<tr>
<td>创建 Windows VM </td><td> <code>azure vm create [options] &lt;dns-name&gt; &lt;image&gt; [userName] [password]</code> </td><td> <code>azure  vm create [options] &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; -y "Windows"</code></td>
</tr>
<tr>
<td>列出 VM </td><td> <code>azure  vm list [options]</code> </td><td> <code>azure  vm list [options]</code></td>
</tr>
<tr>
<td>获取有关 VM 的信息 </td><td> <code>azure  vm show [options] &lt;vm_name&gt;</code> </td><td> <code>azure  vm show [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>启动 VM </td><td> <code>azure vm start [options] &lt;name&gt;</code> </td><td> <code>azure vm start [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>停止 VM </td><td> <code>azure vm shutdown [options] &lt;name&gt;</code> </td><td> <code>azure vm stop [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>释放 VM </td><td> 不可用 </td><td> <code>azure vm deallocate [options] &lt;resource-group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>重新启动 VM </td><td> <code>azure vm restart [options] &lt;vname&gt;</code> </td><td> <code>azure vm restart [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>删除 VM </td><td> <code>azure vm delete [options] &lt;name&gt;</code> </td><td> <code>azure vm delete [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>捕获 VM </td><td> <code>azure vm capture [options] &lt;name&gt;</code> </td><td> <code>azure vm capture [options] &lt;resource_group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>从用户映像创建 VM </td><td> <code>azure  vm create [options] &lt;dns-name&gt; &lt;image&gt; [userName] [password]</code> </td><td> <code>azure  vm create [options] -q &lt;image-name&gt; &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; &lt;os-type&gt;</code></td>
</tr>
<tr>
<td>从专用磁盘创建 VM </td><td> <code>azure  vm create [options]-d &lt;custom-data-file&gt; &lt;dns-name&gt; [userName] [password]</code> </td><td> <code>azue  vm create [options] -d &lt;os-disk-vhd&gt; &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; &lt;os-type&gt;</code></td>
</tr>
<tr>
<td>将数据磁盘添加到 VM </td><td> <code>azure  vm disk attach [options] &lt;vm-name&gt; &lt;disk-image-name&gt;</code> -或- <br/> <code>vm disk attach-new [options] &lt;vm-name&gt; &lt;size-in-gb&gt; [blob-url]</code> </td><td> <code>azure  vm disk attach-new [options] &lt;resource-group&gt; &lt;vm-name&gt; &lt;size-in-gb&gt; [vhd-name]</code></td>
</tr>
<tr>
<td>从 VM 中删除数据磁盘 </td><td> <code>azure  vm disk detach [options] &lt;vm-name&gt; &lt;lun&gt;</code> </td><td> <code>azure  vm disk detach [options] &lt;resource-group&gt; &lt;vm-name&gt; &lt;lun&gt;</code></td>
</tr>
<tr>
<td>将泛型扩展添加到 VM </td><td> <code>azure  vm extension set [options] &lt;vm-name&gt; &lt;extension-name&gt; &lt;publisher-name&gt; &lt;version&gt;</code> </td><td> <code>azure  vm extension set [options] &lt;resource-group&gt; &lt;vm-name&gt; &lt;name&gt; &lt;publisher-name&gt; &lt;version&gt;</code></td>
</tr>
<tr>
<td>将 VM 访问扩展添加到 VM </td><td> 不可用 </td><td> <code>azure vm reset-access [options] &lt;resource-group&gt; &lt;name&gt;</code></td>
</tr>
<tr>
<td>将 Docker 扩展添加到 VM </td><td> <code>azure  vm docker create [options] &lt;dns-name&gt; &lt;image&gt; &lt;user-name&gt; [password]</code> </td><td> <code>azure  vm docker create [options] &lt;resource-group&gt; &lt;name&gt; &lt;location&gt; &lt;os-type&gt;</code></td>
</tr>
<tr>
<td>将 Chef 扩展添加到 VM </td><td> <code>azure  vm extension get-chef [options] &lt;vm-name&gt;</code> </td><td> 不可用</td>
</tr>
<tr>
<td>禁用 VM 扩展 </td><td> <code>azure  vm extension set [options] -b &lt;vm-name&gt; &lt;extension-name&gt; &lt;publisher-name&gt; &lt;version&gt;</code> </td><td> 不可用</td>
</tr>
<tr>
<td>删除 VM 扩展 </td><td> <code>azure  vm extension set [options] -u &lt;vm-name&gt; &lt;extension-name&gt; &lt;publisher-name&gt; &lt;version&gt;</code> </td><td> <code>azure  vm extension set [options] -u &lt;resource-group&gt; &lt;vm-name&gt; &lt;name&gt; &lt;publisher-name&gt; &lt;version&gt;</code></td>
</tr>
<tr>
<td>列出 VM 扩展 </td><td> <code>azure vm extension list [options]</code> </td><td> 不可用</td>
</tr>
<tr>
<td>显示 VM 映像 </td><td> <code>azure vm image show [options]</code> </td><td> 不可用</td>
</tr>
<tr>
<td>获取 VM 资源的使用情况 </td><td> 不可用 </td><td> <code>azure vm list-usage [options] &lt;location&gt;</code></td>
</tr>
<tr>
<td>获取所有可用 VM 大小 </td><td> 不可用 </td><td> <code>azure vm sizes [options]</code></td>
</tr>
</table>


## 后续步骤

* 如需 CLI 命令的其他示例，请参阅[将 Azure 命令行接口用于 Azure 服务管理模式](/documentation/articles/virtual-machines-command-line-tools/)和[将 Azure CLI 用于 Azure 资源管理器模式](/documentation/articles/azure-cli-arm-commands/)。
