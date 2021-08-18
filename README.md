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
* Bruteforce login (done by autorecon.sh with hydra and medusa)
  * manual examples:<br/>
`hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 21 -o "/root/Desktop/tcp_21_ftp_hydra.txt" ftp://19.11.1.2`<br/><br/>
`medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 21 -O "/root/Desktop/tcp_21_ftp_medusa.txt" -M ftp -h 19.11.1.2`
* FTP vulnerable itself? searchsploit it.
## SSH (22/tcp)
* old version of ssh
* public-key with weak crypto
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
* high degree of manual inspection
* Burpsuite
* Be



