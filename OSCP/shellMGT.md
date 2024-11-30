1. `python -c 'import pty; pty.spawn("/bin/bash")'`
2. CTRL + Z
3. `stty raw -echo`
4. `fg`
5. [ENTER]
6. [ENTER]
7. `export TERM=xterm-256color`
8. `stty rows 67 columns 318`