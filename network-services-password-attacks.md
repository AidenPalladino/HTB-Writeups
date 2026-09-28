Network Services Password Attacks Writeup Category: Password Attacks & Remote Services Target: HTB Academy skills-assessment box (WINSRV, 10.129.202.136)

Overview This assessment covered credential attacks against four common Windows network services: WinRM, SSH, RDP, and SMB. A provided username/password wordlist pair was used to spray each service, and for each one, the goal was to recover a valid login, use it to establish a session, and locate a flag file left on the target. No exploitation was required, only credential brute-forcing and knowing how to operate each protocol's client tools.

Note: per HTB's Terms of Service, not all flags are fully shown

**Setup Goal: Prepare the wordlists before attacking any service.**

Downloaded the challenge's wordlist archive and extracted it to get a username list and a password list:

<img width="975" height="187" alt="image" src="https://github.com/user-attachments/assets/c235d27d-68c5-481e-b0da-d52790c8f1ea" />



**WinRM Goal: Find the user for the WinRM service, crack their password, and retrieve the flag.** Sprayed the username and password lists against WinRM using NetExec:


<img width="975" height="54" alt="image" src="https://github.com/user-attachments/assets/190ece67-bb9d-4284-9324-848e6948cea2" />

<img width="486" height="50" alt="image" src="https://github.com/user-attachments/assets/8546d3c4-2ec0-4440-8525-5f1c684f5f41" />


Logged in with Evil-WinRM using the recovered credentials:


<img width="975" height="75" alt="image" src="https://github.com/user-attachments/assets/915eaa96-3a5f-4c8e-8da2-38f397064f5e" />


Moved from the default Documents directory into Desktop and listed its contents:


<img width="975" height="342" alt="image" src="https://github.com/user-attachments/assets/10773dc0-a7bb-4e3f-a177-78493695f092" />


Read the file to get the flag.

Flag: <img width="94" height="56" alt="image" src="https://github.com/user-attachments/assets/2c9a71c9-fa9a-44a8-b7a7-5b88fb01e9ab" />


**SSH Goal: Find the user for the SSH service, crack their password, and retrieve the flag.** Brute-forced SSH using Hydra:


<img width="975" height="52" alt="image" src="https://github.com/user-attachments/assets/31a7ea33-b140-44e6-bd26-badbeb703dbc" />


<img width="975" height="39" alt="image" src="https://github.com/user-attachments/assets/03bf49e2-da49-49cd-90d2-05f188f16024" />


Logged in over SSH with the recovered credentials:


<img width="523" height="52" alt="image" src="https://github.com/user-attachments/assets/7870e2a6-4164-464c-8985-3c99959c150a" />


Since the target is a Windows host, the session drops into a Windows shell rather than bash, so Windows commands (dir, type) were used instead of Unix ones (ls, cat) to reach the flag on the Desktop.


Flag: <img width="102" height="42" alt="image" src="https://github.com/user-attachments/assets/f908d7cc-f768-4feb-a752-efd476b0a7fa" />


**RDP Goal: Find the user for the RDP service, crack their password, and retrieve the flag.** Brute-forced RDP with Hydra, throttled to a single thread with a one-second wait between attempts, since RDP servers do not tolerate many rapid parallel connections:


<img width="975" height="34" alt="image" src="https://github.com/user-attachments/assets/477904e9-8aa5-41bb-a0c0-75d229747942" />


<img width="975" height="36" alt="image" src="https://github.com/user-attachments/assets/f3bff32b-418e-4695-925d-d199f841c6da" />



Connected using xfreerdp with the recovered credentials, ignoring the untrusted self-signed certificate:


<img width="975" height="45" alt="image" src="https://github.com/user-attachments/assets/9fab8c3e-1e15-44eb-8723-cfa2f69b0b79" />



Once inside the RDP session, the flag was right on the desktop


**SMB Goal: Find the user for the SMB service, crack their password, and retrieve the flag.** Sprayed SMB logins with NetExec:


<img width="975" height="48" alt="image" src="https://github.com/user-attachments/assets/45376b9f-8e05-40c3-8669-5455acb29ebe" />




Listed the available shares with the recovered credentials and found one custom, non-default share:


<img width="975" height="71" alt="image" src="https://github.com/user-attachments/assets/4463f67b-43f3-4e3c-a513-2f51496f6a67" />



The share's name pointed toward a different intended user than the one already found, so cassie was targeted directly with the same password list:


<img width="975" height="46" alt="image" src="https://github.com/user-attachments/assets/9cb013d2-5510-40a6-ad6f-805df3c7f3a2" />



Connected to the CASSIE share with smbclient, explicitly specifying the correct NetBIOS domain (WINSRV, not the client's default WORKGROUP):


<img width="975" height="53" alt="image" src="https://github.com/user-attachments/assets/6245f233-6630-47fb-89c5-aba1614bd0a8" />



Retrieved the flag file from within the smbclient session:


smb:  <img width="286" height="31" alt="image" src="https://github.com/user-attachments/assets/10ee40bc-678e-4696-ae48-9c4c29162529" />


Flag: <img width="98" height="63" alt="image" src="https://github.com/user-attachments/assets/8d1b5dc2-b6b4-4cac-b8ce-8fb1cad0a6a0" />


**Reasoning / Chain Summary:**


1. Extracted the provided username and password wordlists before attacking any service
2. NetExec spray against WinRM, valid login found (john:november), used with Evil-WinRM to reach the flag on the Desktop
3. Hydra spray against SSH, valid login found (dennis:rockstar), used with the native ssh client, adjusting for a Windows-style shell
4. Hydra spray against RDP, throttled to avoid overwhelming the service, valid login found (chris:789456123), used with xfreerdp to reach the flag
5. NetExec spray against SMB, first hit (john:november) had no matching share of his own, so the one custom share's name (CASSIE) was used to retarget the spray at that specific user, yielding cassie:12345678910
6. Connected with smbclient using the correct domain to retrieve the final flag

Each service required the same core technique, spraying a wordlist pair and then connecting with the matching client, but each had its own quirks: a Windows shell over SSH, an RDP service that needed throttled connection attempts, and an SMB share whose owner did not match the first valid login found.

**Tools Used:**

- NetExec
- Hydra
- Evil-WinRM
- xfreerdp
- smbclient

**Lessons Learned:**

- NetExec is a strong all-purpose credential-spraying tool across SMB, WinRM, RDP, and more.
- RDP brute-forcing needs to be throttled. Too many parallel connection attempts can cause the service to drop or ignore requests, so limiting threads and adding a wait between attempts (-t 4 -W 1) gets more reliable results.
- A valid login does not guarantee useful access. John's SMB credentials worked, but he had no share of his own on the server. A share's name can be a strong hint toward which account is actually meant to access it.
- Domain and workgroup matter for SMB authentication. A login NetExec confirms as DOMAIN\user can still fail through smbclient if the domain is not passed explicitly with -W, since the client defaults to WORKGROUP, producing a misleading logon failure.
- Windows-over-SSH still gives a Windows shell, not bash. Commands like ls and cat do not work; dir and type do.
