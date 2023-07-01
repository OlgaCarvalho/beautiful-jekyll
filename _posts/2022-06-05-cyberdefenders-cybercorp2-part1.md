---
layout: post
title: CyberDefenders CyberCorp Case 2 - Write-Up Part 1
subtitle: Detailed solution (beware of spoilers!)
tags: [DFIR, tutorial, lab, windows]
---

The [CyberCorp Case 2](https://cyberdefenders.org/blueteam-ctf-challenges/75) challenge from [CyberDefenders](https://cyberdefenders.org/) is all about Threat Hunting 🕵️‍♀️, and as I recently started a Threat Hunter role I thought it would be a fun exercise.

This writeup will have multiple parts as I describe all my thought process and the steps I took for each and every question.

Let's get into it!


## The Challenge
After a cybersecurity incident, CyberCorp's management decided to purchase and deploy EDR (Endpoint Detection and Response) solution. EDR agents were installed on all workstations and servers and forwarded telemetry to a centralized Threat Hunting platform.

The company has also hired a team of highly qualified analysts to build a threat detection process using the Threat Hunting approach. You will have to try on the role of a threat hunter, who decided to verify the hypothesis about one of the attacker's persistence techniques.

Unfortunately, the hypothesis was confirmed, and a persistence technique was discovered on one host, which eventually became the starting point of the investigation.

By analyzing the EDR telemetry in the Threat Hunting platform, you will have to understand how the attacker compromised the network and what he managed to do with the obtained access.

## My Setup
* Link to download the virtual machine (VM) of the challenge: https://cyberdefenders.org/blueteam-ctf-challenges/75
* VM's password: `cyberpolygon`
* Update the VM's keyboard language to fit your own. For me it is "Portuguese".
* Access kibana from VM machine via http://127.0.0.1:5600 (or via the bookmark that already exists on Firefox)
  * Update time range for last 3 years to see when the logs start
  * We can see we have logs starting on Jun 21 (around 10:00) until Jun 22 (around 03:40), 2021.
  * Update time range:

  ```elm
  StartDate: 21-06-2022 10:00 AND EndDate: 22-06-2022 04:00
  ```

## Challenge Questions (1-5)
### 1. What is the name of the WMI Event Consumer used by the attacker?

> The Threat Hunting process usually starts with the analyst making a hypothesis about a possible compromise vector or techniques used by an attacker. In this scenario, your initial hypothesis is as follows: "The attacker used the WMI subscription mechanism to obtain persistence within the infrastructure". Verify this hypothesis and find the name of the WMI Event Consumer used by the attacker to maintain his foothold.

Windows Management Instrumentation (WMI) Event Subscription is a popular technique to establish persistence on an endpoint. Typically persistence via WMI event subscription requires creation of WMI objects, to store the payload (or the arbitrary command) and to specify the event that will trigger the payload.

```elixir
Category: WMI command execution detected
```

We can further filter the results by:

```elixir
*subscription*
```

And we have:

| Date                        | rule_name                    | wmi_command                                                  |
| --------------------------- | ---------------------------- | ------------------------------------------------------------ |
| Jun 21, 2021 @ 23:25:50.000 | WMI - Consumers manipulation | `Start IWbemServices::PutInstance - root\subscription : __EventFilter.Name="PowerControl Event"` |
| Jun 21, 2021 @ 23:25:50.000 | WMI - Consumers manipulation | `Start IWbemServices::PutInstance - root\subscription : CommandLineEventConsumer.Name="PowerControl Consumer"` |
| Jun 21, 2021 @ 23:25:50.000 | WMI - Consumers manipulation | `Start IWbemServices::PutInstance - root\subscription : __FilterToConsumerBinding.Consumer="CommandLineEventConsumer.Name=\"PowerControl Consumer\"",Filter="__EventFilter.Name=\"PowerControl Event\""` |

We can see a new WMI object being created with:

* **_EventFilter** name = `PowerControl Event`. This is the trigger (for instance, new process, failed logon etc.)
* **EventConsumer** name = `PowerControl Consumer`. This is the action to be performed (for instance, execute payload etc.)
* **__FilterToConsumerBinding**. This binds the Filter and Consumer Classes.



Let's remove all our filters and look more into `PowerControl Consumer`:

```elixir
*PowerControl*
```

Analyzing the 7 results we got we can see that:

* >  WMI filter 'powercontrol event'was created by user 🧑‍💻️'cybercorp\john.goldberg'

* The EventFilter `PowerControl Event` is to be triggered after 10 seconds the user logs on (see trigger below).
  ```powershell
  Query = "SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE TargetInstance ISA 'Win32_LoggedOnUser'";
  ```

* The EventConsumer will then execute the command bellow, which executes a PowerShell script (`.ps1`).
  ```powershell
  powershell.exe -noP -ep bypass iex -c \"('C:\\Users\\john.goldberg\\AppData\\Roaming\\Microsoft\\Office\\MSO1033.ps1')\"
  ```


✅ powercontrol consumer



### 2. What is the process that installed the WMI subscription?

> In the previous step, you looked for traces of the attacker's persistence in the compromised system through a WMI subscription mechanism. Now find the process that installed the WMI subscription. Answer the question by specifying the PID of that process and the name of its executable file, separated by a comma without spaces.

The next step is to determine who/what has installed this WMI subscription?

To do this we need to look at what happened prior to the WMI subscription. And so, we need to set the date range to:

```elixir
StartDate: 21-06-2022 at 23:00 AND EndDate: 21-06-2022 23:25:50
```

After analyzing some logs I see an immediate way that can filter the results by excluding whitelisted hashes:

```elixir
NOT enrich.ti.file_md5.categories: Whitelist
```

Browsing trough the remaining results a suspicious activity grabs my attention:

| Date                        | event_type | event_category | event_log_name        | proc_p_cmdline          | proc_file_path                                               | file_path                                                    | proc_id |
| --------------------------- | ---------- | -------------- | --------------------- | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------- |
| Jun 21, 2021 @ 23:25:43.000 | FileCreate | INFO           | BiZone-TH/Operational | C:\Windows\Explorer.EXE | C:\Program Files (x86)\Microsoft Office\Office16\WINWORD.EXE | C:\Users\john.goldberg\AppData\Roaming\Microsoft\Office\Recent\188.135.15.49.url | 5772    |

Why is Word creating a file that looks like an URL or remote address? 🤔

Let's investigate further.

##### Investigating Winword activity

To further filter the results I added another exclusion:

```elixir
NOT event_type: ImageLoad
```

From the result, another log grabbed my attention:

| Date                        | event_type        | event_category | event_log_name        | proc_file_path                                               | net_conn_direction | net_dst_ipv4  | Port |
| --------------------------- | ----------------- | -------------- | --------------------- | ------------------------------------------------------------ | ------------------ | ------------- | ---- |
| Jun 21, 2021 @ 23:25:38.000 | NetworkConnection | INFO           | BiZone-TH/Operational | C:\Program Files (x86)\Microsoft Office\Office16\WINWORD.EXE | Outbound           | 188.135.15.49 | 80   |

Again, why is Word initiating a network connection to an external IP?

From this log we can look at the `enrich.*` fields that look like to be adding more context to the logs.

![image-20220518183407003](/assets/img/image-20220518183407003.png)

Word communication with and IP in the Middle East considered as a malicious domain by ESET? Indeed this looks to be a malicious activity.

As such, the answer is...

✅ 5772,winword.exe



### 3. Which file was extracted and opened from the archive?

> The process described in the previous question was used to open a file extracted from the archive that user received by email. Specify a SHA256 hash of the file extracted and opened from the archive.

The process from the previous question is `winword.exe` (`proc_id:5772`) and it was used to open a file (`event_type: FileOpen`) extracted from the archive (zip maybe?). Keeping the same time range from last question we can query the logs using:

```elixir
proc_id:5772 AND event_type: FileOpen AND *zip*
```

We have only one result:

| Date                        | proc_id | event_type | event_category | proc_file_path                                               | file_path                                                    | file_sha256                                                  |
| --------------------------- | ------- | ---------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Jun 21, 2021 @ 23:25:22.000 | 5772    | FileOpen   | INFO           | C:\Program Files (x86)\Microsoft Office\Office16\WINWORD.EXE | C:\Users\john.goldberg\AppData\Local\Temp\Temp1_Report.zip\Market Forecast EMEA.docx | 54dabbd0a47f5ef839de9183978b9b755c248c8ad7a35aff3fe537990ffb3501 |

It looks like a DOCx file "Market Forecast EMEA.docx" was extracted from an archive named "Temp1_Report.zip".

Expanding the log we have our answer:

✅ 54dabbd0a47f5ef839de9183978b9b755c248c8ad7a35aff3fe537990ffb3501



### 4. From where was the file containing the malicious code downloaded?

> The file mentioned in question 3, is not malicious in and of itself, but when it is opened, another file is downloaded from the Internet that already contains the malicious code. Answer the question by specifying the address, from which this file was downloaded, and the SHA256 hash of the downloaded file, separated by commas without spaces.

The first thing we can do is to readjust the time range to start from when the zip was opened, and ending when the WMI subscription happen.

```elixir
StartDate: 21-06-2022 23:25:22 AND EndDate: 21-06-2022 23:25:50
```

Now we have a much smaller sample of logs to go through.

Next, we want to determine a possible sequence of events such as: Network Connection > File Created (file downloaded) > File Open (to execute the malicious code and subscribe a malicious WMI object). Let's do the following query:

```elixir
event_type: NetworkConnection OR event_type: FileCreate OR event_type: FileOpen
```

From Question 3. we already know there was an outbound communication from the machine to a malicious domain on `188.135.15.49:80`. It is expected that the malicious payload has come from here.

![image-20220518194754074](/assets/img/image-20220518194754074.png)

Looking at the logs above, we see a NetworkConnection event to our suspicious domain and then a file named `FontStyles[1].dotm` (with hash = 65df8039cbd1b3fb40a1cc9198c2ba314dd38ff7d301ee475327d438346d96af) being opened at `AppData\Local\Microsoft\Windows\INetCache\IE` (a Windows cache that contains Internet files).

✅ 188.135.15.49, 65df8039cbd1b3fb40a1cc9198c2ba314dd38ff7d301ee475327d438346d96af



### 5. What was the operating system component whose functionality was used by the attacker to download files from the Internet?

> The malicious code from the file, mentioned in question 4, directly installed a WMI subscription, which we started our hunting with, and also downloaded several files from the Internet to the compromised host. For file downloading, the attacker used a tricky technique that gave him the opportunity to hide the real process, which initiated the corresponding network activity. Specify the SHA256 hash of the operating system component whose functionality was used by the attacker to download files from the Internet.

First let's adjust the time range (always the first step we can make) to focus on what happened after the WMI subscription.

```elixir
StartDate: 21-06-2022 at 23:25:50 AND EndDate: 21-06-2022 23:40:00
```

We have too many logs to go trough, so let's try to filter them according to its severity:

```elixir
enrich.ioa.max_severity: exists
```

Looking at what happened immediately after the WMI subscription, at 23:25:57.000, we have 3 events occurring in the same second:

![image-20220519103045533](/assets/img/image-20220519103045533.png)

Since the logs do not have a millisecond sensibility (notice all the dates end with `.000`), we cannot know for sure the sequence of events that happen in the same second.

In this case, we see `winword.exe` loading a DLL - not really suspicious by itself - followed by DNS request(s) (hummmm 🤔).

This DLL is named `ieproxy.dll` .

> The IE ActiveX Interface Marshaling Library (ieproxy. dll) is a part of Windows Internet Explorer and a system file. 

Researching about the usage of this DLL in attack techniques, an [article from CyberPolygon](https://cyberpolygon.com/materials/hunting-for-advanced-tactics-techniques-and-procedures-ttps/) pops up referring a technique where Internet Explorer COM objects are used in scripts or macros to interact with Internet resources.

It also adds that:

> For the client application to use this COM object (for example, from a macro), the ieproxy.dll library must be loaded by the application process. The loading of this library by an unusual application (such as Microsoft Office or script interpreters) can be indicative of the attacker’s attempt to employ the respective technique. 

So it appears that `winword.exe` loaded the library `iexproxy.dll` which allowed it to use `iexplore.exe` in downloading files from the Internet.

Taking the hash of this DLL we have the answer for the 5th question:

✅ 5516176cd0f4204ef8cf563c1dd6b3991b134d17eef2cc5e62e7f6c7aadfbb37


That's it for now 😉

Part 2 will come soon!