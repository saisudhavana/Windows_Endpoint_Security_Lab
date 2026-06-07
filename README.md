# Windows Endpoint Security Lab – Sysmon + ProcDump + Pypykatz

Windows Endpoint Security Lab demonstrating Sysmon-based event logging, ProcDump-based LSASS memory dump simulation, and credential extraction analysis using Pypykatz with MITRE ATT&CK mapping.

## Project Overview
This project demonstrates Windows endpoint security monitoring and credential dumping simulation using Sysmon, ProcDump, and Pypykatz inside a VirtualBox Windows environment. The goal is to understand how attackers extract credentials from LSASS memory and how Sysmon logs help in detecting malicious behavior.

## VirtualBox & Windows Setup
Installed VirtualBox, downloaded Windows ISO, and created a Windows virtual machine by allocating CPU, RAM, and storage. The Windows OS was installed successfully inside the VM using the ISO file.  
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/c62d3b769249dfe2d9ff6cede58f4fb17a69c1c7/Screenshots/Windows_VM.png)

## Sysmon Installation & Setup
Sysmon (System Monitor) is a Windows Sysinternals tool used for monitoring system activity like process creation, process access, file creation, and registry changes.

Downloaded Sysmon from Microsoft Sysinternals suite and extracted the ZIP file. Also downloaded Sysmon configuration file from GitHub (SwiftOnSecurity community config). This config file defines what Sysmon should log and reduces noise by focusing only on important events like Event ID 1 (Process Creation), Event ID 10 (Process Access), and Event ID 11 (File Creation).  
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/157c6363068fd7420457eac668331ea4a3faffd6/Screenshots/SysmonDownload.png)

![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/157c6363068fd7420457eac668331ea4a3faffd6/Screenshots/sysmonzipfile.png)

SysmonConfig File:
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/87f626afea3341deee37df706a88710aac01c152/Screenshots/Sysmonconfigfile.png)
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/87f626afea3341deee37df706a88710aac01c152/Screenshots/Sysmon%20folder.png)

Opened Command Prompt as Administrator and navigated to Sysmon folder using: cd C:\Users\<User>\Downloads\Sysmon. Installed Sysmon using the command 
Sysmon64.exe -accepteula -i sysmonConfig.xml. 
Here Sysmon64.exe installs Sysmon as a service and SysmonConfig.xml defines monitoring rules. 
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/eb0c2e2b8da8fd0e1a64262ea92b23b05782b860/Screenshots/sysmoninstalltionincmd2.png)


After installation, Sysmon logs are verified in Event Viewer under Applications and Services Logs → Microsoft → Windows → Sysmon → Operational. Event ID 1 shows process creation, Event ID 10 shows process access, and Event ID 11 shows file creation events. 
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/ae50deb0f7a72701bafaceca5f5e9fcbd7385c12/Screenshots/Sysmonineventviewwe.png)

## ProcDump Setup & Credential Dump Simulation
ProcDump is a Microsoft Sysinternals tool used for capturing process memory dumps for debugging and analysis, and is also used in credential dumping scenarios.

Downloaded ProcDump from Microsoft Sysinternals and extracted the ZIP file.
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/ae50deb0f7a72701bafaceca5f5e9fcbd7385c12/Screenshots/ProcDumpDownload.png)
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/ae50deb0f7a72701bafaceca5f5e9fcbd7385c12/Screenshots/procdump1.png)
Opened ProcDump folder and executed ProcDump using administrator CMD. The command used was procdump.exe -ma lsass.exe lsass_dump.dmp where -ma captures full memory of LSASS process and lsass_dump.dmp is the output dump file.  
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/fa6ed520113bdb0580afc2813d4458a9da167ba8/Screenshots/creddumpsuccess.png)

## Sysmon Logs During Attack Simulation
During ProcDump execution, Sysmon captured multiple events. Event ID 1 logs process creation showing ProcDump execution. Event ID 10 logs process access showing ProcDump accessing LSASS memory. Event ID 11 logs file creation showing the creation of lsass_dump.dmp file on disk.  

Process Creation :
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/8fae532bc0224f30894021e8000cc60a9be06cee/Screenshots/procdumpprocesscreation-eventid-1.png)

Process Access :
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/8fae532bc0224f30894021e8000cc60a9be06cee/Screenshots/ProcessAcess.png)

File Creation:
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/8fae532bc0224f30894021e8000cc60a9be06cee/Screenshots/FileCreated.png)

## Dump File Analysis
The generated LSASS dump file is in binary format and when opened in Notepad it appears as unreadable byte data because it contains raw memory content of the LSASS process.  
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/8fae532bc0224f30894021e8000cc60a9be06cee/Screenshots/DUMPFILEINBYTES.png)
 dump file opened in Notepad showing byte format

## Python & Pypykatz Setup
Installed Python from python.org and installed Pypykatz using pip install pypykatz. Pypykatz is a Python-based forensic tool used to analyze LSASS dump files and extract credentials offline. It can extract NTLM hashes, Kerberos tickets, DPAPI keys, usernames, domains, and logon session details.

