---
layout: post
title: How to investigate a web attack
subtitle: Using pandas
tags: [DFIR, tutorial]
---

In this post I showcase samples of useful search queries when investigating a web attack.

## Time for school

### What is a web attack?

A web attack targets vulnerabilities in websites to gain unauthorized access, obtain confidential information, introduce malicious content, or alter the website's content.  

The Open Web Application Security Project, or OWASP, is a popular project dedicated to web application security. They report on the most critical risks concerning web application security, with their [OWASP Top 10](https://owasp.org/www-project-top-ten/ "https://owasp.org/www-project-top-ten/") report focusing on the top 10 security risks.

  

In this post we are going to see how we can identify some of the most common and popular web attacks:

- **SQL Injection (SQLi)**

SQLi is an attack where a web application directly includes unsanitized data provided by the user in SQL queries.  

  

- **Cross-site scripting (XSS)**

XSS is a type of injection based web security vulnerability that enables malicious code to be run.

  

- **Command Injection**

Command Injection attacks happen when the data received from a user is not sanitized and is directly transmitted to the operating system shell.  

  

- **Insecure Direct Object Reference (IDOR)**

IDOR attacks targets the lack (or misconfiguration) of an authorization mechanism. It essentially enables an attacker to gain access to an object that belongs to another.  

Among the highest web application vulnerability security risks published in the 2021 OWASP, IDOR, or “Broken Access Control”, takes 1s place 🥇. 

  

- **Local File Inclusion (LFI) & Remote File Inclusion (RFI)**

File inclusion is a security vulnerability that occurs when a file is included without sanitizing the data obtained from a user. 

On LFI, the file that is intended to be included is on the same web server that the web application is hosted on.

On RFI, the file that is intended to be included is hosted on a different server.

  

### What is pandas?

`pandas` is a software [library](https://pandas.pydata.org/) written for Python, that focuses on data manipulation and analysis. 

In particular, it offers data structures and operations for manipulating numerical tables and time series.

This library allows us to take a large data set (for instance, access logs from a web server) and investigate it using filters as search queries.


## What we need

- Python
- `pandas` library (`pip install pandas`)
- Apache access log

## Tutorial

This tutorial will follow 3 steps:

- **Step 1. Parse server logs**
- **Step 2. Investigate logs**
- **Step 3. Report results**



### Step 1. Parse server logs

We first need to retrieve our server’s log and identify its format.

Check [here](https://www.sumologic.com/blog/apache-access-log/ "https://www.sumologic.com/blog/apache-access-log/") the possible formats you can encounter when analyzing Apache logs.

  

In this case, my server’s logs are being stored with a **Combined Log Format**. Like so:

```
127.0.0.1 - Olga [10/Dec/2019:13:55:36 -0100] "GET /server-status HTTP/1.1" 200 2326 "http://localhost/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
```

  

Where:

- `127.0.0.1` → IP address of the client that made the request;
- `⁠-`⁠ → The hyphen defining the second field in the log file is the identity of the client. This field is often returned as a hyphen and Apache’s HTTP server documentation recommends that this particular field not be relied upon except in the case of a controlled internal network.  
- `Olga` →  userid of the person requesting the resource;⁠⁠
- `[10/Dec/2019:13:55:36 -0700]` → date, time and time zone of the request;
- `"GET /server-status HTTP/1.1"`  → request type and resource being requested;
- `200`  → server HTTP response status code;
- `2326`  → size of the object returned to the client.
- `"http://localhost/"` → This is the HTTP referrer, which represents the address from which the request for the resource originated.
- `"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"` → This is the User Agent, which identifies information about the browser that the client is using to access the resource.

  

Taking the log format into account, we want to parse the logs in order to have them organize like the following table:

| ip  | clientid | userid | datetime | timezone | request | url | response | size | referrer | useragent |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 127.0.0.1<br> | \-⁠<br> | Olga<br> | 10/Dec/2019 13:55:36<br> | \-0700<br> | GET<br> | /server-status HTTP/1.1"<br> | 200 <br> | 2326 <br> | "[http://localhost/](http://localhost/)"<br> | "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"<br> |

  

Let’s see how we can do this 👇

  

1\. Read logs into a dataframe

```python
import pandas as pd
df = pd.read_csv(r"your-path-to-file\apache access logs.txt", sep='\"', engine='python', header=None, skipinitialspace = True)
```


2\. Remove leading and trailing whitespaces

```python
for column in df:
    df[column] = df[column].str.strip()
```

  

3\. Reformat first column:

```python
# create new dataframe from spliting the original dataframe
logs = df[0].str.split(' ', expand=True)

# remove '[' and ']'
logs[3] = logs[3].str.replace('[', '', regex=False)
logs[4] = logs[4].str.replace(']', '', regex=False)

# rename columns
logs.rename(columns = {0:'ip', 1:'clientid', 2:'userid', 3:'datetime', 4:'timezone'}, inplace=True)

# change date to date object
logs['datetime'] = pd.to_datetime(logs['datetime'], format='%d/%b/%Y:%H:%M:%S')
```

  

4\. Reformat request column:

```python
logs[['request', 'url']] = df[1].str.split(' ', n=1, expand=True)
```

  

5\. Divide http response from size

```python
logs[['response', 'size']] = df[2].str.split(' ', n=1, expand=True)
```

  

6\. Add referrer and user-agent columns

```python
logs['referrer'] = df[3]
logs['useragent'] = df[4]
```

  

7\. Delete original dataframe

```python
del df
```

  

Now we have a dataframe named `⁠logs`⁠ with clean values.


### Step 2. Investigate results

#### Get the most relevant results

First, if the attack came from the internet, disregard internal IPs:

```python
maskIP = ~logs.ip.str.contains("10.|127.0.0.1", na=False)
```

Also, we can disregard `.gif` and `.ico`⁠ resources:  

```python
maskICO = ~logs.request.str.contains(".gif|.ico", na=False)
```


#### Look for SQLi payloads

Look for common SQL terms and the symbols. 

You can check some frequently used SQL Injection payloads [here](https://github.com/payloadbox/sql-injection-payload-list). 

  

- SQL Mask:

```python
maskSQL = logs.url.str.contains("%27|SELECT|UNION|SLEEP|AND|OR|CHR|INSERT|WHERE|EXEC", case=False, na=False) 
```

Note: the symbol "`'`" commonly seen in SQLi payloads can appear in percent encoding, as `⁠%27`⁠.

  

Apply masks:  

```python
logs[maskIP & maskICO & maskSQL].sort_values(by="datetime")
```

If this returns results, because these are specific words that belong to SQL, we can determine that we are face to face with a SQL Injection attack.

  

  

#### Look for XSS payloads

Look for keywords which are commonly used in XSS payloads, such as “`alert`” and “`script`” .  

You can examine some frequently used payloads [here](https://github.com/payloadbox/xss-payload-list).   

  

- XSS Mask:

```python
maskXSS = logs.url.str.contains("script|prompt|console.log|alert|confirm|document.write|String.fromCharCode", case=False, na=False)
```

  

Apply masks:  

```python
logs[maskIP & maskXSS].sort_values(by="datetime").tail(60)
```

If this returns results, because these are specific words that belong to typical XSS attacks, we can determine that we are face to face with a XSS attack.  

  

  

#### Look for Command Injection payloads

Look for keywords related to the terminal language, such as:

| Linux | Windows |
| --- | --- |
| `whoami`<br> | `whoami`<br> |
| `ls`<br> | `ver` |
| `cp`<br> | `ipconfig` |
| `cat`<br> | `netstat` |
| `type`<br> | `tasklist` |

  
 
- Command Injection mask:

```python
maskCMD = logs.useragent.str.contains("dir|ls|cp|cat|type|etc|echo|whoami|pwd", case=False, na=False) | logs.url.str.contains("dir|ls|cp|cat|type|etc|echo|whoami|pwd", case=False, na=False)
```

  

Apply masks:  

```python
logs[maskIP & maskCMD].sort_values(by="datetime").tail(60)
```

  

  

#### Look for an IDOR attack

IDOR attacks are more difficult to detect than other attacks because they do not have certain payloads, such as SQL Injection and XSS attacks usually have.  

One approach is for us to look for the IP addresses that connected most to the webserver

  

##### Get the most relevant IP addresses

Search for the IP addresses that connected most to the webserver (remember our `maskIP` to ignore internal IPs):  

```python
logs[maskIP].groupby(["ip"]).count().sort_values(by="datetime").tail(60)
```

  

Take the top 5 source IPs and check what they did. For each do:

```python
logs[logs.ip == "<YOUR_IP>"].sort_values(by="datetime").head(60)
```

  

##### Get the most relevant requests by the most relevant IP addresses

With this mask built, now we can see the most relevant IPs and associated relevant requests:

```python
logs[maskIP & maskICO].groupby(["ip", "url"]).count().sort_values(by="datetime").tail(60)
```

  

  

#### Look for an RFI/LFI attack

Some things we can look for when investigating an LFI attack:

1. **Target files**. Attackers will exploit LFI vulnerabilities by manipulating the file location parameter, in an effort to display the contents of target files on a UNIX / Linux based system, such as `⁠/etc/passwd`.
2. **Directory traversal.** Because attackers do not known what directory the web application is in, they will try to reach the “root” directory using “`../`”.
3. **Null byte injection**. This bypasses application filtering within web applications by adding URL encoded “Null bytes” such as `%00`.
4. LFI Wrapper rot13 and base64 - [](php://filter)`php://filter` . This wrapper allows an attacker include local files and encodes the output (`base64` or `rot13`). Therefore, any base64 output will need to be decoded to reveal the contents.

  

- LFI Mask:

```python
maskLFI = logs.url.str.contains("etc/|passwd|%00|%2500|../|base64|rot13|proc/", case=False, na=False)
```

  

- RFI Mask:

```python
maskRFI = logs.url.str.contains("http:|base64|php:expect:", case=False, na=False)
```

  
Apply both masks:  

```python
logs[maskIP & (maskLFI | maskRFI)].sort_values(by="datetime").tail(60)
```


## Step 3. Report

When we find an evidence of malicious activity, we should include it in our report.

To see the full results in the python console we can simply use the `⁠print`⁠ function:

```python
print(*logs[YOUR-MASK].sort_values(by="datetime").to_csv().split("\n"), sep="\n")
```

  

With pandas, we can directly output a search result to a csv file.

```python
logs[YOUR-MASK].sort_values(by="datetime").to_csv("report.csv", mode='w')
```

_Tip_: use `⁠mode="a+"`⁠ to append multiple results together in the same file.

  

Finally, what should be included in our report?

- The attacker’s IP address
- The date the attack started and ended
- If the attack was successful. If so:
    - The type of the attack
    - If the attack was performed by an automated tool (check time of requests and `UserAgent`)

  

  

And voilà!

Happy Hunting 🕵️‍♀️