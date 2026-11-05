# 03 - Key Vault with Private Endpoint

## Objective

Deploy or update Key Vault to use private access and the same DNS pattern used for AVD storage dependencies.

## Why include Key Vault in this pattern

- AVD operational secrets and certificates should remain private.
- Reusing Hub PE + custom DNS model keeps architecture consistent.

## Step-by-step (Portal-first)

## 1) Create or review Key Vault baseline

1. Go to **Key Vaults** > **Create**.
2. Select RBAC authorization model (recommended for modern governance).
3. Enable soft delete and purge protection per policy.
4. Set public network access according to rollout phase (final target: disabled).

[SCREENSHOT: 03-A - Key Vault basic configuration]  
[SCREENSHOT: 03-B - Key Vault networking baseline]

## 2) Create Private Endpoint for vault

1. Key Vault > **Networking** > **Private endpoint connections** > **+ Private endpoint**.
2. Target sub-resource: **vault**.
3. Select Hub VNet + PE subnet (`Private_EndPoint`).
4. Enable private DNS integration with `privatelink.vaultcore.azure.net`.
5. Create and confirm state = **Approved**.

[SCREENSHOT: 03-C - Key Vault PE subresource=vault]  
[SCREENSHOT: 03-D - Key Vault PE in Hub subnet]  
[SCREENSHOT: 03-E - Key Vault PE approved]

## 3) Confirm DNS records

1. Open `privatelink.vaultcore.azure.net` private DNS zone.
2. Validate `A` record for vault endpoint.
3. Confirm private IP matches the PE NIC.

[SCREENSHOT: 03-F - Key Vault private DNS record]

## 4) Lock down public path

After DNS and private connectivity pass validation:

- Set Key Vault public network access to **Disabled**.
- Verify expected access from AVD/admin hosts via private endpoint only.

## Validation checkpoints

- `nslookup <vaultname>.vault.azure.net` returns private IP.
- Secret operations from authorized hosts succeed.
- Same operations from non-routed networks fail as expected.

## Baseline notes from environment

- Existing AVD service-object resource groups already follow a dedicated Key Vault PE pattern in the Hub subnet.

## Next step

Proceed to [05 - Validation and troubleshooting](05-validation-and-troubleshooting.md).
