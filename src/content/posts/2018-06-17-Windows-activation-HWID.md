---
title: Windows 10  数字许可证激活(HWID)方法——原创翻译
published: 2018-06-17
description: ''
image: ''
tags: [Windows]
category: 'Note'
draft: false 
---

更新：2018-10-04 12:16

### 前言

在 Windows 10 的所有系统中，无论系统是如何被激活的（通过 Windows7/8.1 升级 或者 购买零售密钥版密钥 或者 嵌入BIOS又名MSDN许可证）都会被转换成基于各自的设备硬件 ID (HWID) 的一个数字许可证。这个证书储存在微软服务器并且每次安装系统时都会激活这台设备。只有当这台设备的硬件被更改才会导致许可证失效。通过将它绑定到一个微软账户 (MSA) 你可以在这种情况下转移证书从而激活这台(硬件被更改的)设备。

这个过程在每台机器上只需要执行一次。在以后的安装中只需要跳过任何需要密钥的地方（在安装的时候选择 ‘我没有激活密钥’），之后第一次联网的时候将自动联系微软服务器注册 HWID 并激活这台设备。

注意：如果从 VLSC 或者 MVS Business ISO 中安装了批量许可证版本（VL版），需要输入默认的零售/OEM 密钥 才能重新获得激活。

这个过程实际上非常简单，并且不会篡改任何系统文件和泄露密钥。

许可证的创建流程已经对所有的 MS SKU 版本进行了适配和优化，所以以下的手动激活模式可以完全适用在这些版本上。自动激活模式是一个最简单的激活方式，并且使用在所有 MS SKU 版本上以及为下面的版本进行了专门的设计优化：

支持的 Windows 10 版本 （Skus）：

