<properties
   pageTitle="与安全边界环境配合使用的示例应用程序 | Azure"
   description="创建外围网络以测试流量传送方案后，请部署这个简单的 Web 应用"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
	ms.service="virtual-network"
	ms.date="02/01/2016"
	wacn.date="03/28/2016"/>

# 与安全边界环境配合使用的示例应用程序

这些 PowerShell 脚本可以在 IIS01 和 AppVM01 服务器本地上运行，以安装和设置一个极简单的 Web 应用，显示来自前端 IIS01 服务器的 html 网页和来自后端 AppVM01 服务器的内容。

此应用程序提供简单的测试环境，可测试许多外围网络示例，以及终结点、NSG、UDR 及防火墙规则的更改如何影响流量。

## 允许 ICMP 的防火墙规则
这个简单的 PowerShell 语句可以在任何 Windows VM 上运行，以允许 ICMP (Ping) 流量。由于这可允许 ping 协议通过 Windows 防火墙，因此让测试和疑难解答变得更轻松（ICMP 在多数的 Linux 分发版上默认为打开）。

	# Turn On ICMPv4
	New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" `
		-Protocol ICMPv4 -Enabled True -Profile Any -Action Allow

**注意：**如果你使用以下脚本，添加的这个防火墙规则是第一个语句。

## IIS01 - Web 应用安装脚本
此脚本将：

1.	在本地服务器 Windows 防火墙打开 IMCPv4 (Ping) 以方便测试
2.	安装 IIS 和 .Net Framework v4.5
3.	创建 ASP.NET 网页和 Web.config 文件
4.	更改默认应用程序池以方便访问文件
5.	将匿名用户设置为你的管理员帐户和密码

在通过 RDP 访问 IIS01 时，此 PowerShell 脚本应在本地运行。

	# IIS Server Post Build Config Script
	# Get Admin Account and Password
		Write-Host "Please enter the admin account information used to create this VM:" -ForegroundColor Cyan
		$theAdmin = Read-Host -Prompt "The Admin Account Name (no domain or machine name)"
		$thePassword = Read-Host -Prompt "The Admin Password"
		
	# Turn On ICMPv4
		Write-Host "Creating ICMP Rule in Windows Firewall" -ForegroundColor Cyan
		New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" -Protocol ICMPv4 -Enabled True -Profile Any -Action Allow
		
	# Install IIS
		Write-Host "Installing IIS and .Net 4.5, this can take some time, like 15+ minutes..." -ForegroundColor Cyan
		add-windowsfeature Web-Server, Web-WebServer, Web-Common-Http, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Static-Content, Web-Health, Web-Http-Logging, Web-Performance, Web-Stat-Compression, Web-Security, Web-Filtering, Web-App-Dev, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Net-Ext, Web-Net-Ext45, Web-Asp-Net45, Web-Mgmt-Tools, Web-Mgmt-Console
		
	# Create Web App Pages
		Write-Host "Creating Web page and Web.Config file" -ForegroundColor Cyan
		$MainPage = '<%@ Page Language="vb" AutoEventWireup="false" %>
		<%@ Import Namespace="System.IO" %>
		<script language="vb" runat="server">
			Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
			    Dim FILENAME As String = "\\10.0.2.5\WebShare\Rand.txt"
			    Dim objStreamReader As StreamReader
			    objStreamReader = File.OpenText(FILENAME)
			    Dim contents As String = objStreamReader.ReadToEnd()
			    lblOutput.Text = contents
			    objStreamReader.Close()
		        lblTime.Text = Now()
			End Sub
		</script>
			
		<!DOCTYPE html>
		<html xmlns="http://www.w3.org/1999/xhtml">
		<head runat="server">
			<title>DMZ Example App</title>
		</head>
		<body style="font-family: Optima,Segoe,Segoe UI,Candara,Calibri,Arial,sans-serif;">
		  <form id="frmMain" runat="server">
		    <div>
		      <h1>Looks like you made it!</h1>
		      This is a page from the inside (a web server on a private network),<br />
		      and it is making its way to the outside! (If you are viewing this from the internet)<br />
		      <br />
		      The following sections show:
		      <ul style="margin-top: 0px;">
		        <li> Local Server Time - Shows if this page is or isnt cached anywhere</li>
		        <li> File Output - Shows that the web server is reaching AppVM01 on the backend subnet and successfully returning content</li>
		        <li> Image from the Internet - Doesnt really show anything, but it made me happy to see this when the app worked</li>
		      </ul>
		      <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
		        <b>Local Web Server Time</b>: <asp:Label runat="server" ID="lblTime" /></div>
		      <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
		        <b>File Output from AppVM01</b>: <asp:Label runat="server" ID="lblOutput" /></div>
		      <div style="border: 2px solid #8AC007; border-radius: 25px; padding: 20px; margin: 10px; width: 650px;">
		        <b>Image File Linked from the Internet</b>:<br />
		        <br />
		        <img src="http://sd.keepcalm-o-matic.co.uk/i/keep-calm-you-made-it-7.png" alt="You made it!" width="150" length="175"/></div>
		    </div>
		  </form>
		</body>
		</html>'
		
		$WebConfig ='<?xml version="1.0" encoding="utf-8"?>
		<configuration>
		  <system.web>
		    <compilation debug="true" strict="false" explicit="true" targetFramework="4.5" />
		    <httpRuntime targetFramework="4.5" />
		    <identity impersonate="true" />
		    <customErrors mode="Off"/>
		  </system.web>
		  <system.webServer>
		    <defaultDocument>
		      <files>
		        <add value="Home.aspx" />
		      </files>
		    </defaultDocument>
		  </system.webServer>
		</configuration>'
			
		$MainPage | Out-File -FilePath "C:\inetpub\wwwroot\Home.aspx" -Encoding ascii
		$WebConfig | Out-File -FilePath "C:\inetpub\wwwroot\Web.config" -Encoding ascii
	
	# Set App Pool to Clasic Pipeline to remote file access will work easier
		Write-Host "Updaing IIS Settings" -ForegroundColor Cyan
		c:\windows\system32\inetsrv\appcmd.exe set app "Default Web Site/" /applicationPool:".NET v4.5 Classic"
		c:\windows\system32\inetsrv\appcmd.exe set config "Default Web Site/" /section:system.webServer/security/authentication/anonymousAuthentication /userName:$theAdmin /password:$thePassword /commit:apphost
		
	# Make sure the IIS settings take
		Write-Host "Restarting the W3SVC" -ForegroundColor Cyan
		Restart-Service -Name W3SVC
		
		Write-Host
		Write-Host " Web App Creation Successfull!" -ForegroundColor Green
		Write-Host


