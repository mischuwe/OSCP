# OSCP
All

# Active_Information_Gathering
## autorecon.sh
usage:
autorecon host<br/>
example usage:
`autorecon 10.11.1.209 --output /root/Desktop -vv`
# Enumeration

## FTP (21/tcp)
* anonymous login
## SSH (22/tcp)
* public-key with weak crypto
## SMTP (25/tcp)
* Enumartion of usernames<br/> 
`nc 10.11.1.217 25
[...]
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table`
## DNS (53/tcp)
* Extract information
* Zone transer
`dig axfr @$RHOST DOMAIN.COM
dnsrecon -d DOMAIN.COM`
## RPC / NFS (111/tcp)
## S(a)MB(a) (139/tcp and 445/tcp)
## SNMP (161/udp)
## HTTP(S) (80/tcp, 443/tcp, 8000/tcp, 8080/tcp, 8443/tcp, …)


