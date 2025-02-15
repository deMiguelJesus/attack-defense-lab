- Open ports
- Exploits used
- Security patches applied

## 3.1. Monitor Logs for Suspicious Activity in the target machine

### Option 1: Network connections
Run the following command to see active network connections and identify suspicious IPs:

```bash
netstat -antp
```
- `a` → Show all connections  
- `n` → Show numerical addresses instead of resolving hostnames  
- `t` → Show TCP connections  
- `p` → Show the process associated with the connection  

![[Pasted image 20250211211920.png]]

- **Attacker's IP:** `192.168.1.110`
- **Port Used:** `6200` (Reverse shell port)
- **PID**: 4922 (Reverse shell)

### Option 2: System logs

Check FTP authentication logs for suspicious login attempts:
```bash
cat /var/log/auth.log
```

![[Pasted image 20250213203019.png]]

```bash
cat /var/log/vsftpd.log
```

If you see multiple failed attempts or unexpected logins, note the source IP.

![[Pasted image 20250211212437.png]]

### Option 3:  Check Running Processes
Find active reverse shells or suspicious services:

```bash
ps aux | grep nc
ps aux | grep bash
ps aux | grep sh
```
Attackers often use:
- **Netcat (nc)** for reverse shells
- **Bash shells** (`/bin/bash -i`)
- **Python reverse shell scripts**

![[Pasted image 20250211212857.png]]

### Option 4:  Monitor network traffic

If the attack is ongoing, use **tcpdump** to capture packets:

```bash
sudo tcpdump -i eth0 port 6200
```

This will show real-time FTP traffic, helping you identify the attacker:

![[Pasted image 20250211213232.png]]

---
## 3.2. Defensive measures
### 3.2.1. **Kill malicious processes**

Kill the PID of the reverse shells created by the vsfptd vulnerability and the payload:
  ```bash
  sudo kill -9 <PID>
  ```
 
On the attacker the exploit is stopped:
![[Pasted image 20250211213942.png]]

### 3.2.2. **Block the attacker's IP**
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
![[Pasted image 20250211214221.png]] 

To list all rules in a readable format:
```
sudo iptables -L -v -n
```

`-L` → List rules
`-v` → Show detailed information
`-n` → Show numerical IPs (prevents DNS lookup delays)

### 3.2.3. **Block the infected ports**

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

### 3.2.3. Manage SSH Keys

- Monitor ~/.ssh/authorized_keys for unauthorized entries.
- Disable SSH root login in `/etc/ssh/sshd_config`:
```
PermitRootLogin no
```
- Use SSH key restrictions (e.g., command="restricted_command").

### 3.2.4. Hash passwords differently

In this case, it has hashed with md5. It can be used more secure encriptation like sha-256.