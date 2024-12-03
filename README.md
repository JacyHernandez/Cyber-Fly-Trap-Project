# Cyber-Fly-Trap-Project

# Setting Up a Honeypot on Azure: Step-by-Step Guide

## 1. Create an Azure Account
- **Sign up for a free Microsoft Azure account** using [this link](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=azurefreeaccount).
- 
<img width="468" alt="Picture1" src="https://github.com/user-attachments/assets/c4d2e7c7-3421-4af7-884e-39dac01895ca">

## 2. Setting Up Your Honeypot Virtual Machine
- **Sign in** to the [Azure Portal](https://portal.azure.com).
- Search for **"Virtual Machine"** and click on **"Create"** > **"Azure Virtual Machine"**.

- 
<img width="379" alt="Picture2" src="https://github.com/user-attachments/assets/78ac7aff-6b53-4d3f-8d77-96d39569f5ab">

### Project Details:
- **Resource Group**: Create a new group named `honeypot-project` (helps organize related resources).
- **VM Name**: `honeypot-vm`
- **Region**: `West US 2`
- **Availability**: No infrastructure redundancy required.
- **Security Type**: Standard.
- **Image**: Windows 10 Pro, version 22H2 - x64 Gen2.
- **VM Architecture**: x64.
- **Size**: `Standard_DS1_v2` (1 vcpu, 3.5 GiB memory).

- 
<img width="468" alt="Picture3" src="https://github.com/user-attachments/assets/f098a866-3e55-4c6b-b89a-01c37a858f4a">

### Administrator Account:
- Set a username and password (used for VM login).

### Inbound Port Rules:
- Allow HTTP (80), HTTPS (443), SSH (22), RDP (3389).

### Licensing:
- Confirm the checkbox and proceed to **Next: Disks**.
- Select Next: **Networking**
- 
<img width="468" alt="Picture4" src="https://github.com/user-attachments/assets/3dab94e5-4c15-4ea1-86b7-0be00070bc18">

### Networking:
- NIC network security group: Advanced -> **Create new**
- By clicking on the three dots, delete Inbound rules (1000: default-allow-rdp)
- Click on **+Add an inbound rule**
- On **Destination port ranges**: input * (wildcard for anything)
- Protocol: Any
- Action: Allow
- Priority: 100 (low)
- Name: Anything (**allow-any-inbound**)
- Click on “Add” then on **OK**
- Select **Review + Create**

## 3. Provisioning a Log Analytics Workspace

<img width="468" alt="Picture5" src="https://github.com/user-attachments/assets/8293e790-4aea-4637-a8f7-59eaf636456e">

- Search for **"Log analytics workspaces"** in the Azure Portal.
- Click **"Create"** and select the same resource group (`honeypot-project`).
- Name the workspace `honeypot-law` and choose the region `West US 2`.
- Click **Review + Create** and wait for deployment.

## 4. Setting Up Microsoft Defender for Cloud

<img width="468" alt="Picture6" src="https://github.com/user-attachments/assets/aaeea996-5455-40d6-87b0-f07887321e27">

- Search for **"Microsoft Defender for Cloud"** and navigate to **Environment settings** under the Management drop down on the left hand side.
- Select **Subscription Name** > **Log Analytics Workspace Name** (`honeypot-law`).

### Defender Plans:

<img width="468" alt="Picture7" src="https://github.com/user-attachments/assets/e72c17fe-2127-413e-92be-5c92feb94e4b">

- **Foundational CSPM (Cloud Security Posture Management)**: ON
- **Servers**: ON
- **SQL servers on machines**: OFF
- Click **Save**.

### Data Collection:

<img width="468" alt="Picture8" src="https://github.com/user-attachments/assets/cca18804-5489-4f90-bd71-74d056db2b25">

- Select **"All Events"** and click **Save**.

## 5. Linking VM to Log Analytics Workspace

<img width="468" alt="Picture9" src="https://github.com/user-attachments/assets/2239f4ed-a956-40e5-9cee-824bc3c24c02">

- Search for **"Log Analytics workspaces"** and select `honeypot-law`.
- Go to **Virtual machines** > `honeypot-vm` and click **Connect**.

## 6. Setting Up Microsoft Sentinel

<img width="468" alt="Picture10" src="https://github.com/user-attachments/assets/f2d2d0e3-9cf0-4d47-ad8a-fa36b1ebaae9">

- Search for **"Microsoft Sentinel"** and click **"Create"**.
- Choose `honeypot-law` as the Log Analytics Workspace and click **Add**.

## 7. Turning Off the VM's Firewall

<img width="378" alt="Picture11" src="https://github.com/user-attachments/assets/cb696db1-e0ff-45e3-87c6-896414537793">

- locate the honeypot VM (honeypot-project) under Virtual Machines
- Copy the IP address from the VM
- Using the credentials from step 2, access the virtual machine through Remote Desktop Protocol **(RDP)** on your computer and use the ip address.
- Open **Windows Defender Firewall with Advanced Security**.
- Turn off the firewall for **Domain, Private, and Public profiles**.

## 8. Automating the Security Log Exporter
- Launch **PowerShell ISE** in the VM.
- Copy the PowerShell script from [Josh Madakor’s GitHub](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) and paste it into PowerShell ISE.
- Save and run the script to export logs continually.

## 9. Creating a Custom Log in Log Analytics Workspace
- Go to `C:\ProgramData` and copy the `failed_rdp.log` file.
- In Azure, navigate to **Log Analytics Workspaces** > `honeypot-law` > **Custom logs** > **Add custom log**.
- Configure and click **Create**.

## 10. Querying and Extracting Fields from Custom Logs
- Run a query in **Log Analytics** to filter and extract data:
```kusto
failed_rdp_with_geo_CL 
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

 


