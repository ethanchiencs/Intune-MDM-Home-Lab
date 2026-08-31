# Microsoft Intune MDM Home Lab

## Overview
This project documents the deployment and configuration of a Microsoft Intune environment for managing Windows endpoints, built as a hands-on complement to my Active Directory and Pi-hole DNS home labs. The goal was to gain practical experience with cloud-based device management (MDM), a core skill for modern IT help desk and support roles, using a free Microsoft 365 trial tenant and a Hyper-V test VM.

## Environment
- **Tenant:** Microsoft 365 trial tenant with Microsoft Entra ID P2 and Intune licensing
- **Virtualization:** Hyper-V (Generation 2 VM, virtual TPM enabled)
- **Test device:** Windows 11 Enterprise Evaluation VM
- **Admin tools:** Microsoft Intune admin center, Microsoft Entra admin center, Microsoft 365 admin center

## What This Project Covers
1. Provisioning a Microsoft 365 trial tenant with Intune and Entra ID P2
2. Creating and licensing a test user for device management
3. Building a Windows test VM in Hyper-V configured for Entra ID join (Generation 2, TPM enabled)
4. Joining a device to Microsoft Entra ID and troubleshooting MDM auto-enrollment
5. Creating and assigning a device compliance policy (disk encryption, password requirements)
6. Reviewing device compliance status and configuration in the Intune admin center

## Setup Process

### 1. Tenant and Licensing
Signed up for a Microsoft 365 trial tenant and confirmed Intune and Microsoft Entra ID P2 licensing was active under **Billing > Your products**. Created a dedicated test user account and assigned the Intune/EMS license to it under **Users > Active users > Licenses and apps**.

*[Screenshot: Active licenses in Microsoft 365 admin center]*
*[Screenshot: Test user with license assigned]*

### 2. Building the Test VM
Created a Generation 2 VM in Hyper-V with virtual TPM and Secure Boot enabled — both required for modern Windows enrollment scenarios. Initially provisioned a Windows Server 2022 Evaluation image by mistake, identified that Server editions don't support the client OOBE work/school account enrollment flow, and corrected course by re-provisioning with a Windows 11 Enterprise Evaluation ISO.

*[Screenshot: Hyper-V VM settings — Generation 2, TPM enabled]*

### 3. Entra ID Join
At first boot, signed in using the test user's work account credentials via the "Sign in with a work or school account" option, triggering a Microsoft Entra ID join. Verified the join using:

```
dsregcmd /status
```

confirming `AzureAdJoined: YES`.

*[Screenshot: Work/school account sign-in during OOBE]*
*[Screenshot: dsregcmd /status output confirming Entra ID join]*

### 4. Troubleshooting MDM Auto-Enrollment
While the Entra ID device join succeeded, automatic MDM enrollment into Intune initially did not trigger. Troubleshooting steps taken:

- Confirmed the MDM user scope setting (**Entra admin center > Mobility (MDM and MAM) > Microsoft Intune**) was set to include the test user, correcting it from its default restrictive state.
- Verified Entra ID P2 and Intune licensing were correctly assigned to the test user.
- Attempted manual enrollment via **Settings > Accounts > Access work or school > Enroll only in device management**, using the Intune discovery endpoint (`https://enrollment.manage.microsoft.com/enrollmentserver/discovery.svc`) after automatic discovery failed.
- Investigated and resolved a Windows enrollment error (`801901f4`) tied to stale device objects and MDM scope configuration by clearing duplicate device records from prior enrollment attempts in **Entra admin center > Devices**.
- Confirmed the "Users may join devices to Microsoft Entra ID" device setting was not restricting the test account.

This troubleshooting reflects the kind of real-world enrollment issues IT support staff encounter when managing MDM environments — configuration settings across multiple admin consoles (Entra ID, Intune, Microsoft 365 admin center) all need to align for enrollment to succeed, and diagnosing failures requires checking device state, licensing, and tenant-level policy in parallel.

*[Screenshot: MDM user scope setting]*
*[Screenshot: Enrollment error encountered]*

### 5. Compliance Policy
Created a Windows compliance policy in the Intune admin center requiring:
- Disk encryption (BitLocker)
- Minimum password length

Assigned the policy to the test user/device and reviewed compliance evaluation results.

*[Screenshot: Compliance policy configuration]*
*[Screenshot: Device compliance status]*

## Key Takeaways
- Microsoft Entra ID join and Intune MDM enrollment are related but distinct processes — a successful device join does not guarantee MDM management is active, and troubleshooting requires checking enrollment scope, licensing, and device state separately.
- Enrollment issues often span multiple admin consoles (Entra ID, Intune, Microsoft 365 admin center), reinforcing the importance of understanding how Microsoft's cloud identity and device management services interconnect.
- Diagnostic tools like `dsregcmd /status` and Intune's device compliance views are essential for verifying device state at each stage of the management lifecycle.

## Relevance to IT Support Roles
This lab mirrors day-to-day tasks in help desk and IT support environments: enrolling new devices into management, applying and troubleshooting compliance policies, and diagnosing enrollment failures that end users would otherwise report as "my laptop won't connect to work" tickets.
