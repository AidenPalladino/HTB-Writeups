Inlanefreight: Reconnaissance Skills Assessment Writeup

Category: OSINT & Web Reconnaissance Target: HTB Academy skills-assessment box (inlanefreight.com / inlanefreight.htb)

Overview: 

This assessment focused purely on the reconnaissance phase: domain intelligence, virtual host discovery, and web crawling. No exploitation was required; the goal was to demonstrate a methodical enumeration process to progressively uncover hidden subdomains, admin paths, and sensitive data (an API key and an email address) exposed through misconfigured or forgotten web assets.

Domain Intelligence: 

Goal: Identify the registrar for inlanefreight.com.

Ran a WHOIS lookup, which queries the domain registration database directly:

<img width="624" height="273" alt="Untitled design (3)" src="https://github.com/user-attachments/assets/5ed41de1-3ff9-462e-9623-94408d95e4a8" />



This returned the registrar's IANA ID directly in the output.

Web Server Fingerprinting: 

Goal: Identify the HTTP server software behind inlanefreight.htb.

Since inlanefreight.htb isn't a real, publicly resolvable domain, I first had to map it locally by adding an entry to /etc/hosts:
<img width="417" height="45" alt="image" src="https://github.com/user-attachments/assets/1cc3e685-1864-4b38-8f04-4ca496443db2" />

(target-ip  inlanefreight.htb)
<img width="975" height="436" alt="image" src="https://github.com/user-attachments/assets/42057d66-459c-4d3d-9eb9-f68a0869f4f3" />

With DNS resolution handled locally, I fingerprinted the web stack with whatweb, which inspects HTTP response headers to identify server software and versions:

<img width="624" height="56" alt="Untitled design (4)" src="https://github.com/user-attachments/assets/2f0469b1-988b-4e90-93c6-c6f0b8f1a737" />



This identified the server software.

Virtual Host Discovery: 

Goal: Find a hidden admin directory and extract an exposed API key.

Ran Gobuster in vhost mode to brute-force virtual hosts. This sends requests to the same IP/port while varying the Host header to guess subdomains that route to different content:

<img width="975" height="534" alt="image" src="https://github.com/user-attachments/assets/2cb03f49-f435-4ac0-a755-80a1b6488ca3" />


This revealed a second vhost (web1337.inlanefreight.htb), which I added to /etc/hosts the same way as before.

On the new vhost, I checked robots.txt (which tells crawlers what paths to avoid) and found a disallowed path pointing to a hidden admin directory. Visiting it directly exposed an API key in the page content.
<img width="975" height="201" alt="image" src="https://github.com/user-attachments/assets/cde7e9d2-4235-4a87-8285-f3e44b3be8f2" />


Crawling for Exposed Data: 

Goal: Find an email address and a second, rotated API key.

Ran the same vhost brute-forcing technique again, this time targeting web1337.inlanefreight.htb, which surfaced yet another distinct vhost. Added that to /etc/hosts as well.
<img width="975" height="541" alt="image" src="https://github.com/user-attachments/assets/67e96bc5-0779-4407-b5f6-d53de5562f21" />


Installed Scrapy and used a crawler tool (ReconSpider, provided as part of the HTB Academy module) to crawl the new subdomain and collect embedded artifacts like emails and other sensitive strings:

<img width="975" height="109" alt="image" src="https://github.com/user-attachments/assets/8700b61e-defd-444b-ab52-7b285e181c33" />


<img width="975" height="71" alt="image" src="https://github.com/user-attachments/assets/27097f09-e47b-4982-9085-af70b6ace596" />


Reviewing the crawler's output file surfaced both:

1. The exposed email address
2. A second API key (the one the developers were planning to rotate to)

Reasoning / Chain Summary: 
1. WHOIS → registrar identification (no target interaction needed)
2. Manual /etc/hosts mapping → made the non-public .htb domain resolvable
3. whatweb fingerprinting → identified nginx as the web server
4. Vhost brute-forcing (Gobuster) → uncovered a hidden subdomain (web1337)
5. robots.txt on that subdomain → pointed to a hidden admin path → API key #1
6. Repeating vhost brute-forcing on the new subdomain → uncovered yet another subdomain
7. Crawling that subdomain with ReconSpider → exposed an email address and API key #2

Each step depended on the discovery before it. This assessment demonstrated that reconnaissance is rarely a single scan, but an iterative process of finding one asset that leads to the next.

Tools Used:
- whois
- whatweb
- Gobuster (vhost mode)
- Scrapy / ReconSpider
- /etc/hosts manual DNS mapping
  
Lessons Learned: 

- Virtual host enumeration is essential when a target uses name-based hosting. A single IP can hide multiple distinct applications behind different Host headers.
- robots.txt is a reconnaissance goldmine: it's meant to hide paths from search engines, but it just as easily tells an attacker exactly where to look.
- Automated crawling (vs. manual browsing) surfaces artifacts like API keys, emails, and comments that are easy to miss by hand, especially across multiple subdomains.
