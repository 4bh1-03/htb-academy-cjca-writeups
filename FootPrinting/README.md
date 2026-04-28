# Footprinting

Created by: **4bh1-03**

This write-up documents my completion of the **`Footprinting`** module under the **`Junior Cybersecurity Analyst`** job role path on **`Hack The Box`**. This module serves as a deep dive into the initial phase of the reconnaissance process, focusing on the systematic discovery and enumeration of network services such as `FTP`, `SMB`, `DNS`, `SNMP`, and others to map out a target's attack surface.

![Footprinting.jpeg](images/Footprinting.jpeg)

In this write-up, I provide a structured walkthrough of the active and passive reconnaissance techniques covered throughout the module. It includes the practical application of specialized tools to interact with various protocols, revealing how misconfigurations or default settings can leak sensitive information. Furthermore, I have included step-by-step solutions to the lab exercises and answers to the module questions, illustrating how to transition from initial discovery to actionable data extraction.

---

# Section 6 : FTP

To solve the questions of this section, `ftp` to the given target to recieve the banner which contains the version of FTP. Once you have entered the `ftp` session, recursively list all the files on the server to find the `flag.txt` file. Once you have found it, download it on your local machine (Pwnbox instance) to view the contents. All of this is done below:

![Screenshot 2026-03-04 223115.png](images/Screenshot_2026-03-04_223115.png)

### 1. **Which version of the FTP server is running on the target system? Submit the entire banner as the answer.**

**Answer :** `InFreight FTP v1.1` 

### 2. **Enumerate the FTP server and find the flag.txt file. Submit the contents of it as the answer.**

**Answer :** `HTB{b7skjr4c76zhsds7fzhd4k3ujg7nhdjre}`

---

# Section 7 : SMB

### 1. **What version of the SMB server is running on the target system? Submit the entire banner as the answer.**

Just run an `nmap` scan on the target’s `445` and `139` port as shown below:

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-1uiq1cst3z-htb-cloud-com]─[~]
└──╼ [★]$ nmap 10.129.33.78 -p 445,139 -sCV -Pn -n
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-04 13:36 CST
Nmap scan report for 10.129.33.78
Host is up (0.22s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Host script results:
| smb2-time: 
|   date: 2026-03-04T19:37:06
|_  start_date: N/A
|_nbstat: NetBIOS name: DEVSMB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.06 seconds
```

**Answer :** `Samba smbd 4.6.2` 

### 2. **What is the name of the accessible share on the target?**

We will use `smbmap` with the host argument (`-H`) to list the shares on the target.

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-1uiq1cst3z-htb-cloud-com]─[~]
└──╼ [★]$ smbmap -H 10.129.33.78
[+] IP: 10.129.33.78:445	Name: 10.129.33.78                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	sambashare                                        	READ ONLY	InFreight SMB v3.1
	IPC$                                              	NO ACCESS	IPC Service (InlaneFreight SMB server (Samba, Ubuntu))

```

**Answer :** `sambashare`

### 3. **Connect to the discovered share and find the flag.txt file. Submit the contents as the answer.**

We can connnect to the discovered share (`sambashare`) by using the following command:

```bash
smbclient //<target_IP>/sambashare
```

![image.png](images/image.png)

Now let’s look for the `flag.txt` file on the share.

![image.png](images/image%201.png)

The contents folder seems interesting. Let’s check it out.

![image.png](images/image%202.png)

There we have the `flag.txt` file. We download the file and view it by typing `!cat` . smbclient provides a feature of executing local system commands using an exclamation mark at the beginning (`!<cmd>`) without interrupting the connection.

![image.png](images/image%203.png)

**Answer :** `HTB{o873nz4xdo873n4zo873zn4fksuhldsf}`

### 4. **Find out which domain the server belongs to.**

We can use `rpcclient` which offers us many different requests with which we can execute specific functions on the SMB server to get information. First we have to connect to the target as a `NULL` session.

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-1uiq1cst3z-htb-cloud-com]─[~]
└──╼ [★]$ rpcclient -U '' 10.129.33.78
Password for [WORKGROUP\]:
rpcclient $>
```

There a list of requests that we can make, but to get info about the domain we will use the `querydominfo` .

```bash
rpcclient $> querydominfo
Domain:		DEVOPS
Server:		DEVSMB
Comment:	InlaneFreight SMB server (Samba, Ubuntu)
Total Users:	0
Total Groups:	0
Total Aliases:	0
Sequence No:	1772654266
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
```

**Answer :** `DEVOPS` 

### 5. **Find additional information about the specific share we found previously and submit the customized version of that specific share as the answer.**

Staying in the `rpcclient` session, we will run the `netshareenumall` command to gain info about all the shares present on the domain.

```bash
rpcclient $> netshareenumall
netname: print$
	remark:	Printer Drivers
	path:	C:\var\lib\samba\printers
	password:	
netname: sambashare
	remark:	InFreight SMB v3.1
	path:	C:\home\sambauser\
	password:	
netname: IPC$
	remark:	IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
	path:	C:\tmp
	password:	
```

**Answer :** `InFreight SMB v3.1`

### 6. **What is the full system path of that specific share? (format: "/directory/names")**

From the above output we get the full path as: `C:\home\sambauser\` . But Linux-based operating systems do not have a "C:\" drive. Therefore we have to 

- Remove the `C:\` drive letter (not used in Linux).
- Replace backslashes `\` with forward slashes `/`.

**Answer :** `/home/sambauser`

---

# Section 8 : NFS

First let’s list all the shares that are being exported (`-e`) on the server using `showmount` .

![Screenshot 2026-03-05 034309.png](images/Screenshot_2026-03-05_034309.png)

We will create two separate folders on our local machine (Pwnbox instance) to mount the shares and access the files.

![Screenshot 2026-03-05 034320.png](images/Screenshot_2026-03-05_034320.png)

We can mount both the shares using the below commands:

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-gukonaczqk-htb-cloud-com]─[~]
└──╼ [★]$ sudo mount -t nfs 10.129.202.5:/var/nfs ./nfs-target/ -o nolock
```

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-gukonaczqk-htb-cloud-com]─[~]
└──╼ [★]$ sudo mount -t nfs 10.129.202.5:/mnt/nfsshare ./nfsshare-target/ -o nolock
```

**Options explained:**

| Part | Meaning |
| --- | --- |
| `sudo` | Mounting usually requires root privileges |
| `mount` | Linux command used to attach filesystems |
| `-t nfs` | Specifies the filesystem **type = NFS** |
| `10.129.202.5:/` | Remote NFS server and path |
| `./nfs-target/` | Local directory where the share will be mounted |
| `-o nolock` | Mount option disabling file locking |

### 1. **Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer.**

![Screenshot 2026-03-05 034414.png](images/Screenshot_2026-03-05_034414.png)

**Answer :** `HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze`}

### 2. **Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer.**

![Screenshot 2026-03-05 034423.png](images/Screenshot_2026-03-05_034423.png)

**Answer :** `HTB{8o7435zhtuih7fztdrzuhdhkfjcn7ghi4357ndcthzuc7rtfghu34}`

---

# Section 9 : DNS

### 1. **Interact with the target DNS using its IP address and enumerate the FQDN of it for the "inlanefreight.htb" domain.**

```bash
dig @<target_IP> inlanefreight.htb NS
```

Command Explained:

- **`dig`**: The command name (Domain Information Groper).
- **`@<target_IP>`**: Tells `dig` which server to talk to. Replace `<target_IP>` with the IP of the DNS server (e.g., `10.129.34.194`).
- **`inlanefreight.htb`**: The domain name you are curious about.
- **`NS`**: The **Record Type**. You are specifically asking for "Name Server" records.

![image.png](images/image%204.png)

**Answer :** `ns.inlanefreight.htb`

### 2. **Identify if its possible to perform a zone transfer and submit the TXT record as the answer. (Format: HTB{...})**

Let’s try transfering the `inlanefreight.htb` zone and see the `TXT` records we get:

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-fkgga3aua8-htb-cloud-com]─[~]
└──╼ [★]$ dig axfr @10.129.34.252 inlanefreight.htb

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @10.129.34.252 inlanefreight.htb
; (1 server found)
;; global options: +cmd
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.	604800	IN	TXT	"MS=ms97310371"
inlanefreight.htb.	604800	IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.	604800	IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
app.inlanefreight.htb.	604800	IN	A	10.129.18.15
dev.inlanefreight.htb.	604800	IN	A	10.12.0.1
internal.inlanefreight.htb. 604800 IN	A	10.129.1.6
mail1.inlanefreight.htb. 604800	IN	A	10.129.18.201
ns.inlanefreight.htb.	604800	IN	A	127.0.0.1
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 196 msec
;; SERVER: 10.129.34.252#53(10.129.34.252) (TCP)
;; WHEN: Thu Mar 05 12:51:31 CST 2026
;; XFR size: 11 records (messages 1, bytes 560)
```

We didn’t get the `TXT` record in the mentioned format. Let’s try transfering the other zones. The zones `app.inlanefreight.htb` and `dev.inlanefreight.htb` showed transfer failed. So we are left with `internal.inlanefreight.htb` .

```bash
┌─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-fkgga3aua8-htb-cloud-com]─[~]
└──╼ [★]$ dig axfr @10.129.34.252 internal.inlanefreight.htb

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @10.129.34.252 internal.inlanefreight.htb
; (1 server found)
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN	TXT	"MS=ms97310371"
internal.inlanefreight.htb. 604800 IN	TXT	"HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}"
internal.inlanefreight.htb. 604800 IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN	NS	ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb.	604800 IN A	10.129.34.16
dc2.internal.inlanefreight.htb.	604800 IN A	10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A	10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A	127.0.0.1
vpn.internal.inlanefreight.htb.	604800 IN A	10.129.1.6
ws1.internal.inlanefreight.htb.	604800 IN A	10.129.1.34
ws2.internal.inlanefreight.htb.	604800 IN A	10.129.1.35
wsus.internal.inlanefreight.htb. 604800	IN A	10.129.18.2
internal.inlanefreight.htb. 604800 IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 193 msec
;; SERVER: 10.129.34.252#53(10.129.34.252) (TCP)
;; WHEN: Thu Mar 05 12:52:04 CST 2026
;; XFR size: 15 records (messages 1, bytes 677)
```

**Answer :** `HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}`

### 3. **What is the IPv4 address of the hostname DC1?**

From the above output, it is clear that we have a host named `dc1` with the IPv4 address `10.129.34.16` .

**Answer :** `10.129.34.16`

### 4. **What is the FQDN of the host where the last octet ends with "x.x.x.203"?**

