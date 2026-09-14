
# Day 12 — Azure Administration
### Topics: Azure Backup Architecture | File & Folder Backups | VM Backup & Restore | VM Image Definitions/Versions/Images | Snapshots & Disks | Data Replication Strategies

---

## 1. Azure Backup Architecture

### What is Azure Backup?
Azure Backup <cite index="7-1">helps protect your critical business systems and backup data against a ransomware attack by implementing preventive measures, and provides security to your backup environment, both when your data is in transit and at rest.</cite> <cite index="1-1">It backs up the data, machine state, and workloads running on on-premises machines and Azure virtual machine (VM) instances.</cite>

### The Core Concept: Vaults
<cite index="1-1">A vault is an online-storage entity in Azure used to hold data, such as backup copies, recovery points, and backup policies — vaults make it easy to organize your backup data while minimizing management overhead.</cite>

### Two Types of Vaults
Azure Backup uses **two different vault types** depending on what's being protected:

| Vault Type | Used For |
|---|---|
| **Recovery Services Vault** | <cite index="8-1">IaaS VMs (Linux or Windows), SQL Server in Azure VMs, System Center DPM, Azure Backup Server, and on-premises machines/servers</cite> |
| **Backup Vault** | <cite index="2-1">Newer workloads such as Azure Blob, Azure Database for PostgreSQL servers, and other newer workloads Azure Backup will support</cite> |

### How Backup Actually Works — Three Common Paths
```
Path 1 — On-premises Windows machines:
  MARS agent → directly to Recovery Services vault

Path 2 — On-premises via a backup server:
  Machines → DPM or MABS server → Recovery Services vault

Path 3 — Azure VMs (most common):
  Azure Backup extension (on VM agent) → directly to Recovery Services vault
```
<cite index="1-1">You can back up Azure VMs directly — Azure Backup installs a backup extension to the Azure VM agent that's running on the VM.</cite>

### Key Architectural Features
- <cite index="2-1">Azure RBAC provides fine-grained access control — Azure Backup has three built-in roles to manage recovery points, and backup/restore access is restricted to defined user roles</cite>
- <cite index="2-1">Data isolation — vaulted backup data is stored in a Microsoft-managed subscription and tenant, so external users have no direct access to the backup storage, ensuring backups can't be tampered with or deleted even in a compromised environment</cite>
- <cite index="3-1">Immutable vaults ensure that after recovery points are created, they can't be deleted before their expiry according to backup policy — you can make this immutability irreversible to protect against ransomware attacks</cite>
- <cite index="3-1">By default, vaults use geo-redundant storage (GRS) — if the vault is your primary backup mechanism, Microsoft recommends using GRS</cite>
- <cite index="7-1">Zone-redundancy is available for both the Recovery Services vault/Backup vault itself and, optionally, for the backup data</cite>

### Region Rule for Vaults
<cite index="6-1">The vault must be in the same region as the data source it protects — if you have data sources in multiple regions, you need a separate Recovery Services vault for each region.</cite>

📘 **Official Docs:**
- [Azure Backup architecture overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/backup-architecture)
- [What is Azure Backup? – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/backup-overview)

### 🧪 Practice Lab
1. In Cloud Shell, create a resource group and a **Recovery Services vault**:
   ```bash
   az group create --name rg-backup-day12 --location eastus

   az backup vault create \
     --resource-group rg-backup-day12 \
     --name rsv-day12-demo \
     --location eastus
   ```
2. In the Portal, open the vault → **Properties** → confirm the default storage redundancy shows **Geo-redundant (GRS)**.
3. Explore **Access control (IAM)** on the vault and identify the three built-in Backup RBAC roles available.
4. Discuss: why does Microsoft deliberately store backup data in a *separate, Microsoft-managed* tenant rather than your own subscription's regular storage?

---

## 2. Configuring File and Folder Backups (MARS Agent)

### What is the MARS Agent?
<cite index="11-1">Azure Backup uses the MARS (Microsoft Azure Recovery Services) agent to back up files, folders, and system state from on-premises machines and Azure VMs — those backups are stored in a Recovery Services vault in Azure.</cite>

