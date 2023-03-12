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
nmap -p- -sS --min-rate 5000 -n -Pn 10.10.216.196

```
![scan1](/images/scan1.jpeg)

Once we have a list of open ports, let's scan those ports more in depth.

```
nmap -p 53,135,139,389,445,464,636,3268,5985,9389 -sCV 10.10.216.196 > PortScan
cat PortScan

```

![scan2](/images/scan2.jpeg)


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
