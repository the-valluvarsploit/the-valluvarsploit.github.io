---
title: Fastly Subdomain Takeover $2000
date: 2022-11-21 05:59:00 +0530
categories: [Bug Bounty]
tags: BugBounty
---
## WHOAMI
My name is Alexandar Thangavel AKA ValluvarSploit, a full-time bug hunter and trainer. I love recon. I am the founder and CEO of ValluvarSploit Security. At ValluvarSploit Security, we are providing Bug Bounty training in  one-to-one live sessions. For more information, please check our [LinkedIn page](https://www.linkedin.com/company/valluvarsploit-security).

## OBJECTIVE
Today, I am going to share how I found Fastly subdomain takeover vulnerability and earn my first three digits bounty. Let’s get started.

## BACKSTORY
This was started on October 2nd, 2022 Sunday. The day started as usual. I woke up at 6 AM, finished routine work, checked my Mobile data balance (1.3 GB was remaining), enabled my Mobile Hotspot, connected my Laptop, and resumed hunting on a private program. I spent a few hours on the target application but found nothing so took a short break. I used to revisit my old private programs at least once in six months. So, I reviewed my private invites, picked an old program and started performing subdomain enumeration (Let’s call our target as `redacted.com`).

## SUBDOMAIN ENUMERATION
I have started subdomain enumeration with Google dorking, [OWASP Amass](https://github.com/OWASP/Amass) and [Gobuster](https://github.com/OJ/gobuster).

```bash
# Passive Subdomain Enumeration using Google Dorking
site:*.redacted.com -www -www1 -blog
site:*.*.redacted.com -product

# Passive Subdomain Enumeration using OWASP Amass
amass enum -passive -d redacted.com -config config.ini -o amass_passive_subs.txt

# Subdomain Brute force using Gobuster
gobuster dns -d redacted.com -w wordlist.txt --show-cname --no-color -o gobuster_brute_subs.txt
```

After enumerating subdomains, removed duplicate entries and merged them into a single file (subdomains.txt) using the [Anew](https://github.com/tomnomnom/anew) tool. Then passed the subdomains.txt file to my cname.sh shell script for enumerating CNAME records. 

```bash
# Merging subdomains into one file
cat google_subs.txt amass_passive_subs.txt gobuster_brute_subs.txt | anew subdomains.txt

# Enumerate CNAME records
./cname.sh -l subdomains.txt -o cnames.txt

# We can use HTTPX tool as well
httpx -l subdomains.txt -cname cnames.txt
```

I started analyzing the cnames.txt file and found one subdomain that was pointing to two different CNAME records. I ran dig command on the subdomain and got followings,

```bash
$ dig next.redacted.com CNAME
```



Warm Regards,  
ValluvarSploit