### Where MARS Runs
<cite index="11-1">You can run the agent: directly on on-premises Windows machines (which back up directly to a Recovery Services vault); on Azure VMs that run Windows side-by-side with the Azure VM backup extension, backing up specific files/folders on the VM; or on a Microsoft Azure Backup Server (MABS)/DPM instance, where machines back up to MABS/DPM first, and MABS/DPM then uses the MARS agent to back up to Azure.</cite>

> ⚠️ <cite index="9-1">Linux machines aren't supported by the MARS agent.</cite>

### Important Distinction: MARS vs the VM Backup Extension
<cite index="13-1">By default, Azure VMs enabled for backup use the Azure Backup extension, which backs up the entire VM. You can install and run the MARS agent on an Azure VM alongside the extension if you want to back up specific folders and files rather than the complete VM.</cite>

> 🎯 **Teaching point:** These are two different tools for two different goals — the **extension** = "back up the whole VM," MARS = "back up just these specific files/folders," and you can run **both simultaneously** on the same VM.

### How the Backup Process Works
<cite index="9-1">The MARS agent only uses the Windows System Writer operation to capture the snapshot — it doesn't use any application VSS writers, and doesn't capture app-consistent snapshots.</cite> <cite index="9-1">After the VSS agent takes the snapshot, the MARS agent creates a virtual hard disk (VHD) in the specified cache folder, and stores checksums for each data block. Incremental backups run according to the configured schedule — changed files are identified, a new VHD is created, compressed, and encrypted, then sent to the vault, where it's merged with the previous VHD.</cite>

### Setup Steps (High-Level)
1. <cite index="9-1">Create a Recovery Services vault and choose "Files, folders, and system state" from the Backup goals</cite>
2. <cite index="9-1">Configure the vault to securely save the backup passphrase to Azure Key Vault</cite>
3. <cite index="9-1">Download the vault credentials and agent installer to the on-premises machine</cite>
4. <cite index="9-1">Install the agent and use the downloaded credentials to register the machine to the vault</cite>
5. <cite index="9-1">From the agent console, configure what to back up, the schedule, and retention</cite>

### Critical Security Note — The Encryption Passphrase
<cite index="16-1">The encryption passphrase is critical — it encrypts your backup data before it leaves the server. Without this passphrase, you cannot restore data, not even Microsoft can recover it. Store it securely (e.g., in a password manager or Azure Key Vault).</cite>

### Retention and Recovery
<cite index="15-1">Azure Backup prunes recovery points according to the configured policy, and you can restore backed-up data from existing recovery points.</cite> <cite index="15-1">If you stop protection but choose to retain backup data, deleted backup data is retained for 14 days before permanent deletion, and re-enabling backup requires a security PIN generated from the vault's Properties page (valid only 5 minutes).</cite>

📘 **Official Docs:**
- [About the MARS agent – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/backup-azure-about-mars)
- [Install the MARS agent – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/install-mars-agent)
- [Back up Windows machines using the MARS agent – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/backup-windows-with-mars-agent)

### 🧪 Practice Lab (Conceptual Walkthrough — MARS requires a Windows machine to fully complete)
1. In the Portal, open your `rsv-day12-demo` vault → **Getting Started → Backup** → choose **On-Premises** as the backup goal → select **Files and folders**.
2. Download the vault credentials file and note the agent installer download link presented.
3. Discuss (without necessarily installing): what would happen to your ability to restore data if you lost the encryption passphrase, given Microsoft explicitly cannot recover it for you?
4. Discuss: for a company with 50 branch-office Windows file servers, would MARS agent (direct-to-vault) or a MABS/DPM intermediary server likely scale better, and why?

---

## 3. VM Backup and Restore

### The Standard Flow (Portal)
<cite index="3-1">The steps are: prepare Azure VMs → create a vault → discover VMs and configure a backup policy → enable backup for Azure VMs → run the initial backup.</cite> <cite index="3-1">Alternatively, you can back up a single Azure VM directly from the VM's own settings pane, without pre-creating a vault first.</cite>

### How the Backup Extension Works
<cite index="3-1">Azure Backup backs up Azure VMs by installing an extension to the Azure VM agent that runs on the machine — if your VM was created from an Azure Marketplace image, this agent is already installed and running.</cite>