```bash
─[eu-academy-4]─[10.10.14.96]─[htb-ac-2120529@htb-fkgga3aua8-htb-cloud-com]─[~]
└──╼ [★]$ dnsenum --dnsserver 10.129.34.252 --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/wordlists/seclists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb
dnsenum VERSION:1.2.6

-----   dev.inlanefreight.htb   -----

Host's addresses:
__________________

Name Servers:
______________

ns.inlanefreight.htb.                    604800   IN    A         127.0.0.1

Mail (MX) Servers:
___________________

Trying Zone Transfers and getting Bind Versions:
_________________________________________________

unresolvable name: ns.inlanefreight.htb at /usr/bin/dnsenum line 900 thread 2.

Trying Zone Transfer for dev.inlanefreight.htb on ns.inlanefreight.htb ... 
AXFR record query failed: no nameservers

Brute forcing with /usr/share/wordlists/seclists/Discovery/DNS/fierce-hostlist.txt:
____________________________________________________________________________________

dev1.dev.inlanefreight.htb.              604800   IN    A         10.12.3.6
ns.dev.inlanefreight.htb.                604800   IN    A         127.0.0.1
win2k.dev.inlanefreight.htb.             604800   IN    A        10.12.3.203

Launching Whois Queries:
_________________________

dev.inlanefreight.htb_____________________

Performing reverse lookup on 0 ip addresses:
_____________________________________________

0 results out of 0 IP addresses.

dev.inlanefreight.htb ip blocks:
_________________________________

done.

```

Command breakdown

| **Part** | **What it does** |
| --- | --- |
| **`dnsenum`** | The name of the program. It stands for "DNS Enumerator." |
| **`--dnsserver 10.129.34.194`** | Forces the tool to ask this specific IP address for answers, rather than using your default internet settings. |
| **`--enum`** | This is a "shortcut" flag. It tells the tool to perform a full enumeration, including checking for zone transfers and Google scraping. |
| **`-p 0`** | Sets the number of pages to scrape from Google to zero (effectively disabling the search engine scraping part). |
| **`-s 0`** | Sets the number of subdomains to scrape from Google to zero. |
| **`-o subdomains.txt`** | **Output:** Saves the final results into a text file named `subdomains.txt`. |
| **`-f [path/to/list]`** | **Brute Force:** This points to a "wordlist." The tool will try every single word in that file (e.g., `dev`, `admin`, `mail`) to see if `dev.inlanefreight.htb` actually exists. |
| **`dev.inlanefreight.htb`** | The target sub domain you are investigating. |

**Answer :** `win2k.dev.inlanefreight.htb`

---

# Section 11 : IMAP/POP3

### **1. Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.**

First let’s run an `nmap` scan on the ports `110`, `143`, `993` and `995` on the target.

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-xzzo1fhuqk-htb-cloud-com]─[~]
└──╼ [★]$ nmap 10.129.2.141 -p110,143,993,995 -sCV
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-07 01:32 CST
Nmap scan report for 10.129.2.141
Host is up (0.21s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA SASL AUTH-RESP-CODE STLS PIPELINING UIDL RESP-CODES TOP
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more OK ENABLE LITERAL+ ID have post-login listed IDLE capabilities IMAP4rev1 LOGIN-REFERRALS LOGINDISABLEDA0001 Pre-login SASL-IR STARTTLS
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd
|_imap-capabilities: OK ENABLE LITERAL+ IMAP4rev1 more have post-login IDLE listed capabilities AUTH=PLAINA0001 ID Pre-login SASL-IR LOGIN-REFERRALS
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
995/tcp open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
|_pop3-capabilities: CAPA SASL(PLAIN) AUTH-RESP-CODE PIPELINING USER UIDL RESP-CODES TOP
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.82 seconds
```

**Answer :** `InlaneFreight Ltd`

### 2. **What is the FQDN that the IMAP and POP3 servers are assigned to?**

From the above `nmap` scan output, it is clearly visible from the `ssl-cert` that the IMAP and POP3 servers are assigned to `dev.inlanefreight.htb` .

**Answer :** `dev.inlanefreight.htb`

### 3. **Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...})**

To interact with the IMAP service, we will use `openssl` .

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-xzzo1fhuqk-htb-cloud-com]─[~]
└──╼ [★]$ openssl s_client -connect 10.129.2.141:imaps
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
verify error:num=18:self-signed certificate
verify return:1
depth=0 C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
verify return:1
---
Certificate chain
 0 s:C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
   i:C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov  8 23:10:05 2021 GMT; NotAfter: Aug 23 23:10:05 2295 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEUzCCAzugAwIBAgIUDf35PqFuv6Uv0EECM8dFmNSZoY8wDQYJKoZIhvcNAQEL
BQAwgbcxCzAJBgNVBAYTAlVLMQ8wDQYDVQQIDAZMb25kb24xDzANBgNVBAcMBkxv
bmRvbjEaMBgGA1UECgwRSW5sYW5lRnJlaWdodCBMdGQxHDAaBgNVBAsME0Rldk9w
-SNIP-

-SNIP-
8Oh7vSfzvqvHLE7HHAO0G5Q81zo+wWsrQF0s40HEF/raEMfOy2Htm79YjyjAlLWf
ueS+u8rX2smOYdRIpL3UPx7+yZPGu47vYoetde1Z5cfTCgmeS05BQ2qMOp6Tw6+G
xUuqg8nK1Q==
-----END CERTIFICATE-----
subject=C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
issuer=C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1667 bytes and written 377 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 2048 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 3687A009B8627EF257DCD3F52B5670B75DB551DD8865D2FD30C444D1BCB0B3D0
    Session-ID-ctx: 
    Resumption PSK: 09222E27A33CA9C2FAABC24D57C9283EB2E10F7A2B678BDB835AAC7CF7A4BD2AC13179CD4A643BB6029122EF56A160B4
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 8f fe 27 10 ff 3f 91 b3-73 0e 7b 20 39 e7 a3 ed   ..'..?..s.{ 9...
    0010 - b5 0b e9 29 ad 56 29 49-bf 14 eb 01 06 f3 0b a7   ...).V)I........
	  -SNIP-
	  
	  -SNIP-
    00a0 - 36 c9 c1 f5 8e 17 81 85-c3 30 17 e6 3b f3 a7 70   6........0..;..p
    00b0 - 00 9f aa 8e 64 16 38 86-21 b3 57 c5 ee fa ff 75   ....d.8.!.W....u

    Start Time: 1772869058
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: AEEDBF670B2324983BE74AF9CBAB25FD6D9DBB347B6F6873CA57EEC73E3038FE
    Session-ID-ctx: 
    Resumption PSK: E4A81378B18E11D9A73F3AB43AFBDCD0DC752FB54E1ADD3C4086BBDED9CF414F63E2A75708D7C443EC5E48968108D8CF
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 8f fe 27 10 ff 3f 91 b3-73 0e 7b 20 39 e7 a3 ed   ..'..?..s.{ 9...
    0010 - 85 a8 e2 d7 ce 7a 44 8e-cc 1a ba 1c c9 8b e9 fa   .....zD.........
    -SNIP-
    
    -SNIP-
    00a0 - d8 e0 9c bf 31 51 cf 8c-28 38 66 8a 24 16 f0 6e   ....1Q..(8f.$..n
    00b0 - aa 96 1c ea fd f5 d0 fe-7e 0d 87 3e da 15 55 1e   ........~..>..U.

    Start Time: 1772869058
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] 
HTB{roncfbw7iszerd7shni7jr2343zhrj}

```

> **`OpenSSL`** is a robust, commercial-grade, and full-featured toolkit for the Transport Layer Security (TLS) and Secure Sockets Layer (SSL) protocols. It is also a general-purpose cryptography library. In the context of security auditing, it is used to manually negotiate handshakes, verify certificates, and test for protocol vulnerabilities.
> 

**The Three Main Pillars**

OpenSSL isn't just one thing; it’s a toolkit that serves three primary roles:

1. **The Protocol (SSL/TLS):** It implements the rules for how computers "shake hands" and agree on a secret code before sending data.
2. **The Library (Cryptography):** It contains the actual math (algorithms) used to lock and unlock data, such as **AES**, **RSA**, and **SHA-256**.
3. **The Command Line Tool:** The command which we have been using (`openssl s_client`). It’s a way for humans to manually talk to servers, create security certificates, or encrypt files.

```bash
openssl s_client -connect <target_IP>:imaps
```

The Command Breakdown

- **`openssl s_client`**: Again, this invokes the OpenSSL client tool to initiate a secure handshake.
- **`connect <IP>:imaps`**: This tells OpenSSL to connect to the specified IP address using the shorthand for the IMAPS port.
    - **Note:** `imaps` usually maps to **Port 993**. If your system doesn't recognize the name, you would use `<IP>:993`.

**Answer :** `HTB{roncfbw7iszerd7shni7jr2343zhrj}`

### 4. **What is the customized version of the POP3 server?**

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-c2jvqqm5vv-htb-cloud-com]─[~]
└──╼ [★]$ openssl s_client -connect 10.129.42.195:pop3s
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
verify error:num=18:self-signed certificate
verify return:1
depth=0 C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
verify return:1
---
Certificate chain
 0 s:C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
   i:C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov  8 23:10:05 2021 GMT; NotAfter: Aug 23 23:10:05 2295 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEUzCCAzugAwIBAgIUDf35PqFuv6Uv0EECM8dFmNSZoY8wDQYJKoZIhvcNAQEL
BQAwgbcxCzAJBgNVBAYTAlVLMQ8wDQYDVQQIDAZMb25kb24xDzANBgNVBAcMBkxv
bmRvbjEaMBgGA1UECgwRSW5sYW5lRnJlaWdodCBMdGQxHDAaBgNVBAsME0Rldk9w
-SNIP-

