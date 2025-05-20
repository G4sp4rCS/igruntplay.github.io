# Shell Management
1. `python3 -c 'import pty; pty.spawn("/bin/bash")'`
2. CTRL + Z
3. `stty raw -echo; fg`
6. `export TERM=xterm-256color`
7. `stty sane`
8. `stty rows 40; stty cols 80`


## alternativa
- `rlwrap nc -lvnp $port`