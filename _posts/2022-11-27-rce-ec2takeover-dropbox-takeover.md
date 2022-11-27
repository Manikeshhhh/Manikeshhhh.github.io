---
title: RCE,SSRF-to-EC2takeover and dropbox account takeover
published: true
---

### RCE,SSRF-to-EC2takeover and dropbox account takeover
**Published on 27-Nov-2022**

### Description
So in this Blog post i will walk you through some of high severity issues i found in one of the Public Program. This was a vehicle rental application mostly used in India. During this period I found several issues but I will mostly be talking about some of the interesting issue i found during this period.

### Inital Recon
I don't recon much initally and I will definitely improve my recon soon,even during recon phase i don't automate stuffs, I don't use nuclie either most of the time(sometimes running nuclie can really give you fruitful result but i don't always do), sometimes i also use gauplus to find some url. 
so, I always start of with subdomain enumeration.First i find some doamin from subfinder then i bruteforce to find some non listed sub-domain using ffuf.

```
root@x71n0:~$ subfinder -d xxx.com | httpx --status-code
```
I grep subdomains from here first then I use ffuf to bruteforce some new domains
```
root@x71n0:~$ ffuf -u https://FUZZ.xxxx.com -w bruteforcelist.txt -ac -mc 200,302,403
```