-SNIP-
8Oh7vSfzvqvHLE7HHAO0G5Q81zo+wWsrQF0s40HEF/raEMfOy2Htm79YjyjAlLWf
ueS+u8rX2smOYdRIpL3UPx7+yZPGu47vYoetde1Z5cfTCgmeS05BQ2qMOp6Tw6+G
xUuqg8nK1Q==
-----END CERTIFICATE-----
subject=C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
issuer=C = UK, ST = London, L = London, O = InlaneFreight Ltd, OU = DevOps Dep\C3\83artment, CN = dev.inlanefreight.htb, emailAddress = cto.dev@dev.inlanefreight.htb
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1667 bytes and written 377 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 2048 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 7ABAF17E3FAB664C7EF13B48EF701E0CF7307652F85A5A0B2001359FD387B81B
    Session-ID-ctx: 
    Resumption PSK: 39720DC98E5D0637E980933D1594BC63E7679EFD8F5ECF8B47D15EC14C0305F2F3F46BA815423C8AFEC09A8A6D86C53B
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 5d 0b 83 ac b6 16 47 d3-08 44 63 dd 3c 4e bc 76   ].....G..Dc.<N.v
    0010 - fe 9f 23 7d c0 eb 14 76-3c 28 62 01 3d 65 c0 59   ..#}...v<(b.=e.Y
		-SNIP-
		
		-SNIP-
    00a0 - 0d 6c 61 3e 09 f3 78 8a-e0 06 14 99 1d f1 4b e0   .la>..x.......K.
    00b0 - 22 05 fd 12 1e a7 e7 c7-31 c9 58 a9 c2 22 b6 be   ".......1.X.."..

    Start Time: 1772904919
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 3C91A4A18B784882A6B1E62F7F7FA94AE6958C367300D7DB8F1EBDFB5C4029BC
    Session-ID-ctx: 
    Resumption PSK: D00B8D651960E769F1D51C78B57A87F693CE6BB286ECF315C6888759CE342B4BBFBE9B52F7D26589B372179FC92E4DD0
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 5d 0b 83 ac b6 16 47 d3-08 44 63 dd 3c 4e bc 76   ].....G..Dc.<N.v
    0010 - 7e 6e 44 a9 c3 fd f0 ce-48 54 b7 12 cd 74 e4 8c   ~nD.....HT...t..
    0020 - fb 65 56 78 5e 15 39 26-a1 bd 48 7c a4 8a b1 d0   .eVx^.9&..H|....
    0030 - 05 ea 69 3d 53 de 17 93-b7 ba a5 90 e5 8f 46 ce   ..i=S.........F.
    0040 - 3a 7c be b7 09 0c df 96-04 e9 77 de a0 0f 0e 70   :|........w....p
    0050 - 23 d3 b3 91 af cb 7a 87-e7 8a 87 05 cb 62 a9 3c   #.....z......b.<
    0060 - 51 21 60 1f 55 fb cd 13-d9 76 d4 da e9 d4 a4 37   Q!`.U....v.....7
    0070 - 34 11 8c 5f ee 1f 8e 43-49 4e c1 e8 27 92 e3 b2   4.._...CIN..'...
    0080 - 2f 81 7a ca ce 46 c8 0c-c0 33 7f 13 e3 c6 0b 76   /.z..F...3.....v
    0090 - da c9 2c 2c b7 48 50 7b-c6 e8 a6 f1 38 35 b7 37   ..,,.HP{....85.7
    00a0 - 67 cb 1c 0a 4e 61 25 b1-9e 84 92 25 c7 36 f8 6e   g...Na%....%.6.n
    00b0 - 30 2d ce 8b d0 55 5f fc-21 cb 35 5b d9 4d b8 5d   0-...U_.!.5[.M.]

    Start Time: 1772904919
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
+OK InFreight POP3 v9.188

```

**Answer :** `InFreight POP3 v9.188`

### 5. **What is the admin email address?**

For this question, we have to login into the imap server with the given credentials (`robin:robin`).

```bash
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ 
AUTH=PLAIN] HTB{roncfbw7iszerd7shni7jr2343zhrj}
1 LOGIN robin robin
1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT 
SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND 
URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 
CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY 
MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE] 
Logged in
```

Once we are logged in, we can further investigate and check which mailboxes are available on the server as done below:

![image.png](images/image%205.png)

We have got four mailboxes, namely `DEV`, `DEPARTMENT`, `INT` and `INBOX` .

**General form of the output:**

```bash
* LIST (attributes) "delimiter" mailbox_name
```

**Attributes:**

- `\HasNoChildren` - Does not contain subfolders
- `\Noselect` - Cannot store messages
- `\HasChildren` - Contains subfolders

**Understanding the folder hierarchy:**

Because the delimiter is `"."`, the folder tree looks like this:

```
DEV
 └── DEPARTMENT
      └── INT
```

So the full hierarchy:

```
DEV
DEV.DEPARTMENT
DEV.DEPARTMENT.INT
INBOX
```

Now let’s select the `DEV.DEPARTMENT.INT` mailbox.

![image.png](images/image%206.png)

We find that the mailbox contains a single mail through the response `* 1 EXISTS` . Let’s print the entire body of the mail.

![image.png](images/image%207.png)

There we have the admin email.

**Answer :** `devadmin@inlanefreight.htb`

### 6. **Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...})**

From the above mail body we also find thr flag and the answer to this question.

**Answer :** `HTB{983uzn8jmfgpd8jmof8c34n7zio}` 

---

# Section 12 : SNMP

After running an `nmap` scan, we see that the `snmp` service is running on the port `161` . Now, we have to find the community string using the tool `onesixtyone` along with a wordlist as to enumerate the `snmp` objects using `snmpwalk` .

```bash
onesixtyone -c community_strings_wordlist.txt <target_IP>
```

![image.png](images/image%208.png)

community_string : `public`

After discovering that **SNMP is running on UDP port 161** and identifying the valid **community string**, we can enumerate SNMP objects using `snmpwalk`.

```bash
snmpwalk -v2c -c <community_string> <target_IP>
```

![image.png](images/image%209.png)

The answer to the first two questions is right here in the image.

### 1. **Enumerate the SNMP service and obtain the email address of the admin. Submit it as the answer.**

**Answer :** `devadmin@inlanefreight.htb`

### **2. What is the customized version of the SNMP server?**

**Answer :** `InFreight SNMP v0.91` 

### **3. Enumerate the custom script that is running on the system and submit its output as the answer.**

Take a look at this part of the `snmpwalk` output.

```bash
iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.4.70.76.65.71 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.4.4.70.76.65.71 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.5.4.70.76.65.71 = INTEGER: 5
iso.3.6.1.2.1.25.1.7.1.2.1.6.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.7.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.20.4.70.76.65.71 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.2.1.21.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.1.4.70.76.65.71 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
iso.3.6.1.2.1.25.1.7.1.3.1.2.4.70.76.65.71 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
iso.3.6.1.2.1.25.1.7.1.3.1.3.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.4.4.70.76.65.71 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.1.4.1.2.4.70.76.65.71.1 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
```

The first important entry is:

```bash
iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
```

This reveals that the SNMP agent is configured to execute the script:

```bash
/usr/share/flag.sh
```

This is typically done using the **NET-SNMP "extend" feature**, which allows administrators to expose the output of custom scripts through SNMP queries.

A typical configuration inside `/etc/snmp/snmpd.conf` would look like:

```bash
extend FLAG /usr/share/flag.sh
```

When queried, SNMP executes this script and returns its output.

- **Understanding the OID Suffix (70.76.65.71):**
    
    The numbers at the end of the OID represent the **ASCII values of the script identifier**.
    
    ```bash
    70 = F
    76 = L
    65 = A
    71 = G
    ```
    
    Therefore:
    
    ```bash
    70.76.65.71 → "FLAG"
    ```
    
    This confirms that the script registered in SNMP is called **FLAG**, which corresponds to the command `/usr/share/flag.sh`.
    
- **Understanding STRING vs INTEGER in SNMP Output:**
    
    In SNMP responses, each object contains a **data type** describing the kind of value being returned.
    
    - `STRING`
        
        A **`STRING`** represents textual data.
        
        Example:
        
        ```bash
        STRING: "/usr/share/flag.sh"
        ```
        
        This indicates that the value stored in that OID is a **text string**, which in this case is the **path to the script**.
        
        Another example:
        
        ```bash
        STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
        ```
        
        Here, the SNMP agent returns the **text output produced by the script**.
        
    
    - `INTEGER`
        
        An **INTEGER** represents a numeric value.
        
        Example:
        
        ```
        INTEGER: 1
        INTEGER: 5
        INTEGER: 0
        ```
        
        These numbers usually represent **status values or configuration parameters** related to the script execution, such as:
        
        - execution state
        - result status
        - exit codes
        - configuration flags
        
        In this case, they are metadata describing the execution of the extend command rather than the actual output of the script.
        
- **Extracting the Script Output**

The critical entries are:

```bash
iso.3.6.1.2.1.25.1.7.1.3.1.1.4.70.76.65.71 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
```

and

```bash
iso.3.6.1.2.1.25.1.7.1.4.1.2.4.70.76.65.71.1 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
```

These OIDs return the **output generated by the script `/usr/share/flag.sh`**.

The returned value is clearly the flag.

**Answer :** `HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}`

---

# Section 13 : MySQL

### 1. **Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX)**

```bash
nmap <target_IP> -sCV -p 3306
```

![image.png](images/image%2010.png)

**Answer :** `MySQL 8.0.27`

### Useful `MySQL` commands for Enumeration

| Command | Description |
| --- | --- |
| `mysql -u <user> -p<password> -h <IP_address>` | Connect to a remote MySQL server. Note: there should be **no space between `-p` and the password**. |
| `show databases;` | Lists all databases available on the MySQL server. |
| `use <database>;` | Selects a specific database to interact with. |
| `show tables;` | Displays all tables inside the selected database. |
| `describe <table>;` | Shows the structure of a table, including column names and data types. |
| `show columns from <table>;` | Lists all columns in the specified table. |
| `select * from <table>;` | Retrieves all records from a table. |
| `select <column> from <table>;` | Retrieves data from a specific column in a table. |
| `select * from <table> limit 10;` | Displays the first 10 rows from a table. Useful when tables are large. |
| `select * from <table> where <column> = "<value>";` | Filters rows based on a condition. |
| `select user, host from mysql.user;` | Lists MySQL users and allowed hosts. Useful for privilege enumeration. |
| `show grants for '<user>'@'<host>';` | Displays privileges assigned to a specific user. |
| `select database();` | Displays the current database being used. |
| `select version();` | Shows the MySQL server version. Useful for identifying vulnerabilities. |
| `exit;` | Exits the MySQL interactive shell. |

### 2. **During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"?**

Login to the `mysql` server with the given credentials using the below command.

```bash
mysql -u robin -probin -h <target_IP>
```

![image.png](images/image%2011.png)

List all the databases in the server.

![image.png](images/image%2012.png)

The `customers` database seems interesting, let’s enumerate it further.

![image.png](images/image%2013.png)

Once the desired database is selected, we can list the tables in the database.

![image.png](images/image%2014.png)

There is a single table named `myTable` . This makes our task easier. Let’s list down all the columns in the table `myTable` .

![image.png](images/image%2015.png)

