# Building and Securing a Honeynet in Microsoft Azure

### Objective

The objective of this project was to build a honeynet in Microsoft Azure open to the public internet, run it for 24 hours to observe traffic, and then secure it to comply with NIST 800-53. A diagram of the honeynet can be seen below.

### Skills Learned

- Provisioning resources in the cloud
-	Secure cloud design and best practices
-	Managing virtual machines using RDP and SSH
-	Logging and monitoring using Windows, Linux, and Azure
-	Understanding and using KQL



### Tools Used

- Microsoft Azure
  - Active Directory
  -	Log Analytics Workspace
  -	Sentinel
  -	Key Vault
  -	Storage Account
-	Windows
    -	Powershell
    -	Event Viewer
    -	Registry
-	SQL Server Studio
-	Ubuntu Server 20.04

### Honeynet Diagram

![project-1](https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/d2acfa3c-df4a-44d7-9315-6824e6ba690a)

### Environment Set-up

The steps that follow are a high-level overview. The intention is to minimize tedium while still giving a clear idea of the tasks involved.

1. Create Windows and Linux virtual machines.

<img width="1260" alt="Azure Intro Step 1 - Creating VMs" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/104520c2-bd8e-4ef6-b876-55072c1b79c2">

2. Open virtual machine network security groups (NSGs) to all public traffic. Shown here on Windows VM.

<img width="1269" alt="Azure Intro Step 3 - Create NSG rule opening both machines to inbound traffic (Windows)" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/0b60cdb5-8580-41c3-8d7d-a9d7350577df">

3. Next, I RDP into the Windows VM and turn off the firewall.

<img width="1046" alt="Installing MSQL Server - Step 2 RDP to Windows VM and Turn off Firewall" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/d81d6833-006d-4e91-bf25-36f2b2df17e8">

4. Still inside the Windows VM, I install SQL server and Server Management Studio to create and easily manage another attack surface. I edit log settings in Management Studio so authentication logs are collected any time someone attempts to log in and I edit the registry to make sure these are sent to Event Viewer.

<img width="1319" alt="Installing MSQL Server - Step 8 Log into Server using Server Management Studio" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/328201ca-b125-41ad-b371-f4e399fd87e9">

<img width="1319" alt="Installing MSQL Server - Step 10 Confirm logs generated in Event Viewer" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/87e25186-27bf-4120-85dd-b6c68401e593">

5. I create a Log Analytics Workspace, so that logs created by the Windows and Linux machines including the SQL server. are available directly in Azure. Logs from the NSGs are also stored here. This was quite an involved process. You can see the end-result is I am able to query events in Azure from the Log Analytics workspace.

<img width="1267" alt="Enable Log Collection for VMs and Network Security Groups - Step 12 Test query NSG logs" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/e63beb57-9a46-4c3c-98a5-4cc77814877a">

<img width="1267" alt="Enable Log Collection for VMs and Network Security Groups - Step 13 Test query Windows logs" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/e75f2350-b960-468f-9265-3b91e5be48da">

<img width="1267" alt="Enable Log Collection for VMs and Network Security Groups - Step 14 Test query Linux" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/a9315894-1716-4d3a-9081-e8f138e20539">

6. With traffic logs being sent to Azure, it would be good to create a visualization so we can see what hackers get up to over the next 24 hours. Here are the maps I came up with showing RDP attempts, Linux SSH activity, traffic passing through the NSGs and authentication attempts against the SQL server. The results are shown below along with the query used to generate one of the maps. The hackers were very busy!

<img width="1267" alt="World Maps Construction Step 2 - Paste in Advanced Editor" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/fdc1c361-2aa8-41e2-8052-3f1202b1d59a">

<img width="1258" alt="World Maps Construction Step 3 - Linux Map" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/4066750c-c323-4705-ac30-0e7ef99a9d53">

<img width="1258" alt="World Maps Construction Step 4 - MSSQL Map" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/2e9e82a6-cdfd-4efc-a0a9-0abc1f6e2044">

<img width="1258" alt="World Maps Construction Step 5 - NSG Map" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/8ab3cc12-a4cc-47d6-82d2-fde27cdb9488">

<img width="1258" alt="World Maps Construction Step 6 - RDP Map" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/6a48b3fe-e91c-4a3b-9d8d-61a78ac8a44b">

7. I also imported a ruleset into Sentiel Analytics that automatically generates alerts for events such as privilege escalation, secret viewing in Key Vault, editing files in Storage Blobs, and Brute force attacks


<img width="1267" alt="Automatic Alert Import Step 3 - Confirm rules show up properly" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/3131f117-b0f1-4547-a3d9-0c6a3cb3fc69">

### Hardening the Environment

Now that the VMs have been attacked for 24 hours, its time to implement proper security measures and compare the results!

1. Step one is to enable Microsoft Defender for Cloud's Regulatory Compliance feature for NIST 800-53. This will recommend lots of different measures we can take to make our environment more secure but we will focus on SC-7, Boundary protection. This takes a few days to activate.

<img width="1268" alt="Regulatory Compliance - Step 2 Enabling NIST Compliance in Defender for Cloud" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/13299bb4-75f7-409c-9828-233fa6e70960">

2. One simple measure will cut down on the vast majority of malicious traffic. We edit our NSG to only allow traffic from my IP address.

<img width="1258" alt="Regulatory Compliance - Step 1 Changing NSGs to allow only traffic from my IP" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/0e43702a-168f-4006-8add-a70bb866f147">

3. Next, we create Private Links to my resources. This makes sure that only machines inside my subnet can access the resources, ensuring they cannot be accessed through the public internet. Here's an example of doing this on a my storage account. 

<img width="1268" alt="Azure Private Link and Firewall - Step 3 - Block Public Access to Storage Account" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/7dc40670-a760-4d56-acf2-8cbd3debcf40">

4. Finally, for good measure, we put the entire subnet behind its own NSG.

<img width="1266" alt="Azure Private Link and Firewall - Step 6 - Assign NSG to Subnet" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/4dc74c51-c4b4-4ea1-82d2-e3aea8bed140">

### Results

The table below compares the pre-hardened with the post-hardened environment. You can see that virtually all malicious traffic has been eliminated.

<img width="537" alt="Screenshot 2024-06-08 at 6 54 37 PM" src="https://github.com/benjuliano1/Building-and-Securing-a-Honeynet-in-Microsoft-Azure/assets/170282669/5642f2c7-49da-4cad-a064-627be9b5a956">



