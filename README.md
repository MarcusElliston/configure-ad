<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory logo"/>
</p>

# On-Premises Active Directory Deployed in the Cloud (Azure)

This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.

---
---

## Environments and Technologies Used

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

---

## Operating Systems Used

- Windows Server 2022
- Windows 10 (21H2)

---

## High-Level Deployment and Configuration Steps

- Set up Domain Controller and Client VMs in Azure
- Install and promote Active Directory Domain Services
- Create admin and standard user accounts in Active Directory
- Join the client VM to the domain and verify login access

---

## Deployment and Configuration Steps

### Step 1: Create the Domain Controller and Client VMs in Azure

<img width="1527" height="704" alt="Gemini_Generated_Image_7xmp9h7xmp9h7xmp" src="https://github.com/user-attachments/assets/2186cf5c-2373-439d-ab81-560fa8ce25ac" />

Log into the Azure Portal and create a new Resource Group. Inside it, create two Virtual Machines: one running **Windows Server 2022** (this will be your Domain Controller, name it `DC-1`) and one running **Windows 10** (this will be your client machine, name it `Client-1`). Make sure both VMs are on the **same virtual network and subnet**. After creation, set DC-1's private IP address to **static** so it doesn't change — go to the VM's Network Interface settings and set the IP allocation to Static.

---

### Step 2: Ensure Connectivity Between Client and Domain Controller

<img width="1664" height="986" alt="1000030318" src="https://github.com/user-attachments/assets/c893454a-34ef-471b-9587-952641723bd1" />

Log into `Client-1` via Remote Desktop and open Command Prompt. Attempt to ping DC-1's private IP address (e.g. `ping 10.0.0.4`). If the ping times out, log into `DC-1` and enable **ICMPv4** through the Windows Firewall by opening **Windows Defender Firewall with Advanced Security > Inbound Rules** and enabling the Core Networking Diagnostics (ICMPv4) rules. Return to Client-1 and confirm the ping now succeeds.

---

### Step 3: Install Active Directory Domain Services

<img width="1645" height="980" alt="1000030307" src="https://github.com/user-attachments/assets/44bbe348-4896-4f03-b1c8-dbb927767a5d" />

Log into `DC-1` and open **Server Manager**. Click **Add Roles and Features** and install **Active Directory Domain Services**. Once installed, click the notification flag in Server Manager and select **Promote this server to a domain controller**. Choose **Add a new forest** and set a domain name (e.g. `mydomain.com`). Set a Directory Services Restore Mode password, complete the wizard, and let the server restart. Log back in as `mydomain.com\labuser` (or whatever admin account you created).

---

### Step 4: Create Admin and User Accounts in Active Directory

<img width="1785" height="1042" alt="1000030309" src="https://github.com/user-attachments/assets/709ae1ef-b066-485b-a862-5e534ba2647d" />

On `DC-1`, open **Active Directory Users and Computers (ADUC)**. Create two Organizational Units (OUs): `_EMPLOYEES` and `_ADMINS`. Inside `_ADMINS`, create a new user (e.g. Jane Doe, username: `jane_admin`) and add her to the **Domain Admins** security group. Log out of DC-1 and log back in as `mydomain.com\jane_admin` — use this account going forward instead of the default admin account.

---

### Step 5: Join Client-1 to the Domain

<img width="1655" height="1004" alt="1000030313" src="https://github.com/user-attachments/assets/a251775c-6440-49af-be3d-771ebc5b2b98" />

Log into the Azure Portal and update `Client-1`'s DNS settings to point to **DC-1's private IP address** (under Networking > DNS Servers). Restart Client-1. Then log into Client-1, go to **System > Rename this PC (Advanced) > Change**, select **Domain**, and enter `mydomain.com`. Enter the admin credentials when prompted (e.g. `mydomain.com\jane_admin`). The machine will restart and is now joined to the domain. Verify in ADUC on DC-1 that Client-1 appears under the **Computers** container.

---

### Step 6: Set Up Remote Desktop for Non-Admin Users

<img width="1634" height="995" alt="1000030322" src="https://github.com/user-attachments/assets/29ca723e-fa56-4aa6-8b10-74c9b4a6583c" />

Log into `Client-1` as `mydomain.com\jane_admin`. Open **System Properties > Remote Desktop** and click **Select users that can remotely access this PC**. Add **Domain Users** to allow all standard domain users to log into Client-1 via Remote Desktop. In a real environment this would typically be done with Group Policy, but this manual method works for lab purposes.

---

### Step 7: Create Additional Users with PowerShell and Test Login

<img width="1647" height="995" alt="1000030324" src="https://github.com/user-attachments/assets/912de0e8-a755-4127-be07-b1d92070517a" />

Log into `DC-1` as `jane_admin` and open **PowerShell ISE as Administrator**. Run a script to bulk-create user accounts in the `_EMPLOYEES` OU (your instructor will provide this script, or you can write one that creates users with a default password). Once the script completes, open ADUC and confirm the accounts appear in `_EMPLOYEES`. Then attempt to log into `Client-1` using one of the newly created domain user accounts to verify everything is working end-to-end.
