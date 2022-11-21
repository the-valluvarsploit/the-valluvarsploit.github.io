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
I started subdomain enumeration with Google Dorking, [OWASP Amass](https://github.com/OWASP/Amass) and [Gobuster](https://github.com/OJ/gobuster) tools.

```bash
# Passive Subdomain Enumeration using Google Dorking
site:*.redacted.com -www -www1 -blog
site:*.*.redacted.com -product

# Passive Subdomain Enumeration using OWASP Amass
amass enum -passive -d redacted.com -config config.ini -o amass_passive_subs.txt

# Subdomain Brute force using Gobuster
gobuster dns -d redacted.com -w wordlist.txt --show-cname --no-color -o gobuster_subs.txt
```

After enumerating subdomains, removed duplicate entries and merged them into a single file (subdomains.txt) using the [Anew](https://github.com/tomnomnom/anew) tool. Then passed the subdomains.txt file to my cname.sh shell script for enumerating CNAME records. 

```bash
# Merging subdomains into one file
cat google_subs.txt amass_passive_subs.txt gobuster_subs.txt | anew subdomains.txt

# Enumerate CNAME records
./cname.sh -l subdomains.txt -o cnames.txt

# We can use HTTPX tool as well
httpx -l subdomains.txt -cname cnames.txt
```

Then passed the subdomains.txt file to the HTTPX tool for probing live websites.

```bash
# Probe for live HTTP/HTTPS servers
httpx -l subdomains.txt -p 80,443,8080,3000 -status-code -title -o servers_details.txt
```

## ANALYSIS
I started analyzing the cnames.txt file and found one subdomain that was pointing to two different CNAME records. I ran dig command on the subdomain and got followings,

```bash
$ dig next.redacted.com CNAME
```

![dig command](/assets/posts_assets/2022-04-07-Meow/fastly_subdomain_takeover_dig_command.png)

This subdomain had two CNAME records. The first CNAME record was pointing to webflow.io domain and the second CNAME record was pointing to fastly.net domain. Whenever we have multiple CNAME records, the first CNAME record will redirect us to next CNAME record and so on. 

I started analyzing the servers_details.txt file for interesting stuff and found this line. Notice status code and website title.

```bash
https://next.redacted.com [500] [246] [Fastly error: unknown domain next.redacted.com]
```

The status code was 500 and the title was “Fastly error: unknown domain next.redacted.com”.  I came across this Fastly error many times before and knew that Fastly is not completely vulnerable. It's vulnerable only when certain conditions are met, called the edge case.

Most of the time we cannot takeover Fastly service. For example, below case,

![dig command](/assets/posts_assets/2022-04-07-Meow/fastly_subdomain_takeover_fastly_error.png)

But if the domain is not already taken by another customer then we can claim the domain and takeover the subdomain completely.

## COFIRMING THE VULNERABILITY
I went to fastly.com to check if it is vulnerable or not.  I already had an account on fastly.com that was created 2 years ago but deleted this account 6 months ago because I believed Fastly is secure (I was wrong).

Steps to detect if fastly is vulnerable or not,
1. I went to fastly.com and created an account.
2. Logged in to my Fastly Dashboard and clicked on the "Create a Delivery Service" button.
3. Entered the target subdomain (next.redacted.com) and clicked on Add button.

I was expecting the error message (domain is already taken by another customer) but there was no error message. I was redirected to the next page “Hosts page”. I was surprised by seeing this.

## PoC CREATION STEPS
Once the vulnerability was confirmed, I logged into my VPS server and created a directory called hosting. Then within the hosting directory created a simple HTML file called index.html.

```bash
$ mkdir hosting
$ cd hosting
$ nano index.html
```

Index.html file contained below code.

```html
<h1>ValluvarSploit PoC</h1>
```

After that, I started a simple Python web server in the current directory

```bash
$ python3 -m http.server 80
```

After that, I started a simple Python web server in the current directory. Then I went to the Fastly dashboard and Added the public IP address of the VPS server in the hostname. After a few seconds, I opened up a new browser window and visited http://next.redacted.com/index.html page. My PoC file was rendered successfully. I have written a detailed report and submitted it. I kept my Fastly service running for 3 days and monitored server logs for sensitive information.

## REWARD
My report was triaged as a HIGH severity vulnerability and rewarded $2000 within 10 days.

## TAKEAWAYS
1. Revisit your old targets at least once in 6 months.
2. Subdomain Enumeration is key.
3. Don’t give up.

Thanks for taking time to read my post.

Warm Regards,  
ValluvarSploit