Now we can easily query the `email` of the customer `Otto Lang` .

![image.png](images/image%2016.png)

**Answer :** `ultrices@google.htb`

---

# Section 14 : MSSQL

### Important MSSQL Commands:

| Command | Description |
| --- | --- |
| `lcd {path}` | Changes the current **local directory** to the specified `{path}`. |
| `exit` | Terminates the **server process** and ends the current session. |
| `enable_xp_cmdshell` | Enables the **xp_cmdshell** stored procedure, allowing execution of system commands from SQL Server. |
| `disable_xp_cmdshell` | Disables the **xp_cmdshell** stored procedure to prevent command execution through SQL Server. |
| `enum_db` | Enumerates all **databases** available on the SQL Server instance. |
| `enum_links` | Enumerates **linked SQL servers** configured on the current server. |
| `enum_impersonate` | Lists logins that can be **impersonated** using SQL Server impersonation privileges. |
| `enum_logins` | Enumerates all **login accounts** configured on the SQL Server. |
| `enum_users` | Enumerates **users in the current database**. |
| `enum_owner` | Displays the **owner of the current database**. |
| `exec_as_user {user}` | Impersonates the specified **database user** using `EXECUTE AS USER`. |
| `exec_as_login {login}` | Impersonates the specified **server login** using `EXECUTE AS LOGIN`. |
| `xp_cmdshell {cmd}` | Executes an **operating system command** via the `xp_cmdshell` stored procedure. |
| `xp_dirtree {path}` | Executes `xp_dirtree` to **list directories and files** in the specified `{path}`. |
| `sp_start_job {cmd}` | Executes a command through **SQL Server Agent jobs** (often used for blind command execution). |
| `use_link {link}` | Selects a **linked SQL Server** to interact with. `use_link localhost` returns to the local server, while `use_link ..` goes back one step. |
| `! {cmd}` | Executes a **local shell command** on the attacker's machine. |
| `upload {from} {to}`  | Uploads a file from the attacker's machine `{from}` to the SQL Server host `{to}`. |
| `show_query` | Displays the **SQL query being executed** by the tool. |
| `mask_query` | Hides the SQL query from being displayed. |

### 1. **Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server.**

We will use `msfconsole` to enumerate the `mssql` server. The steps are as shown below:

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-omfspf88oe-htb-cloud-com]─[~]
└──╼ [★]$ msfconsole
Metasploit tip: The use command supports fuzzy searching to try and 
select the intended module, e.g. use kerberos/get_ticket or use 
kerberos forge silver ticket
                                                  
                                   ____________
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%| $a,        |%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%]
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%| $S`?a,     |%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%]
 [%%%%%%%%%%%%%%%%%%%%__%%%%%%%%%%|       `?a, |%%%%%%%%__%%%%%%%%%__%%__ %%%%]
 [% .--------..-----.|  |_ .---.-.|       .,a$%|.-----.|  |.-----.|__||  |_ %%]
 [% |        ||  -__||   _||  _  ||  ,,aS$""`  ||  _  ||  ||  _  ||  ||   _|%%]
 [% |__|__|__||_____||____||___._||%$P"`       ||   __||__||_____||__||____|%%]
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%| `"a,       ||__|%%%%%%%%%%%%%%%%%%%%%%%%%%]
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%|____`"a,$$__|%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%]
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%        `"$   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%]
 [%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%]

       =[ metasploit v6.4.71-dev                          ]
+ -- --=[ 2529 exploits - 1302 auxiliary - 431 post       ]
+ -- --=[ 1669 payloads - 49 encoders - 13 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/

[msf](Jobs:0 Agents:0) >> search mssql_ping

Matching Modules
================

   #  Name                                Disclosure Date  Rank    Check  Description
   -  ----                                ---------------  ----    -----  -----------
   0  auxiliary/scanner/mssql/mssql_ping  .                normal  No     MSSQL Ping Utility

Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/mssql/mssql_ping

[msf](Jobs:0 Agents:0) >> use 0
[msf](Jobs:0 Agents:0) auxiliary(scanner/mssql/mssql_ping) >> options

Module options (auxiliary/scanner/mssql/mssql_ping):

   Name                 Current Setting  Required  Description
   ----                 ---------------  --------  -----------
   PASSWORD                              no        The password for the specified username
   RHOSTS                                yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit
                                                   /basics/using-metasploit.html
   THREADS              1                yes       The number of concurrent threads (max one per host)
   USERNAME             sa               no        The username to authenticate as
   USE_WINDOWS_AUTHENT  false            yes       Use windows authentication (requires DOMAIN option set)

View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) auxiliary(scanner/mssql/mssql_ping) >> set RHOSTS 10.129.3.255
RHOSTS => 10.129.3.255
[msf](Jobs:0 Agents:0) auxiliary(scanner/mssql/mssql_ping) >> run
[*] 10.129.3.255          - SQL Server information for 10.129.3.255:
[+] 10.129.3.255          -    ServerName      = **ILF-SQL-01**
[+] 10.129.3.255          -    InstanceName    = MSSQLSERVER
[+] 10.129.3.255          -    IsClustered     = No
[+] 10.129.3.255          -    Version         = 15.0.2000.5
[+] 10.129.3.255          -    tcp             = 1433
[+] 10.129.3.255          -    np              = \\ILF-SQL-01\pipe\sql\query
[*] 10.129.3.255          - Scanned 1 of 1 hosts (100% complete)
```

**Answer :** **`ILF-SQL-01`** 

### 2. **Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.**

Connect to the MSSQL instance with the given credentials (`backdoor:Password1`)

![image.png](images/image%2017.png)

Since this is not `MySQL`, we cannot use normal `SQL` commands. Let’s type `help` to know the shell commands.

![image.png](images/image%2018.png)

We can use `enum_db` to list the databases.

![image.png](images/image%2019.png)

The non-default database in these is `Employees` .

**Answer :** `Employees` 

---

# Section 15 : Oracle TNS

Install the below packages and tools if not done already on your VM or Pwnbox Instance.

```bash
sudo apt-get update
sudo apt-get install -y build-essential python3-dev libaio1
cd ~
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz
cd cx_Oracle-8.3.0
python3 setup.py build
sudo python3 setup.py install
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

### 1. **Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.**

We can use the `odat.py` tool to perform a variety of scans to enumerate and gather information about the Oracle database services and its components. Those scans can retrieve database names, versions, running processes, user accounts, vulnerabilities, misconfigurations, etc. Let us use the `all` option and try all modules of the `odat.py` tool.

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-nueofojdbi-htb-cloud-com]─[~/odat]
└──╼ [★]$ ./odat.py all -s 10.129.205.19
[+] Checking if target 10.129.205.19:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.205.19:1521 is well configured. Continue...

[1] (10.129.205.19:1521): Is it vulnerable to TNS poisoning (CVE-2012-1675)?
[+] Impossible to know if target is vulnerable to a remote TNS poisoning because SID is not given.

[2] (10.129.205.19:1521): Searching valid SIDs
[2.1] Searching valid SIDs thanks to a well known SID list on the 10.129.205.19:1521 server
[+] 'XE' is a valid SID. Continue...                              #########################################################################################################  | ETA:  00:00:03 
100% |#######################################################################################################################################################################| Time: 00:06:07 
[2.2] Searching valid SIDs thanks to a brute-force attack on 1 chars now (10.129.205.19:1521)
100% |#######################################################################################################################################################################| Time: 00:00:13 
[2.3] Searching valid SIDs thanks to a brute-force attack on 2 chars now (10.129.205.19:1521)
[+] 'XE' is a valid SID. Continue...                              ########################################################################################                   | ETA:  00:00:36 
100% |#######################################################################################################################################################################| Time: 00:05:33 
[+] SIDs found on the 10.129.205.19:1521 server: XE

[3] (10.129.205.19:1521): Searching valid Service Names
[3.1] Searching valid Service Names thanks to a well known Service Name list on the 10.129.205.19:1521 server
[+] 'XE' is a valid Service Name. Continue...                     #########################################################################################################  | ETA:  00:00:03 
[+] 'XEXDB' is a valid Service Name. Continue...                  
100% |#######################################################################################################################################################################| Time: 00:06:15 
[3.2] Searching valid Service Names thanks to a brute-force attack on 1 chars now (10.129.205.19:1521)
100% |#######################################################################################################################################################################| Time: 00:00:10 
[3.3] Searching valid Service Names thanks to a brute-force attack on 2 chars now (10.129.205.19:1521)
[+] 'XE' is a valid Service Name. Continue...                     ########################################################################################                   | ETA:  00:00:35 
100% |#######################################################################################################################################################################| Time: 00:05:22 
[+] Service Name(s) found on the 10.129.205.19:1521 server: XE,XEXDB
[!] Notice: SID 'XE' found. Service Name 'XE' found too: Identical database instance. Removing Service Name 'XE' from Service Name list in order to don't do same checks twice

[4] (10.129.205.19:1521): Searching valid accounts on the XE SID
The login cis has already been tested at least once. What do you want to do:                                                                                                 | ETA:  00:18:52 
- stop (s/S)
- continue and ask every time (a/A)
- skip and continue to ask (p/P)
- continue without to ask (c/C)
c
[!] Notice: 'ctxsys' account is locked, so skipping this username for password                                                                                               | ETA:  00:37:03 
[!] Notice: 'dbsnmp' account is locked, so skipping this username for password                                                                                               | ETA:  00:34:18 
[!] Notice: 'dip' account is locked, so skipping this username for password                                                                                                  | ETA:  00:30:45 
[!] Notice: 'hr' account is locked, so skipping this username for password 49% |#################################################################################                             [!] Notice: 'mdsys' account is locked, so skipping this username for password
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password########################                                                                   | ETA:  00:11:41 
[!] Notice: 'outln' account is locked, so skipping this username for password###################################                                                             | ETA:  00:10:21 
[+] Valid credentials found: scott/tiger. Continue...             #########################################################################                                  | ETA:  00:05:30 
[!] Notice: 'xdb' account is locked, so skipping this username for password###########################################################################################       | ETA:  00:01:04 
100% |#######################################################################################################################################################################| Time: 00:26:26 
[+] Accounts found on 10.129.205.19:1521/sid:XE: 
**scott/tiger**
```

We find an account with the username `scott` with the password `tiger` for the SID `XE` .

Now we use the tool `sqlplus` as System Database Admin (`sysdba`)to connect with the Oracle database and interact with it.

<aside>
<img src="https://www.notion.so/icons/light-bulb_green.svg" alt="https://www.notion.so/icons/light-bulb_green.svg" width="40px" />

