# Moving an Azure Virtual Machine to Availability Zones – Step-by-Step Experience
<br>

## High availability is no longer optional for production workloads. Azure Availability Zones provide built‑in infrastructure redundancy, but what if your existing Virtual Machine was deployed without any availability configuration?
In this article, I’ll walk through my hands‑on experience of moving a single-instance Azure VM from a regional deployment to an Availability Zone within the same Azure region.

<br>

## Scenario Overview
I had a Virtual Machine named testvmun01 deployed in the UAE North region.
- Single instance VM
- No Availability Set
- No Availability Zone
- No infrastructure redundancy

In short, the VM was running with zero resiliency at the infrastructure level.
The goal was to move this VM to an Availability Zone to achieve higher availability and fault isolation, without rebuilding the VM from scratch.

<br>

## Current State – No Availability Zone
When I checked the VM configuration:
- testvmun01 had no availability options configured
- It was deployed as a regional VM
This can be verified from:
Azure Portal → Virtual Machine → Availability + scaling
<img width="940" height="506" alt="image" src="https://github.com/user-attachments/assets/982b884b-b80a-489c-ba0c-d25c25bfa2cb" />
<img width="523" height="239" alt="image" src="https://github.com/user-attachments/assets/25453543-738e-4507-8f3d-4059c5511494" />

<br>
<br>

## 1. Initiate the Availability Zone Move

Go to **Azure Portal** \
Select **testvmun01** \
Navigate to **Availability + scaling** \
Click **Edit** \
Select the **Target Availability Zone** \
Click **Next**

At this point, Azure begins the Managed Service Identity (MSI) authentication process.

⏳ The MSI authentication process takes a few minutes to complete.

<img width="940" height="312" alt="image" src="https://github.com/user-attachments/assets/fc77e0b9-0231-4b23-b5e0-253b1730c08e" />


## 2. Validation Phase
Once authentication completes:
- Azure starts validation
- A temporary resource group is created automatically: **ZonalMove-MC-RG-SourceRegion**
- Example: **ZonalMove-MC-RG-UAENorth**

<img width="940" height="290" alt="image" src="https://github.com/user-attachments/assets/fe1e7574-8e21-4034-b089-389e381fb2b9" />

<img width="316" height="53" alt="image" src="https://github.com/user-attachments/assets/6651fbc4-fddd-4846-900c-6ab2e4c24167" />

## 3. Move
Once the validation is completed > click **Move**

<img width="940" height="500" alt="image" src="https://github.com/user-attachments/assets/e232ee8f-085b-4ae5-bb9e-c1b9dd8d7452" />

## 4. What Happens During the Move?
Azure performs a copy-based move, not an in-place conversion. Here’s exactly what happens behind the scenes:

✅ **Virtual Machine**

A new VM is created in the selected Availability Zone
The source VM is stopped after the move
Source VM ARM ID is not retained

✅ **Resource Group**

A new resource group is created by default
Reason: you cannot have two VMs with the same name in the same resource group
You can later move the VM to an existing resource group if needed

✅ **Network Interface (NIC)**

A new NIC is created for the zonal VM
The source NIC is stopped
Source NIC ARM ID is not retained

✅ **Disks**

OS and data disks are recreated with new disk names
Disks are attached to the new zonal VM

✅ **Load Balancer**

Basic SKU Load Balancer is not supported with Availability Zones
A Standard SKU Load Balancer is created by default
You can modify it or attach an existing Standard Load Balancer

✅ **Public IP**

Basic SKU Public IPs are not retained
A Standard SKU Public IP is created by default
You can select an existing Standard Public IP if required