- 核心（家庭）版(N）
- 核心单语言版(N）
- 专业版(N）
- 专业教育版(N）
- 专业工作站版(N）
- 教育版(N）
- 企业版(N）
- 企业 S 版(N）【2015/2016：数字证书】
- 企业 S 版(N）【2019：19年的离线 KMS 激活（和企业G版相似】
- 服务器标准（核心）版(N）【2016/2019：19年的离线 KMS 激活（和企业G版相似】
- 服务器数据中心（核心）版(N）【2016/2019：19年的离线 KMS 激活（和企业G版相似】
- 服务器解决方案（核心）版(N）【2016/2019：19年的离线 KMS 激活（和企业G版相似】

-----

### 手动模式

1. 从 Windows 10 17134 ISO 内 获取 GatherOsState.exe 

2. 从 [GitHub](https://github.com/vyvojar/slshim/releases) 获取最新的 slshim 

3. 提取 slshim32.dll (适用于 x86 ISO) 或者 slshim64.dll (适用于 x64 ISO)

4. 将 GatherOsState.exe 和 提取的 slshim(32/64).dll 放置在相同的文件夹内

5. 将 slshim(32/64).dll 重命名为 slc.dll

6. 导入注册表

   1. 从下列列表中选择对应的 %sku% 值

      ```
      edition=Cloud
      sku=178
      	
      edition=CloudN
      sku=179
      
      edition=Core
      sku=101
      
      edition=CoreCountrySpecific
      sku=99
      
      edition=CoreN
      sku=98
      
      edition=CoreSingleLanguage
      sku=100
      
      edition=Education
      sku=121
      
      edition=EducationN
      sku=122
      
      edition=Enterprise
      sku=4
      
      edition=EnterpriseN
      sku=27
      
      edition=EnterpriseS
      sku=125
      
      edition=EnterpriseSN
      sku=126
      
      edition=Professional
      sku=48
      
      edition=ProfessionalEducation
      sku=164
      
      edition=ProfessionalEducationN
      sku=165
      	
      edition=ProfessionalN
      sku=49
      
      edition=ProfessionalWorkstation
      sku=161
      
      edition=ProfessionalWorkstationN
      sku=162
      ```

   2. 使用对应的 %sku%值 替换 XXX 部分 。 如果使用导入注册表的方法请确保字符串长度为7位，CMD 的方式直接使用上面的值即可。

      CMD：

      ```
      reg add "HKLM\SYSTEM\Tokens" /v "Channel" /t REG_SZ /d "Retail" /f
      reg add "HKLM\SYSTEM\Tokens\Kernel" /v "Kernel-ProductInfo" /t REG_DWORD /d XXX /f
      reg add "HKLM\SYSTEM\Tokens\Kernel" /v "Security-SPP-GenuineLocalStatus" /t REG_DWORD /d 1 /f
      ```

      REG：

      ```
      Windows Registry Editor Version 5.00
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens]
      "Channel"="Retail"
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens\Kernel]
      "Kernel-ProductInfo"=dword:0000XXX
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens\Kernel]
      "Security-SPP-GenuineLocalStatus"=dword:00000001
      ```

7. 从密钥列表中找到对应的默认 零售/OEM 密钥 并安装

   [密钥列表](https://pastebin.com/rYakstDc)

   如果你是用 企业版N 或者 LTSB 2016 N 版请在管理员模式下运行 Powershell 运行以下代码

   ```
   ::EnterpriseN
   ((Get-Content '.\gatherosstate.exe') -replace "`0" | Select-String -Pattern "(.....-){4}C372T" -AllMatches).Matches | Select-Object -ExpandProperty Value
   
   ::EnterpriseSN
   ((Get-Content '.\gatherosstate.exe') -replace "`0" | Select-String -Pattern "(.....-){4}VMJWR" -AllMatches).Matches | Select-Object -ExpandProperty Value
   ```

   这会从 GatherOsState.exe 获取密钥

8. 运行 GatherOsState.exe 。几秒钟后你会获得 GenuineTicket.xml

9. (可选)从注册表中移除 HKEY_LOCAL_MACHINE\SYSTEM\Tokens 

   CMD：

   ```
   reg delete "HKLM\SYSTEM\Tokens" /f
   ```

   REG：

   ```
   Windows Registry Editor Version 5.00
   
   [-HKEY_LOCAL_MACHINE\SYSTEM\Tokens]
   ```

10. 把创建的 GenuineTicket.xml 移动到 C盘根目录 然后 管理员 CMD 执行

    ```
    clipup -v -o -altto c:\
    ```

11. 然后使用以下命令强制激活

    ```
    cscript /nologo %windir%\system32\slmgr.vbs -ato
    ```

完成。恭喜。

-----

### 自动模式

**请不要使用任何代理**

注意：这个工具需要一些时间（取决于你的设备性能）执行系统检查，莫方，稍等即可。谢谢。

**v20.10** 添加了 1809 的 GatherOsState.exe，添加了对 LTSC/Server（2016/19）的离线 KMS 激活支持（19年期限）

**v10.24** 修复删除错误的注册表的问题（感谢 angelkyo 的提醒）

**v10.21** 更正了 Win7  的 compat 条目（感谢 alert 的资源）[ 译者注：compat 怀疑是作者笔误？]

**v10.18** 修复了 破解 LTSB 2016 的过程

**v10.15** 增加了 Enterprise LTSB 2015 （N）的支持  （仅在非 N 的版本上进行了测试）（感谢 hwidmod 提供所需的 gatherosstate.exe） 

**v10.08** 添加了密钥安装模式（下拉菜单中），对于使用 VL ISO 重装，已经有 HWID 和不需要整个激活过程的设备快速切换到 零售/OEM 版本 ，工具将会把此密钥显示在系统信息中如果这个密钥没有被安装的话。

**v10.01** 稍微改变了在 Win 7 兼容模式下运行 GatherOsState.exe 的过程，所以创建票据的过程将会把系统信息设置为 Windows 7，这将会更好地模仿从一个 Win 7 系统获得原始票据。优化了 界面

**v9.32** 添加了 nsane 和 aiowares 论坛的链接以获取信息和支持

**v9.25** 在没有用户干预的情况下将更改初始信息界面改为启动屏幕（注：我并没有使用此工具也不知道是什么意思）

**v9.18** 重做了系统检查部分

**v9.11** 添加了对 LTSB 2015 版的支持（仅支持非N版本并且目前尚未测试）添加了静默模式

……（太久远了的更新不想翻译了(╯‵□′)╯︵┻━┻）……

**v9.04** fixed spelling error in splash pic

**v9.01** fixed the KMS detection (will work on activated KMS systems now) amd added silent mode

**v8.13** added Messagebox to inform user tool-start-up might need a moment, fixed tool not closing when done via the 'X'

**v8.06** changed disabled WU handling to: set to auto, start service, activate, stop service and set back to disabled

**v7.99** added last checks and some code cleanup

**v7.77** implemented disabled WU handling.

 使用静默模式

```
hwidgen.mk3 silent
```

 下载地址（本人做了搬运并且将持续更新）

[Google Drive (包含历史版本](https://drive.google.com/open?id=14swGfWlpXVcfE9U_mu8InPbAqtf78KzO))

解压密码

3Fs44Rv#tZ4u3UOij656NgF____

各种哈希值

```
BLAKE2sp: 6532912a2006d902eae31518586b1a752746b9e9cc413927dbdf85c07e9b1888
CRC64: b53b2c47ec894c4e
MD5: 52718308d772440a8c1796460f751c35
SHA-1: a4001818941631c6c7d92c4d21d79e2962ab18ca
SHA-256: ef22089f5267d151950761c3c76426514450f59fd4c5a342ad856f3954d2a846
SHA-384: 2b0e41572fcbf83ed9b67b5221e79757d7970fe17620984bd0c07a8fd4a1e4d31ed977d3bf0da34a4809b749481dbb65
SHA-512: 4c9e6ecf7b4be0186a6f49c51716df595f8fc871685c35205ec1bfe39435157d54ef23ea7fe20aa40db76015a331d3c59f7d87d89814022b4aca589b40d4d854
SHA3-224: 9f771530c10ee392a824eb29971a3b3742e686c2b3a0a0ec5c028053
SHA3-256: 70d6481756e8625f01ec7c3417e8483bce88bc695d31ff5e20f8b09746f1e539
```

[病毒检测报告](https://www.virustotal.com/de/file/d6117778cd38325cc1572f625a276e286f91f6eff98724456060bcbacf4f8c1f/analysis/1526509270/)

[**原帖地址**](http://www.nsaneforums.com/topic/312871-windows-10-digital-license-hwid-generation-without-kms-or-predecessor-installupgrade/)