If you are getting errors with `sqlplus` as shown below:

![image.png](images/image%2020.png)

And still facing problems to install it, then execute the below script:

```bash
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip && 
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip && 
sudo mkdir -p /opt/oracle && 
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip && 
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip && 
cd /opt/oracle/instantclient_21_4 && 
find . -type f | sort && 
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH && 
export PATH=$LD_LIBRARY_PATH:$PATH && 
source ~/.bashrc && 
sqlplus -V
```

This will fix all your errors.

</aside>

![image.png](images/image%2021.png)

To get the password hash of the `DBSNMP` user we will run the following query:

```sql
select name, password from sys.user$ where name='DBSNMP';
```

![image.png](images/image%2022.png)

**Answer :** `E066D214D5421CCC`

---

# Section 16 : IPMI

```bash
┌─[eu-academy-4]─[10.10.14.59]─[htb-ac-2120529@htb-imd4dp5fdv-htb-cloud-com]─[~]
└──╼ [★]$ msfconsole -q
[msf](Jobs:0 Agents:0) >> search ipmi

Matching Modules
================

   #   Name                                                                Disclosure Date  Rank    Check  Description
   -   ----                                                                ---------------  ----    -----  -----------
   0   auxiliary/scanner/ipmi/ipmi_cipher_zero                             2013-06-20       normal  No     IPMI 2.0 Cipher Zero Authentication Bypass Scanner
   1   auxiliary/scanner/ipmi/ipmi_dumphashes                              2013-06-20       normal  No     IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval
   2   auxiliary/scanner/ipmi/ipmi_version                                 .                normal  No     IPMI Information Discovery
   3   exploit/multi/upnp/libupnp_ssdp_overflow                            2013-01-29       normal  No     Portable UPnP SDK unique_service_name() Remote Code Execution
   4     \_ target: Automatic                                              .                .       .      .
   5     \_ target: Supermicro Onboard IPMI (X9SCL/X9SCM) Intel SDK 1.3.1  .                .       .      .
   6     \_ target: Axis Camera M1011 5.20.1 UPnP/1.4.1                    .                .       .      .
   7     \_ target: Debug Target                                           .                .       .      .
   8   auxiliary/scanner/http/smt_ipmi_cgi_scanner                         2013-11-06       normal  No     Supermicro Onboard IPMI CGI Vulnerability Scanner
   9   auxiliary/scanner/http/smt_ipmi_49152_exposure                      2014-06-19       normal  No     Supermicro Onboard IPMI Port 49152 Sensitive File Exposure
   10  auxiliary/scanner/http/smt_ipmi_static_cert_scanner                 2013-11-06       normal  No     Supermicro Onboard IPMI Static SSL Certificate Scanner
   11  exploit/linux/http/smt_ipmi_close_window_bof                        2013-11-06       good    Yes    Supermicro Onboard IPMI close_window.cgi Buffer Overflow
   12  auxiliary/scanner/http/smt_ipmi_url_redirect_traversal              2013-11-06       normal  No     Supermicro Onboard IPMI url_redirect.cgi Authenticated Directory Traversal

Interact with a module by name or index. For example info 12, use 12 or use auxiliary/scanner/http/smt_ipmi_url_redirect_traversal

[msf](Jobs:0 Agents:0) >> use 1
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> options

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                                        Required  Description
   ----                  ---------------                                        --------  -----------
   CRACK_COMMON          true                                                   yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                          no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                             no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_p  yes       File containing common passwords for offline cracking, one per line
                         asswords.txt
   RHOSTS                                                                       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.
                                                                                          html
   RPORT                 623                                                    yes       The target port
   SESSION_MAX_ATTEMPTS  5                                                      yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                                      yes       Delay between session retries in seconds
   THREADS               1                                                      yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_u  yes       File containing usernames, one per line
                         sers.txt

View the full module info with the info, or info -d command.
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> set RHOST 10.129.202.5
RHOST => 10.129.202.5
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE => /usr/share/wordlists/rockyou.txt
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run
[+] 10.129.202.5:623 - IPMI - Hash found: admin:41bfb1a082240000661e69c851c94969c755945281dfc9e99efd5e905a0d04b081fb2d6fd33a9c4fa123456789abcdefa123456789abcdef140561646d696e:0e87a36f15335c54a54b9b1101a5e14ca01150e7
[+] 10.129.202.5:623 - IPMI - Hash for user 'admin' matches password 'trinity'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### 1. **What username is configured for accessing the host via IPMI?**

**Answer :** `admin` 

### 2. **What is the account's cleartext password?**

**Answer :** `trinity` 

---

# Section 19 : Footprinting Lab - Easy

> 
> 
> 
> We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.
> 
> The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.
> 
> Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.
> 
> The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.
> 

### **Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer.**

The first thing which we will do is run an `nmap` scan to list the ports and services that are open.

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-rosscfjzxh-htb-cloud-com]─[~]
└──╼ [★]$ sudo nmap -sCV -T4 -p- -vv <Target_IP>
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-10 11:01 CDT

PORT     STATE SERVICE      REASON         VERSION
21/tcp   open  ftp          syn-ack ttl 63 ProFTPD
22/tcp   open  ssh          syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDa9RJRoAShv6FzLx23WYUh5z5vpaC1W0jTGGJuVfOVmOiwXu7d+eLRcf51dFwqe2J4OZ7z70w6Lrbm3RyKjNSZmY0ekPqbXyP0P6KqYn4eFdJkYp74zPUEvC/Y5U9gYmvCpoQ8gvqgAImYwhBXAlAmGDptcfRWRJ3KaRG/bbmfg0vsWqwYvDVfxEcCfbO1m7v6a9EiWELRTynHS26+oJbjY7tX5X05XMvj6L53JMWodHVsFf/vD4/qP2Ic0lafSBXuyKOcN5Tnx0DpExUwqj7GPLaM/ljG5LjF8y2yqZ85GeNQsgnsSxIL6dHiWkbUP4RXogUVI/prXLDU8307Wn/LWJQl3hxjJmunJfC5qw4a/JPLd9ydFSwadjYhztQoYIsSp41mr/wEVns8owxcKzBju74T9FptZ4I4UAzZLIWg1RJzpnJ7wpnFSUXFbvOa6V+nzeMesjYvKK1vx+UuNtrUuXPJm3BoYKjRJd2msog1KX4CguQNGZMS6LegiRIGde0=
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNAdY+PFLa0XBlXCp3lL+mrrQKkU6bxWjDVEsljltzBYtugbDuER3AyIq1igFdgQPn+uKh5RtNQvPvX1Al8pA0Y=
|   256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGKKM5saOYH/Fq3lWY1P4fchdWaH60Ib5/VQk6A00nAP
53/tcp   open  domain       syn-ack ttl 63 ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
2121/tcp open  ccproxy-ftp? syn-ack ttl 63
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 967.45 seconds
           Raw packets sent: 72092 (3.172MB) | Rcvd: 70934 (2.879MB)

```

**Findings:**

- `ftp` service open on ports
    - `21`
    - `2121`
- `ssh` service open on port `22`
- `dns` service open on port `53`

First we will check if we can `ssh` into the target using the given credentials (`ceil:qwer1234`)

![image.png](images/image%2023.png)

So we have to enumerate the `ftp` service now. The port `2121` seems unusual, let’s login into the `ftp` service with the given credentials.

![image.png](images/image%2024.png)

We see there is a `.ssh` directory which contains the private `SSH` key of the user we have logged in as and also the public part of the key pair.

![image.png](images/image%2025.png)

We can tranfer the `id_rsa` file on to our local machine and again try to `ssh` into the target using the key.

![image.png](images/image%2026.png)

Make sure to fix the permissions of the key because SSH refuses insecure keys.

Run:

```
chmod 600 id_rsa
```

- **`chmod`** → change permissions
- `600` → only **you** can read/write the key

Now we can easily `ssh` into the target.

![image.png](images/image%2027.png)

Now let’s search for something interesting that the administrators have left for us.

![image.png](images/image%2028.png)

**There we go, we have the flag!**

**Answer :** `HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}`

---

# Section 20 : Footprinting Lab - Medium

> 
> 
> 
> This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.
> 
> Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named `HTB` has been created. Accordingly, we need to obtain the credentials of this user as proof.
> 

### **Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer.**

