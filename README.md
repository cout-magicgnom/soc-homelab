# Soc-homelab (DIY)
>[!WARNING]
>The following content is for educational purposes only and is not intended to teach how to hack enterprise infrastructure Do good hack, not bad hack +🤠.
---

## 🧱 Archtecture
```
Wazuh Security Lab
│
├── 01-file-integrity
├── 02-ssh-bruteforce
├── 03-user-creation
├── 04-privilege-escalation
├── 05-service-manipulation
├── 06-permission-change
└── 07-attack-timeline
```
--- 

## 📎Setting Up Wazuh SIEM (VM Box)

1 – [Click here to download](https://packages.wazuh.com/4.x/vm/wazuh-4.14.7.ova) the latest version of Wazuh VM on their offical [website](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) .

2 – Once download is finish, find the wazuh.ova directory and double click on it

3 – The machine comes with Every configuration pre-installed, just hit the “finish” button on VM Box.

4 – Just make sure that the VM has your network device on bridge mode and you are good to go.

5 - When your server starts up, make sure to check if the services are up and running:
```
systemctl status wazuh-indexer
systemctl status wazuh-dashboard
systemctl status wazuh-manager
```
6 - Open the dashboard on your browser: https://<WAZUH-IP>

## 🕵️‍♀️ Setting up a Wazuh Agent 
1 - When opening the dashboard for the first time, the first button will be to add a new agent

2 - On the "Deploy new agent" tab select your Operating System and check the server address (Wazuh IP)

<!-- Deploy new agent image -->

3 - copy/paste the next 2 commands (1st to install all wazuh agent packages / 2nd to start wazuh service)

> [!NOTE]
> When using a VM it doesn't require the UFW, so you should not have any trouble with "closed doors"

---
> [!TIP]
> Do not forget to reserve an IP address using your router/modem's DHCP for your Wazuh & Agents

## 🗂️ 01-File-Integrity (File Integrity Monitoring)

A simple, step-by-step guide to add a specific folder on a Windows agent to Wazuh's File Integrity Monitoring (FIM) and verify that changes generate events.

This guide uses `C:\FIM-Test` as the example folder.

## Table of Contents

- [1. Create the Folder in Windows](#1-create-the-folder-in-windows)
- [2. Locate ossec.conf](#2-locate-ossecconf)
- [3. Locate the `<syscheck>` Configuration](#3-locate-the-syscheck-configuration)
- [4. Add the Folder](#4-add-the-folder)
- [5. Enable Real-Time Monitoring (Optional)](#5-enable-real-time-monitoring-optional)
- [6. Save the File](#6-save-the-file)
- [7. Restart the Agent](#7-restart-the-agent)
- [8. Run a Test](#8-run-a-test)
- [9. Check the Event in the Wazuh Dashboard](#9-check-the-event-in-the-wazuh-dashboard)
- [10. Next Step: Enable Who-Data](#10-next-step-enable-who-data)
- [Full Flow](#full-flow)

---

## 1. Create the Folder in Windows

Open PowerShell as Administrator:

```powershell
New-Item -ItemType Directory -Path "C:\FIM-Test"
```

Verify it was created:

```powershell
Get-Item "C:\FIM-Test"
```

You should see the folder's information printed out.

## 2. Locate ossec.conf

On the Windows agent, open:

```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

If you installed the agent in a different directory, look for the `ossec.conf` file inside the installation folder.

Back up the file before making changes:

```powershell
Copy-Item `
  "C:\Program Files (x86)\ossec-agent\ossec.conf" `
  "C:\Program Files (x86)\ossec-agent\ossec.conf.bak"
```

## 3. Locate the `<syscheck>` Configuration

Inside `ossec.conf`, look for:

```xml
<syscheck>
```

You will likely already have a configuration similar to:

```xml
<syscheck>
    <disabled>no</disabled>
    ...
</syscheck>
```

> **Note:** Don't create a separate `<syscheck>` block if one already exists. Add the new configuration inside the existing block.

## 4. Add the Folder

To get started, add:

```xml
<directories>C:\FIM-Test</directories>
```

For example:

```xml
<syscheck>
    <disabled>no</disabled>

    <frequency>300</frequency>

    <scan_on_start>yes</scan_on_start>

    <directories>C:\FIM-Test</directories>
</syscheck>
```

The `<directories>` parameter defines which directories Syscheck/FIM should monitor.

## 5. Enable Real-Time Monitoring (Optional)

For a lab environment, it's recommended to use `realtime="yes"`:

```xml
<directories realtime="yes">C:\FIM-Test</directories>
```

So the full block becomes:

```xml
<syscheck>
    <disabled>no</disabled>

    <frequency>300</frequency>

    <scan_on_start>yes</scan_on_start>

    <directories realtime="yes">C:\FIM-Test</directories>
</syscheck>
```

This way, you don't need to wait for the next scheduled scan to test changes. Wazuh supports real-time directory monitoring on Windows.

## 6. Save the File

Save `ossec.conf`.

Before restarting the agent, it's worth double-checking that you haven't introduced any XML syntax errors.

## 7. Restart the Agent

Open PowerShell as Administrator:

```powershell
Restart-Service -Name wazuh
```

Verify the service status:

```powershell
Get-Service wazuh
```

The output should show:

```
Status   Name
------   ----
Running  wazuh
```

## 8. Run a Test

Create a file inside the folder:

```powershell
New-Item -ItemType File -Path "C:\FIM-Test\teste.txt"
```

Write some content to it:

```powershell
Set-Content `
    -Path "C:\FIM-Test\teste.txt" `
    -Value "My first FIM test"
```

Modify it again:

```powershell
Set-Content `
    -Path "C:\FIM-Test\teste.txt" `
    -Value "Modified file"
```

And finally, delete it:

```powershell
Remove-Item "C:\FIM-Test\teste.txt"
```

With `realtime="yes"`, these changes should be detected by FIM almost immediately.

## 9. Check the Event in the Wazuh Dashboard

In the Dashboard, look for events related to the agent.

You should find Syscheck/FIM events related to the file:

```
C:\FIM-Test\teste.txt
```

One particularly useful field is:

```
syscheck.path
```

It should show:

```
C:\FIM-Test\teste.txt
```

Also look for:

```
syscheck.event
```

which may show events such as:

- `added`
- `modified`
- `deleted`

## 10. Next Step: Enable Who-Data

Once the basic test is working, the recommended next step is:

```xml
<directories realtime="yes" whodata="yes">
    C:\FIM-Test
</directories>
```

This provides additional information about who made the change and which process was involved, making the experiment much more interesting from a security standpoint.

For example, instead of simply finding out:

```
C:\FIM-Test\teste.txt
        ↓
     modified
```

you can get information like:

```
File:    teste.txt
Event:   modified
User:    Administrator
Process: powershell.exe
```

## Full Flow

```
Windows Agent
     │
     │
     ▼
C:\FIM-Test
     │
     │ file created/modified/deleted
     ▼
Syscheck / FIM
     │
     ▼
Wazuh Agent
     │
     ▼
Wazuh Manager
     │
     ▼
Wazuh Dashboard
     │
     ▼
File Integrity Event
```

---

**Recommendation:** For your lab, start with just `C:\FIM-Test` + `realtime="yes"`. Once you've confirmed events are coming through, add `whodata`, and only then move on to real folders such as `C:\Windows\System32` or Registry keys. This avoids generating a huge volume of events before you understand the basic workflow.

>[!NOTE]
> For more information, please visit [my website](https://cout-magicgnom.github.io/website/projects/documents.html) for full vulnerabilites walkthrough
> and if you're interested in some videos, there's some repertoires that's helped to make this repo:
> (PT-BR) - https://www.youtube.com/watch?v=ZvfSGRnfYfo&t=334s
> (ENG) - https://www.youtube.com/watch?v=3CaG2GI1kn0&t=151s



