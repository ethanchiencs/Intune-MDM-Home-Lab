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

<img width="1127" height="622" alt="Screenshot 2026-08-31 182314" src="https://github.com/user-attachments/assets/19606dbf-e3d9-409c-8884-0baaf7967cd4" />
<img width="912" height="852" alt="Screenshot 2026-08-31 175633" src="https://github.com/user-attachments/assets/7560245b-20ee-4673-922f-bf0f53ed5f66" />

### 2. Building the Test VM
Created a Generation 2 VM in Hyper-V with virtual TPM and Secure Boot enabled — both required for modern Windows enrollment scenarios. Initially provisioned a Windows Server 2022 Evaluation image by mistake, identified that Server editions don't support the client OOBE work/school account enrollment flow, and corrected course by re-provisioning with a Windows 11 Enterprise Evaluation ISO.

<img width="704" height="650" alt="Screenshot 2026-08-30 170709" src="https://github.com/user-attachments/assets/d950f830-147e-4719-9785-b6094268f3b7" />

### 3. Entra ID Join
At first boot, signed in using the test user's work account credentials via the "Sign in with a work or school account" option, triggering a Microsoft Entra ID join. Verified the join using:

```
dsregcmd /status
```

confirming `AzureAdJoined: YES`.

<img width="936" height="767" alt="Screenshot 2026-08-30 173507" src="https://github.com/user-attachments/assets/d931edfb-0bc6-4cf8-8662-b30b6026965e" />
<img width="1007" height="396" alt="Screenshot 2026-08-30 185353" src="https://github.com/user-attachments/assets/554c54d0-5b7c-4fee-ad61-c6d99cf56395" />

### 4. Troubleshooting MDM Auto-Enrollment
While the Entra ID device join succeeded on the first attempt, automatic MDM enrollment into Intune did not trigger immediately. Troubleshooting steps taken:

- Confirmed the MDM user scope setting (**Entra admin center > Mobility (MDM and MAM) > Microsoft Intune**) was set to include the test user, correcting it from its default restrictive state.
- Verified Entra ID P2 and Intune licensing were correctly assigned to the test user.
- Attempted manual enrollment via **Settings > Accounts > Access work or school > Enroll only in device management**, using the Intune discovery endpoint (`https://enrollment.manage.microsoft.com/enrollmentserver/discovery.svc`) after automatic discovery initially failed.
- Investigated and resolved a Windows enrollment error (`801901f4`) tied to stale device objects and MDM scope configuration by clearing duplicate device records from prior enrollment attempts in **Entra admin center > Devices**.
- Confirmed the "Users may join devices to Microsoft Entra ID" device setting was not restricting the test account.
- Rebuilt the test VM with the corrected MDM user scope already in place, then re-ran the OOBE work/school account sign-in. This time, MDM enrollment completed successfully alongside the Entra ID join.

Verified full enrollment using:

```
dsregcmd /status
```

confirming both `AzureAdJoined: YES` and a populated `MdmUrl`, and confirmed the device management connection in **Settings > Accounts > Access work or school**.

<img width="931" height="448" alt="Screenshot 2026-08-31 175242" src="https://github.com/user-attachments/assets/8ab03fb0-ba23-4403-9111-1416c6498528" />
<img width="1689" height="868" alt="Screenshot 2026-08-31 175424" src="https://github.com/user-attachments/assets/4289134e-d17d-4ee6-a107-c470048a5c0c" />

This troubleshooting reflects real-world enrollment issues IT support staff encounter when managing MDM environments — configuration settings across multiple admin consoles (Entra ID, Intune, Microsoft 365 admin center) all need to align for enrollment to succeed, and diagnosing failures requires checking device state, licensing, and tenant-level policy in parallel rather than assuming a single cause.

### 5. Compliance Policy
Created a Windows compliance policy in the Intune admin center requiring:
- Firewall enabled
- Antivirus and Microsoft Defender Antimalware active
- Password complexity, minimum length, expiration, and reuse restrictions
- Disk encryption (BitLocker)

Assigned the policy to the test user/device and reviewed compliance evaluation results.

**Result:** 7 of 8 settings evaluated as **Compliant** — firewall, antivirus, Defender, and all password policy settings passed. **Disk encryption returned a remediation error** (code `2016281112`).

<img width="845" height="572" alt="Screenshot 2026-08-30 191853" src="https://github.com/user-attachments/assets/22e04da6-2500-4a11-9276-2adbdb49965a" />
<img width="1027" height="609" alt="Screenshot 2026-08-31 175740" src="https://github.com/user-attachments/assets/4b0a4350-6be6-416e-9e3b-aec13ad06e32" />

This partial result is expected rather than a configuration mistake: enforcing BitLocker inside a Hyper-V VM relies on a virtualized TPM, and BitLocker policy remediation is known to behave inconsistently in nested/virtualized environments compared to physical hardware. The compliance policy itself was configured and evaluated correctly — the error reflects a virtualization limitation, not a policy or enrollment failure.

## Key Takeaways
- Microsoft Entra ID join and Intune MDM enrollment are related but distinct processes — a successful device join does not guarantee MDM management is active, and troubleshooting requires checking enrollment scope, licensing, and device state separately.
- Enrollment issues often span multiple admin consoles (Entra ID, Intune, Microsoft 365 admin center), reinforcing the importance of understanding how Microsoft's cloud identity and device management services interconnect.
- Diagnostic tools like `dsregcmd /status` and Intune's device compliance views are essential for verifying device state at each stage of the management lifecycle.
- Not every compliance failure indicates a misconfiguration — understanding *why* a policy setting fails (e.g., virtualized TPM limitations with BitLocker) is as important as getting a policy to evaluate as compliant.

## Relevance to IT Support Roles
This lab mirrors day-to-day tasks in help desk and IT support environments: enrolling new devices into management, applying and troubleshooting compliance policies, and diagnosing enrollment failures that end users would otherwise report as "my laptop won't connect to work" tickets.
