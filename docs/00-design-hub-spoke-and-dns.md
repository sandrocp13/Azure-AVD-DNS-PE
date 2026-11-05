# 00 - Design: Hub-Spoke with Private Endpoints and Custom DNS

## Design goal

Provide a single architecture view that explains:

1. Hub VNet with dedicated Private Endpoint subnet.
2. Spoke VNet with AVD subnet and Domain Controller subnet.
3. VNet DNS settings pointing to AD DS DNS servers.
4. Conditional forwarders on AD DS DNS to Azure DNS (`168.63.129.16`).
5. Private DNS zones for Storage and Key Vault.

## Architecture diagram (Mermaid)

```mermaid
flowchart LR
    subgraph HUB["Hub VNet (10.0.0.0/16)"]
        subgraph PESN["Private_EndPoint Subnet (10.0.100.0/24)"]
            SAPE["PE - Storage (file)"]
            KVPE["PE - Key Vault (vault)"]
        end
        PDNSF["Private DNS Zone\nprivatelink.file.core.windows.net"]
        PDNSK["Private DNS Zone\nprivatelink.vaultcore.azure.net"]
    end

    subgraph SPOKE["Spoke VNet (10.1.0.0/16)"]
        subgraph DCNET["Domain-Controller Subnet (10.1.1.0/24)"]
            ADDS1["AD DS DNS 1\n10.1.1.4"]
            ADDS2["AD DS DNS 2\n10.1.1.5"]
        end
        subgraph AVDNET["AVD Subnet (10.1.2.0/24)"]
            AVDH["AVD Session Host"]
        end
    end

    AZDNS["Azure DNS\n168.63.129.16"]
    SA["Azure Files Premium\n(FileStorage)"]
    KV["Azure Key Vault"]

    AVDH -->|"DNS query"| ADDS1
    AVDH -->|"DNS query"| ADDS2
    ADDS1 -->|"Conditional forwarder\nprivatelink.file.core.windows.net"| AZDNS
    ADDS1 -->|"Conditional forwarder\nprivatelink.vaultcore.azure.net"| AZDNS
    ADDS2 -->|"Same forwarders"| AZDNS

    AZDNS --> PDNSF
    AZDNS --> PDNSK
    PDNSF --> SAPE
    PDNSK --> KVPE

    AVDH -->|"Private IP path via peering/routing"| SAPE
    AVDH -->|"Private IP path via peering/routing"| KVPE
    SAPE --> SA
    KVPE --> KV
```

## Screenshot mapping for Step 0

- [SCREENSHOT: 00-A - Hub VNet and PE subnet]
  - Capture the Hub VNet overview and show `Private_EndPoint` subnet.
- [SCREENSHOT: 00-B - Spoke VNet and AVD subnet]
  - Capture the Spoke VNet overview and show `Domain-Controller` + `AVD` subnets.
- [SCREENSHOT: 00-C - VNet DNS settings using AD DS DNS]
  - Capture DNS server settings showing AD DS DNS IPs (`10.1.1.4`, `10.1.1.5`) on Hub/Spoke VNets.

## Design decisions summary

- Private Endpoints are centralized in Hub for consistent policy and operations.
- AVD workloads stay in Spoke with segmented subnets.
- DNS remains authoritative in AD DS; private-link names are conditionally forwarded to Azure DNS.