Let’s run an `nmap` scan on all ports as usual to discover running services.

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-qb5cd5ewus-htb-cloud-com]─[~]
└──╼ [★]$ nmap 10.129.202.41 -sCV -p- -Pn -T4 -vv
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-11 00:52 CDT

Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
111/tcp   open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
2049/tcp  open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2026-03-11T06:06:28+00:00
| ssl-cert: Subject: commonName=WINMEDIUM
| Issuer: commonName=WINMEDIUM
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-03-10T05:51:05
| Not valid after:  2026-09-09T05:51:05
| MD5:   5b33:45f7:1ea7:7a2f:a699:c1d8:e08c:79ef
| SHA-1: 09a7:306f:5ecd:b380:0eed:1c71:891e:05ac:903b:bd75
| -----BEGIN CERTIFICATE-----
| MIIC1jCCAb6gAwIBAgIQcx441INXiZJLgscfAj6HqzANBgkqhkiG9w0BAQsFADAU
| MRIwEAYDVQQDEwlXSU5NRURJVU0wHhcNMjYwMzEwMDU1MTA1WhcNMjYwOTA5MDU1
| MTA1WjAUMRIwEAYDVQQDEwlXSU5NRURJVU0wggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDY+y/Hui5JhoKKWQdqg2EpBGjlFnCwQJ5avirKlmcOT5i9MMwf
| mbak3Zyy5rju+IO7067EOyKYn4Om70iuWuHBulo1E0SFGEMkoUgAWVMxnj+Vu03j
| S5NDVHLzKLv5rp5OXduuvr17UW8UxA/lt4jCpxKF5QQC/LqH3h1ukMGbXI9gudfK
| JIDSWGDHcDEqo7l2H4PGVMnqaz4tnt9RNNob2SXSjEIaB1Px2lLm/5h1c+I4E05G
| h8qESrgi+s0BO6ztyrAM6rfn1Utjmyl5lAx7dj49t9TiW///IzrWKeFCo5Zfm03x
| 27H32gpO/4A2PcTmU/wkEM6oMfFkT3YHN2oxAgMBAAGjJDAiMBMGA1UdJQQMMAoG
| CCsGAQUFBwMBMAsGA1UdDwQEAwIEMDANBgkqhkiG9w0BAQsFAAOCAQEAqD2wbuqh
| ET8ZqpkccCidHowlbxlRPO1MOtOXd91Hxe4Sru/72mI0tqM0hJ/BqVdjlbuqaJwh
| XX44bmS45fNxWDWuljhZnfI4cSKEZZ9+kX1lHiMmaFmor/iABOqJUwnp7nNxxnUS
| H3dd9f9wVc/Xp0z/snOQFXFVHinnii/ozyeuRcftbbSBkDp2gVXSfvV44uXgAHSE
| QSNEqEsRX4EleDDZnkzpbHuOTxkvriuXqubRjBEyQMCieN7FzbC5T6n3gl/Uszpe
| KgvMMru+QvTEc88x4NHCfcpZT0hCprN+i3WXmqsN1bCEKRi2EgrQAF20k92bZW8V
| rHiGK5hfDXjZcg==
|_-----END CERTIFICATE-----
|_ssl-date: 2026-03-11T06:06:37+00:00; 0s from scanner time.
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49679/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49680/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-03-11T06:06:29
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 47689/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 61014/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 28088/udp): CLEAN (Timeout)
|   Check 4 (port 3163/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: 0s, deviation: 0s, median: 0s

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 01:07
Completed NSE at 01:07, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 01:07
Completed NSE at 01:07, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 01:07
Completed NSE at 01:07, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 942.75 seconds
           Raw packets sent: 70588 (3.106MB) | Rcvd: 70315 (2.980MB)
```

**Findings:**

- `smb` service open on ports
    - `445`
    - `139`
- `nfs` service open on port `111`
- `rdp` service open on port `3389`

First let’s enumerate the Network File System (`NFS`) and list down the available shares.

![image.png](images/image%2029.png)

We found a share named `TechSupport` which is open to connect for everyone. Mount the share on your local machine.

![image.png](images/image%2030.png)

The user and group is set to `nobody` and `nogroup` respectively. This is a common feature of nfs shares which happens usually because of NFS root squashing.

> NFS root squashing prevents remote root users from gaining root-level access on an NFS server by mapping them to the unprivileged `nobody` user.
> 

So we switch to root user by executing `sudo su` .

![image.png](images/image%2031.png)

We see a lot of ticket, but most of them are empty. You can iterate through all the files and print the contents one by one or find the file which is not empty and print its contents.

```bash
┌─[root@htb-qb5cd5ewus-htb-cloud-com]─[/home/htb-ac-2120529/myMount]
└──╼ cat ticket4238791283*.txt
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17    	withUserNameSupport=false
18    	sendWelcomeEmail=true
19    	enableGravatarSupport=true
20    	signupSkipLogin=true
21    	tokenDuration=60
22    	tokenDeleteInterval=5
23    	minimumPasswordLength=8
24    	enableTokenJob=true
25    	hasher=bcrypt
26	}
27
28     cookie {
29            name=id
30            path=/login
31            domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }   

---
```

We find the credentials of a user named `alex` .

Now, let’s enumerate the `smb` service using `crackmapexec` and alex’s credentials.

![image.png](images/image%2032.png)

We will connect to the `devshare` share and list the contents.

![image.png](images/image%2033.png)

We find an interesting file `important.txt` . Transfer this to the local machine and view its contents.

![image.png](images/image%2034.png)

We have found the credentials for the server administrator (`sa:87N1ns@slls83`).

Now, let’s `rdp` to the given target using alex’s credentials.

![image.png](images/image%2035.png)

![Screenshot 2026-03-11 124111.png](images/Screenshot_2026-03-11_124111.png)

Run the `Microsoft SQL Server Management Studio` as `Administrator` .

![Screenshot 2026-03-11 124123.png](images/Screenshot_2026-03-11_124123.png)

Enter the password we found : `87N1ns@slls83` 

![image.png](images/image%2036.png)

Expand the `accounts` database and Select the top 1000 rows of `dbo.devsacc` table.

![Screenshot 2026-03-11 124041.png](images/Screenshot_2026-03-11_124041.png)

There we have it, the password of user `HTB`

**Answer :** `lnch7ehrdn43i7AoqVPK4zWR`

---

# Section 21 : Footprinting Lab - Hard

> The third server is an MX and management server for the internal network. Subsequently, this server has the function of a backup server for the internal accounts in the domain. Accordingly, a user named `HTB` was also created here, whose credentials we need to access.
> 

### **Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.**

We will run both `tcp` scan and `udp` (only ports `161` and `162` narrowing it down to only those concepts which we have learnt) scan using `nmap` on the target.

**TCP Scan:**

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-7eaxeko72l-htb-cloud-com]─[~]
└──╼ [★]$ nmap 10.129.202.20 -sCV -p- -Pn -T4 -vv
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-11 12:00 CDT

PORT    STATE SERVICE  REASON         VERSION
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDa9RJRoAShv6FzLx23WYUh5z5vpaC1W0jTGGJuVfOVmOiwXu7d+eLRcf51dFwqe2J4OZ7z70w6Lrbm3RyKjNSZmY0ekPqbXyP0P6KqYn4eFdJkYp74zPUEvC/Y5U9gYmvCpoQ8gvqgAImYwhBXAlAmGDptcfRWRJ3KaRG/bbmfg0vsWqwYvDVfxEcCfbO1m7v6a9EiWELRTynHS26+oJbjY7tX5X05XMvj6L53JMWodHVsFf/vD4/qP2Ic0lafSBXuyKOcN5Tnx0DpExUwqj7GPLaM/ljG5LjF8y2yqZ85GeNQsgnsSxIL6dHiWkbUP4RXogUVI/prXLDU8307Wn/LWJQl3hxjJmunJfC5qw4a/JPLd9ydFSwadjYhztQoYIsSp41mr/wEVns8owxcKzBju74T9FptZ4I4UAzZLIWg1RJzpnJ7wpnFSUXFbvOa6V+nzeMesjYvKK1vx+UuNtrUuXPJm3BoYKjRJd2msog1KX4CguQNGZMS6LegiRIGde0=
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNAdY+PFLa0XBlXCp3lL+mrrQKkU6bxWjDVEsljltzBYtugbDuER3AyIq1igFdgQPn+uKh5RtNQvPvX1Al8pA0Y=
|   256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGKKM5saOYH/Fq3lWY1P4fchdWaH60Ib5/VQk6A00nAP
110/tcp open  pop3     syn-ack ttl 63 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL(PLAIN) RESP-CODES TOP USER STLS CAPA PIPELINING UIDL AUTH-RESP-CODE
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Issuer: commonName=NIXHARD
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T01:30:25
| Not valid after:  2031-11-08T01:30:25
| MD5:   2b45:ec3c:508f:3cfb:9f6a:750c:63f8:2077
| SHA-1: ed43:7d5a:3c46:54ac:9902:8dc4:9d86:6efb:2ae3:357c
| -----BEGIN CERTIFICATE-----
| MIIC0zCCAbugAwIBAgIUC6tYfrtqQqCrhjYv11bUtaKet3EwDQYJKoZIhvcNAQEL
| BQAwEjEQMA4GA1UEAwwHTklYSEFSRDAeFw0yMTExMTAwMTMwMjVaFw0zMTExMDgw
| MTMwMjVaMBIxEDAOBgNVBAMMB05JWEhBUkQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDEBpDfkH4Ro5ZXW44NvnF3N9lKz27V1hgRppyUk5y/SEPKt2zj
| EU+r2tEHUeHoJHQZBbW0ybxh+X2H3ZPNEG9nV1GtFQfTBVcrUEpN5VV15aIbdh+q
| j53pp/wcL/d8+Zg2ZAaVYWvQHVqtsAudQmynrV1MHA39A44fG3/SutKlurY8AKR0
| MW5zMPtflMc/N3+lH8UUMBf2Q+zNSyZLiBEihxK3kfMW92HqWeh016egSIFuxUsH
| kk4xpGmyG9NDYna47dQzoHCg+42KgqFvWrGw2nIccaEIX5XA8rU9u53C7EQzDzmQ
| vAtHpKWBwNmiivxAz/QC7MPExWIWtZtOqxmfAgMBAAGjITAfMAkGA1UdEwQCMAAw
| EgYDVR0RBAswCYIHTklYSEFSRDANBgkqhkiG9w0BAQsFAAOCAQEAG+Dm9pLJgNGC
| X1YmznmtBUekhXMrU67tQl745fFasJQzIrDgVtK27fjAtQRwvIbDruSwTj47E7+O
| XdS7qyjFNBerklWNq4fEAVI7BmkxnTS9542okA/+UmeG70LdKjzFS+LjjOnyWzTh
| YwU8uUjLfnRca74kY0DkVHOIkwZQha0J+BrKSADq/zDjkG0g4v0vzHINOmHx9eiE
| 67NoJKJPY5S3RYWxl/4x8Kphx7PNJBPC75gYjlxxDhxdYu9a3daqJUa58/qOm6P8
| w1P9nA6lkg7NopyqepulLAzIcqnTjb/nMD2Pd9b6vgWc3IqSfFreqjzshZ+FjNZo
| zR+tR6z4TQ==
|_-----END CERTIFICATE-----
143/tcp open  imap     syn-ack ttl 63 Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: ENABLE more ID have post-login IMAP4rev1 SASL-IR IDLE listed AUTH=PLAINA0001 Pre-login OK capabilities STARTTLS LOGIN-REFERRALS LITERAL+
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Issuer: commonName=NIXHARD
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T01:30:25
| Not valid after:  2031-11-08T01:30:25
| MD5:   2b45:ec3c:508f:3cfb:9f6a:750c:63f8:2077
| SHA-1: ed43:7d5a:3c46:54ac:9902:8dc4:9d86:6efb:2ae3:357c
| -----BEGIN CERTIFICATE-----
| MIIC0zCCAbugAwIBAgIUC6tYfrtqQqCrhjYv11bUtaKet3EwDQYJKoZIhvcNAQEL
| BQAwEjEQMA4GA1UEAwwHTklYSEFSRDAeFw0yMTExMTAwMTMwMjVaFw0zMTExMDgw
| MTMwMjVaMBIxEDAOBgNVBAMMB05JWEhBUkQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDEBpDfkH4Ro5ZXW44NvnF3N9lKz27V1hgRppyUk5y/SEPKt2zj
| EU+r2tEHUeHoJHQZBbW0ybxh+X2H3ZPNEG9nV1GtFQfTBVcrUEpN5VV15aIbdh+q
| j53pp/wcL/d8+Zg2ZAaVYWvQHVqtsAudQmynrV1MHA39A44fG3/SutKlurY8AKR0
| MW5zMPtflMc/N3+lH8UUMBf2Q+zNSyZLiBEihxK3kfMW92HqWeh016egSIFuxUsH
| kk4xpGmyG9NDYna47dQzoHCg+42KgqFvWrGw2nIccaEIX5XA8rU9u53C7EQzDzmQ
| vAtHpKWBwNmiivxAz/QC7MPExWIWtZtOqxmfAgMBAAGjITAfMAkGA1UdEwQCMAAw
| EgYDVR0RBAswCYIHTklYSEFSRDANBgkqhkiG9w0BAQsFAAOCAQEAG+Dm9pLJgNGC
| X1YmznmtBUekhXMrU67tQl745fFasJQzIrDgVtK27fjAtQRwvIbDruSwTj47E7+O
| XdS7qyjFNBerklWNq4fEAVI7BmkxnTS9542okA/+UmeG70LdKjzFS+LjjOnyWzTh
| YwU8uUjLfnRca74kY0DkVHOIkwZQha0J+BrKSADq/zDjkG0g4v0vzHINOmHx9eiE
| 67NoJKJPY5S3RYWxl/4x8Kphx7PNJBPC75gYjlxxDhxdYu9a3daqJUa58/qOm6P8
| w1P9nA6lkg7NopyqepulLAzIcqnTjb/nMD2Pd9b6vgWc3IqSfFreqjzshZ+FjNZo
| zR+tR6z4TQ==
|_-----END CERTIFICATE-----
993/tcp open  ssl/imap syn-ack ttl 63 Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE ID more have IMAP4rev1 SASL-IR AUTH=PLAINA0001 post-login listed Pre-login OK capabilities LOGIN-REFERRALS IDLE LITERAL+
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Issuer: commonName=NIXHARD
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T01:30:25
| Not valid after:  2031-11-08T01:30:25
| MD5:   2b45:ec3c:508f:3cfb:9f6a:750c:63f8:2077
| SHA-1: ed43:7d5a:3c46:54ac:9902:8dc4:9d86:6efb:2ae3:357c
| -----BEGIN CERTIFICATE-----
| MIIC0zCCAbugAwIBAgIUC6tYfrtqQqCrhjYv11bUtaKet3EwDQYJKoZIhvcNAQEL
| BQAwEjEQMA4GA1UEAwwHTklYSEFSRDAeFw0yMTExMTAwMTMwMjVaFw0zMTExMDgw
| MTMwMjVaMBIxEDAOBgNVBAMMB05JWEhBUkQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDEBpDfkH4Ro5ZXW44NvnF3N9lKz27V1hgRppyUk5y/SEPKt2zj
| EU+r2tEHUeHoJHQZBbW0ybxh+X2H3ZPNEG9nV1GtFQfTBVcrUEpN5VV15aIbdh+q
| j53pp/wcL/d8+Zg2ZAaVYWvQHVqtsAudQmynrV1MHA39A44fG3/SutKlurY8AKR0
| MW5zMPtflMc/N3+lH8UUMBf2Q+zNSyZLiBEihxK3kfMW92HqWeh016egSIFuxUsH
| kk4xpGmyG9NDYna47dQzoHCg+42KgqFvWrGw2nIccaEIX5XA8rU9u53C7EQzDzmQ
| vAtHpKWBwNmiivxAz/QC7MPExWIWtZtOqxmfAgMBAAGjITAfMAkGA1UdEwQCMAAw
| EgYDVR0RBAswCYIHTklYSEFSRDANBgkqhkiG9w0BAQsFAAOCAQEAG+Dm9pLJgNGC
| X1YmznmtBUekhXMrU67tQl745fFasJQzIrDgVtK27fjAtQRwvIbDruSwTj47E7+O
| XdS7qyjFNBerklWNq4fEAVI7BmkxnTS9542okA/+UmeG70LdKjzFS+LjjOnyWzTh
| YwU8uUjLfnRca74kY0DkVHOIkwZQha0J+BrKSADq/zDjkG0g4v0vzHINOmHx9eiE
| 67NoJKJPY5S3RYWxl/4x8Kphx7PNJBPC75gYjlxxDhxdYu9a3daqJUa58/qOm6P8
| w1P9nA6lkg7NopyqepulLAzIcqnTjb/nMD2Pd9b6vgWc3IqSfFreqjzshZ+FjNZo
| zR+tR6z4TQ==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3 syn-ack ttl 63 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA USER SASL(PLAIN) RESP-CODES TOP PIPELINING AUTH-RESP-CODE UIDL
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Issuer: commonName=NIXHARD
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T01:30:25
| Not valid after:  2031-11-08T01:30:25
| MD5:   2b45:ec3c:508f:3cfb:9f6a:750c:63f8:2077
| SHA-1: ed43:7d5a:3c46:54ac:9902:8dc4:9d86:6efb:2ae3:357c
| -----BEGIN CERTIFICATE-----
| MIIC0zCCAbugAwIBAgIUC6tYfrtqQqCrhjYv11bUtaKet3EwDQYJKoZIhvcNAQEL
| BQAwEjEQMA4GA1UEAwwHTklYSEFSRDAeFw0yMTExMTAwMTMwMjVaFw0zMTExMDgw
| MTMwMjVaMBIxEDAOBgNVBAMMB05JWEhBUkQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDEBpDfkH4Ro5ZXW44NvnF3N9lKz27V1hgRppyUk5y/SEPKt2zj
| EU+r2tEHUeHoJHQZBbW0ybxh+X2H3ZPNEG9nV1GtFQfTBVcrUEpN5VV15aIbdh+q
| j53pp/wcL/d8+Zg2ZAaVYWvQHVqtsAudQmynrV1MHA39A44fG3/SutKlurY8AKR0
| MW5zMPtflMc/N3+lH8UUMBf2Q+zNSyZLiBEihxK3kfMW92HqWeh016egSIFuxUsH
| kk4xpGmyG9NDYna47dQzoHCg+42KgqFvWrGw2nIccaEIX5XA8rU9u53C7EQzDzmQ
| vAtHpKWBwNmiivxAz/QC7MPExWIWtZtOqxmfAgMBAAGjITAfMAkGA1UdEwQCMAAw
| EgYDVR0RBAswCYIHTklYSEFSRDANBgkqhkiG9w0BAQsFAAOCAQEAG+Dm9pLJgNGC
| X1YmznmtBUekhXMrU67tQl745fFasJQzIrDgVtK27fjAtQRwvIbDruSwTj47E7+O
| XdS7qyjFNBerklWNq4fEAVI7BmkxnTS9542okA/+UmeG70LdKjzFS+LjjOnyWzTh
| YwU8uUjLfnRca74kY0DkVHOIkwZQha0J+BrKSADq/zDjkG0g4v0vzHINOmHx9eiE
| 67NoJKJPY5S3RYWxl/4x8Kphx7PNJBPC75gYjlxxDhxdYu9a3daqJUa58/qOm6P8
| w1P9nA6lkg7NopyqepulLAzIcqnTjb/nMD2Pd9b6vgWc3IqSfFreqjzshZ+FjNZo
| zR+tR6z4TQ==
|_-----END CERTIFICATE-----
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:16
Completed NSE at 12:16, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:16
Completed NSE at 12:16, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:16
Completed NSE at 12:16, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 916.63 seconds
           Raw packets sent: 73400 (3.230MB) | Rcvd: 71581 (2.929MB)

```

