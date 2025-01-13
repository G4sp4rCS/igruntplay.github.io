# Pivoting & Port forwarding
## WORK IN PROGRESS

- [SSH port forwarding + proxychains](./sshPortForwardingProxychains.md)
- [Reverse port forwarding SSH](./reversePortForwardingSSH.md)
    - [Chisel Alternative](./chisel.md)
- [Ligolo](./ligolo.md)

- Ping sweep linux: `for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done`
- Ping sweep windows cmd: `for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"`
- Ping sweep windows powershell: `1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}`