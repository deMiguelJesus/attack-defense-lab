Disable unused services:
```
sudo systemctl disable vsftpd
```

Apply firewall rules (UFW on Ubuntu)
```
sudo ufw enable
sudo ufw deny 21/tcp
```

Install fail2ban (to block brute-force attacks)

```
sudo apt install fail2ban
```