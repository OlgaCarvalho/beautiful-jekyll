---
layout: post
title: How ransomware works
subtitle: A high-level overview of how ransomware works
tags: [DFIR, howto]
---

Ransomware attacks have become increasingly more sophisticated, making it more difficult for organizations to defend against and recover from these attacks 🏴‍☠️.

A good cyber hygiene and protection measures ensure your organization's resilience to these types of attacks, but understanding how these attacks work helps you discern the value of investing in cyber security and what to prioritize.

In this post I will talk about:

* What is ransomware
* What is a general timeline for a ransomware attack
* A real-life cyber-attack applied on the timeline
 


## What is ransomware

[Ransomware](https://cyber.gc.ca/en/glossary#r) is a type of malware that denies a user access to a system or data until a sum of money is paid. 

When it infects a device, it either locks the screen or encrypts the files, preventing access to your information and systems.

You will receive a message on your screen indicating your files have been encrypted, or your device locked, and the instructions to pay the ransom in order to have them decrypted or unlocked.

This payment is often requested in the form of digital currency. To coerce the victim, threat actors often threaten to release the data or authentication credentials to publicly embarrass the organization.

On the image below you can see a [real sample](https://blog.malwarebytes.com/cybercrime/malware/2019/01/ryuk-ransomware-attacks-businesses-over-the-holidays/) of Ryuk ransomware infection message given.

![ryuk-ransom-note](/assets/img/ryuk-ransom-note.png)


However, what most don't realize, is when the ransomware is deployed, it means the attacker has been inside the network for weeks or months before the final load (ransomware) is deployed.


## Ransomware Timeline

In the figure below I created a general timeline and methodology of how a ransomware attack takes place, starting on the beginning stages of obtaining access to a system, to the final goal of deploying the ransomware malware, as well as the visibility and detection probability from a defender point-of-view.

![ransomware-timeline](/assets/img/oc_ransomware_timeline.png)

##### Initial Access

The first step of any ransomware attack is to get the malware installed on the host system. This typically occurs using specific techniques for initial access, most common being:

* **Phishing attack** - where users of the victim organization receive an e-mail with a malicious attachment or link. Trough that attachment or link, malware can be deployed, or the user can be tricked in providing their credentials.
* **Brute-force attack** - where users' accounts are submitted to scripts that use a trial-and-error technique to guess login information. When a match is found, the attacker can use the compromised user and password combination to access the system.
* **Vulnerability Exploitation** - where a vulnerability is exploited by the attacker in order to gain system access.



##### Privilege Escalation

When access is gained, the attackers use post-exploitation frameworks to scan the infrastructure and its vulnerabilities, in order to gain elevated privilege and have a higher control of the system.



##### Nesting

With administrator privileges, the attackers are better equipped to infect other devices with the ransomware payload. During this stage, the attacker will also elevate their privileges on other assets and create backdoors in case their main access vector is closed.



##### Information Gathering

Once the attackers take control of the organization's systems and connected devices, they usually take time in searching for valuable data and critical systems in order to better understand the most destructive targets. With this information, they know what data to steal and which systems that need to be fully compromised in order to have the most impact in the business.



##### Automation Testing

In this stage, the attackers start to run their scripts and tools. This includes dumping credentials, extract data from the company and erase backups.




##### Final load
Once the attackers have stolen everything they want and erased backups to make sure reconstruction is impossible, they launch the ransomware and the instructions to pay it.



## A real-life cyber-attack

Let's take as an example, the cyber attack [Cisco reported in August 10](https://blog.talosintelligence.com/2022/08/recent-cyber-attack.html) to have suffered back in May.

*Note:* many of this adversary techniques happen in parallel to each other. This is a simple exercise of how this general timeline can help us, not only to analyze an incident but to also investigate it during an attack, knowing what to expect from the TTPs being used by the attacker.

#### Initial Access

1. The attacker gain access to a personal Gmail account of a Cisco's employee. In the Gmail account, the credentials for Cisco's VPN access were stored.
> Initial access to the Cisco VPN was achieved via the successful compromise of a Cisco employee’s personal Google account. The user had enabled password syncing via Google Chrome and had stored their Cisco credentials in their browser (...)
2. Since the VPN required MFA, the attacker used a combination of MFA push spamming (sending multiple MFA prompts to the victim's phone) and impersonating Cisco IT support team they called the victim in order for them to accept the prompt.
> After obtaining the user's credentials, the attacker attempted to bypass multifactor authentication (MFA) using a variety of techniques, including voice phishing (aka "vishing") and MFA fatigue (...)
3. After connecting to the VPN, the attacker enrolled other devices for MFA, eliminating the need to spam the user with MFA prompts. With MFA enrolled devices they were able to connect to the network whenever they wanted.
> Once the attacker had obtained initial access, they enrolled a series of new devices for MFA and authenticated successfully to the Cisco VPN.

#### Privilege Escalation

4. Once on the system, the attacker started by scanning the infrastructure to understand what privileges the compromised account can obtain.
> Once on a system, the threat actor began to enumerate the environment, using common built-in Windows utilities to identify the user and group membership configuration of the system, hostname, and identify the context of the user account under which they were operating. 
5. Using the compromised account on different systems they were able to gain privileged access.
> After establishing access to the VPN, the attacker then began to use the compromised user account to logon to a large number of systems before beginning to pivot further into the environment. They moved into the Citrix environment, compromising a series of Citrix servers and eventually obtained privileged access to domain controllers. 



#### Nesting

6. Once they obtained privileged access they were able to get their hands on other credentials for further access.
> After obtaining access to the domain controllers, the attacker began attempting to dump NTDS.  
> (...)  
> After obtaining access to credential databases, the attacker was observed leveraging machine accounts for privileged authentication and lateral movement across the environment. 
7. To get full access to other systems the attacker created an administrator account for them. 
>  (...) the adversary created an administrative user called “z” on the system using the built-in Windows “net.exe” commands. This account was then added to the local Administrators group. 
8. In the meantime, the attacker also infected other devices and made sure they have backdoors in case any access is lost.
> The actor in question dropped a variety of tools, including remote access tools like LogMeIn and TeamViewer, offensive security tools such as Cobalt Strike, PowerSploit, Mimikatz, and Impacket, and added their own backdoor accounts and persistence mechanisms. 



#### Information Gathering

9. Once they have administrator privileges and access to multiple systems it is time to gather the most information possible. This includes critical systems like directory services environment, and high-value data like registry information, etc.
> (The administrator account they created) was then used in some cases to execute additional utilities, such as adfind or secretsdump, to attempt to enumerate the directory services environment and obtain additional credentials. Additionally, the threat actor was observed attempting to extract registry information, including the SAM database on compromised windows hosts.  
> (...)  
> On some systems, the attacker was observed employing MiniDump from Mimikatz to dump LSASS. 



#### Automation Testing

10. Once the attackers have a pretty good knowledge of the environment it is time to steal data.
> Throughout the attack, we observed attempts to exfiltrate information from the environment
> To move files between systems within the environment, the threat actor often leveraged Remote Desktop Protocol (RDP) and Citrix. 
11. Also, the attacker can be seen here hiding their traces.
> The attacker also took steps to remove evidence of activities performed on compromised systems by deleting the previously created local Administrator account. 



#### Final Load

The final load would be expected to be a ransomware, but the Cisco Security Incident Response Team (CSIRT) responded to the incident, when they were alerted when an account with administrative privileges was logging in to multiple systems.

However, data exfiltration did ending up to occur.





# Wrapping-up
Unfortunately there isn't a silver bullet in cyber security. 

Even as more and more organizations roll out defenses like MFA, attackers will find a way to bypass it.

This is the reality security professionals live in - there is no finish line in cyber security - it is an endless game of survival ⚔️.

However there is always ways to improve, adapt and stay alert 🕵️‍♀️.

Next post I'll talk about:
* How to respond to a ransomware
* How to prevent a ransomware
