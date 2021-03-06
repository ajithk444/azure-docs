---
title: Azure Backup support matrix for Azure VM backup
description: Provides a summary of support settings and limitations when backing up Azure VMs with the Azure Backup service.
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 02/17/2019
ms.author: raynew
---

# Support matrix for Azure VM backup
You can use the [Azure Backup service](backup-overview.md) to back up on-premises machines and workloads, and Azure VMs. This article summarizes support settings and limitations when backing up Azure virtual machines (VMs) with Azure Backup.

Other support matrices:

- [General support matrix](backup-support-matrix.md) for Azure Backkup
- [Support matrix](backup-support-matrix-mabs-dpm.md) for Microsoft Azure Backup server/DOM backup
- [Support matrix](backup-support-matrix-mars-agent.md) for backup with the MARS agent



## Supported scenarios

Here's how you can back up and restore Azure VMs with the Azure Backup service.


**Scenario** | **Backup** | **Agent** |**Restore** 
--- | --- | --- | ---
**Direct backup of Azure VMs** | Back up the entire VM  | No agent is needed on the Azure VM. Azure Backup installs and uses an extension to the [Azure VM agent](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows) running on the VM. | Restore as follows:<br/><br/> - **Create a basic VM**. This is useful if the VM has no special configuration such as mutiple IP addresses.<br/><br/> - **Restore the VM disk**. Restore the disk. Then attach it to an existing VM, or create a new VM from the disk using PowerShell.<br/><br/> - **Replace VM disk**. If a VM exists and it uses managed disks (unencrypted), you can restore a disk and use it to replace an existing disk on the VM.<br/><br/> - **Restore specific files/folders**. You can restore files/folders from a VM, instead of the entire VM.  
**Direct backup of Azure VMs (Windows only)** | Back up specific files/folders/volume | Install the [Microsoft Azure Recovery Services (MARS) agent](backup-azure-file-folder-backup-faq.md).<br/><br/> You can run the MARS agent alongside the backup extension for the Azure VM agent to back up the VM at file/folder level. | Restore specific folders/files.
**Back up Azure VM to backup server** |  Back up files/folders/volumes; system state/bare metal files; app data to System Center DPM or Microsoft Azure Backup Server (MAB server).<br/><br/> DPM/MABS then backs up to the backup vault | Install the MABS/DPM protection agent on the VM. The MARS agent is installed on DPM/MABS.| Restore files/folders/volumes; system state/bare metal files; app data. 

