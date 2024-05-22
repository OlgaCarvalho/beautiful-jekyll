---
layout: post
title: Cyber lab for high-school students
subtitle: Engineers for 1 Day
tags: [lab, review, tutorial]
---

Today I participated in a public initiative called [Engineers for a day](https://engenheirasporumdia.pt) that promotes to high-schools students the option for engineering and technologies, deconstructing the idea that these are male domains, by having **only women specialists** to present different technological and engineering industries to the students.

  

An initiative of the Portuguese Government that is coordinated by the:
* **Commission for Citizenship and Gender Equality** (CIG)
* **INCoDe.2030**,  

in conjunction with the:  
* **Portuguese Association for Diversity and Inclusion** (APPDI), 
* university **Instituto Superior Técnico** (my alma mater) 
* **Engineers' Order**,

and it involves a network of:
* **101 Partner entities** (15 Of which municipalities), 
* **62 Primary and secondary schools**
* **23 Higher education institutions**.

  
One of these partners is the company I work for, [Noesis](https://noesis.pt/), that invited me to present my area of expertise: Cybersecurity.

  

They asked me to put together a proposal of some practical presentation, to which I thought - "Easy! there’s no lack of practical labs in cybersecurity". HOWEVER... the students wouldn’t have computers, not even pen and paper, which did complicate things a _little bit_.

  

But, in the end, we did have amazing feedback from the students, and even the teachers, that were so worried as most appeared in [haveibeenpwned.com](https://haveibeenpwned.com "https://haveibeenpwned.com") (a little tool useful to scare people into protecting their account 🙊).

  

So, after a quick presentation of cybersecurity and what is working in a SOC, here’s the two labs we presented in this event.

## Lab1. Find The Red Flag 🚩

- Teams of 4/5 students
- 6 phishing emails
- Goal: find all the red flags
- Prize: a literal red flag 🚩 + a water bottle

Ideas of what to introduce in the examples:

- Source email address from a popular company but using gmail
- Link that changes a character “o” to the digit “0”
- Attachments with unexpected invoice
- Outlook URL but \* different \*
- (very) wrong orthography
- Urgency for need of action
    - “Your account is being closed. Log in now!”
- Typosquatting
    - E.g. [netflix.com](https://netflix.com "https://netflix.com") → [netflx.com](https://netflx.com "https://netflx.com")
- Homograph/homoglyph attack
    - Use “a” from the Cyrillic alphabet

  
*Review:* I was worried that too many teams would win (and not have enough prizes) however the opposite happened, and no teams found all the red flags. As such, the prizes were given to unique or first findings to keep things interesting. 😊
  

### What to do?

1. Always check sender address
2. Beware of attachments
3. Use [VirusTotal.com](https://VirusTotal.com "https://VirusTotal.com") to validate URLs

  

  

## Lab2. Steal Passwords 🔓

### Activities

#### 1\. Steal Chrome Passwords

- Simple overview how Chrome stores passwords and encryption key locally on our computers
- On a Windows VM pre-installed with the repo 
 [https://github.com/ohyicong/decrypt-chrome-passwords](https://github.com/ohyicong/decrypt-chrome-passwords):
    - Ask voluntaries for “victims” and have them store passwords in Chrome
    - Ask voluntary for “hacker” and have them run the decryptor

  

#### 2\. Crack Passwords

- Simple overview of a Dictionary attack vs. a Brute-force attack and times to decode a password according to the number of characters.
- On a Kali VM pre-prepared with a file “Top Secret” and JohnTheRipper ready to run:
    - Ask a “victim” student to create a ZIP archive protected with a (simple) password. 
    *Note:* It must really be ***simple*** so JohnTheRipper does not take a long time
    - Ask a “hacker” student to run JohnTheRipper to crack the ZIP file



### What hackers do with our stolen credentials?

- Showcase Darkweb and Telegram posts


### What to do?

1. Check [haveibeenpwned.com](https://haveibeenpwned.com "https://haveibeenpwned.com")
2. Use strong passwords
3. Use a password manager
4. Use multi factor authentication


And voilá!

It was a really fun event, where we got to present a bit about, not only the **importance** of cybersecurity, but how truly **fun** it can be 😁