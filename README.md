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
* anonymous login (anonymous anonymous)

* FTP vulnerable itself? searchsploit it.
## SSH (22/tcp)
* old version of ssh
* public-key with weak crypto
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 21 -o "/root/Desktop/tcp_21_ftp_hydra.txt" ftp://19.11.1.2`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 21 -O "/root/Desktop/tcp_21_ftp_medusa.txt" -M ftp -h 19.11.1.2`
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
* Zone transer
```
dig axfr @$RHOST DOMAIN.COM<br/>
dnsrecon -d DOMAIN.COM
```
## RPC / NFS (111/tcp)

## S(a)MB(a) (139/tcp and 445/tcp)
* anonymous login
## SNMP (161/udp)
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
* SSLScan<br/>
example usage: `sslscan 10.11.1.2`

* Burpsuite
* Be



