1. 将安装程序复制到要保护的服务器上的某个本地文件夹（如 \\Temp），然后在权限提升的命令提示符下运行以下命令。

  ```
  cd C:\Temp
  ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
  MobilityServiceInstaller.exe /q /xC:\Temp\Extracted
  cd C:\Temp\Extracted.
  ```
2. 现在，可使用以下命令行安装移动服务

  ```
  UnifiedAgent.exe /Role "Agent" /CSEndpoint "IP Address of Configuration Server" /PassphraseFilePath <Full path to the passphrase file>``
  ```

#### 移动服务安装程序命令行参数

```
Usage :
UnifiedAgent.exe [/Role <Agent/MasterTarget>] [/InstallLocation <Installation Directory>] [/CSIP <IP address>] [/PassphraseFilePath <Passphrase file path>] [/LogFilePath <Log File Path>]<br/>
```

  | 参数|类型|说明|可能的值| 
  |-|-|-|-| 
  |/Role|必需|指定是否应安装移动服务|Agent </br> MasterTarget| 
  |/InstallLocation|必需|移动服务的安装位置|计算机上的任意文件夹| 
  |/CSIP|必需|配置服务器的 IP 地址| 任何有效的 IP 地址| 
  |/PassphraseFilePath|必需|通行短语的存储位置 |任何有效的 UNC 或本地文件路径| 
  |/LogGilePath|可选|安装日志的位置|计算机上的任意有效文件夹|

#### 示例用法

```
  UnifiedAgent.exe /Role "Agent" /CSEndpoint "I192.168.2.35" /PassphraseFilePath "C:\Temp\MobSvc.passphrase"
```

<!---HONumber=Mooncake_0327_2017-->