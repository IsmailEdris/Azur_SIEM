# Azure Sentinel SIEM Implementation 
![honeypot](https://files.oaiusercontent.com/file-BfhHUgBfosTsaKP4PwVmux?se=2024-11-27T19%3A59%3A21Z&sp=r&sv=2024-08-04&sr=b&rscc=max-age%3D604800%2C%20immutable%2C%20private&rscd=attachment%3B%20filename%3D6e609c99-c8bd-4819-b1a1-1475280560e9.webp&sig=qruy4PZV0K1I5uD%2Ba41owxvAy%2BvGZ7DBdDxo%2BMuzsvQ%3D)
## Description
This guide provides an overview of setting up and using an educational Homelab with Microsoft Azure Sentinel. Honeypots, which are decoy systems designed to attract and monitor malicious activity, play a key role in gaining insights into potential threats and attackers' tactics. A Security Information and Event Management (SIEM) solution facilitates the collection, analysis, and response to security events in real-time. Azure Sentinel, a cloud-native SIEM solution from Microsoft, enables a comprehensive view of security events while automating threat detection and response. This Homelab showcases the integration of cybersecurity and innovation, offering the ability to track and log attacks from across the globe while visualizing them on an interactive attack map. Through this setup, key aspects of modern cyber defense and threat monitoring are explored. A third party API [ipgeolocation](https://ipgeolocation.io/) is also used to track API queries. 

## Index
1. [Creating Azure Account](https://camo.githubusercontent.com/a99e124ad9661c14f6657210f64603c465e1bc52ba3e5cf3c6c6c515af1b9c00/68747470733a2f2f692e696d6775722e636f6d2f427a6d6a5661342e706e67)
2. [Creating a Log Analytics Workspace (LAW) and setting up Microsoft Defender](LINK)
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

## Step 2: 






