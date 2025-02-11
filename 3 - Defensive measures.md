## 3.1. Monitor Logs for Suspicious Activity in the target machine

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
- PID: 4922 (Reverse shell)


Check FTP authentication logs for suspicious login attempts:
```bash
cat /var/log/auth.log
```

![[Pasted image 20250211221337.png]]


```bash
cat /var/log/vsftpd.log
```

If you see multiple failed attempts or unexpected logins, note the source IP.

![[Pasted image 20250211212437.png]]

---

### **3. Check Running Processes**
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

---

### **4. Use Tcpdump to Monitor Traffic**
If the attack is ongoing, use **tcpdump** to capture packets:

```bash
sudo tcpdump -i eth0 port 6200
```
This will show real-time FTP traffic, helping you identify the attacker.

![[Pasted image 20250211213232.png]]


## 3.2. Block attack

- **Kill malicious processes** (PID = 4922):  
  ```bash
  sudo kill -9 <PID>
  ```

On Kali Linux the exploit is stopped
![[Pasted image 20250211213942.png]]
- **Block the attacker's IP**:  
  ```bash
  sudo iptables -A INPUT -s <ATTACKER_IP> -j DROP
  ```

Kali cannot perform the attack
![[Pasted image 20250211214221.png]]





