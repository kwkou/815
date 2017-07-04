1. 将安装程序复制到要保护的服务器上的某个本地文件夹（例如 C:\Temp）。 以管理员身份在命令提示符处运行以下命令：

      cd C:\Temp
      ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
      MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
      cd C:\Temp\Extracted.

2. 若要安装移动服务，请运行以下命令：

      UnifiedAgent.exe /Role "Agent" /CSEndpoint "IP address of the configuration server" /PassphraseFilePath <Full path to the passphrase file>``

#### <a name="mobility-service-installer-command-line-arguments"></a>移动服务安装程序命令行参数

    Usage :
    UnifiedAgent.exe [/Role <Agent/MasterTarget>] [/InstallLocation <Installation directory>] [/CSIP <IP address>] [/PassphraseFilePath <Passphrase file path>] [/LogFilePath <Log file path>]<br/>

  | 参数|类型|说明|可能的值|
  |-|-|-|-|
  |/Role|必需|指定是否应安装移动服务|代理 </br> MasterTarget|
  |/InstallLocation|必需|移动服务安装到的位置|计算机上的任意文件夹|
  |/CSIP|必需|配置服务器的 IP 地址| 任何有效的 IP 地址|
  |/PassphraseFilePath|必需|通行短语的位置 |任何有效的 UNC 或本地文件路径|
  |/LogFilePath|可选|安装日志的位置|计算机上的任意有效文件夹|

#### <a name="example"></a>示例

      UnifiedAgent.exe /Role "Agent" /CSEndpoint "I192.168.2.35" /PassphraseFilePath "C:\Temp\MobSvc.passphrase"