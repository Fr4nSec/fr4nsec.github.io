---
layout: post
author: Fr4nSec
tags: [Windows, SMB,]
---

CTF challenge having a Windows target. SMB is key to solve it.

## Objectives

- User flag
- Root flag

## Recon

Let's scan the target IP address..

```
nmap -p- -sS --min-rate 5000 -n -P <IP Address>
```

![theme logo](https://github.com/Fr4nSec/fr4nsec.github.io/blob/master/images/roasted.jpeg)
![theme logo](http://www.abhinavsaxena.com/images/abhinav.jpeg)

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