Learn more about backup using a backup server(backup-architecture.md#architecture-back-up-to-dpmmabs), and [support requirements](backup-support-matrix-mabs-dpm.md).


## Supported backup actions

**Action** | **Support**
--- | ---
Enable backup when you create a Windows Azure VM | Supported for:  Windows Server 2016 (Datacenter/Datacenter Core); Windows Server 2012 R2 DataCenter; Windows Server 2008 R2 SP1
Enable backup when you create a Linux VM | Supported for:<br/><br/> - Ubuntu Server: 1710, 1704, 1604 (LTS), 1404 (LTS)<br/><br/> - Red Hat: RHEL 6.7, 6.8, 6.9, 7.2, 7.3, 7.4<br/><br/> - SUSE Linux Enterprise Server: 11 SP4, 12 SP2, 12 SP3<br/><br/> - Debian: 8, 9<br/><br/> - CentOS: 6.9, 7.3<br/><br/> Oracle Linux: 6.7, 6.8, 6.9, 7.2, 7.3
Back up a VM that's shutdown/offline/seeking VM | Supported.<br/><br/> Snapshot is crash-consistent only, not app-consistent.
Back up disks after migrating to managed disks. | Supported.<br/><br/> Backup will continue to work. No action is required.
Back up managed disks after enabling resource group lock | Not supported.<br/><br/> Backup can't delete the older resource points and backups will start to fail when the maximum limit of restore points is reached.
Modify backup policy for a VM | Supported.<br/><br/> The VM will be backed up using the schedule and retention settings in new policy. If retention settings are extended, existing recovery points are marked and kept. If they're reduced, existing recovery points will be pruned in the next cleanup job, and eventually deleted.
Cancel a backup job	| Supported during snapshot process.<br/><br/> Not supported when the snapshot is being transferred to the vault. 
Back up the VM to a different region or subscription |	Not supported.
Backups per day (via the Azure VM extension) | One scheduled backup per day.<br/><br/> You can take up to four on-demand backups per day.
Backups per day (via the MARS agent) | Three scheduled backups per day. 
Backups per day (via DPM/MABS) | Two scheduled backups per day.
Monthly/yearly backup	| Not supported when backing up with Azure VM extension. Only daily and weekly is supported.<br/><br/> You can set up the policy to retain daily/weekly backups for monthly/yearly retention period.
Automatic clock adjustment | Not supported.<br/><br/> Azure Backup doesn't take automatic adjustment of clock for daylight-saving changes into account when backing up VM.<br/><br/>  Modify the policy manually as needed.
[Security features for hybrid backup](https://docs.microsoft.com/azure/backup/backup-azure-security-feature) |	Disabling the features isn't supported.



## Operating system support (Windows)

The table summarizes the supported operating systems when backing up Windows Azure VMs.

**Scenario** | **OS support** 
--- | ---
Back up with Azure VM agent extension | Windows Client: Not supported<br/><br/> Windows Server: Windows Server 2008 R2 or later 
Back up with MARS agent | [Supported](backup-support-matrix-mars-agent.md#support-for-direct-backups) operating systems.
Back up via DPM/MABS | Supported operating systems for backup with [MABS](backup-mabs-protection-matrix.md) and [DPM](https://docs.microsoft.com/system-center/dpm/dpm-protection-matrix?view=sc-dpm-1807).

## Support for Linux backup

Here's what's supported if you want to back up Linux machines.

**Action** | **Support**
--- | --- 
Back up Linux Azure VMs with the Linux Azure VM agent | File consistent backup.<br/><br/> App-consistent backup using [custom scripts](backup-azure-linux-app-consistent.md).<br/><br/> During restore you can create a new VM, restore a disk and use that to create a VM, or restore a disk and use it to replace a disk on an existing VM. You can also restore individual files and folders.
Back up Linux Azure VMs with MARS agent | Not supported.<br/><br/> The MARS agent can only be installed on Windows machines.
Back up Linux Azure VMs with DPM/MABS | Not supported.

## Operating system support (Linux)

For Azure VM Linux backups, Azure Backup supports the list of Linux [distributions endorsed by Azure](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros). Note that:

- Azure Backup doesn't support Core OS Linux.
- Azure Backup doesn't support 32-bit operating systems.
- Other bring-your-own Linux distributions might work as long as the [Azure VM agent for Linux](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux) is available on the VM, and Python is supported.



## Backup frequency and retention

**Setting** | **Limits** 
--- | --- 
Maximum recovery points per protected instance (machine/workload | 9999
Maximum expiry time for a recovery point | No limit
Maximum backup frequency to vault (Azure VM extension) | Once a day
Maximum backup frequency to vault (MARS agent) | Three backups per day.
Maximum backup frequency to DPM/MABS | Every 15 minutes for SQL Server<br/><br/> Once a hour for other workloads.
Recovery point retention | Daily, weekly, monthly, yearly.
Maximum retention period | Depends on backup frequency.
Recovery points on DPM/MABS disk | 64 for file servers, 448 for app servers.<br/><br/> Tape recovery points are unlimited for on-premises DPM.


## Supported restore methods

**Restore method** | **Details**
--- | ---
**Create a new VM** | You can create a VM during the restore process. <br/><br/> This option gets a basic VM up and running. You can specify the VM name, resource group,  virtual network, subnet, and storage.  
**Restore a disk** | You can restore a disk and use it to create a VM<br/><br/> When you select this option, Azure Backup copies data from the vault to a storage account that you select. The restore job generates a template. You can download this template and use it to specify custom VM settings, and create a VM.<br/><br/> This option allows you to specify more settings that the previous option to create a VM.<br/><br/>
**Replace an existing disk** | You can restore a disk and then use the restored disk to replace a disk that's currently on a VM.
**Restore files** | You can recover files from a selected recovery point. You download a script to mount the VM disk from the recovery point. You then browse through the disk volumes to find the files/folders you want to recover, and unmount the disk when you're done.

## Support for file-level restore

**Restore** | **Supported**
--- | --- 
Restoring files across operating systems | You can restore files on any machine that has the same (or compatible) OS as the backed-up VM. see the [Compatible OS table](backup-azure-restore-files-from-vm.md#system-requirements).
Restoring files on classic VMs | Not supported. 
Restoring files from encrypted VMs | Not supported.
Restoring files from network-restricted storage accounts | Not supported.
Restoring files on VMs using Windows Storage Spaces | Restore not supported on same VM.<br/><br/> Instead, restore the files on a compatible VM.
Restore files on Linux VM using LVM/raid arrays | Restore not supported on same VM.<br/><br/> Restore on a compatible VM.
Restore files with special network settings | Restore not supported on same VM. <br/><br/> Restore on a compatible VM.

## Support for VM management

The table summarizes support for backup during VM management tasks, such as adding or replacing VM disks.

**Restore** | **Supported** 
--- | --- 
Restore across subscription/region/zone. | Not supported 
Restore to an existing VM | Use replace disk option.
Restore disk with storage account enabled for Azure Storage Service Encryption (SSE) | Not supported.<br/><br/> Restore to an account that doesn't have SSE enabled.
Restore to mixed storage accounts |	Not supported.<br/><br/> Based on the storage account type, all restored disks will be either premium or standard, and not mixed. 
Restore to storage account using ZRS | Not supported
Restore VM directly to an availability set | For managed disks, you can restore the disk, and use the availability set option in the template.<br/><br/> Not supported for unmanaged disks. For unmanaged disks, restore the disk, and then create a VM in the availability set.
Restore backup of unmanaged VMs after upgrading to managed | Supported.<br/><br/> You can restore disks, and then create a managed VM.
Restore VM to restore point before the VM was migrated to managed disks | Supported.<br/><br/> You restore to unmanaged disks (default), convert the restored disks to managed disk, and create a VM with the managed disks.
Restore a VM that's been deleted. | Supported.<br/><br/> You can restore the VM from a recovery point.
Restore a domain controller (DC) VM that is part of a multi-DC configuration through portal | Supported if you restore the disk and create a VM using PowerShell.
Restore VM in different VNet |	Supported.<br/><br/> The VNet must be in the same subscription and region.




## VM compute support

**Compute** | **Support** 
--- | --- 
VM size |	Any Azure VM size with at least 2 CPU cores and 1-GB RAM.<br/><br/> [Learn more](https://docs.microsoft.com/azure/virtual-machines/windows/sizes) 
Back up VMs in [availability sets](https://docs.microsoft.com/azure/virtual-machines/windows/regions-and-availability#availability-sets) | Supported.<br/><br/> You can't restore a VM in an available set using the option to quickly create a VM. Instead, when you restore the VM you need to either restore the disk and use it to deploy a VM, or restore a disk and use it to replace an existing disk. 
Back up VMs in [availability zones](https://docs.microsoft.com/azure/availability-zones/az-overview) |	Not supported.	
Back up VMs deployed with [Hybrid Use Benefit (HUB)](https://docs.microsoft.com/azure/virtual-machines/windows/hybrid-use-benefit-licensing) | Supported.
Back up VMs deployed in a [scale set](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview) |	Not supported	
Back up VMs deployed from the [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?filters=virtual-machine-images)<br/><br/> (Published by Microsoft, third-party) |	Supported.<br/><br/> The VM must be running a supported operating system.<br/><br/> When recovering files on the VM, you can restore only to a compatible OS (not an earlier or later OS).
Back up VMs deployed from a custom image (third-party) |	Supported.<br/><br/> The VM must be running a supported operating system.<br/><br/> When recovering files on the VM, you can restore only to a compatible OS (not an earlier or later OS).
Back up VMs migrated to Azure	| Supported.<br/><br/> To back up the VM, the VM agent must be installed on the migrated machine. 



## VM storage support

**Component** | **Support**
--- | ---
Azure VM data disks | Back up a VM with 16 or less data disks.
Data disk size | Individual disk can be up to 4095 GB.<br/><br/> If you're running the latest version of Azure VM backup (known as Instant Restore), disk sizes up to 4TB are supported. [Learn more](backup-instant-restore-capability.md).
Storage type | Standard HDD, standard SSD, premium SSD <br/><br/> Standard SSD is supported if you're running the latest version of Azure VM backup (known as Instant Restore), standard SSD is supported. [Learn more](backup-instant-restore-capability.md).
Managed disks | Supported
Encrypted disks | Supported.<br/><br/> Azure VMs enabled with Azure Disk Encryption (ADE) can be backed up (with or without the Azure AD app).<br/><br/> Encrypted VMs can’t be recovered at the file/folder level. You need to recover the entire VM.<br/><br/> You can enable encryption on VMs that are already protected by Azure Backup.
Disks with Write Accelerator enabled | Not supported.<br/><br/> If you're running the latest version of Azure VM backup (known as [Instant Restore](backup-instant-restore-capability.md)), you can exclude disks with Write Accelerator enabled from backup.
Back up deduplicated disks | Not supported.
Add disk to protected VM | Supported.
Resize disk on protected VM | Supported.

## VM network support


**Component** | **Support**
--- | ---
Number of NICs | Up to maximum number of NICs supported for a specific Azure VM size.<br/><br/> NICs are created when the VM is created during the restore process.<br/><br/> The number of NICs on the restored VM mirrors the number of NICs on the VM when you enabled protection. Removing NICs after enabling doesn't affect the count.
External/internal load balancer |	Supported. <br/></br> [Learn more](backup-azure-arm-restore-vms.md#restore-vms-with-special-network-configurations) about restoring VMs with special network settings.
Multiple reserved IP address |	Supported. <br/></br> [Learn more](backup-azure-arm-restore-vms.md#restore-vms-with-special-network-configurations) about restoring VMs with special network settings.
VMs with multiple network adapters	| Supported. <br/></br> [Learn more](backup-azure-arm-restore-vms.md#restore-vms-with-special-network-configurations) about restoring VMs with special network settings.
VMs with public IP addresses	| Supported.<br/><br/> You need to associate an existing public IP address with the NIC, or create an address and associate it with the NIC after restore is done.
Network security group (NSG) on NIC/subnet. |	Supported.
Reserved IP address (static) | Not Supported.<br/><br/> You can't back up a VM with an reserved IP address and no defined endpoint.
Dynamic IP address |	Supported.<br/><br/> If the NIC on the source VM uses dynamic IP addressing, by default the NIC on the restored VM will too.
Traffic Manager	| Supported<br/><br/>. If the backed up VM is in Traffic Manager, you'll need to manually add the restored VM to the same Traffic Manager. 
Azure DNS |	Supported.
Custom DNS |	Supported.
Outbound connectivity via HTTP proxy | Supported.<br/><br/> An authenticated proxy isn't supported.	
VNet service endpoints	| Supported.<br/><br/> Firewall and VNet storage account settings should allow access from All Networks. 



## VM security/encryption support

Azure Backup supports encryption for in-transit and at-rest data:

Network traffic to Azure:
- Backup traffic from servers to the Recovery Services vault is encrypted using Advanced Encryption Standard 256.
- Backup data is sent over a secure HTTPS link.
- The backup data is stored in the Recovery Services vault in encrypted form.
- Only you have the passphrase to unlock this data. Microsoft can't decrypt the backup data at any point.
    > [!WARNING]
    > After setting up the vault, only you have access to the encryption key. Microsoft never maintains a copy and doesn't have access to the key. If the key is misplaced, Microsoft can't recover the backup data.
Data security:
- When backing up Azure VMs you need to set up encryption *within* the virtual machine.
- Azure Backup supports Azure Disk Encryption, which uses BitLocker on Windows virtual machines and **dm-crypt** on Linux virtual machines.
- On the back end, Azure Backup uses [Azure Storage Service encryption](../storage/common/storage-service-encryption.md), which protects data at rest.


**Machine** | **In transit** | **At rest**
--- | --- | ---
On-premises Windows machines without DPM/MABS | ![Yes][green] | ![Yes][green]
Azure VMs | ![Yes][green] | ![Yes][green] 
On-premises/Azure VMs with DPM | ![Yes][green] | ![Yes][green]
On-premises/Azure VMs with MABS | ![Yes][green] | ![Yes][green]



## VM compression support

Backup supports the compression of backup traffic, as summarized in the table below. Note that:

- For Azure VMs, the VM extension reads the data directly from the Azure storage account over the storage network, so it is not necessary to compress this traffic.
- If you're using DPM or MABS, you can compress the data before it's backed up to DPM/MABS, to save bandwidth. 

**Machine** | **Compress to MABS/DPM (TCP)** | **Compress (HTTPS) to vault**
--- | --- | ---
On-premises Windows machines without DPM/MABS | NA | Yes
Azure VMs | NA | NA
On-premises/Azure VMs with DPM | ![Yes][green] | ![Yes][green]
On-premises/Azure VMs with MABS | ![Yes][green] | ![Yes][green]




## Next steps

- [Back up Azure VMs](backup-azure-arm-vms-prepare.md)
- [Back up Windows machines directly](tutorial-backup-windows-server-to-azure.md), without a backup server.
- [Set up MABS](backup-azure-microsoft-azure-backup.md) for backup to Azure, and then back up workloads to MABS.
- [Set up DPM](backup-azure-dpm-introduction.md) for backup to Azure, and then back up workloads to DPM.

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png