**Findings:**

- `ssh` service on port `22`
- `imap` service on ports
    - `143`
    - `993`
- `pop3` service on ports
    - `110`
    - `995`

**UDP Scan:**

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-7eaxeko72l-htb-cloud-com]─[~]
└──╼ [★]$ nmap 10.129.202.20 -sCVU -p161,162 -Pn -T4 -vv
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-11 12:23 CDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:23
Completed NSE at 12:23, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:23
Completed NSE at 12:23, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:23
Completed NSE at 12:23, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 12:23
Completed Parallel DNS resolution of 1 host. at 12:23, 0.00s elapsed
Initiating UDP Scan at 12:23
Scanning 10.129.202.20 [2 ports]
Discovered open port 161/udp on 10.129.202.20
Completed UDP Scan at 12:23, 2.31s elapsed (2 total ports)
Initiating Service scan at 12:23
NSE: Script scanning 10.129.202.20.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:23
Completed NSE at 12:24, 10.07s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:24
Completed NSE at 12:24, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:24
Completed NSE at 12:24, 0.00s elapsed
Nmap scan report for 10.129.202.20
Host is up, received user-set (0.21s latency).
Scanned at 2026-03-11 12:23:49 CDT for 12s

PORT    STATE  SERVICE  REASON              VERSION
161/udp open   snmp     udp-response ttl 63 net-snmp; net-snmp SNMPv3 server
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 5b99e75a10288b6100000000
|   snmpEngineBoots: 10
|_  snmpEngineTime: 29m01s
162/udp closed snmptrap port-unreach ttl 63

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 12:24
Completed NSE at 12:24, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 12:24
Completed NSE at 12:24, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 12:24
Completed NSE at 12:24, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.58 seconds
           Raw packets sent: 4 (223B) | Rcvd: 6 (503B)

```

**Findings:**

- `snmp` service on port `161`

The `ssh`, `imap` and `pop3` cannot be enumerated until and unless we get any user credentials.

So let’s first enumerate the `snmp` service, by first finding the community string using the `onesixtyone` tool. 

![image.png](images/image%2037.png)

We found a community string - `backup` . We will use this to execute an snmp walk.

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-7eaxeko72l-htb-cloud-com]─[~]
└──╼ [★]$ snmpwalk -v2c -c backup 10.129.202.20
iso.3.6.1.2.1.1.1.0 = STRING: "Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (588471) 1:38:04.71
iso.3.6.1.2.1.1.4.0 = STRING: "Admin <tech@inlanefreight.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "NIXHARD"
iso.3.6.1.2.1.1.6.0 = STRING: "Inlanefreight"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (8) 0:00:00.08
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (590022) 1:38:20.22
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 EA 03 0B 12 20 16 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.4.0-90-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro ipv6.disable=1 maybe-ubiquity
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 159
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.1.1.0 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
iso.3.6.1.2.1.25.1.7.1.2.1.4.6.66.65.67.75.85.80 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.5.6.66.65.67.75.85.80 = INTEGER: 5
iso.3.6.1.2.1.25.1.7.1.2.1.6.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.7.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.20.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.2.1.21.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.1.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.3.1.2.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:
Authentication token manipulation error
chpasswd: (line 1, user tom) password not changed
Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.3.1.3.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.3.1.4.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.1 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.2 = STRING: "Authentication token manipulation error"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.3 = STRING: "chpasswd: (line 1, user tom) password not changed"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = STRING: "Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = No more variables left in this MIB View (It is past the end of the MIB tree)
```

