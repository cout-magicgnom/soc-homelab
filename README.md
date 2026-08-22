# soc-homelab (DIY)

## 🟢 Entrance Level
```
    This is a tutorial/documentation on how to set up a SIEM homelab with Wazuh and Kali Linux in a controlled environment.

```
--- 
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
---
> [!TIP]
> Do not forget to reserve an IP address using your router/modem's DHCP for your Wazuh & Agents

## 👟Running the Server
```
When you initialize the Wazuh for the fisrt time 






