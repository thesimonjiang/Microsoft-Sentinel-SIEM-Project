# Honeypot Attack Map via Microsoft Sentinel (Cloud Native SIEM)
<br>

## Project Overview

SIEM, Security Information and Event Management, is an integral part of any cybersecurity ecosystem. It collects, aggregates and analyze all the organization's data in one place. It gives cybersecurity analysts an overview of the entire organization's security status. In this project, we are going to collect attack data from our honeypot Windows 10 virtual machine (failed remote logins) using Microsoft Sentinel, a cloud-native SIEM, and project the attackers' origin on a world map.
<br>

## Project Architecture
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Honeypot%20Attack%20Map.drawio.jpg)

The concept is simple:

- Create a honeypot to lure in attackers by spin up a vulnerable Windows virtual machine and expose it to the internet.

- Log the attempted remote logins from attackers around the world

- Create a custom log by combining geolocation data with Windows security logs

- Collect the data from our custom log using Microsoft Sentinel

- Use Microsoft Sentinel to plot the origins of the attackers on a world map
<br>

## Project Process

### Signing Up for Microsoft Azure Cloud

Go to [Microsoft Azure's official website]([Cloud Computing Services | Microsoft Azure](https://azure.microsoft.com/en-us)) and sign up for a free account. You will need a Microsoft/Outlook account in order to do so. You will get some free resources along with a $200 credit.
<br>
<br>

### Getting API Key from IPGeolocation.io

Go to [IPGeolocation's official website](https://ipgeolocation.io/) and sign up for a free account. Locate the API Key in the dashboard of your account (you will see it immediately upon login). We will need it for our PowerShell code.
<br>
<br>

### Create Windows 10 Virtual Machine

Now let's create a Windows 10 VM inside Azure cloud. Follow the steps shown in the GIF. During the creation of the VM, let's also create a firewall rule to allow all inbound traffic so that once our VM has been discovered, the attackers can start to exploit it through the open RDP port.
<br>
<br>

**WARNING: In hindsight, creating the "ALLOW ALL" rule at this stage is a mistake.** This should be done just before you are ready to plot the attacker map. The attackers discovered my VM very quickly and has already started to exploiting the VM before I even had a chance to set everything up properly. I was forced to create a firewall rule to block all ports temporarily so I can finish setting up the project.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Create%20Virtual%20Machine%201.gif)
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Create%20Virtual%20Machine%202.gif)
<br>
<br>

### Create Log Analytic Workspace

We will need to create an Analytic Workspace to process our custom log. Follow the steps shown in the GIF.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Create%20Log%20Analytics%20Workspaces.gif)
<br>
<br>

### Enable Microsoft Defender

Now let's enable Microsoft Defender on the Log Analytic Workspace we've created.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Connect%20Microsoft%20Defender%20to%20Honeypot-log.gif)
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Connect%20Sentinel%20to%20Log%20Space.gif)
<br>
<br>

### Connect Virtual Machine to Log Analytic Workspace

Let's connect our VM to the Analytic Workspace.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Connect%20Honeypot-log%20to%20VM.gif)
<br>
<br>

### Access Windows Virtual Machine (Inside Honeypot VM)

Now let's see if our virtual machine running Windows 10 is working. Head to **Virtual Machines** and select the virtual machine you've created (*honeypot-vm1*). We see that the VM is up and running. Let's copy its public IP address and connect to it remotely. If you are using a Windows PC like I am, you can use the default "Remote Desktop Connection" already exists on your windows 10/11 PC. If you are using Mac, you can use the "[Microsoft Remote Desktop for Mac](https://apps.apple.com/us/app/microsoft-remote-desktop/id1295203466?mt=12)". If you are using Linux, you can use any VNC such as "[Remmina]([The fast, stable, and always free Linux VNC client - Remmina](https://remmina.org/remmina-vnc/))" if your distribution didn't already include it.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/RDP%20into%20VM%201.gif)
<br>
<br>

Once inside your VM, type `event viewer` inside the search field in the taskbar and hit **Enter**. This will take your to the **Windows Event Viewer**. We will be able to find the security information we seek in here. Expand the **Windows Logs** folder on the left hand side. Then select **Security**. We will find all the login related logs here. Specifically, we are looking for [**Event ID 4625**](%5B4625(F) An account failed to log on. - Windows Security | Microsoft Learn]([4625(F) An account failed to log on. - Windows Security | Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625))). This id number is associated with failed log on. Any information on the failed attempt will be logged; this includes items such as date and time, computer name, credential used, and IP address. Here I will try to logon using a made up account and password. As you can see, Windows has logged this failed login attempt.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Event%20Viewer%204625%20Logon.gif)
<br>
<br>

### Run PowerShell Script (Inside Honeypot VM)

**WARNING: Do this in your honeypot VM, NOT your own machine.**

Now let's create the PowerShell script that will help us to automatically export the security log information to the IPGeolocation website that will feed us the geolocation data on the intrusions. We will need the API key for this. So go to your IPGeolocation dashboard and copy the API Key into your PowerShell code. The PowerShell code file is included in this repository. In taskbar's search field, type `powershell ISE` and hit "Enter". Copy and paste the PowerShell code. On line 5, `$API_KEY = "ENTER YOUR API KEY HERE"` , replace the text with your IPGeolocation API Key.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Run%20PowerShell%20Script.gif)
<br>
<br>

### Turning off Windows Firewall (Inside Honeypot VM)

**WARNING: Do this in your honeypot VM, NOT your own machine.**

Now let's turn off the Windows Firewall so that our honeypot VM is discoverable to anyone on the internet. In taskbar's search field, type `wf.msc` then hit "Enter". Go to "Windows Defender Firewall Properties" and change the Firewall Status from "On" to "Off" for Domain Profile, Private Profile and Public Profile. **Warning: DO NOT** do this in real life!
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Turning%20off%20Windows%20Firewall.gif)
<br>
<br>

### Create Custom Log

Now we need to create a custom log that will be used by Microsoft Sentinel to extract the data from in order to plot our attacker map.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Create%20Custom%20Log.gif)
<br>
<br>

### Plot the Origins of the Attackers on Map

Now all we have to do is to set up the attacker map in Microsoft Sentinel and sit back to watch all the fun actions. Copy and paste the KQL query into the editor following the steps shown in GIF.

```kusto
FAILED_RDP_GEOLOCATION_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country
```
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/Plotting%20Windows%20Sentinel%20Attack%20Map.gif)
<br>
<br>

## Final Result

Here's the final result after about 12 hours.
<br>
<br>

![](https://github.com/thesimonjiang/Microsoft-Sentinel-SIEM-Project/blob/2f41df11f6b8db30f438a799077590eb9decffd8/honeypotAttackMapFinal.png)
