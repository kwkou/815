<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle">资源</th>
   <th align="left" valign="middle">默认限制</th>
   <th align="left" valign="middle">最大限制</th>
</tr>
<tr>
   <td valign="middle"><p>每个云服务的<a href="/services/virtual-machines">虚拟机数</a><sup>1</sup></p></td>
   <td valign="middle"><p>50</p></td>
   <td valign="middle"><p>50</p></td>
</tr>
<tr>
   <td valign="middle"><p>每个云服务的输入终结点数<sup>2</sup></p></td>
   <td valign="middle"><p>150</p></td>
   <td valign="middle"><p>150</p></td>
</tr>
</table>

<sup>1</sup>当你在 Azure 资源组外部创建一个虚拟机时，系统会自动创建一个云服务来包含该虚拟机。然后可以在该相同的云服务中添加多个虚拟机。

<sup>2</sup>输入终结点用于允许包含云服务的到外部虚拟机的通信。同一个云服务中的虚拟机自动允许所有 UDP 和 TCP 端口之间的通信，以实现内部通信。

<!---HONumber=HO63-->