![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/7970164ceb1fcecdc2def733f7f6054dfc8c7211/Screenshots/Pypykatzdownload.png)

Ran the command pypykatz lsa lsass_dump.dmp 
which parses the dump file and displays credential information in structured format.  
![image alt](https://github.com/saisudhavana/Windows_Endpoint_Security_Lab/blob/7970164ceb1fcecdc2def733f7f6054dfc8c7211/Screenshots/dumpfileextracted.png)
IMAGE: Pypykatz output showing extracted credentials

## Key Observations
Sysmon logs all process activities, ProcDump accesses LSASS memory and generates dump file, and Pypykatz successfully extracts credentials from the dump file.

## MITRE ATT&CK Mapping
T1003 – OS Credential Dumping (LSASS Memory)  
T1055 – Process Memory Access  
T1070 – Indicator of Activity via Logs  


Incident Response, Containment & Remediation PlaybookTactic: Containment, Eradication, and Recovery (NIST SP 800-61 Phase 3 & 4)Once Sysmon Event ID 10 triggers a high-fidelity alert for unauthorized access to lsass.exe, the following structured containment and eradication playbook must be initiated immediately:1. Immediate Host Isolation (Containment)Action: Issue an isolation command via the Endpoint Detection and Response (EDR) console (or disconnect the network adapter inside VirtualBox/Switch configurations).Why: The moment an attacker dumps the memory, they will attempt to exfiltrate the .dmp file over the network to their own machine. Isolating the host immediately cuts off their outbound communication channel and prevents lateral movement.2. Process Termination & Artifact Eradication (Eradication)Action: Forcefully terminate the parent process that initiated the dump (e.g., the specific compromised cmd.exe or powershell.exe instance found in Sysmon Event ID 1).Action: Locate and permanently purge the memory dump file from the hard drive (e.g., deleting lsass_dump.dmp).Action: Scan memory structures to ensure no residual malicious injection hooks or secondary threads are left running inside legitimate background processes.3. Identity Invalidation & Session Revocation (Recovery)Action: Because the NTLM hashes and Kerberos tickets inside that memory dump must now be treated as fully compromised, all user accounts logged into that specific machine must undergo an immediate forced password reset.Action: Revoke all active authentication tokens, active Azure AD / M365 session cookies, and Kerberos Ticket Granting Tickets (TGTs) globally across the domain. This breaks any active Pass-the-Hash or replay setups the attacker might try offline.4. Host Hardening & Post-Incident ActivityAction: Audit the system configuration to find out why the dump was allowed to happen.Action: Re-enforce Protected Process Light (PPL) for LSASS by ensuring the registry key RunAsPPL is set to 1 under the LSA control panel.Action: Verify that Windows Defender Credential Guard is fully enabled via Group Policy to prevent standard administrative memory handle requests from succeeding in the future.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------
## 🛡️ Incident Response, Containment & Eradication Playbook
Operational Framework: NIST SP 800-61 Rev. 2 (Containment, Eradication, and Recovery)
Once a high-fidelity behavioral signature triggers for unauthorized memory access to lsass.exe via Sysmon Event ID 10, the SOC team must immediately initiate the following structured playbooks to disrupt the threat actor's objective:
## Immediate Network Isolation (Host Containment)

* Action: Issue a host isolation command via the active Endpoint Detection and Response (EDR) dashboard or forcefully disconnect the network adapter configuration.
* Objective: Sever active outbound network sessions to completely block the attacker's data exfiltration pipeline. This prevents them from transferring the raw .dmp container to an offline environment for decryption.

## Process Termination & Artifact Purging (Eradication)

* Action: Trace the process ancestry tree via Sysmon Event ID 1 and forcefully terminate the parent binary hierarchy responsible for spawning the dump tool (e.g., the specific compromised cmd.exe or powershell.exe thread context).
* Action: Locate the exact filesystem location captured by Sysmon Event ID 11 and permanently purge the .dmp or .dat dump file from the hard drive layer.

## Identity Invalidation & Token Revocation (Recovery)

* Action: Treat all user accounts, active domain credentials, and administrative profiles cached or active on that endpoint as fully compromised. Enforce an immediate mandatory password reset across the domain.
* Action: Globally revoke all active authentication cookies, active M365/Azure AD session tokens, and active Kerberos Ticket Granting Tickets (TGTs) to completely disrupt offline Pass-the-Hash (MITRE T1550.002) or credential replay operations.

## Defensive Baselining & System Hardening (Post-Incident Review)

* Action: Conduct a configuration audit to identify why baseline guardrails failed. Re-enable Protected Process Light (PPL) by modifying the Windows Registry value RunAsPPL to 1 under HKLM\SYSTEM\CurrentControlSet\Control\Lsa.
* Action: Enforce group policy objects (GPOs) to validate that Windows Defender Credential Guard and Virtualization-Based Security (VBS) metrics are live across the subnet scope to prevent future user-mode memory handle attachments.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Conclusion
This lab demonstrates how credential dumping works in Windows environments and how Sysmon provides visibility into such activities. It shows how attackers extract credentials from LSASS memory and how defenders can detect and analyze these events using Windows logs and forensic tools.
