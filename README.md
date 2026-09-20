# Azure-cloud-security-and-monitoring-Lab
Hands-on Azure cloud security lab covering Log Analytics, Azure Monitor alerts, RBAC, Key Vault, Defender for Cloud and Azure Firewall.

# Azure Cloud Security & Monitoring Lab (Logging, Alerting, Access Control, Key Vault and Firewall Documentation)

`Microsoft Azure` · `Azure Monitor` · `Log Analytics` · `Key Vault` · `Defender for Cloud` · `Azure Firewall` · `RBAC`

## Overview
This lab was hands-on practice with **cloud security and monitoring in Azure**, completed as part of the Microsoft Applied Skills assessment *Get started with cloud security and monitoring tasks*. The work covered five areas that show up in almost every real Azure environment: centralised logging, alerting, access control, secrets and key management, and network security documentation.

Rather than just clicking through the portal, I focused on understanding *why* each setting exists. A few things failed on the first attempt (a storage link, a role search and two rotation policy validation errors). I've kept those in the write-up because reading an error message and working out the cause is a big part of the job.

## Objective
Configure and document monitoring and security controls in an Azure environment: link a Log Analytics workspace to storage, assign a least-privilege role, update alert rules, export a security recommendations report, manage keys and secrets in Key Vault, and gather firewall details for documentation.

## Environment
- **Platform:** Microsoft Learn Applied Skills assessment, hosted lab environment
- **Client:** Windows lab VM with the Azure portal in a browser, File Explorer, and a lab-provided app called **Contoso Data Editor** for recording results
- **Azure scope:** one subscription, one resource group (`RG1`), all resources in **Australia East**
- **Resources worked with:** a Log Analytics workspace, a storage account, two Key Vaults (`kv1`, `kv2`), Azure Monitor alert rules (`RGAlert`, `KVHitsAlert`), an Azure AI Search service, and two Azure Firewalls (`Firewall2`, `FW1`)
- **Local output folder:** `C:\Documentation`

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Log Analytics workspace** | Central store for logs and saved queries | Linked it to a storage account for saved log alert queries |
| **Azure Storage (networking settings)** | Blob/file storage with its own firewall | Fixed the failed link by allowing trusted Microsoft services |
| **Azure RBAC (Access control / IAM)** | Controls who can do what on a resource | Assigned the Log Analytics Contributor role |
| **Azure Monitor (alert rules, action groups)** | Detects conditions and sends notifications | Added an email recipient and changed an alert's scope |
| **Microsoft Defender for Cloud** | Security posture recommendations | Exported the recommendations as a CSV report |
| **Azure Key Vault** | Stores keys and secrets | Enabled and backed up a key, set an expiry notification, versioned a secret |
| **Azure AI Search (Keys)** | Search service with admin keys | Retrieved the primary admin key so it could be stored safely in Key Vault |
| **Azure Firewall / Public IP addresses** | Network filtering and its public addresses | Identified the firewall on a given virtual network and its IPs |
| **Contoso Data Editor** | Lab app for recording results | Saved the secret identifier and the firewall details |

## What I Did

### Logging: Linking a Log Analytics Workspace to Storage
1. Opened **Linked storage accounts** on the workspace. It offers three types: *Custom logs & IIS logs*, *Saved queries* and *Saved log alert queries*.
2. Linked only **Saved log alert queries**. The portal warns that linking a storage account for saved queries permanently moves all queries and functions out of the workspace, so I was careful to pick the right type.
3. The first attempt **failed**. The error said Azure Monitor couldn't access the storage account, and pointed at either a region mismatch or the account not being set to bypass its firewall for Azure services.
4. Worked through it in order:
   - Confirmed the storage account and workspace were both in **Australia East** (region mismatch ruled out).
   - Confirmed public network access was enabled on the storage account.
   - Ticked **Allow trusted Microsoft services to access this resource** under the storage account's networking exceptions, saved, waited a minute, and retried.
5. The link then worked.

