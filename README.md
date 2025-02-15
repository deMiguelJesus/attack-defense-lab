# Attack-Defense Lab

## Project Objectives

1. Set up the lab: Create two virtual machines: (1) **Kali Linux VM** (Attacker machine) – Used for penetration testing; and (2) **Metasploitable 2 VM** (Target machine) – A deliberately vulnerable Linux system.
2. Attack the target VM (Ethical Hacking Test)
	1. Find vulnerabilities
	2. Exploit VSFTPD vulnerability
	3. Post-exploitation: SSH Key persistence
	4. Post-exploitation: Password and user details cracking
3. Secure the target VM (Analysis and defensive measures)
	1. Open ports
	2. Exploits used
	3. Security patches applied

## Considerations

Important security measures to take into account before running this project:
- Run in an isolated network (no internet).
- Disable auto-bridging to your host.
- Ensure Metasploitable 2 is never exposed to external networks.



**Tools**: KVM, libvirt, QEMU, Kali Linux, Metasploitable 2, Metasploit, msfvenom, nmap, netstat, tcpdump, John The Ripper.

Concepts: 