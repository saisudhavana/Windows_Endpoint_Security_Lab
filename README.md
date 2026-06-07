# Windows Endpoint Security Lab – Sysmon + ProcDump + Pypykatz

Windows Endpoint Security Lab demonstrating Sysmon-based event logging, ProcDump-based LSASS memory dump simulation, and credential extraction analysis using Pypykatz with MITRE ATT&CK mapping.

## Project Overview
This project demonstrates Windows endpoint security monitoring and credential dumping simulation using Sysmon, ProcDump, and Pypykatz inside a VirtualBox Windows environment. The goal is to understand how attackers extract credentials from LSASS memory and how Sysmon logs help in detecting malicious behavior.

## VirtualBox & Windows Setup
Installed VirtualBox, downloaded Windows ISO, and created a Windows virtual machine by allocating CPU, RAM, and storage. The Windows OS was installed successfully inside the VM using the ISO file.  
IMAGE: VirtualBox Windows VM setup screenshot

## Sysmon Installation & Setup
Sysmon (System Monitor) is a Windows Sysinternals tool used for monitoring system activity like process creation, process access, file creation, and registry changes.

Downloaded Sysmon from Microsoft Sysinternals suite and extracted the ZIP file. Also downloaded Sysmon configuration file from GitHub (SwiftOnSecurity community config). This config file defines what Sysmon should log and reduces noise by focusing only on important events like Event ID 1 (Process Creation), Event ID 10 (Process Access), and Event ID 11 (File Creation).  
IMAGE: Sysmon download and config file screenshot

Opened Command Prompt as Administrator and navigated to Sysmon folder using: cd C:\Users\<User>\Downloads\Sysmon. Installed Sysmon using the command Sysmon64.exe -i SysmonConfig.xml. Here Sysmon64.exe installs Sysmon as a service and SysmonConfig.xml defines monitoring rules.  
IMAGE: Sysmon installation command execution

After installation, Sysmon logs are verified in Event Viewer under Applications and Services Logs → Microsoft → Windows → Sysmon → Operational. Event ID 1 shows process creation, Event ID 10 shows process access, and Event ID 11 shows file creation events.  
IMAGE: Event Viewer Sysmon logs

## ProcDump Setup & Credential Dump Simulation
ProcDump is a Microsoft Sysinternals tool used for capturing process memory dumps for debugging and analysis, and is also used in credential dumping scenarios.

Downloaded ProcDump from Microsoft Sysinternals and extracted the ZIP file. Opened ProcDump folder and executed ProcDump using administrator CMD. The command used was procdump.exe -ma lsass.exe lsass_dump.dmp where -ma captures full memory of LSASS process and lsass_dump.dmp is the output dump file.  
IMAGE: ProcDump execution screenshot

## Sysmon Logs During Attack Simulation
During ProcDump execution, Sysmon captured multiple events. Event ID 1 logs process creation showing ProcDump execution. Event ID 10 logs process access showing ProcDump accessing LSASS memory. Event ID 11 logs file creation showing the creation of lsass_dump.dmp file on disk.  
IMAGE: Event ID 1, 10, 11 logs

## Dump File Analysis
The generated LSASS dump file is in binary format and when opened in Notepad it appears as unreadable byte data because it contains raw memory content of the LSASS process.  
IMAGE: dump file opened in Notepad showing byte format

## Python & Pypykatz Setup
Installed Python from python.org and installed Pypykatz using pip install pypykatz. Pypykatz is a Python-based forensic tool used to analyze LSASS dump files and extract credentials offline. It can extract NTLM hashes, Kerberos tickets, DPAPI keys, usernames, domains, and logon session details.

Ran the command pypykatz lsa lsass_dump.dmp which parses the dump file and displays credential information in structured format.  
IMAGE: Pypykatz output showing extracted credentials

## Key Observations
Sysmon logs all process activities, ProcDump accesses LSASS memory and generates dump file, and Pypykatz successfully extracts credentials from the dump file.

## MITRE ATT&CK Mapping
T1003 – OS Credential Dumping (LSASS Memory)  
T1055 – Process Memory Access  
T1070 – Indicator of Activity via Logs  

## Conclusion
This lab demonstrates how credential dumping works in Windows environments and how Sysmon provides visibility into such activities. It shows how attackers extract credentials from LSASS memory and how defenders can detect and analyze these events using Windows logs and forensic tools.