## AppVM01 - 文件服务器安装脚本
此脚本设置此简单应用程序的后端。此脚本将：

1.	在防火墙打开 IMCPv4 (Ping) 以方便测试
2.	创建新目录
3.	创建上述网页要远程访问的文本文件
4.	将目录和文件的权限设为匿名以允许访问
5.	关闭 IE 增强的安全性以方便从此服务器浏览 

>[AZURE.IMPORTANT]**最佳实践**：切勿在生产服务器上关闭“IE 增强的安全性”，而且从生产服务器浏览网页通常不是个好主意。此外，最好不要打开文件共享来进行匿名访问，此处这样做是为了简单起见。

在通过 RDP 访问 AppVM01 时，此 PowerShell 脚本应在本地运行。必须以管理员身份运行 PowerShell 才能确保成功执行。
	
	# AppVM01 Server Post Build Config Script
	# PowerShell must be run as Administrator for Net Share commands to work
	
	# Turn On ICMPv4
	    New-NetFirewallRule -Name Allow_ICMPv4 -DisplayName "Allow ICMPv4" -Protocol ICMPv4 -Enabled True -Profile Any -Action Allow
	
	# Create Directory
	    New-Item "C:\WebShare" -ItemType Directory
	
	# Write out Rand.txt
	    $FileContent = "Hello, I'm the contents of a remote file on AppVM01."
	    $FileContent | Out-File -FilePath "C:\WebShare\Rand.txt" -Encoding ascii
	
	# Set Permissions on share
	    $Acl = Get-Acl "C:\WebShare"
	    $AccessRule = New-Object system.security.accesscontrol.filesystemaccessrule("Everyone","ReadAndExecute, Synchronize","ContainerInherit, ObjectInherit","InheritOnly","Allow")
	    $Acl.SetAccessRule($AccessRule)
	    Set-Acl "C:\WebShare" $Acl
	
	# Create network share
	    Net Share WebShare=C:\WebShare "/grant:Everyone,READ"
	
	# Turn Off IE Enhanced Security Configuration for Admins
	    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0
	
	    Write-Host
	    Write-Host "File Server Setup Successfull!" -ForegroundColor Green
	    Write-Host
	

## DNS01 - DNS 服务器安装脚本
本示例应用程序中未包含设置 DNS 服务器的脚本。如果测试防火墙规则、NSG 或 UDR 时需要包含 DNS 流量，必须手动安装 DNS01 服务器。这两个示例的网络配置 XML 文件都包含 DNS01 作为主要 DNS 服务器，而由第 3 级托管的公共 DNS 服务器则作为备份 DNS 服务器。第 3 级 DNS 服务器是非本地流量使用的实际 DNS 服务器，若未安装 DNS01，就不会有本地 DNS。

<!--Link References-->
[HOME]: /documentation/articles/best-practices-network-security/

<!---HONumber=82-->