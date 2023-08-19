---
layout: post
title: TweetDeck for Security Researchers
subtitle: Curated threat intelligence via Twitter
tags: [threathunting, threatintel, OSINT]
---

On Jun 12 I got the sad news from [https://twitter.com/simonbyte](https://twitter.com/simonbyte) that, because of new restrictions to Twitter’s API, my favorite web tool to daily monitor vulnerabilities - [CVETrends](https://cvetrends.com/) - is down.

While waiting to see if there will be a plan to restore the tool, I started to look for an alternative, but unfortunately nothing really compares.

However, one idea that stood out is to use TweetDeck, a Twitter tool that allows us to create multiple dashboards to monitor specific keywords.

Immediately I can see multiple use-cases for a tool like this:

- Monitor your company’s keywords; or, as a MSSPs, use this data to monitor your customer base
- Monitor news about your (and your client’s) supply chain (vendors, products)
- Monitor news about zero-days and ransomware attacks
- and many others

  
In TweetDeck you can have multiple decks, and in each deck you can have multiple columns with your searches.

Below I share some examples of my decks and searches that I use and that you can quickly set up.  

  

## My Setup

My two main decks are named “Home” and “CVEs”. The first includes general searches and the second includes searches related to CVEs.

  

### Deck 1 - “Home”

Column 1 - “Home” that shows you tweets from the accounts you follow

```
filter:follows
```

  

Column 2 - “Portugal” to monitor cyber security news in my country, Portugal. You can adapt this search respective to your country.

```
("ataque informático" OR "ciberataque" OR "ransomware" OR "ciberattack" OR "ciber attack") AND ("portugal" OR "português" OR "portuguesa" OR "portuguese")
```

  

Column 3 - “<your-industry> to follow news related to your industry vertical

```
("attack" OR "cyberattack" OR "ransomware") AND ("<your-industry>")
```

  

Column 4 - “Outages” to follow big outages or system crashes

```
(
 ("computer system" OR "computers" OR "systems") AND 
 (
   ("outage" OR "failure" OR "offline" OR "down") OR 
   ("Breached" OR "hacked" OR "compromise" OR "long line" OR "waiting" OR "globally" OR "nationwide") OR 
   ("Hacked" OR "Breached" OR "compromised") OR 
   ("dump" OR "exposed records")
  )
) OR from:DownDetector
```



### Deck 2 - “CVEs”

Column 1 - “CVE Trends” to follow CVE news in the past 7 days

```sql
(cve OR #CVE) min_faves:10 min_retweets:5 within_time:7d lang:en
```

  

Column 2 - “Vendors & Products” to monitor vulnerabilities in your supply chain. For instance:

```
(vulnerability OR #cve OR cve) AND (firefox OR chrome OR IE OR "internet explorer")
```

  

Column 3 - “0-days” which is self-explanatory

```
#0day OR #zeroday
```

  

Column 4 - “Specific CVEs” to follow any specific CVE activity

```
<CVE> OR "<cve-name>"
```

  


## Other samples searches

- Monitoring for Windows 10 vulnerabilities.

```sql
"Windows 10" AND (vulnerability OR #cve)
```

- Shodan Safari

```
(#shodan OR shodan.com) OR (#zoomeye OR zoomeye.org) OR (censys.io AND -FROM:censysio)
```

- Sandbox Runs

```sql
virustotal.com OR app.any.run OR alienvault
```

- Malware and Red Team

```sql
(#offsec OR #malware OR #redteam OR #pentest OR #hacking OR #bugbounty) AND (min_faves:5 OR min_retweets:5)
```

- Blue team

```sql
(#DFIR OR "digital forensics" OR #ThreatHunting OR #blueteam OR #threatintel OR #threatintelligence) AND (min_faves:10 or min_retweets:5)
```

- Fresh IOCs

```
from:drb_ra include:nativeretweets -filter:replies
```

- Live ransomware updates

```
from:Ransom_Diary OR from:RansomwareLeaks OR from:FalconFeedsio include:nativeretweets -filter:replies
```

  

  

Happy Hunting 🕵️‍♀️