### Enabling Backup — CLI Example
```bash
az backup vault create --resource-group rg-backup-day12 --name rsv-day12-demo --location eastus

# Create a backup policy
az backup policy create \
  --resource-group rg-backup-day12 \
  --vault-name rsv-day12-demo \
  --name DailyBackupPolicy \
  --policy '{"schedulePolicy":{"scheduleRunFrequency":"Daily"},"retentionPolicy":{"dailySchedule":{"retentionDuration":{"count":30,"durationType":"Days"}}}}' \
  --backup-management-type AzureIaasVM

# Enable backup for a VM using that policy
az backup protection enable-for-vm \
  --resource-group rg-backup-day12 \
  --vault-name rsv-day12-demo \
  --vm <vm-name> \
  --policy-name DailyBackupPolicy
```

### Restore Options
When restoring an Azure VM, admins typically choose from:

| Restore Type | What It Does |
|---|---|
| **Create a new VM** | Restores to a brand-new VM from a recovery point |
| **Restore disk** | Recovers only the disk(s), which you then attach manually |
| **Replace existing VM** | Restores over the current VM, replacing its disks |
| **File recovery** | Mounts the recovery point as a drive to recover individual files without a full VM restore |

### Vault Storage and Ransomware Protection (Recap Link to Section 1)
<cite index="3-1">Azure Backup now supports immutable vaults, ensuring recovery points can't be deleted before their expiry according to the backup policy — this can be made irreversible to help protect against ransomware attacks and malicious actors.</cite>

📘 **Official Docs:**
- [Back up Azure VMs in a Recovery Services vault – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/backup-azure-arm-vms-prepare)
- [Guidance and best practices – Azure Backup – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/guidance-best-practices)

### 🧪 Practice Lab
1. Create/reuse a test VM (e.g., from Day 5), then create a backup policy and enable protection using the CLI commands above.
2. Trigger an on-demand backup:
   ```bash
   az backup protection backup-now \
     --resource-group rg-backup-day12 \
     --vault-name rsv-day12-demo \
     --container-name <vm-container-name> \
     --item-name <vm-name> \
     --backup-management-type AzureIaasVM \
     --workload-type VM \
     --retain-until $(date -u -d "+30 days" '+%d-%m-%Y')
   ```