### Access Control: Least-Privilege Role Assignment
1. On the workspace, opened **Access control (IAM) → Add role assignment** and chose **Log Analytics Contributor**.
2. Searched for the admin user to add as a member. The search returned **no results** because I'd typed part of the username wrongly. Typing the correct name fixed it.
3. Assigned the role at the **workspace scope** rather than the resource group or subscription, so the user only gets the access they need.

### Alerting: Azure Monitor
1. **Action group:** opened the alert rule's action group and configured an **Email/SMS/Push/Voice** notification to send to an admin email address. The portal notes that emails must be verified before the action group can send to them.
2. **Alert scope:** edited a Key Vault alert rule to point at `kv2` instead of `kv1`. Ticking `kv2` wasn't enough on its own: the panel showed **2 key vaults selected**, so I had to expand the *Selected resources* list and remove `kv1` as well.
3. Saved both rules with **Review + save**.

### Security Posture: Defender for Cloud Report
1. Opened Defender for Cloud **Recommendations** and switched to the **classic view**.
2. Used **Download CSV report** to export the recommendations. The environment filter showed connectors for other platforms too (AWS, GCP, GitHub, GitLab, Docker Hub, JFrog), which gave a good sense of how Defender covers multi-cloud.
3. The browser saved the file to Downloads, so I moved it to `C:\Documentation`. Along the way I dismissed a "select an app to open this .csv file" pop-up, since the file only needed to be saved, not opened.

### Secrets and Key Management: Key Vault
1. **Key1:** confirmed the current version was **Enabled** and downloaded a **backup** (`.keybackup`) to `C:\Documentation`.
2. **Key2, expiry notification:** opened the **Rotation policy** and set a notification **15 days before expiry**. My first try put 15 in the wrong field, which threw two validation errors. The *Expiry time* has to be at least 28 days, and the *Rotation time* can't exceed 8 days. The 15 days belongs in the **Notification** section, which is a separate setting from how long the key lasts.
3. **Key2, public key:** downloaded the current version's **public key in PEM format** to `C:\Documentation`.
4. **Secret1:** created a **new version** with a new value, then **disabled the older version** so only the new one can be used.
5. Copied the new version's **secret identifier URI** into the lab's `Connection Info` file.
6. **Storing a credential properly:** copied the Azure AI Search service's **primary admin key** and saved it as a new Key Vault secret (`aisearchadmin`), so the key lives in Key Vault instead of in a file or a config.

### Network Security: Documenting the Firewall
1. Opened **Network security → Azure Firewalls**. Two firewalls were listed (`Firewall2` and `FW1`), both in `RG1`. The list doesn't show the virtual network, so I opened each one and checked its Overview.
2. `FW1` was the firewall on **VNET3**. Its Overview showed the **Basic** SKU, a firewall policy, a private IP, and both an `AzureFirewallSubnet` and an `AzureFirewallManagementSubnet`.
3. Found its **public IP** from the Public IP configuration page (resource `IPFW`).
4. The **management public IP** wasn't on that page. Basic-SKU firewalls use a separate management subnet and management IP. I found it by searching **Public IP addresses** and checking the **Associated to** column for the second IP tied to `FW1` (resource `ManagementIP`).
5. Recorded the firewall name, public IP and management IP in the lab's `VNET Config` file and saved.

## What's in This Repo

```
azure-cloud-security-lab/
├── README.md                                   # This file
└── screenshots/
    ├── 01-linked-storage-accounts.png          # Three link types, none linked yet
    ├── 02-link-storage-error.png               # Failed to link storage account
    ├── 03-storage-networking-exceptions.png    # Trusted Microsoft services exception
    ├── 04-role-assignment-member-search.png    # Log Analytics Contributor, member search
    ├── 05-alert-action-group-email.png         # Email notification on the action group
    ├── 06-alert-scope-selected-resources.png   # 2 key vaults selected in the scope
    ├── 07-defender-recommendations-csv.png     # CSV report download
    ├── 08-keyvault-key1-versions.png           # Key1 enabled
    ├── 09-key2-rotation-policy-errors.png      # Expiry/rotation validation errors
    ├── 10-secret1-new-version.png              # Creating a new secret version
    ├── 11-firewall-list-and-fw1-overview.png   # Firewalls list and FW1 on VNET3
    └── 12-public-ip-addresses.png              # Management IP resource
```

