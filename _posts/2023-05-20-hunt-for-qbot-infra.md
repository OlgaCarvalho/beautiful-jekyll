---
layout: post
title: Hunt for a QBot infrastructure
subtitle: Using JARM and HTTP Response hash (step-by-step guide)
tags: [threathunting, DFIR, tutorial]
---

In this post I will show you how we can hunt for a malicious infrastructure using a few standard concepts and tools.

These include:

- The Qbot malware
- The JARM fingerprint
- The Shodan tool

  

## Lecture Time

### What is QBot?

QakBot (aka QBot, QuackBot, and Pinkslipbot) has historically been known as a banking Trojan that was first observed in 2007, with the main purpose of stealing banking data (banking credentials, online banking session information, victim’s personal details, etc.). 

However, since then, QBot has continued to be developed, with more capabilities and new techniques that transcended the financial sector to other sectors. In addition to its first purpose, it now has functionalities that allow QBot to spread itself, evade detection and debugging, and to function as a loader, using C2 servers to install additional malware on compromised machines, such as Cobalt Strike, REvil, ProLock, and Egregor ransomware.

With this, it has become one of the leading trojans globally.

  

### What is JARM and how can it help?

**JARM** is an active Transport Layer Security (TLS) server fingerprinting tool, released in 2020 as an open source project by [Salesforce](https://engineering.salesforce.com/easily-identify-malicious-servers-on-the-internet-with-jarm-e095edac525a/ "https://engineering.salesforce.com/easily-identify-malicious-servers-on-the-internet-with-jarm-e095edac525a/").

JARM works by actively sending 10 TLS `ClientHello` packets to TLS instances (HTTPS servers and services) to create a fingerprint of their TLS configuration, according to a uniqueness of the responses. Then it aggregates and hashes these responses in such a way as to create a **JARM fingerprint**. 

  

A JARM is a 62-character fingerprint, and it is in itself a concatenation of two fingerprints:

- First 30 bytes: The output of a hybrid fuzzy hash of the service’s TLS version and cryptographic cipher usage.
    
- Second 32 bytes: A SHA-256 digest of the service’s TLS extension usage.
    

  

JARM fingerprints can be used for a number of purposes. As the [Salesforce blog post](https://engineering.salesforce.com/easily-identify-malicious-servers-on-the-internet-with-jarm-e095edac525a) says:

> - Quickly verify that all servers in a group have the same TLS configuration.
> - Group disparate servers on the internet by configuration, identifying that a server may belong to Google vs. Salesforce vs. Apple, for example.
> - Identify default applications or infrastructure.
> - Identify malware command and control infrastructure and other malicious servers on the Internet.

  

_So, how can JARM help us to hunt for a malicious infrastructure?_

Turns out, when malware deploys a TLS-enabled service, its configuration tend to be the same across multiple deployments. As such, fingerprinting one of these deployments allows us to escalate from one malicious C2 to hundreds, or sometimes thousands, of them.

As such, this tool can be proactively used for threat hunting and incident response.  

  

### Why Shodan?

[Shodan](https://www.shodan.io/ "https://www.shodan.io/") is a widely used tool that allows us to search for servers connected to the internet.

In addition, Shodan has integrated JARM and has generated JARM fingerprints for all TLS instances they have discovered. You can check them on a [Shodan facet](https://www.shodan.io/search/facet?query=http&facet=ssl.jarm "https://www.shodan.io/search/facet?query=http&facet=ssl.jarm"). 

So, once we find a suitable JARM fingerprint, Shodan is an excellent resource to locate additional servers corresponding to that fingerprint.

  

  

## Let’s Hunt

First, the path we are taking for this investigation:

1. Find the first node
2. Characterize the malware deployment
3. Hunt for the infrastructure
4. Analyze the results 
5. Pivot to new JARMs

  

### 1\. Finding the first node

There are various sources of threat intelligence that can give us a starting point. VirusTotal, ThreatFox and Twitter are all good sources.

Let’s look at VirusTotal user [drb\_ra](https://www.virustotal.com/gui/user/drb_ra/comments "https://www.virustotal.com/gui/user/drb_ra/comments") and check for recent Qakbot reports.

_Note_: look for reports on Qakbot served on port 443/8433 since we want to base our investigation on TLS-enabled service (HTTPS).

  

Found one from a few hours ago:

> Qakbot Found  
> C2: 183\[.\]87\[.\]163\[.\]165:**443**  
> Country: India (AS132220)  
> ASN: JPRDIGITAL-IN JPR Digital Pvt. Ltd.  
>   
> #c2 #Qakbot  

  

✅ 183.87.163\[.\]165 → [https://www.shodan.io/host/183.87.163.165](https://www.shodan.io/host/183.87.163.165)

  

### 2\. Characterizing the infrastructure

Now we need to analyze our first node to characterize the infrastructure we are hunting.

Pay attention to:

- The server and version
- Certificates
- Open Ports
- HTTP Response
- Vulnerabilities 

  

![](/assets/img/9b0400fa-69cc-4995-93cb-cb3845810944.png)  

From here we now know that the infrastructure we’re hunting probably will have a:

✅ Nginx 1.9.12 server running  

✅ Certified by `CN=epooohruieo.us` or similar   

✅ With HTTP response hash = `501510358`  

  

Still, we still need the JARM fingerprint:

- Click on “>\_Raw Data” and then “Expand All”:

![](/assets/img/01655744-9731-4137-86df-3a6ed1806ef4.png)  

- Search (`ctrl`+`f`) for “jarm” and under the field `ssl` you will find the `jarm` property:

![](/assets/img/3b655cba-a39f-4f5d-9089-8deab8dfb589.PNG)  

✅ JARM = `"21d14d00021d21d21c42d43d0000007abc6200da92c2a1b69c0a56366cbe21"`  

  

  

### 3\. Hunting the infrastructure

The characterization of the first node is important to understand how the threat actors have built their infrastructure, and based on that, how we can build our first hunting rule.

  

Our first hunting rule will take the HTTP response hash and the JARM fingerprint we have found:

```yaml
http.html_hash:501510358 ssl.jarm:"21d14d00021d21d21c42d43d0000007abc6200da92c2a1b69c0a56366cbe21"
```

  

This results in 88 QBots (see image below) all associated with the same JARM.

![](/assets/img/8826ec8a-6c4c-4399-976a-2e7558a39c1a.PNG)  

This result already gives a good visualization of this QBot infrastructure in particular, as we can see :

- the server and version used (the same as our first node)  
- origin countries they are operating from
- the certificates being used
- ports used to communicate with compromised hosts

  

### 4\. Analyzing the results

In addition, we can use [Shodan Facet](https://www.shodan.io/search/facet "https://www.shodan.io/search/facet") to visualize more similarities: by taking our query and select the property we want to analyze for all results.

  

For instance, selecting the property `vuln`  to see the vulnerabilities the Qbot servers have in common:

![](/assets/img/9bd197c6-d40b-4f08-9025-85594b22f46d.png)  

From our 88 results, we can see that 57 of those have exactly the same vulnerabilities in their deployment, helping conclude they all belong to the same configuration deployment.

  

Another example is the property `ssl.cert.issuer.cn` .

![](/assets/img/f824be6f-869c-4755-a86a-4009096eece5.png)  

We can see that only 4 issuers are shared by a pair of QBots. This helps us understand that this property would not be a good global detection rule, which is an important insight to take away too!

  

One more property that is always interesting to see is the `http.headers_hash` :

![](/assets/img/8727d45d-c215-48b0-997a-5e6a00c89ded.png)  

We can see that all 88 results have the same http headers, with `http.headers_hash` : `-1219739159`

We can repeat the process for any of the properties we want to analyze in more depth.  

  

### 5\. Finding other JARMs

Our hunting rule was based in one JARM, but it could be interesting to pivot from one JARM to other ones.

  

We can do this by modifying our hunting rule to query Shodan based on `http.html_hash` and `http.headers_hash` (found in the image above).

```yaml
http.html_hash:501510358 http.headers_hash:-1219739159
```

  

![](/assets/img/4c56c7c4-ddf1-4f2d-bd73-e1abfa2f2790.png)  

  

We can see that now we have eyes on 3 other JARMs corresponding to the same HTML and headers used for the infrastructure we found before.

We can now use these JARMs to continue following the infrastructure.

  

  

Happy Hunting 🕵️‍♀️

  

  

## To Read

About QBot

- [https://www.cisa.gov/sites/default/files/2023-02/202010221030\_qakbot\_tlpwhite.pdf](https://www.cisa.gov/sites/default/files/2023-02/202010221030_qakbot_tlpwhite.pdf)  
    

About why JARM is not foolproof

- [https://conference.hitb.org/hitbsecconf2021ams/materials/D2%20COMMSEC%20-%20JARM%20Randomizer%20Evading%20JARM%20Fingerprinting%20-%20Dagmawi%20Mulugeta.pdf](https://conference.hitb.org/hitbsecconf2021ams/materials/D2%20COMMSEC%20-%20JARM%20Randomizer%20Evading%20JARM%20Fingerprinting%20-%20Dagmawi%20Mulugeta.pdf)

