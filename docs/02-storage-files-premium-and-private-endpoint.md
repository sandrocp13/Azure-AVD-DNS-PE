# 02 - Storage Files Premium and Private Endpoint

## Objective

Create a **Premium Azure Files** storage account and bind it to a **Private Endpoint** hosted in the Hub VNet.

## Scope

- Storage account type: `FileStorage`
- SKU: `Premium_LRS`
- Access path: private only (public network disabled)
- Private DNS zone integration: `privatelink.file.core.windows.net`

## Step-by-step (Portal-first)

## 1) Create the storage account

1. Go to **Storage accounts** > **Create**.
2. Select target subscription and resource group.
3. Set:
   - Performance: **Premium**
   - Redundancy: **LRS**
   - Account kind: **FileStorage**
4. In **Networking**:
   - Public network access: **Disabled** (or selected networks per policy before final hardening)
5. Review + create.

[SCREENSHOT: 02-A - Storage account basics tab]  
[SCREENSHOT: 02-B - Storage account networking (public disabled)]

## 2) Create a file share for FSLogix

1. Open storage account > **File shares** > **+ File share**.
2. Choose Premium tier and quota aligned with capacity planning.
3. Record UNC path for AVD profile configuration.

[SCREENSHOT: 02-C - File share creation parameters]

## 3) Create the Private Endpoint in Hub

1. Storage account > **Networking** > **Private endpoint connections** > **+ Private endpoint**.
2. Select:
   - Resource type: `Microsoft.Storage/storageAccounts`
   - Sub-resource: **file**
3. Network:
   - VNet: Hub VNet
   - Subnet: Dedicated PE subnet (e.g., `Private_EndPoint`)
4. DNS:
   - Integrate with private DNS zone: **Yes**
   - Zone: `privatelink.file.core.windows.net`
5. Create and validate state = **Approved**.

[SCREENSHOT: 02-D - PE creation target subresource=file]  
[SCREENSHOT: 02-E - PE placed in Hub PE subnet]  
[SCREENSHOT: 02-F - PE connection approved]

## 4) Confirm DNS record auto-registration

1. Open private DNS zone `privatelink.file.core.windows.net`.
2. Validate `A` record for `<storageaccountname>.file.core.windows.net` target.
3. Confirm record points to PE private IP in Hub subnet.

[SCREENSHOT: 02-G - Private DNS A record for Storage file endpoint]

## 5) Harden storage account

- Require secure transfer: **Enabled**
- Minimum TLS: `1.2`
- Keep public network access disabled after private path validation
- Configure RBAC/NTFS access model for AVD profiles (separate identity guide if needed)

## Validation checkpoints

- `nslookup <storage>.file.core.windows.net` from AVD session host returns private IP.
- `Test-NetConnection <storage>.file.core.windows.net -Port 445` succeeds from allowed hosts.
- Access denied behavior aligns with RBAC/ACL model, not network failure.

## Baseline notes from environment

- Existing pattern shows `FileStorage` + `Premium_LRS` with one private endpoint per storage account.
- Private endpoints are placed in Hub subnet `Private_EndPoint`, which aligns with this guide.

## Next step

Proceed to [04 - Custom DNS forwarders on AD DS](04-custom-dns-forwarders-on-ad-ds.md).
