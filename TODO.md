
What is a CRONNNNNNNNNNN?

**TODO: Investigate why firewalls not working**


To block an attack, you need to deny all traffic from the attacker's IP:

```
sudo ufw deny from <ATTACKER_IP>
```

- **Close suspicious ports**:
remove ip
```
sudo iptables -D INPUT -s 192.168.1.150 -j DROP
```

  ```bash
  sudo ufw deny <PORT>
  ```

Check active rules:
```
sudo ufw status verbose
```



To check if the firewall is blocking attacks, enable logging:

sudo ufw logging on

Then view the logs:

sudo cat /var/log/ufw.log | grep BLOCK

- **Uninstall vsftpd 2.3.4** and install a secure version.





. Block the Attacker's IP Completely


Example:

sudo ufw deny from 192.168.1.150

This will drop all incoming connections from that IP.
1. Block Specific Port for All Attackers

If you know the port being attacked (e.g., port 21 for vsftpd), use:

sudo ufw deny 21/tcp

For a UDP attack:

sudo ufw deny 53/udp

2. Enable Logging to Monitor Attacks


3. Use Rate Limiting to Prevent Brute-Force Attacks

If the attacker is repeatedly trying to connect (like SSH brute force), use:

sudo ufw limit ssh

This will block an IP after too many failed attempts.
4. Enable UFW and Reload

Ensure the firewall is enabled:

sudo ufw enable
sudo ufw reload








. Check for Persistent Backdoors

Attackers often create cron jobs, rootkits, or hidden services to keep access.
Check for Suspicious Cron Jobs

crontab -l
ls -la /etc/cron.*

If you find something suspicious, remove it:

crontab -r