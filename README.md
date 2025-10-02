Metasploitable2 — Lab Notebook & Learning Outcomes

Broad topic: Practical offensive security lab using Metasploitable2 — a deliberately vulnerable VM used to learn recon, service enumeration, exploitation, and safe post‑exploit analysis.

Overview

This repository documents a hands‑on three‑day lab exploring Metasploitable2. The goal was not to produce exploits for real systems, but to practice the full penetration‑testing workflow in a safe, isolated environment: planning → reconnaissance → enumeration → exploitation → post‑exploit enumeration → documentation and cleanup.

Lab goals / learning aims

Build a repeatable, professional lab workflow for offensive security practice.

Learn and practice recon & service fingerprinting (Nmap outputs saved as .nmap / .xml / .gnmap).

Gain hands‑on experience exploiting real, known vulnerabilities in a controlled environment (Metasploit + PoC).

Practice post‑exploit enumeration and evidence collection (whoami, id, uname, processes, network, SUID files, sudo rights).

Understand different payload styles (reverse, bind, single‑command) and when to use each.

Develop safe lab hygiene (host‑only networking, snapshots, sanitization before committing logs).

What I did (high level)

Day 1 — Setup & initial recon

Downloaded/imported Metasploitable2 to VirtualBox, configured host‑only networking and snapshots.

Performed initial Nmap reconnaissance and identified core attack surface (FTP, SSH, Telnet, HTTP/Tomcat, SMB, VNC, IRC, distcc, PostgreSQL).

Conducted first practical exploit: vsftpd 2.3.4 backdoor via Metasploit and obtained a shell.

Day 2 — Targeted exploitation

Built structured lab folders for scans/, exploits/, results/, screenshots/, notes/.

Re‑ran baseline scans and exploited vsftpd again for practice.

Exploited UnrealIRCd backdoor — tested and obtained both reverse and bind shells.

Practiced consistent logging (msfconsole spools) and core post‑exploit enumeration.

Day 3 — Full service coverage

Completed exploitation and documentation for remaining high‑value services:

Tomcat (manager WAR deploy) — meterpreter via tomcat_mgr_deploy.

distcc — RCE PoC.

SSH / Telnet / PostgreSQL / VNC — accessed via default/weak credentials.

Samba/SMB — enumeration (enum4linux, smbclient, smbmap) and module use (usermap_script where applicable).

Saved all scans, msf spools, post‑exploit outputs and screenshots (screenshots tracked or handled with Git LFS).

Key commands / patterns used (examples)

Baseline scan:

sudo nmap -sS -sV -O -Pn <TARGET> -oA metasploitable_initial


Metasploit session logging:

msfconsole
spool ~/labs/metasploitable/exploits/<service>-session.txt
use <exploit/module>
set RHOSTS <TARGET>
set PAYLOAD <payload>
exploit
spool off


Post‑exploit enumeration (save outputs to results/):

whoami; id; uname -a; ps aux | head -n 30; ss -tulpen || netstat -tulpen; sudo -l
find / -perm -4000 -type f | head -n 40

Skills & takeaways

Repeatable, auditable pen‑test workflow and good lab discipline.

Practical experience with Metasploit modules and manual PoCs.

Clear understanding of payload types (reverse vs bind) and practical trade‑offs.

Stronger Linux post‑exploit enumeration skills and artifact collection practices.

Awareness of safe sharing practices: never commit unredacted credentials or private keys.

Safety & ethics

Only run these exercises on systems you own or have explicit permission to test.

Always isolate the lab (VirtualBox Host‑only or internal network).

Take snapshots before risky actions and revert after testing.