Take a look at the OID `iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80` . It is some sort of account recovery script, where we can see the user named `tom` is trying to change his password. We can also see the username and password being `tom` and `NMds732Js2761` . Let’s try out these credentials on the `imap` ,`ssh` and `pop3` services.

![image.png](images/image%2038.png)

`ssh` needs a private key. Let’s enumerate `imap` service.

```bash
┌─[eu-academy-4]─[10.10.14.233]─[htb-ac-2120529@htb-7eaxeko72l-htb-cloud-com]─[~]
└──╼ [★]$ openssl s_client -connect 10.129.202.20:imaps
CONNECTED(00000003)
----SNIP----

----SNIP----
read R BLOCK
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] Dovecot (Ubuntu) ready.
1 LOGIN tom NMds732Js2761
1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE] Logged in
```

So now that we have logged in successfully, we can list all the mailboxes.

![image.png](images/image%2039.png)

The Important mailbox seems interesting, let’s select it and list all the mails.

![image.png](images/image%2040.png)

There are zero mails in the mailbox, let’s check `INBOX` .

```bash
1 SELECT INBOX
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636509064] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
1 OK [READ-WRITE] Select completed (0.369 + 0.000 + 0.368 secs).
1 FETCH 1 BODY[]
* 1 FETCH (BODY[] {3661}
HELO dev.inlanefreight.htb
MAIL FROM:<tech@dev.inlanefreight.htb>
RCPT TO:<bob@inlanefreight.htb>
DATA
From: [Admin] <tech@inlanefreight.htb>
To: <tom@inlanefreight.htb>
Date: Wed, 10 Nov 2010 14:21:26 +0200
Subject: KEY

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAgEA9snuYvJaB/QOnkaAs92nyBKypu73HMxyU9XWTS+UBbY3lVFH0t+F
+yuX+57Wo48pORqVAuMINrqxjxEPA7XMPR9XIsa60APplOSiQQqYreqEj6pjTj8wguR0Sd
hfKDOZwIQ1ILHecgJAA0zY2NwWmX5zVDDeIckjibxjrTvx7PHFdND3urVhelyuQ89BtJqB
abmrB5zzmaltTK0VuAxR/SFcVaTJNXd5Utw9SUk4/l0imjP3/ong1nlguuJGc1s47tqKBP
HuJKqn5r6am5xgX5k4ct7VQOQbRJwaiQVA5iShrwZxX5wBnZISazgCz/D6IdVMXilAUFKQ
X1thi32f3jkylCb/DBzGRROCMgiD5Al+uccy9cm9aS6RLPt06OqMb9StNGOnkqY8rIHPga
H/RjqDTSJbNab3w+CShlb+H/p9cWGxhIrII+lBTcpCUAIBbPtbDFv9M3j0SjsMTr2Q0B0O
jKENcSKSq1E1m8FDHqgpSY5zzyRi7V/WZxCXbv8lCgk5GWTNmpNrS7qSjxO0N143zMRDZy
Ex74aYCx3aFIaIGFXT/EedRQ5l0cy7xVyM4wIIA+XlKR75kZpAVj6YYkMDtL86RN6o8u1x
3txZv15lMtfG4jzztGwnVQiGscG0CWuUA+E1pGlBwfaswlomVeoYK9OJJ3hJeJ7SpCt2GG
cAAAdIRrOunEazrpwAAAAHc3NoLXJzYQAAAgEA9snuYvJaB/QOnkaAs92nyBKypu73HMxy
U9XWTS+UBbY3lVFH0t+F+yuX+57Wo48pORqVAuMINrqxjxEPA7XMPR9XIsa60APplOSiQQ
qYreqEj6pjTj8wguR0SdhfKDOZwIQ1ILHecgJAA0zY2NwWmX5zVDDeIckjibxjrTvx7PHF
dND3urVhelyuQ89BtJqBabmrB5zzmaltTK0VuAxR/SFcVaTJNXd5Utw9SUk4/l0imjP3/o
ng1nlguuJGc1s47tqKBPHuJKqn5r6am5xgX5k4ct7VQOQbRJwaiQVA5iShrwZxX5wBnZIS
azgCz/D6IdVMXilAUFKQX1thi32f3jkylCb/DBzGRROCMgiD5Al+uccy9cm9aS6RLPt06O
qMb9StNGOnkqY8rIHPgaH/RjqDTSJbNab3w+CShlb+H/p9cWGxhIrII+lBTcpCUAIBbPtb
DFv9M3j0SjsMTr2Q0B0OjKENcSKSq1E1m8FDHqgpSY5zzyRi7V/WZxCXbv8lCgk5GWTNmp
NrS7qSjxO0N143zMRDZyEx74aYCx3aFIaIGFXT/EedRQ5l0cy7xVyM4wIIA+XlKR75kZpA
Vj6YYkMDtL86RN6o8u1x3txZv15lMtfG4jzztGwnVQiGscG0CWuUA+E1pGlBwfaswlomVe
oYK9OJJ3hJeJ7SpCt2GGcAAAADAQABAAACAQC0wxW0LfWZ676lWdi9ZjaVynRG57PiyTFY
jMFqSdYvFNfDrARixcx6O+UXrbFjneHA7OKGecqzY63Yr9MCka+meYU2eL+uy57Uq17ZKy
zH/oXYQSJ51rjutu0ihbS1Wo5cv7m2V/IqKdG/WRNgTFzVUxSgbybVMmGwamfMJKNAPZq2
xLUfcemTWb1e97kV0zHFQfSvH9wiCkJ/rivBYmzPbxcVuByU6Azaj2zoeBSh45ALyNL2Aw
HHtqIOYNzfc8rQ0QvVMWuQOdu/nI7cOf8xJqZ9JRCodiwu5fRdtpZhvCUdcSerszZPtwV8
uUr+CnD8RSKpuadc7gzHe8SICp0EFUDX5g4Fa5HqbaInLt3IUFuXW4SHsBPzHqrwhsem8z
tjtgYVDcJR1FEpLfXFOC0eVcu9WiJbDJEIgQJNq3aazd3Ykv8+yOcAcLgp8x7QP+s+Drs6
4/6iYCbWbsNA5ATTFz2K5GswRGsWxh0cKhhpl7z11VWBHrfIFv6z0KEXZ/AXkg9x2w9btc
dr3ASyox5AAJdYwkzPxTjtDQcN5tKVdjR1LRZXZX/IZSrK5+Or8oaBgpG47L7okiw32SSQ
5p8oskhY/He6uDNTS5cpLclcfL5SXH6TZyJxrwtr0FHTlQGAqpBn+Lc3vxrb6nbpx49MPt
DGiG8xK59HAA/c222dwQAAAQEA5vtA9vxS5n16PBE8rEAVgP+QEiPFcUGyawA6gIQGY1It
4SslwwVM8OJlpWdAmF8JqKSDg5tglvGtx4YYFwlKYm9CiaUyu7fqadmncSiQTEkTYvRQcy
tCVFGW0EqxfH7ycA5zC5KGA9pSyTxn4w9hexp6wqVVdlLoJvzlNxuqKnhbxa7ia8vYp/hp
6EWh72gWLtAzNyo6bk2YykiSUQIfHPlcL6oCAHZblZ06Usls2ZMObGh1H/7gvurlnFaJVn
CHcOWIsOeQiykVV/l5oKW1RlZdshBkBXE1KS0rfRLLkrOz+73i9nSPRvZT4xQ5tDIBBXSN
y4HXDjeoV2GJruL7qAAAAQEA/XiMw8fvw6MqfsFdExI6FCDLAMnuFZycMSQjmTWIMP3cNA
2qekJF44lL3ov+etmkGDiaWI5XjUbl1ZmMZB1G8/vk8Y9ysZeIN5DvOIv46c9t55pyIl5+
fWHo7g0DzOw0Z9ccM0lr60hRTm8Gr/Uv4TgpChU1cnZbo2TNld3SgVwUJFxxa//LkX8HGD
vf2Z8wDY4Y0QRCFnHtUUwSPiS9GVKfQFb6wM+IAcQv5c1MAJlufy0nS0pyDbxlPsc9HEe8
EXS1EDnXGjx1EQ5SJhmDmO1rL1Ien1fVnnibuiclAoqCJwcNnw/qRv3ksq0gF5lZsb3aFu
kHJpu34GKUVLy74QAAAQEA+UBQH/jO319NgMG5NKq53bXSc23suIIqDYajrJ7h9Gef7w0o
eogDuMKRjSdDMG9vGlm982/B/DWp/Lqpdt+59UsBceN7mH21+2CKn6NTeuwpL8lRjnGgCS
t4rWzFOWhw1IitEg29d8fPNTBuIVktJU/M/BaXfyNyZo0y5boTOELoU3aDfdGIQ7iEwth5
vOVZ1VyxSnhcsREMJNE2U6ETGJMY25MSQytrI9sH93tqWz1CIUEkBV3XsbcjjPSrPGShV/
H+alMnPR1boleRUIge8MtQwoC4pFLtMHRWw6yru3tkRbPBtNPDAZjkwF1zXqUBkC0x5c7y
XvSb8cNlUIWdRwAAAAt0b21ATklYSEFSRAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
)
1 OK Fetch completed (0.005 + 0.000 + 0.004 secs).

```

We find a private which supposingly belongs to user `tom` .

Create a new file named `id_rsa` and copy paste the private key starting from `-----BEGIN OPENSSH PRIVATE KEY-----` and ending at `-----END OPENSSH PRIVATE KEY-----` . Make sure to give proper permissions to the key `chmod 600 id_rsa` . Once everything is done, you can use it ot ssh into the target as user `tom` .

![image.png](images/image%2041.png)

Now let’s start enumerating the target further by listing all the files.

![image.png](images/image%2042.png)

There are 2 interesting files:

- `.bash_history`
- `.mysql_history`

Let’s view both of them.

![image.png](images/image%2043.png)

![image.png](images/image%2044.png)

This shows that we can login into the `mysql` server using tom’s credentials. There is also a database named `users` which we can enumerate further.

![image.png](images/image%2045.png)

We are successfully logged in. Let’s list down all the databases.

![image.png](images/image%2046.png)

We have select the `users` database, now we can list down all the tables in the database.

 

![image.png](images/image%2047.png)

There is only a single table and this makes our work much easier.

![image.png](images/image%2048.png)

There we have the password!

**Answer :** `cr3n4o7rzse7rzhnckhssncif7ds`

---