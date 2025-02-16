# Attack-Defense Lab

## Project Objectives

1. **Set up the lab** - Create two virtual machines: 
	1. Kali Linux VM (Attacker machine) – Used for penetration testing.
	2. Metasploitable 2 (Target machine) – A deliberately vulnerable Linux system.
2. **Attack the target VM** - Ethical Hacking Test
	1. Find vulnerabilities in the target system
	2. Exploit vulnerability
	3. Post-exploitation: SSH key persistence
	4. Post-exploitation: Password and user details cracking
3. **Secure the target VM** - Analysis and defensive measures
	1. Analyze open ports, logs and exploits used.
	2. Security patches applied.


## Dependencies

| Name                 | Version    |
| -------------------- | ---------- |
| QEMU-KVM             | 6.2.0      |
| libvirt              | 8.0.0      |
| Kali Linux           | 2024.4     |
| Metasploitable 2     | 2.0        |
| Metasploit-Framework | 6.4.34-dev |
| nmap                 | 7.94       |
| netstat              | 1.42       |
| tcpdump              | 3.9.8      |
| John The Ripper      | 1.9.0      |
