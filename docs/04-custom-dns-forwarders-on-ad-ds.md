# 04 - Custom DNS forwarders on AD DS (Azure-hosted DC)

## Objective

Configure AD DS DNS servers (running on Azure VMs) to resolve Azure private endpoint FQDNs through Azure DNS.

## Important requirement

Custom DNS forwarders for private endpoint names require DNS servers that can reach Azure DNS virtual IP `168.63.129.16`.  
This is why deploying DC/DNS servers **inside Azure** is the practical requirement for this model.

## Resolution model

1. Client in AVD/Spoke queries AD DS DNS.
2. AD DS DNS conditionally forwards `privatelink.*` zones to `168.63.129.16`.
3. Azure DNS resolves against linked Private DNS Zones.
4. Client receives private IP and connects through Hub PE path.

## Step-by-step (Windows DNS Manager)

## 1) Verify DNS servers configured on VNets

- Hub/Spoke VNets should point to AD DS DNS server private IPs.
- Example baseline pattern: two DNS servers in Domain Controller subnet.

[SCREENSHOT: 04-A - VNet DNS server configuration]

## 2) Create conditional forwarders

On each AD DS DNS server:

1. Open **DNS Manager**.
2. Right-click **Conditional Forwarders** > **New Conditional Forwarder**.
3. Add zones:
   - `privatelink.file.core.windows.net`
   - `privatelink.vaultcore.azure.net`
4. Master server IP: `168.63.129.16`
5. Store in AD (forest/domain scope) if required by your DNS design.

[SCREENSHOT: 04-B - Conditional forwarder for privatelink.file.core.windows.net]  
[SCREENSHOT: 04-C - Conditional forwarder for privatelink.vaultcore.azure.net]

## 3) Optional PowerShell method

```powershell
Add-DnsServerConditionalForwarderZone `
  -Name "privatelink.file.core.windows.net" `
  -MasterServers 168.63.129.16 `
  -ReplicationScope "Forest"

Add-DnsServerConditionalForwarderZone `
  -Name "privatelink.vaultcore.azure.net" `
  -MasterServers 168.63.129.16 `
  -ReplicationScope "Forest"
```

## 4) Validate DNS path

From AVD session host and from DNS server:

```powershell
nslookup <storageaccount>.file.core.windows.net
nslookup <keyvault>.vault.azure.net
```

Expected: private IP answers that map to PE NICs.

[SCREENSHOT: 04-D - nslookup from AVD session host (private IP)]  
[SCREENSHOT: 04-E - nslookup from DNS server (private IP)]

## Common pitfalls

- Conditional forwarder typo or wrong zone suffix.
- VNet not linked to private DNS zone.
- DNS servers not configured on VNet DHCP options.
- NSG/UDR blocking DNS flow between client and AD DS DNS servers.
- Attempting this design with DNS servers outside Azure without proper relay/resolver path.

## Next step

Proceed to [03 - Key Vault with Private Endpoint](03-key-vault-private-endpoint.md) and then [05 - Validation and troubleshooting](05-validation-and-troubleshooting.md).
