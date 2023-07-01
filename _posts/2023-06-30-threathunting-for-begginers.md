---
layout: post
title: Threat Hunting for Beginners
subtitle: 1-year review as a Threat Hunter
tags: [threathunting, review, learning]
---
 
This month caught my 1-year anniversary as a Threat Hunter (and my actually birthday 😁) so I thought it would be a nice idea to share what I have learned this year about threat hunting.
  

## From SOC to Threat Hunter (my story)

When my current company proposed me this position, I actually didn’t even know “threat hunting” was a thing 😅.

But since I am not one to turn away from a challenge I gladly accepted!

I promptly started reading and studying about Threat Hunting and, to my dismay, there was actually nothing really concrete about this position/activity. There are some general guidelines but nothing to really help a beginner to get in action.

So, my obvious next step: reach out to my network to try find Threat Hunters.

Nothing. 😬

All right... let’s expand the search. Fortunately, my team lead had some connections to professionals that dabble in threat hunting, so I got some interesting (but, again, general) feedback.

This made me understand that, not only I was to start a new role, but I would have to write-out, and implement from the beginning, the process “Threat Hunting” in regards to the client I would be working for.

  

With this goal in mind, I got to work on a process of what “Threat Hunting” would look like in the client’s context and what they could expect from me and this activity.

During this time, my impostor syndrome reached an all time high, since “who am I to define what Threat Hunting is and how it should be done!?”😶 . But at the same time, it was a good opportunity to practice being patient with myself and “trust the process” (pun intended 😋).

  

So, with my experience as a security researcher, SOC analyst and incident responder, and insights given by other professionals, I share here my Threat Hunting guide for beginners.

  
## What is Threat Hunting?

First things first:

- Threat hunting is the practice of **proactively** searching for cyber threats that may be lurking undetected in a network.
- The threat hunter is responsible for monitoring security patterns to identify, isolate, and detect the threats **before** attackers tend to exploit them.
- It is **not** an automated process; it’s driven by **questions** and **hypotheses** that lead to **investigations**. For instance:
    

> \[**Question**\] “Is data exfiltration happening?”  
> \[**Hypothesis**\] “If there is data exfiltration happening, it is most likely going on through \<this\> part of the network.”  
> \[**Investigation**\]   
> 
> - What protocols attackers uses to exfiltrate data?
> - What would that activity look like in the logs? Ex: stealing data by FTPing it straight out OR using HTTP OR DNS exfiltration

- In the end, the main goal of threat hunting is to decrease the gap between initial compromise by an attacker (T=1) and the detection of that attacker in the environment (T=2).

![](/assets/img/th-1-timeline.png)  
<p style="text-align: center;"><i>Defensive activity timeline</i></p>
  
  

## Threat Hunting Phases

Basing off of the [Targeted Hunting integrating Threat Intelligence (TaHiTI) framework by FI-ISAC](https://www.betaalvereniging.nl/en/safety/tahiti/ "https://www.betaalvereniging.nl/en/safety/tahiti/"), a good rule is to divide this activity in three main phases. I propose the following:

1\. **Trigger**

A trigger points threat hunters to a specific system or area of the network for further investigation. 

A trigger can take various forms, for instance (1) a malicious activity identified by detection systems (AVs, EDR, etc.); (2) a new TTP using fileless malware to evade defense mechanisms was id-ed; or (3) a new ATP targeting a specific industry of interest was identified.  

2\. **Hunting**

During the hunting phase, the threat hunter uses all technology available to take a **deep dive** into potential malicious compromise of a system.

3\. **Output**

The output phase involves (1) communicating relevant intelligence to operations and security teams so they can respond to the incident and mitigate threats, (2) generate threat intelligence, (3) update use-cases and other relevant recommendations.

  

The image below illustrates this process and what are some of the inputs, action points and results for each phase, respectively.

![](/assets/img/th-2-phases.png)
<p style="text-align: center;"><i>Threat Hunting phases - Trigger, Hunting and Output</i></p>    

One interesting thing to point out is that any output of a hunt can (and 9/10 will) become a trigger to another hunt. Yes, it is an endless job.

  

### 1\. Trigger

As you can see in the image above, a trigger for a hunt may come from multiple different sources, by multiple different categories.

One of the main triggers come from **threat intelligence**.

Since my company does not have a threat intelligence team, I accumulate the roles of Threat Intelligence and Threat Hunting. So, after spending some months collecting and analyzing different sources of threat intelligence, I ended up with 5 main intel categories that can help you make sure you have visibility in all areas: (1) Specific targets, (2) CVEs, (3) IOCs, (4) Threat Actors and (5) News.

In the image below I also share some OSINT sources examples for each category.

![](/assets/img/th-3-triggers.png)    
<p style="text-align: center;"><i>Main categories of triggers, with source examples</i></p>
  

Also, check out the [awesome-threat-intelligence](https://github.com/hslatman/awesome-threat-intelligence "https://github.com/hslatman/awesome-threat-intelligence") repo for more great sources of threat intelligence.  

  

### 2\. Hunting

To be able to perform a thorough investigation, the threat hunter needs:

- Tools and logs to deep dive into abnormal behavior
- Tools and logs that allow pivoting and filtering

When implementing a threat hunting process it is important to identify all available technology, tools and logs, and also, those that could be missing to be able to further investigate a trigger.

Take time to really understand what you can do with the tools and how you can access/analyze the logs.

  

### 3\. Output

The final phase is the documentation of results and handover to other processes.

![](/assets/img/th-4-output.png)
<p style="text-align: center;"><i>Processes triggered by threat hunting investigations, according to the TaHiTI framework</i></p>    

According to the TaHiTI framework, the threat hunter hands over their results to 5 processes: security incident response, security monitoring, threat intelligence, vulnerability management and others.

The image above gives good examples of what action items and outputs this activity can lead to.

  

## Final thoughts

Even though I fell in love with this job, I recognize there are some particular aspects that may not be for everyone.

Coming from a SOC and incident responder for 30+ customers, I was used to clear processes and procedures, and quick/big wins. 

Threat hunting for only one client rarely provides this sense of accomplishment (which is a good signal! since you are not finding clear signs of compromise) and is something you have to get accustomed to.

But what this means is, this job is perfect for someone that loves to **read** a lot everyday (news, threat reports, malware analysis, etc.); that loves to **write** reports to give visibility on your activity and findings for your team, and that **never gives up** trying to find the needle in in the haystack (while hoping it isn’t there!).

  

Hopefully this guide has given you the confidence to dive into threat hunting, and I will keep sharing the awesome things I learn as a Threat Hunter.

  

Happy Hunting 🕵️‍♀️