## 3.1. Analysis of suspicious activity

## 3.1.1. Analysis during exploitation
### Option 1: Network connections
Run the following command to see active network connections and identify suspicious IPs:

```bash
netstat -antp
```
- `a` → Show all connections  
- `n` → Show numerical addresses instead of resolving hostnames  
- `t` → Show TCP connections  
- `p` → Show the process associated with the connection  

![Alt text](/images/Pasted%20image%2020250211211920.png)

We found the following suspicious activity:
- **Protocol**: `tcp`
- **Attacker's IP:** `192.168.1.110`
- **Port Used:** `6200`
- **PID**: `4922` 
- **Program name**: `sh` (Reverse shell)

### Option 2:  Check Running Processes
Find active reverse shells or suspicious services. Attackers often use:
- **Netcat (nc)** for reverse shells
- **Bash shells** (`/bin/bash -i`)
- **Python reverse shell scripts**

```bash
ps aux | grep nc
ps aux | grep bash
ps aux | grep sh
```

We have found the unrecognized process PID `4922` named `sh` as we have found in option 1.

![Alt text](images/Pasted%20image%2020250211212857.png)

### Option 3:  Monitor network traffic
If the attack is ongoing, use **tcpdump** to capture packets through the infected port:

```bash
sudo tcpdump -i eth0 port 6200
```

This will show real-time traffic, helping you identify the attacker:

![Alt text](images/Pasted%20image%2020250211213232.png)

## 3.1.2. Analysis after exploitation
### Option 1: System logs

Check authentication logs for suspicious login attempts:
```bash
cat /var/log/auth.log
```

Someone with root access has read the file `testfile.txt`:
![Alt text](images/Pasted%20image%2020250213203019.png)
And executed an unknown executable named `hola`:
![Alt text](images/Pasted%20image%2020250216164835.png)

Moreover, attacker has injected a public ssh key in the system to connect to ssh as root user:
![Alt text](images/Pasted%20image%2020250216164120.png)

Check the `vsftp.log` to see multiple failed attempts or unexpected logins:
```bash
cat /var/log/vsftpd.log
```

![Alt text](images/Pasted%20image%2020250211212437.png)

## 3.1.3. Conclusions and consequences of the exploit

The infected `tcp`port is `6200` with the service `vsftpd`. By using this vulnerability the attacker has run a reverse shell with PID `4922` from the ip `192.168.56.110`. If we go to the [NIST database](https://nvd.nist.gov/vuln/detail/CVE-2011-2523) to investigate about the possible exploitation, we see that the vulnerability is called `CVE-2011-2523`:

> vsftpd 2.3.4 downloaded between 20110630 and 20110703 contains a backdoor which opens a shell on port 6200/tcp.

Moreover, the attacker has injected a payload named `hola` to upload their public ssh key to the system to have root access. This exploit would have high priority as **the attacker has root access to the system.**


---
## 3.2. Defensive measures against vulnerability vsftpd 2.3.4
### 3.2.1. Kill malicious processes

Kill the PID of the reverse shells created by the vsfptd vulnerability and the payload:
  ```bash
  sudo kill -9 <PID>
  ```
 
On the attacker the exploit is stopped:
![Alt text](images/Pasted%20image%2020250211213942.png)

### 3.2.2. Block the attacker's IP
```bash
sudo iptables -A INPUT -s <ATTACKER_IP> -j DROP
```

- **`-A INPUT`**:
    - **`-A`**: Stands for "append," which adds the rule to the end of the specified chain.
    - **`INPUT`**: Specifies the chain where the rule will be added. The `INPUT` chain handles incoming traffic to the local system. This means the rule applies to all incoming traffic.
- **`-s <ATTACKER_IP>`**:
    - **`-s`**: This option is used to specify the **source IP address**.
    - **`<ATTACKER_IP>`**: This is a placeholder for the IP address of the attacker (you would replace `<ATTACKER_IP>` with the actual IP address of the attacker). The rule applies only to traffic coming from this specific IP address.
- **`-j DROP`**:
    - **`-j`**: This stands for "jump," which specifies what action to take if the rule matches the traffic.
    - **`DROP`**: This action drops the packets (i.e., it silently discards the incoming traffic). No response will be sent back to the attacker, and the connection is effectively blocked.

Kali cannot perform the attack
![Alt text](images/Pasted%20image%2020250211214221.png)

To list all rules in a readable format:
```
sudo iptables -L -v -n
```

`-L` → List rules
`-v` → Show detailed information
`-n` → Show numerical IPs (prevents DNS lookup delays)

### 3.2.3. **Block the infected port**

Block the port that is being exploited by the VSFPTD vulneravility (6200):
```bash
sudo iptables -I INPUT 1 -p tcp --dport 6200 -j DROP
```

- **`-I INPUT 1`**:
    - **`-I`**: This option stands for "insert" and tells `iptables` to add a new rule to the chain.
    - **`INPUT`**: This specifies the chain in which the rule is added. The `INPUT` chain is for traffic that is destined for the local machine (i.e., incoming traffic).
    - **`1`**: This specifies the position where the rule should be inserted. In this case, the rule is added at the **first position** in the `INPUT` chain, meaning it will be checked first when evaluating incoming traffic.
- **`-p tcp`**: This specifies the **protocol**. The rule applies to **TCP** traffic (as opposed to other protocols like UDP, ICMP, etc.).
- **`--dport 6200`**: This option defines the **destination port**. The rule applies to incoming traffic on **port 6200**.
- **`-j DROP`**: This specifies the **action** to take if the rule matches. The `DROP` action means that the incoming traffic on port 6200 will be silently discarded (not allowed through). The connection will not be established, and no response will be sent.

---
## 3.3. Defensive measures against SSH Key persistence
### 3.3.1. Disable root logging in SSH connection

- Disable SSH root login in `/etc/ssh/sshd_config`:
```
PermitRootLogin no
```

### 3.3.2. Manage SSH connection

- Monitor ~/.ssh/authorized_keys for unauthorized entries.
### 3.3.3. Block the attacker's IP
Same as in 3.2.2.
### 3.3.4. Block the infected port
Same as in 3.2.3. In this case port 22.

---
## 3.4. Defensive measures against password and user details cracking
### 3.4.1. Hash passwords differently

In this case, it has hashed with md5. It can be used more secure encriptation like sha-256.
