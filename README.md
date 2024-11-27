# Azure Sentinel SIEM Implementation 
![honeypot](https://files.oaiusercontent.com/file-BfhHUgBfosTsaKP4PwVmux?se=2024-11-27T19%3A59%3A21Z&sp=r&sv=2024-08-04&sr=b&rscc=max-age%3D604800%2C%20immutable%2C%20private&rscd=attachment%3B%20filename%3D6e609c99-c8bd-4819-b1a1-1475280560e9.webp&sig=qruy4PZV0K1I5uD%2Ba41owxvAy%2BvGZ7DBdDxo%2BMuzsvQ%3D)
## Description
This guide provides an overview of setting up and using an educational Homelab with Microsoft Azure Sentinel. Honeypots, which are decoy systems designed to attract and monitor malicious activity, play a key role in gaining insights into potential threats and attackers' tactics. A Security Information and Event Management (SIEM) solution facilitates the collection, analysis, and response to security events in real-time. Azure Sentinel, a cloud-native SIEM solution from Microsoft, enables a comprehensive view of security events while automating threat detection and response. This Homelab showcases the integration of cybersecurity and innovation, offering the ability to track and log attacks from across the globe while visualizing them on an interactive attack map. Through this setup, key aspects of modern cyber defense and threat monitoring are explored. A third party API [ipgeolocation](https://ipgeolocation.io/) is also used to track API queries. 

## Index
1. [Creating Azure Account](https://github.com/IsmailEdris/Azur_SIEM/blob/main/README.md#step-1-create-a-microsoft-azure-account)
2. [Creating and Setting up the Honey Pot](LINK)
3. [Connecting Log analytics to the VM](LINK)
4. [Setting up Microsoft Sentinel](LINK)
5. [Logging in to the VM](LINK)
6. [Event Viewer](LINK)
7. [Firewall Configurations](LINK)
8. [Powershell Script and API Key](LINK)
9. [Creating Custom Logs](LINK)
10. [Creating Custom Fields and Extracting Raw Log Data](Link)
11. [Setting up Sentinel Map](LINK)
12. [Creating SIEM Alerts](LINK)

## How does the project work?
1.	We are going to build a **Vulnerable VM** in Azure.
2.	We are going to create a **Log Analytics Workspace (LAW)** to transfer our logs from the VM to Azure so we can have a record of the logs on Azure. After, we will connect the LAW to **Microsoft Defender for Cloud**.
3.	Next, we will **connect** the Log Analytics to the VM.
4.	After, we are going to set up **Microsoft Sentinel** on Azure which we can use to create a map of where the attacks are coming from. 
5.	After we set up Sentinel we are going to Login to the VM. However, we will **fail one login attempt** on purpose so we can see it on the VM’s log. 
6.	Then, we will open up **Event Viewer** on the VM to view the logs on the VM. 
7.	After that, we will **turn off the firewall** on the VM so it is vulnerable and open to attackers. 
8.	Then we will create a **PowerShell script** to track the login attempts for us.  
9.	Next, we will create **custom logs** in Log Analytics Workspace to view our VMs custom logs. 
10.	Then, we will create **custom fields and extract data** from the log’s raw data. 
11.	Then, we will set up a **map** in Azure Sentinel configured with Latitude and Longitude so we can visually see where on the globe we are being attacked from. 
12.	Finally, we will create **SIEM alerts.**

## Step 1: Create a Microsoft Azure Account

> Microsoft offers $200 in Azure credit for 30 days when you initially sign up

![set](https://camo.githubusercontent.com/a99e124ad9661c14f6657210f64603c465e1bc52ba3e5cf3c6c6c515af1b9c00/68747470733a2f2f692e696d6775722e636f6d2f427a6d6a5661342e706e67)

## Step 2: Create and Set Up a Honeypot

1. After signing up, go to the [Azure Portal](portal.azure.com), or click "Go to the Azure Portal" then create a resoruce group and give it a name (honeypot-lab)
> A resource group is a container that helps organize and manage related cloud resources.
3. In the search bar type "Virtual Machines"
4. Under the "Create" tab click on Azure Virtual Machine.

> VM Console

![VM](https://camo.githubusercontent.com/50ba2b93e0ad2021154004f7746374856472cbf6154b6d96cb51a247c6d1b5ea/68747470733a2f2f692e696d6775722e636f6d2f6e5544726e627a2e706e67)

### Instance details
- Give your virtual machine a name (honeypot-vm)
- Choose a recommended region: ((US) West 3)
- Availability options: No infrastructure redundancy required
- Security type: Standard
- Image: Windows 10 Pro, version 22H2 - x64 Gen2
- VM Architecture: x64
- Size: Default is fine (Standard_D2s_v3 – 2vcpus, 8 GiB memory)
- Click Next

> Instance Details

![id](https://camo.githubusercontent.com/6cacf3312f1b7beba7a0065ae3dfa3e34b26d426a86b2cf1e4693f6f056974be/68747470733a2f2f692e696d6775722e636f6d2f4576766e724e512e706e67)

### Adminstrator Account
- Set up a username and password for the VM. These credentials will be used to remotly log in to the VM.
- Click next
### Inbound Port Rules
- Change the "Inboud Port Rules" to Publick inbound ports -> Allow selected ports: RDP (3389)
- Click Next
### Licensing 
- Confirm the Licensing
- Click Next

### Disks
- Leave everything as is
- Click Next

### Networking

-NIC network security group: Advanced -> Create new
  >A Network Security Group (NSG) in Azure is a virtual firewall that filters and controls network traffic to protect Azure resources.

- By clicking the three dots, delete Inbound rules (1000: default-allow-rdp)
- Add an inbound rule
- Destination port ranges: * (wildcard for anything)
- Protocol: Any
- Action: Allow
- Priority: 100 (low)
- Name: Anything (allow-any-inbound)
- Select Review + Create

>NSG

![nic](https://camo.githubusercontent.com/de615c6f72bf73666048bd38ffdc73b668f23e8f3891f7c4d0cbab8df4064ee7/68747470733a2f2f692e696d6775722e636f6d2f6a4d6c723455452e706e67)

## Step 3: Creat a Log Analytics Workspace (LAW)

1. Search for "Log analytics workspaces"
2. Select Create Log Analytics workspace
3. Place it in the identical resource group as the VM (honeypot-lab)
4. Give it the name you choose (honeypot-law)
5. Add to the same region (West US 3)
6. Select Review + Create

>LAW Details

![law](https://camo.githubusercontent.com/7d5e7599aae81b6845c3cdb2005740d4e0304c056982e635032bacf4266036cf/68747470733a2f2f692e696d6775722e636f6d2f4e336677546b522e706e67)

## Step 4: Set up Microsoft Defender for Cloud
1. Search for "Microsoft Defender for Cloud"
2. Under Management click on "Environment settings" -> Subscription Name -> Log Analytics Workspace Name (honeypot-law)

> Microsoft Defender for Cloud Interface

![def](https://camo.githubusercontent.com/a1f5da3f3517f5d6b310506c72b6d76401a477bebee11d7ea58e28a126bf15a2/68747470733a2f2f692e696d6775722e636f6d2f795878537679522e706e67)

### Settings | Defender Plans

- Foundational CSPM (Cloud Security Posture Management): ON
- Servers: ON
- SQL servers on machines: OFF
- Click Save

> Defender Plans

![plan](https://camo.githubusercontent.com/4156ff4c6afa72c443f7ce45139b692a5d73ff6e2b1b9358c42682f9069e1421/68747470733a2f2f692e696d6775722e636f6d2f6f6e79327861792e706e67)

### Settings | Data collection

- Select "All Events"
- Click Save'

## Step 5: Linking the VM to LAW

- At the search bar type "Log Analytics workspaces"
- Select workspace name (honeypot-law) -> "Virtual machines" -> virtual machine name (honeypot-vm)
- Click Connect

![hit](https://camo.githubusercontent.com/bf1a258daccef2d2b7c8b0dbea289b2748331521e2b0b08f6c4e704e237c1117/68747470733a2f2f692e696d6775722e636f6d2f493078793637382e706e67)

## Step 6: Setup Microsoft Sentinel 
- At the serach bar type "Microsoft Sentinel"
- Click Create Microsoft Sentinel
- Choose Log Analytics Workspace name (honeypot-law)
- Click Add
![click](https://camo.githubusercontent.com/cc45082491f5ec1edec0be1b204fdf279ca7b0bdb20e2ea9ce8c133ff3dc84e9/68747470733a2f2f692e696d6775722e636f6d2f74366a62564c542e706e67)

## Step 7: Turn off the VM's Firewall

1. Locate the honeypot VM (honeypot-vm) under Virtual Machines.
2.Copy the IP address from the VM
3. Using the credentials from step 2, access the virtual machine through Remote Desktop Protocol (RDP). Note: if your on a Mac you can download the "Microsoft Remote Desktop" application or use another VM host for Microsoft that supports the protocol.
4. Accept Certificate warning
5. Select NO for all Choose privacy settings for your device
6. Hit Start and search for "wf.msc" (Windows Defender Firewall)
7. Click "Windows Defender Firewall Properties"
8. Turn Firewall State OFF for Domain Profile | Private Profile | and Public Profile
9. Click Apply and Ok
10. To check if VM is reachable, ping it using the command line of the host ping -t (ip-adress)

>Windows Defender Firewall Interface

![firewall](https://camo.githubusercontent.com/b02fb64eef48e54f75a662626e8242465ec22c140a83d3fe13a848b63aeaf2a3/68747470733a2f2f692e696d6775722e636f6d2f443475466963582e706e67)

## Step 8 : Automating the Secuirty Log Exporter

1. In your VM launch Powershell ISE
2. Configure Edge browser without logging in
3. Copy Powershell Script and insert into Virtual Machine's Powershell (authored by Josh Madakor)
4. Choose New Script in Powershell ISE and paste script
5. Give it a name and save it to the desktop (log_exporter)


