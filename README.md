Metasploitable2 — Offensive Security Lab Notes
📖 Overview

This repository documents a three-day hands-on lab with Metasploitable2 — a deliberately vulnerable Linux VM used for practicing penetration testing skills.
The exercises followed a structured penetration-testing workflow:

Planning → Reconnaissance → Enumeration → Exploitation → Post-Exploitation → Documentation & Cleanup

⚠️ Disclaimer: This project was performed entirely in an isolated VirtualBox lab environment. Do not attempt these techniques on systems you do not own or have explicit permission to test.

🎯 Lab Goals

Practice reconnaissance & service fingerprinting with Nmap.

Learn service enumeration (FTP, SSH, Telnet, HTTP, SMB, PostgreSQL, etc.).

Exploit known, intentional vulnerabilities with Metasploit and manual tools.

Perform post-exploitation checks (whoami, id, uname, sudo rights, SUID files).

Reinforce safe lab hygiene: snapshots, bridged/host-only networking, revert after exploitation.

Focus deeply on FTP, SSH, Telnet, and HTTP to understand real-world risks.

🗂️ Lab Progress
🔹 Day 1 — Lab Setup & First Exploitation

Imported Metasploitable2 into VirtualBox, configured host-only networking for isolation.

Verified default credentials (msfadmin/msfadmin).

Performed initial Nmap scan to identify open services.

First exploitation: vsftpd 2.3.4 backdoor (FTP) → gained a shell.

🔹 Day 2 — FTP & IRC Exploitation

Organized lab structure (scans/, exploits/, results/).

Re-scanned services for detailed version info.

FTP (vsftpd): Practiced enumeration (including anonymous login), then re-exploited vsftpd backdoor.

UnrealIRCd (IRC): Exploited backdoored version 3.2.8.1 → obtained both reverse and bind shells.

Learned differences between reverse vs bind payloads.

Practiced post-exploitation: system info, process listing, privilege escalation checks.

🔹 Day 3 — Complete Service Exploitation (Focus: FTP, SSH, Telnet, HTTP)

Completed exploitation of remaining high-value services.

Extra focus given to FTP, SSH, Telnet, and HTTP (Apache/PHP):

FTP (21)

Tested anonymous login → successful.

Exploited vsftpd 2.3.4 backdoor → gained remote shell.

SSH (22)

Banner confirmed OpenSSH running.

Exploited using weak default credentials (msfadmin/msfadmin).

Telnet (23)

Logged in with default creds.

Verified that credentials travel in plaintext, captured via Wireshark.

HTTP (80 — Apache 2.2.8 + PHP)

Enumerated with curl and Gobuster → discovered /phpinfo.php and exposed credentials.

Exploited PHP-CGI argument injection via Metasploit → gained Meterpreter session.

Other services covered: Tomcat (manager deploy), distcc RCE, Samba/SMB enum + usermap_script, VNC (weak password), PostgreSQL (weak creds).

🛠️ Key Tools & Commands

Nmap: sudo nmap -sS -sV -O -Pn <TARGET>

FTP: ftp <TARGET> (tested anonymous login)

SSH: sshpass -p 'msfadmin' ssh msfadmin@<TARGET>

Telnet: telnet <TARGET> (captured plaintext creds in Wireshark)

HTTP: curl -I http://<TARGET>/, gobuster dir -u http://<TARGET>/

Metasploit: Used relevant modules for vsftpd, UnrealIRCd, Tomcat, PHP CGI, etc.

📚 Skills & Takeaways

Built a repeatable penetration-testing workflow.

Gained hands-on practice with service enumeration & exploitation.

Understood risks of:

Anonymous FTP access.

Weak/default SSH credentials.

Telnet plaintext transmission.

Web app misconfigurations (Apache/PHP).

Learned systematic post-exploitation: checking users, processes, network, SUIDs, configs.

Strengthened documentation discipline (all scans, exploits, results stored).

Practiced safe lab hygiene: snapshots, revert after exploitation, bridged/host-only networks.

✅ Conclusion

Over three days, this lab provided end-to-end exposure to penetration testing in a safe environment.
I now have practical experience in:

Reconnaissance & service fingerprinting.

Exploiting multiple real, known vulnerabilities.

Post-exploitation enumeration and evidence collection.

Documenting findings in a professional format suitable for GitHub and presentations.
