# OSCP
* Use of autorecon.sh
# Active_Information_Gathering
Make a scan over all ports.<br/>
example:
`nmap -p- 10.11.1.2 10.11.1.3 10.11.1.4 10.11.1.5 -o /root/Desktop/all_ports.txt`
## autorecon.sh
example usage:
`autorecon --profile quick 10.11.1.2 10.11.1.3 10.11.1.4 10.11.1.5 --output /root/Desktop -vv`
# Enumeration
## FTP (21/tcp)
* anonymous login: `ftp 10.11.1.2 21` (anonymous anonymous)
* FTP vulnerable itself? searchsploit it.
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 21 -o "/root/Desktop/tcp_21_ftp_hydra.txt" ftp://10.11.1.2`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 21 -O "/root/Desktop/tcp_21_ftp_medusa.txt" -M ftp -h 10.11.1.2`
## SSH (22/tcp)
* old version of ssh
* public-key with weak crypto
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 22 -o "/root/Desktop/tcp_22_ssh_hydra.txt" ssh://10.11.1.2`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 22 -O "/root/Desktop/tcp_22_ssh_medusa.txt" -M ssh -h 10.11.1.2`
* Login with known username<br/>
`ssh username@10.11.1.2`<br/>
## SMTP (25/tcp)
* Enumartion of usernames
```
nc 10.11.1.217 25
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
dig axfr @10.11.1.2 DOMAIN.COM<br/>
dnsrecon -d DOMAIN.COM
```
## RPC / NFS (111/tcp)
* Enumerate (done by autorecon.sh with nmap, rpcclient)
  * manual example:<br/>
  `nmap -vv --reason -Pn -sV -p 111 --script=banner,msrpc-enum,rpc-grind,rpcinfo -oN /root/Desktop/tcp_111_rpc_nmap.txt 10.11.1.2`<br/><br/>
  `rpcclient -p 111 -U "" 10.11.1.2`
## MSRPC (135/tcp, 593/tcp)  
* Enumerate (done by autorecon.sh with nmap, rpcclient)
  * manual example:<br/>
  `nmap -vv --reason -Pn -sV -p 135 --script=banner,msrpc-enum,rpc-grind,rpcinfo -oN /root/Desktop/tcp_111_rpc_nmap.txt 10.11.1.2`<br/><br/>
  `rpcclient -p 111 -U "" 10.11.1.2`
## SMB (139/tcp and 445/tcp)
* anonymous login: `smbclient -N -L \\\\10.11.1.2 `
* List out files (if anonymous login is enabled): `smbmap -H 10.11.1.2 -R`<br/>
* Connect to share (if accessible): `smbclient //10.11.1.2/sharename$`<br/>
* Download all folders recursivly:<br/>
`mask ""`<br/>
`recurse ON`<br/>
`prompt OFF`<br/>
`lcd /root/Desktop/smb/`<br/>
`mget *`<br/>
* Enumerate (done by autorecon.sh with nmap)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms06-025" --script-args="unsafe=1" -oN "/root/Desktop/scans/tcp_139_smb_ms06-025.txt" 10.11.1.2`<br/><br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms07-029" --script-args="unsafe=1" -oN "/root/Desktop/tcp_139_smb_ms07-029.txt" 10.11.1.2`<br/><br/>
 `nmap -vv --reason -Pn -sV -p 139 --script="smb-vuln-ms08-067" --script-args="unsafe=1" -oN "/root/Desktop/tcp_139_smb_ms08-067.txt" 10.11.1.2`

## SNMP (161/udp)

## ldap (389/tcp,636/tcp,3268/tcp,3296/tcp)
* Enumerate (done by autorecon.sh with nmap and ldapsearch)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 389 "--script=banner,(ldap* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN /root/Desktop/tcp_389_ldap_nmap.txt 10.11.1.2`<br/><br/>
 `ldapsearch -x -D "<username>" -w "<password>" -p 389 -h 10.11.1.2 -b "dc=example,dc=com" -s sub "(objectclass=*) 2>&1 | tee > "/root/Desktop/scans/tcp_389_ldap_all-entries.txt"`
 
 ## ms-sql (1433/tcp)
