# 01 - Architecture and prerequisites

## Objective

Build a repeatable private-access baseline for AVD that uses:

- **Azure Files Premium** for FSLogix.
- **Private Endpoints** in the **Hub** VNet.
- **Private DNS Zones** integrated with custom DNS.
- **AD DS DNS servers in Azure** to forward private link queries to Azure DNS (`168.63.129.16`).

## Why place Private Endpoints in the Hub subnet

1. **Centralized control**: One endpoint strategy for many spoke workloads.
2. **Operational consistency**: Same DNS and routing model for Storage and Key Vault.
3. **Security**: Easier to enforce NSG/UDR/inspection policies in one place.
4. **Scalability**: New AVD pools/spokes consume existing private link patterns.

## Baseline assumptions

- AVD session hosts run in spoke subnet(s).
- Hub and Spoke are peered.
- VNets use **custom DNS** (AD DS DNS servers inside Azure).
- DNS servers can reach Azure DNS virtual IP `168.63.129.16`.

## Prerequisites checklist

- [ ] Azure RBAC: Contributor (or higher) on target resource groups.
- [ ] Existing Hub and Spoke VNets with peering in place.
- [ ] Dedicated subnet in Hub for Private Endpoints (network policies disabled for PE).
- [ ] AD DS DNS server VMs deployed in Azure (at least two recommended).
- [ ] Private DNS zones available or approved for creation:
  - `privatelink.file.core.windows.net`
  - `privatelink.vaultcore.azure.net`
- [ ] Naming convention finalized for RGs/resources.
- [ ] Redaction policy agreed for screenshots.

## Portal checkpoints to capture (screenshots)

- [SCREENSHOT: 01-A - Hub VNet overview + subnets]
- [SCREENSHOT: 01-B - Spoke VNet overview + AVD subnet]
- [SCREENSHOT: 01-C - VNet peering status (Hub <-> Spoke)]
- [SCREENSHOT: 01-D - Custom DNS servers on both VNets]

## Implementation notes

- Keep Private Endpoint subnet NSG rules explicit; deny-by-default with required allow rules as needed.
- Avoid mixing broad service endpoints and private endpoints for the same access path unless intentionally designed.
- Keep zone linking and DNS forwarding simple first, then add split-brain complexity only if required.

## Next step

Proceed to [02 - Storage Files Premium + Private Endpoint](02-storage-files-premium-and-private-endpoint.md).
