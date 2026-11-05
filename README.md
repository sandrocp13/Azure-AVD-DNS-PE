# Azure AVD Private Endpoint + Custom DNS Baseline

Professional, portal-first implementation guide for deploying **Azure Files Premium** and **Key Vault** with **Private Endpoints** in an Azure Virtual Desktop (AVD) environment, using a **Hub/Spoke** network design and **Active Directory DNS forwarders**.

This repository is designed to be reusable across Azure environments, while grounded in a real AVD baseline pattern.

## What this guide covers

- End-to-end prerequisites and architecture decisions.
- Storage Account (**FileStorage + Premium**) setup for AVD FSLogix profiles.
- Private Endpoint placement in the **Hub** network.
- Private DNS Zone creation and linkage.
- AD DS DNS forwarder configuration (custom DNS scenario).
- Key Vault private access pattern aligned with the same DNS and networking model.
- Validation and troubleshooting.

## Reference baseline (sanitized)

> Sensitive identifiers should stay redacted in documentation and screenshots.

| Component | Baseline Pattern |
|---|---|
| Region | East US 2 |
| Network model | Hub/Spoke |
| Hub VNet CIDR | 10.0.0.0/16 |
| Hub PE subnet | `Private_EndPoint` (`10.0.100.0/24`) |
| Spoke VNet CIDR | 10.1.0.0/16 |
| Domain Controller subnet | `10.1.1.0/24` |
| AVD session host subnet | `10.1.2.0/24` |
| Custom DNS servers used by VNets | `10.1.1.4`, `10.1.1.5` |
| Private DNS zones | `privatelink.file.core.windows.net`, `privatelink.vaultcore.azure.net` |

## Repository structure

```text
.
├─ README.md
├─ diagrams/
│  └─ avd-private-endpoint-dns-topology.mmd
├─ docs/
│  ├─ 01-architecture-and-prereqs.md
│  ├─ 02-storage-files-premium-and-private-endpoint.md
│  ├─ 03-key-vault-private-endpoint.md
│  ├─ 04-custom-dns-forwarders-on-ad-ds.md
│  ├─ 05-validation-and-troubleshooting.md
│  ├─ 06-screenshot-redaction-checklist.md
│  └─ 07-microsoft-reference-links.md
└─ assets/
   └─ screenshots/
      └─ .gitkeep
```

## Recommended execution order

1. Read [Architecture and prerequisites](docs/01-architecture-and-prereqs.md).
2. Deploy [Storage Files Premium + Private Endpoint](docs/02-storage-files-premium-and-private-endpoint.md).
3. Apply [Custom DNS forwarders on AD DS](docs/04-custom-dns-forwarders-on-ad-ds.md).
4. Configure [Key Vault Private Endpoint](docs/03-key-vault-private-endpoint.md).
5. Run [Validation and troubleshooting](docs/05-validation-and-troubleshooting.md).
6. Complete [Screenshot redaction QA](docs/06-screenshot-redaction-checklist.md).

## Screenshot placeholders

Each deep-dive page includes screenshot markers in this format:

`[SCREENSHOT: <ID> - <what to capture>]`

Store images in `assets/screenshots/` and keep naming consistent (example: `01-hub-vnet-overview.png`).

## Why this Hub-based PE model is preferred for AVD

- Centralizes private ingress/egress control and policy.
- Scales across multiple AVD pools/spokes without duplicating endpoint strategy.
- Keeps DNS resolution predictable with one custom DNS authority plane (AD DS DNS).
- Simplifies operations by standardizing private link patterns for both Storage and Key Vault.

---

For source references, see [Microsoft reference links](docs/07-microsoft-reference-links.md).
