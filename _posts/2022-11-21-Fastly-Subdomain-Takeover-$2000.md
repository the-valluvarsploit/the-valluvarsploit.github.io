---
title: Fastly Subdomain Takeover $2000
date: 2022-11-21 05:59:00 +0530
categories: [Bug Bounty]
tags: BugBounty
---

Hello Bug Hunters,
My name is Alexandar Thangavel AKA ValluvarSploit, a full-time bug hunter and trainer. I love recon. I am the founder and CEO of ValluvarSploit Security. At ValluvarSploit Security, we are providing Bug Bounty training in  one-to-one live sessions. For more information, please check our LinkedIn page.

Today, I am going to share how I found Fastly subdomain takeover vulnerability and earn my first three digits bounty. Let’s get started.

This was started on October 2nd, 2022 Sunday. The day started as usual. I woke up at 6 AM, finished routine work, checked my Mobile data balance (1.3 GB was remaining), enabled my Mobile Hotspot, connected my Laptop, and resumed hunting on a private program. I spent a few hours on the target application but found nothing so took a short break. I used to revisit my private programs at least once in six months. So I picked an old private program  and started performing subdomain enumeration (Let’s call it as redacted.com).

```bash
# Passive Subdomain Enumeration using Google Dorking
site:*.redacted.com -www -www1 -blog

site:*.*.redacted.com -product

# Passive Subdomain Enumeration using OWASP Amass
amass enum -passive -d redacted.com -config config.ini -o amass_passive_subs.txt

# Subdomain Brute force using Gobuster
gobuster dns -d redacted.com -w wordlist.txt --show-cname --no-color -o gobuster_brute_subs.txt
```

Warm Regards,  
ValluvarSploit
