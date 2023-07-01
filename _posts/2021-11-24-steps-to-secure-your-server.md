---
layout: post
title: How would you secure a server?
subtitle: The most popular interview question, for good reason.
---
Servers are vital elements in organizations.

They hold confidential data and information, with the primary goal of providing data and services. As such, they are open for interaction with clients, so hackers are always on the lookout for open servers, and server vulnerabilities.

Here are 6 quick steps to secure your server (or to answer in your interview 😉).



## Secure Server Connectivity

### Step 1. Establish and Use a Secure Connection

SSH, or **S**ecure **Sh**ell, is an encrypted protocol used to administer and communicate with servers.

With SSH, any authentication is encrypted, however password-based logins nowadays can be easily brute-forced if password requirements are not rigorous. A more secure alternative is to use SSH keys to login to your server. The server will hold allowed users' public keys, and users will authenticate using their private key. Moreover, you can prohibit SSH connections with passwords altogether and only allow SSH connections with keys to be accepted.

By default, SSH uses port 22. Everyone, including hackers, knows this. Changing the port number is an easy way to reduce the chances of automatic reconnaissance mechanisms flagging your server for open SSH ports.



### Step 2. Set Up a Firewall

A firewall is a must-have to ensure that your server is safe. They control how services are exposed to the network, and what types of traffic are allowed in and out of a server.

You should define the specific services you need to remain open and which rules they require:

* External-facing public services, should be left open and available to the internet. For instance, your organization's public website.
* Private services should only be accessed by authorized accounts. For example, your organization's website where employees manage their time, vacations, sick days, etc.
* Internal services that should be made completely inaccessible to the network or the internet. For instance, a database that should only accept local connections.



### Step 3. Close Unused Ports

Attacks can come through open ports that you don’t even realize are open. It is important to remove or turn off unused network-facing services and close any other port that is not absolutely necessary.

Both Windows and Linux servers share a command - `netstat` - that can be used to determine which ports are listening.



## Server Access Management

### Step 4. Manage Users

Every server has a root/admin user who can execute any command. As such, it is widespread practice to remove remote access from the default root/administrator accounts. Moreover, the use of general named accounts, like "root" and "admin" should be discouraged.

Another way to manage server access is to limit what users/accounts can access. Not all employees should be given access to all the resources of your organization, and this also apply to users accessing servers. Access controls allows you to specify access privileges to directories, networks, files, and other server elements.

This approach to limiting permissions is known as the *principle of least privilege*, and is an excellent move in securing your servers.



### Step 5. Monitor Login Attempts

An Intrusion Prevention System (IPS) can monitor login attempts in order to protect the server against brute-force attacks. If the number of attempts exceeds the set value you define, the IPS can block the IP address either indefinitely or for a certain period of time.



### Step 6. Hide Server Information

Information is gold; Even knowing software versions can aid hackers when searching for weaknesses.

On a web server, it is a good idea to hide software version numbers of any software that is installed on the server, by removing this information from the HTTP header.

Also, most web servers are configured by default to display directory indexes when a user accesses a directory that lacks an index file. This exposure of the server directory listings could yield potentially confidential data. As such, disabling directory indexes is also a very good practice.



## Other best practices
As a general rule, the more security measures you have, the safer your server will become.
Other security measures to think of:
1. Update and upgrade software and OS regularly
2. Setup a regular file backup and restoration strategy
3. Setup password best practices
4. Use Virtual Private Networks (VPNs)