* Enumerate (done by autorecon.sh with nmap and sqsh)
  * manual example:<br/>
 `nmap -vv --reason -Pn -sV -p 1433 "--script=banner,(ms-sql* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" --script-args=mssql.instance-port=1433,mssql.username=sa,mssql.password=sa -oN /root/Desktop/tcp_1433_mssql_nmap.txt 10.11.1.2`<br/><br/>
 `sqsh -U <username> -P <password> -S 10.11.1.2:1433`<br/><br/>
 
## HTTP(S) (80/tcp, 443/tcp, 8000/tcp, 8080/tcp, 8443/tcp, …)
* close enumeration (done by autorecon.sh with nikto, feroxbuster, dirsearch, dirb, gobuster, wpscan, hydra, medusa)
  * manual examples:<br/>
 `nikto -ask=no -h http://10.11.1.2:8080 2>&1 | tee "/root/Desktop/tcp_8080_http_nikto.txt"`<br/><br/>
 `feroxbuster -u http://10.11.1.2:8080 -t 10 -w /usr/share/seclists/Discovery/Web-Content/big.txt -x "txt,html,php,asp,aspx,jsp" -v -k -n -o /root/Desktop/tcp_8080_http_feroxbuster_big.txt`<br/><br/>
 `dirsearch -u http://10.11.1.2:8080/ -t 16 -r -e txt,html,php,asp,aspx,jsp -f -w /usr/share/seclists/Discovery/Web-Content/big.txt -o /root/Desktop/tcp_8080_http_dirsearch_big.txt`<br/><br/>
 `dirb http://10.11.1.2:8080/ /usr/share/seclists/Discovery/Web-Content/big.txt -l -r -S -X ",.txt,.html,.php,.asp,.aspx,.jsp" -o "/root/Desktop/tcp_8080_http_dirb_big.txt"`<br/><br/>
 `gobuster dir -u http://10.11.1.2:8080/ -w /usr/share/seclists/Discovery/Web-Content/big.txt -e -k -s "200,204,301,302,307,403,500" -x "txt,html,php,asp,aspx,jsp" -z -o "/root/Desktop/tcp_8080_http_gobuster_big.txt"`<br/><br/>
 `wpscan --url http://10.11.1.2:8080/ --no-update -e vp,vt,tt,cb,dbe,u,m --plugins-detection aggressive --plugins-version-detection aggressive -f cli-no-color 2>&1 | tee "/root/Desktop/tcp_8080_http_wpscan.txt"`<br/><br/>
 `hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 8080 -o "/root/Desktop/tcp_8080_http_auth_hydra.txt" http-get://10.11.1.2/path/to/auth/area` <br/><br/>
 `hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 8080 -o "/root/Desktop/scans/tcp_8080_http_form_hydra.txt" http-post-form://10.11.1.2/path/to/login.php:username=^USER^&password=^PASS^:invalid-login-message`<br/><br/>
  `medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 8080 -O "/root/Desktop/tcp_8080_http_auth_medusa.txt" -M http -h 10.11.1.2 -m DIR:/path/to/auth/area`<br/><br/>
  `medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 8080 -O "/root/Desktop/tcp_8080_http_form_medusa.txt" -M web-form -h 10.11.1.2 -m FORM:/path/to/login.php -m FORM-DATA:"post?username=&password=" -m DENY-SIGNAL:"invalid login message"`
* high degree of manual inspection
  * Check for error pages  
  * Try some easy logins
  * Search web for standard logins
  * Try or directory traversal
* SSLScan<br/>
example usage: `sslscan 10.11.1.2`
* Burpsuite<br/>
  * Basic Authentication<br/>
  1. Intercept Request<br/>
  2. Send Request to Intruder<br/>
  3. Payload Positions -> Add two positions<br/>
  4. Payloads -> Payload type: Custom iterator ¦ Separator for position 1: ':' ¦ Payload Processing: Base64-encode ¦ Payload Encoding: Delete '='
# Privilege_Escalation_Linux
Spawn a better shell:
`python -c 'import pty; pty.spawn("/bin/bash")`