3. In the Portal, monitor the backup job under **Backup jobs**.
4. Once complete, explore the **Restore VM** wizard (don't need to complete a full restore) and note the four restore options above.
5. Clean up: disable and stop protection when done to avoid ongoing storage charges.

---

## 4. Azure VM Image Definitions, Image Versions, and Images (Azure Compute Gallery)

### What is Azure Compute Gallery?
<cite index="19-1">An Azure Compute Gallery (formerly Shared Image Gallery) simplifies custom image sharing across your organization. Custom images are like marketplace images, but you create them yourself, from a VM, VHD, snapshot, managed image, or another image version.</cite> <cite index="19-1">You choose which images to share, which regions to make them available in, and who to share them with.</cite>

### The Three Core Resources

| Resource | What It Is |
|---|---|
| **Gallery** | <cite index="23-1">Like the Azure Marketplace, but a repository you control — a container for managing and sharing images and VM applications</cite> |
| **Image Definition** | <cite index="23-1">Created within a gallery, and carries information about the image and requirements for using it — including whether it's Windows or Linux, release notes, and minimum/maximum memory requirements. It defines a *type* of image, not a specific usable image itself.</cite> |
| **Image Version** | <cite index="23-1">What you actually use to create a VM from the gallery — you can have multiple versions of the same image definition, each representing a specific build/update</cite> |

### The Relationship Between Them
```
Gallery ("myCompanyGallery")
   └── Image Definition ("web-server-ubuntu")
          ├── Image Version 1.0.0  (initial release)
          ├── Image Version 1.1.0  (patched)
          └── Image Version 1.2.0  (latest, in production)
```
This structure lets teams **deploy consistently** across the organization — everyone uses the same governed, tested image, and can always tell exactly which version they're running.

### Where an Image Version's Data Comes From
<cite index="23-1">An "image source" is a resource used to create an image version in a gallery — it can be an existing Azure VM (generalized or specialized), a managed image, a snapshot, or an image version from another gallery.</cite>

### Generalized vs Specialized Images — Critical Distinction
| Type | Meaning |
|---|---|
| **Generalized** | The VM has been "cleaned" (Sysprep for Windows, `waagent -deprovision` for Linux) — no machine-specific data remains, so it can be used to spin up **many new VMs** |
| **Specialized** | An exact copy of a specific VM's current disk (keeps hostname, user accounts, etc.) — used to restore/clone **that particular** VM |

### Creating an Image Definition and Version — CLI Example
```bash
az sig create --resource-group rg-images-day12 --gallery-name myCompanyGallery

az sig image-definition create \
  --resource-group rg-images-day12 \
  --gallery-name myCompanyGallery \
  --gallery-image-definition web-server-ubuntu \
  --publisher Contoso --offer WebServer --sku ubuntu-24-04 \
  --os-type Linux --os-state Generalized

az sig image-version create \
  --resource-group rg-images-day12 \
  --gallery-name myCompanyGallery \
  --gallery-image-definition web-server-ubuntu \
  --gallery-image-version 1.0.0 \
  --virtual-machine <source-vm-resource-id>
```
<cite index="20-1">You need to wait for the image version to completely finish being built and replicated before you can use the same managed image to create another image version.</cite>

### Scale Limits to Teach
<cite index="19-1">Per subscription, per region, you can have a maximum of 100 galleries, 1,000 image definitions, and 10,000 image versions — and a maximum of 100 replicas per image version.</cite>

### Sharing Scope
<cite index="19-1">You can share at the gallery, definition, or version level using Azure RBAC roles — sharing with users, service principals, and groups, even across tenants. Microsoft recommends sharing at the gallery level for the best experience.</cite> <cite index="19-1">A direct shared gallery distributes images widely to all users in a subscription or tenant, while a community gallery distributes images publicly — use caution with IP-sensitive images.</cite>

📘 **Official Docs:**
- [Overview of Azure Compute Gallery – Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/azure-compute-gallery)
- [Create an image definition and image version – Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/image-version)

### 🧪 Practice Lab
1. Create a gallery, image definition, and image version using the CLI commands above (use a VM from earlier days as the source — it must first be **generalized** via `az vm deallocate` + `az vm generalize`).
2. List the image definitions in your gallery:
   ```bash
   az sig image-definition list --resource-group rg-images-day12 --gallery-name myCompanyGallery --output table
   ```
3. Deploy a **new** VM directly from your image version:
   ```bash
   az vm create --resource-group rg-images-day12 --name vm-from-gallery \
     --image "/subscriptions/<sub-id>/resourceGroups/rg-images-day12/providers/Microsoft.Compute/galleries/myCompanyGallery/images/web-server-ubuntu" \
     --admin-username azureuser --generate-ssh-keys
   ```
4. Discuss: why would a company want a governed "approved image" gallery rather than letting every team pick their own base OS image from the public Marketplace?

---

## 5. Azure Snapshots and Disks

### What is a Snapshot?
<cite index="25-1">Azure managed disk snapshots provide point-in-time backups of disks that can be used during software upgrades, disaster recovery, or to create new environments — Azure automatically copies the data from the disk to the snapshot in the background.</cite>

### Full vs Incremental Snapshots
| Type | Billing | Storage |
|---|---|---|
| **Full Snapshot** | Billed for the **entire** disk size, every time | <cite index="28-1">Can use Premium SSDs</cite> |
| **Incremental Snapshot** (Recommended) | <cite index="26-1">Billed only for the delta changes since the last snapshot</cite> | <cite index="26-1">Always stored on the most cost-effective storage, Standard HDD, regardless of the parent disk's storage type — and stored on Zone-Redundant Storage (ZRS) by default in supporting regions</cite> |

<cite index="30-1">The subsequent incremental snapshots occupy only delta changes to disks since the last snapshot. When you restore a disk from an incremental snapshot, the system reconstructs the full disk representing the point-in-time backup of the disk when that snapshot was taken.</cite>

### Instant Access — A Nuance by Disk Type
<cite index="25-1">Snapshots of Premium SSD, Standard SSD, and Standard HDD are, by default, instant access — immediately usable to restore new disks, download data, or copy to other regions. Snapshots of Ultra Disk and Premium SSD v2 aren't instant access by default and require the background data copy to complete first.</cite>

### Why Incremental Snapshots Matter for DR
<cite index="26-1">Incremental snapshots provide differential capability, allowing you to get the changes between two snapshots of the same disk — copying only changed data between snapshots across regions, reducing time and cost for backup and disaster recovery.</cite>

### Creating a Snapshot — CLI Example
```bash
az snapshot create \
  --resource-group rg-images-day12 \
  --name snap-vm-osdisk-01 \
  --source <managed-disk-resource-id> \
  --incremental true
```

### Snapshots vs Azure Disk Backup — Which to Use?
<cite index="24-1">Azure Disk Backup is a turnkey solution providing snapshot lifecycle management for managed disks — automating periodic snapshot creation and retention using a backup policy, with zero infrastructure cost and no custom scripting needed. It's a crash-consistent backup solution using incremental snapshots, with support for multiple backups per day, and doesn't impact production application performance.</cite>

> 🎯 **Teaching point:** Manually scripting `az snapshot create` on a schedule works, but **Azure Disk Backup** (a feature within Backup vaults) automates the entire lifecycle — creation, retention, and cleanup — with no custom scripting required. This is the same "roll your own vs managed service" trade-off seen throughout the course.

### Snapshots vs. Full VM Backup — Recap Comparison
| Aspect | Disk Snapshot | Azure VM Backup (Section 3) |
|---|---|---|
| Scope | One disk at a time | Entire VM (all disks + config) |
| Consistency | <cite index="24-1">Crash-consistent</cite> | Can be app-consistent (VSS-aware) |
| Automation | Manual, or via Azure Disk Backup | Fully policy-driven via Recovery Services vault |
| Best for | Quick pre-change safety points, cloning environments | Full disaster recovery of an entire VM |

📘 **Official Docs:**
- [Create an incremental snapshot – Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-incremental-snapshots)
- [Overview of Azure Disk Backup – Microsoft Learn](https://learn.microsoft.com/en-us/azure/backup/disk-backup-overview)

### 🧪 Practice Lab
1. Create an incremental snapshot of a VM's OS disk using the CLI command above.
2. Confirm it's stored on Standard HDD regardless of the source disk's tier:
   ```bash
   az snapshot show --resource-group rg-images-day12 --name snap-vm-osdisk-01 --query "sku" --output json
   ```
3. Create a new disk from the snapshot:
   ```bash
   az disk create --resource-group rg-images-day12 --name disk-from-snap --source snap-vm-osdisk-01
   ```
4. Discuss: before applying a risky OS update to a production VM, would you take a **snapshot** or trigger a **full VM backup**? What's the trade-off in speed vs completeness?

---

## 6. Data Replication Strategies (Azure Site Recovery)

### What is Azure Site Recovery (ASR)?
<cite index="34-1">The Azure Site Recovery service contributes to your business continuity and disaster recovery (BCDR) strategy by keeping your business applications online during planned and unplanned outages. Site Recovery manages and orchestrates disaster recovery of on-premises machines and Azure VMs, including replication, failover, and recovery.</cite>

### Replication Scenarios ASR Supports
- **Azure-to-Azure** — <cite index="35-1">replicate Azure VMs from one region to another</cite>
- **VMware-to-Azure** — replicate on-premises VMware VMs to Azure
- **Physical-to-Azure** — <cite index="37-1">replicate physical on-premises Windows/Linux servers to Azure</cite> (note: <cite index="37-1">after failover to Azure, physical servers can't fail back to physical machines — only to VMware VMs</cite>)
- **Hyper-V-to-Azure** — <cite index="38-1">replication frequency as low as 30 seconds for Hyper-V</cite>

### How It Fits Into a BCDR Strategy
<cite index="40-1">ASR continuously replicates VMs to a second Azure region (or from on-premises into Azure) and fails them over when the primary goes down, so you recover with minimal downtime — enabling organizations to implement disaster recovery strategies aligned with their RTO and RPO requirements.</cite>

### Recovery Point Options
<cite index="38-1">You can choose between crash-consistent and application-consistent recovery options, with recovery points of up to 15 days.</cite>

### Networking Considerations
<cite index="36-1">Networks are typically protected using firewalls and NSGs — service tags (recap from Day 9!) should be used to control network connectivity, rather than IP address-based filtering, which isn't recommended.</cite> <cite index="35-1">If multi-VM consistency is enabled, machines in the replication group communicate over port 20004, and no firewall appliance should block this internal communication.</cite>

### Target VM Sizing
<cite index="35-1">The default SKU for the target region VM matches the source VM's SKU (or the next best available comparable size), and this can be modified either before or after replication begins — though the availability type (single instance, set, or zone) can't be updated later.</cite>

### Test Failover — A Critical Best Practice
<cite index="38-1">You can validate your ongoing replication and disaster recovery strategy without any downtime by running a test failover</cite> — this spins up an isolated copy of the failed-over environment so you can confirm everything works **without impacting production**.

### ASR vs Azure Backup — Different Purposes
| Aspect | Azure Backup | Azure Site Recovery |
|---|---|---|
| **Purpose** | Point-in-time recovery of data (restore to an earlier state) | Business continuity — keep the app **running** during an outage |
| **Recovery Speed** | Slower — restore process from a recovery point | Fast — failover to an already-replicated, running copy |
| **Typical Trigger** | Data corruption, accidental deletion, ransomware | Regional outage, disaster, planned maintenance |
| **Frequency** | Scheduled (e.g., daily) | Continuous/near-continuous replication |

> 🎯 **Teaching point:** Backup answers *"how do I recover data from the past?"* — Site Recovery answers *"how do I keep the application running right now if this region goes down?"* Most mature organizations use **both together**.

### Cost Model
<cite index="38-1">Azure Site Recovery is billed based on the number of instances protected, and every instance is free for the first 31 days.</cite>

📘 **Official Docs:**
- [About Azure Site Recovery – Microsoft Learn](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
- [Set up Azure VM disaster recovery to a secondary region – Microsoft Learn](https://learn.microsoft.com/en-us/azure/site-recovery/azure-to-azure-quickstart)

### 🧪 Practice Lab (Conceptual — full ASR setup takes significant time and isn't ideal for a quick classroom lab)
1. In the Portal, open your `rsv-day12-demo` vault → **Site Recovery** → **Azure virtual machines** → **Enable replication** → walk through the wizard (Source region, target region, target resource group) without necessarily completing it.
2. Note the **target VM SKU** field — observe it defaults to matching your source VM.
3. Discuss/design (on paper): your company runs a critical order-processing app on a VM in East US. Design a BCDR strategy combining **Azure Backup** (for data protection) and **Azure Site Recovery** (for regional failover) — specify your target failover region and roughly how often you'd run a test failover.
4. Clean up all lab resources:
   ```bash
   az group delete --name rg-backup-day12 --yes
   az group delete --name rg-images-day12 --yes
   ```

---

## Quick Recap Table

| Concept | One-Line Summary |
|---|---|
| **Azure Backup Architecture** | Recovery Services vaults (VMs, on-prem, SQL) and Backup vaults (newer workloads like Blob) store backup data in an isolated, Microsoft-managed tenant |
| **File & Folder Backup (MARS)** | Agent-based backup for specific files/folders — separate from full-VM backup, and requires securely storing an unrecoverable encryption passphrase |
| **VM Backup & Restore** | Extension-based full-VM protection via policy; restore options include new VM, disk-only, replace, or file-level recovery |
| **Compute Gallery (Images)** | Gallery → Image Definition (the "type") → Image Version (the deployable build) — governs consistent image sharing org-wide |
| **Snapshots & Disks** | Incremental snapshots are cheap, ZRS-backed, delta-only backups of a single disk; Azure Disk Backup automates their lifecycle |
| **Data Replication (ASR)** | Continuous replication for business continuity/disaster recovery — fast failover, complements (doesn't replace) Azure Backup |

> 🎯 **Key takeaway:** A complete data protection strategy layers multiple tools — **Backup** for point-in-time data recovery, **Snapshots** for quick disk-level safety points, **Compute Gallery** for consistent, governed VM deployment, and **Site Recovery** for keeping applications running through a regional disaster. Each tool answers a different question about "what happens when something goes wrong," and mature Azure environments use them together, not as substitutes for one another.

---