Files produced during the lab (the key backup, PEM file and Defender CSV) are **not** committed here on purpose. Key backups and exported security reports don't belong in a public repo.

## Skills I Picked Up
- **Reading Azure error messages properly.** The linking error named its own cause in the last line (Azure services not allowed to bypass the storage firewall), and I only solved it once I stopped guessing and worked through each cause in the message.
- **Applying least privilege.** Assigning a specific role at the narrowest useful scope instead of a broad one.
- **Understanding what a setting actually does before clicking.** Noticing that linking storage for saved queries is a one-way move, and choosing the right link type.
- **Key lifecycle basics.** The difference between a key's *expiry time*, its *rotation time* and its *expiry notification*, and why a new secret version plus disabling the old one is how you rotate a secret safely.
- **Keeping credentials out of plain sight.** Putting a service admin key into Key Vault instead of leaving it wherever it was copied from.
- **Checking the whole picture, not just the item I changed.** The alert scope showed two key vaults selected even though I'd only ticked one.
- **Tracing a resource to its details.** Following a firewall to its public IP and its separate management IP through the linked resources.
- **Documenting as I went.** Saving outputs to a set folder and recording results in the required format.

## How This Applies in the Real World
Logging, alerting and access control are the basics behind almost every security investigation. If alert queries and logs aren't stored and monitored properly, an incident can't be reconstructed later. The role and Key Vault tasks mirror everyday cloud security work: give people only the access they need, keep secrets out of code and documents, rotate them, and get warned before a key expires rather than after something breaks.

The firewall task is a small example of asset documentation. In a real environment, knowing which firewall protects which network, and which public IPs belong to it, is what lets a team respond quickly when something looks wrong.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in **healthcare**. It's a different field on paper, but a lot of the habits carry over: following procedures carefully, protecting sensitive information, and staying calm and methodical when something isn't working. I'm currently studying for **CompTIA Security+** and completing labs like this one to get hands-on experience, since that's what my CV is missing compared to my experience.

I've kept the things that went wrong in this write-up rather than tidying them away. I'd rather show the real process than pretend everything worked first time.

## What I Want to Learn Next
- Building alert rules and action groups from scratch, and testing that notifications actually arrive
- Reviewing and prioritising Defender for Cloud recommendations, not just exporting them
- Using Key Vault with managed identities and access policies or RBAC, so applications never handle secrets directly
- Locking storage accounts down with private endpoints instead of relying on public network access
- Azure Firewall rules and policies, not just documenting the firewall

## Limitations & What I'd Do Differently in Production
- **This was a guided assessment.** The resources already existed and each task was set for me. I haven't designed this environment from scratch.
- **Public network access was enabled on the storage account.** That got the link working, but in production I'd prefer a private endpoint or a restricted network scope, keeping only the trusted-services exception.
- **I exported the Defender report but didn't triage it.** A real assessment would sort the recommendations by severity and assign owners.
- **I didn't test the alert notification.** An action group email must be verified before it can send, so in production I'd confirm the recipient is verified and send a test notification.
- **Key backups need careful handling.** They should go in controlled, encrypted storage, not a shared documentation folder.
- **Troubleshooting help.** When I got stuck, I used documentation and an AI assistant to work through blockers, then confirmed the fix in the portal myself.

## References
- [Microsoft Learn: Log Analytics workspace overview](https://learn.microsoft.com/azure/azure-monitor/logs/log-analytics-workspace-overview)
- [Microsoft Learn: Azure Monitor action groups](https://learn.microsoft.com/azure/azure-monitor/alerts/action-groups)
- [Microsoft Learn: Introduction to Microsoft Defender for Cloud](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-cloud-introduction)
- [Microsoft Learn: Azure Key Vault overview](https://learn.microsoft.com/azure/key-vault/general/overview)
- [Microsoft Learn: Azure Firewall documentation](https://learn.microsoft.com/azure/firewall/)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
