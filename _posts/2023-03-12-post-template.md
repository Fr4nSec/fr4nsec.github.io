---
layout: post
author: Fr4nSec
tags: [Windows, SMB,]
---

CTF challenge having a Windows target. SMB is key to solve it.

## Objectives

- Obtain the user flag
- Obtain the system flag (root)

## Recon

Let's scan the target IP address..

```
nmap -p- -sS --min-rate 5000 -n -P <IP Address>
```

![screenshot1](/images/roasted.jpeg)


## Exploitation

We can use Hydra to brute force HTTP-POST-FORM:


```
hydra -l <user> -P <password list> <IP Address> ssh
```



## Privilege escalation

Let's check if sudo has any additional configuration..

```
sudo cat /root/root.txt
```
