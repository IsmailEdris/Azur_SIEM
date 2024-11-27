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
3. Copy [PowerShell Script](https://github.com/IsmailEdris/Azur_SIEM/blob/9afee62ba18f4adfa56b5ea84e6773b577c14b91/powershell%20_script.ps1) (authored by Josh Makador) and paste it into the VM's Powershell 
4. Choose New Script in Powershell ISE and paste script
5. Give it a name and save it to the desktop (log_exporter)

> PowerShell Scirpt

![script](https://camo.githubusercontent.com/1b1cd6cfaa0b93662bdad02cd069887dc686d37fe3f95c37840694b96f19c803/68747470733a2f2f692e696d6775722e636f6d2f4f437a624e41732e706e67)

6. Create an acount with [ipgeolocation.io](ipgeolocation.io)
> ipgeolocation allows you to track 1,000 API calls per day for free, however, you can upgrade to 150,000 API calls for $15/month. For this tutorial we will be using the free 1,000 API calls.

7. Once logged in, copy the API key and paste it into line 2 of the script.  ```$API_KEY = "<API key>"```
8. Click Save
9. To generate log data continually, run the PowerShell ISE script (green play button) in the virtual machine

>ipgeolocation Interface

![inter](https://camo.githubusercontent.com/11e3d26dc010ceaacf81df0f031993321155c5821a03ceefa2f7cbc250e7461c/68747470733a2f2f692e696d6775722e636f6d2f3765676f4869312e706e67)
> Data will be exported from Windows Event Viewer and imported into the IP Geolocation service by the script. The latitude and longitude will then be extracted, and a new log file called failed_rdp.log will be created in the location specified below: C:\ProgramData\failed_rdp.log

## Step 9: Make a custom Log
- To add the extra information from the IP Geolocation service to Azure Sentinel, create a custom log
1. Search "Run" in VM and type "C:\ProgramData"
2. Open file named "failed_rdp" hit **CTRL + A** to select all and **CTRL + C** to copy selection
3. On the host PC, open notepad and paste the information
4. Save to desktop as "failed_rdp.log" Note: make sure it's saved as a (.txt) text file. I had issues with formatting when saving in (.rtf) rich text format.
5. In Azure go to Log Analytics Workspaces -> Log Analytics workspace name (honeypot-law) -> Custom logs -> Add **custom log**

### Sample
- Select Sample log saved to Desktop (failed_rdp.log) and click Next
### Record delimiter
- Look over sample logs -> Click Next
## Collection paths
- Type: Windows
- Path: "C:\ProgramData\failed_rdp.log
## Details
- Name and describe the custom log (FAILED_RDP_WITH_GEO) before pressing the Next button
- Click Create

> Custom Log Interface

![custom](https://camo.githubusercontent.com/9747f35f18f1ebc2d125f9f26b66d6bd047b5a3526e754f70d432c8da00164fd/68747470733a2f2f692e696d6775722e636f6d2f344c53717268492e706e67)

## Step 10 : Query + Extract Fields from Custom Log
- Navigate to the newly established workspace (honeypot-law) in Log Analytics Workspaces -> Logs
- We then can run a query and extract the different data filtering by different fields such as latitude, longitude, destinationhost, etc.
- As of March 31st, 2023, Microsoft has disabled the creation of new custom fields and has migrated to KQL. You can learn more about it here
- Copy/Paste the following query into the query window and Run Query

```KQL
FAILED_RDP_WITH_GEO_CL 
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
| summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude
```
> KQL Query

![kql](https://camo.githubusercontent.com/fa8d218fd1c75db92c5735f377043103b63819302fb9958c7ddc4cb591e12fb8/68747470733a2f2f692e696d6775722e636f6d2f514a6c41494e392e706e67)


## Step 11: Create World Attack Map in Microsoft Sentinel
- Access Microsoft Sentinel to view the Overview page and available events
- Click on Workbooks and Add workbook then click Edit
- Delete default widgets (three dots -> remove)
- Click Add->Add query
- You can Copy/Paste the previous query or this one into the query window and Run Query

```kql
Failed_RDP_Geolocation_CL
| parse RawData with * "latitude:" Latitude ",longitude:" Longitude ",destinationhost:" DestinationHost ",username:" Username ",sourcehost:" Sourcehost ",state:" State ", country:" Country ",label:" Label ",timestamp:" Timestamp
| where DestinationHost != "samplehost"
| where Sourcehost != ""
| summarize event_count=count() by Sourcehost, Latitude, Longitude, Country, Label, DestinationHost

```
- When results appear, select Map from the Visualization drop-down box.
- Choose Map Settings to make additional adjustments

### Layout Settings
- Location info using: Latitude/Longitude
- Latitude: latitude
- Longitude: longitude
- Size by: event_count

### Color Settings
- Coloring Type: Heatmap
- Color by: event_count
- Aggregation for color: Sum of Values
- Color palette: Green to Red

### Metric Settings
- Metric Label: label
- Metric Value: event_count
- Click Apply button and Save and Close
- Save as "Failed RDP International Map" in the same region and under the resource group (honeypot-lab)
- Keep refreshing the map to show more inbound failed RDP attacks

> Only failed login attempts will be shown on the map, not any additional attacks the VM might be facing.

![internationa](https://camo.githubusercontent.com/30ff0df166aa43d4d83b8c251c8c24a6c2e4addd4b4dcee0ffece87305940646/68747470733a2f2f692e696d6775722e636f6d2f734251506a56792e706e67)

> Event Viewer demonstrating failed RDP login attempts. EVENT ID: 4625

![failed](https://camo.githubusercontent.com/335503d5dbee224e58719dbd95dec9ac678342b0aad132fd5e4e71f6b2669c4e/68747470733a2f2f692e696d6775722e636f6d2f536c436c7979492e706e67)

> Data is also presented in the PowerShell script using ipgeolocation.io (third pary API)

![third](https://camo.githubusercontent.com/caf562ed9a65fbb9c65a674d2ac8fa7c639870a019f5561f7db41edb8ec6cac2/68747470733a2f2f692e696d6775722e636f6d2f35513038666a4c2e706e67)

### Step 12: Creating SIEM Alerts

In this step we are going to create SIEM alerts that will notify us every time a failed login attempt occurs. We are going to see these alerts through our phone. So we will get a push notification from the Azure App every time someone tries to login. For this step you will need the Azure App downloaded on your phone. Make sure to login to the same Azure account on your phone and your PC so you can get the notifications. 

1.	Search up “Log Analytics Workspace” in the search bar.
2.	Click on the workspace that you made for this project. In this case “law-honeypot”
3.	On the left hand side look for “Alerts” and click on it. 
4.	Once you are on the Alerts page, at the top click “Create” and then “Alert Rule.”
5.	You should now be in the page where you create alert rules. For the “Signal name” select Custom log search. 
6.	Copy and Paste the code from the previous step in the “Search Query” entry. 
7.	Run the query and then click “continue editing alert”
8.	Change the “Aggregation Granularity” to 1 minute or 5 minutes.
9.	Under “Alert Logic” in the “Operator” box click Greater than. The “Threshold Value” should be 0. And the “Frequency of Evaluation” should be 1 minute or 5 minutes. This means that you will get an alert every 1 minute if the login attempt failed is greater than 0. So basically, if anybody fails to login then you will get an alert. 
10.	Click “Next” and go to actions. Click Create New action Group.
11.	Select the right resource group (HoneyPotLab), the right region (US East), and then name the action group. 
12.	Click next to go to the Notifications tab. This tab is where you select what type of notification you want to receive. You can set the notification type however you want. Whether that is an email, text message, push notification, or voice notification. However, in this case we are doing a push notification. 
13.	Under “Notification type” select Email/SMS message/Push/Voice
14.	Name the Notification. I named it “Failed_Login”
15.	At the right hand side select “Azure Mobile App Notification” and enter the email address for the account. MAKE SURE IT IS THE SAME ACCOUNT THAT THE ALERTS ARE ON.  
16.	Review and Create it.

###  Step 13 : Shut Down Resources

>IMPORTANT: DO NOT SKIP THIS STEP OR ELSE YOU WILL GET CHARGED MONEY

- Type "Resource groups" in the serach bar -> name of resource group
- Key in the name of the resource group (honeypot-lab) to verify removal of resources
- Select the Apply force delete for selected Virtual machines and Virtual machine scale sets box
- Click Delete

![delte](https://camo.githubusercontent.com/a1b196dcb1e839bf547b8e967043afc7ca07bac1bbd1a4512084b2d02c94dbbc/68747470733a2f2f692e696d6775722e636f6d2f6b54736468354d2e706e67)









