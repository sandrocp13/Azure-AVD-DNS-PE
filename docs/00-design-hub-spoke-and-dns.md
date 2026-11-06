# 00 - Design: Hub-Spoke with Private Endpoints and Custom DNS

## Design goal

Provide a single architecture view that explains:

1. Hub VNet with a dedicated Private Endpoint subnet.
![00-A - Hub VNet and PE subnet](./00-A-Hub-VNet-and-PE-subnet.png)
2. Spoke VNet with AVD subnet and Domain Controller subnet.
![00-B - Spoke VNet and AVD subnet](./00-B-Spoke-VNet-and-AVD-subnet.png)
3. VNet DNS settings pointing to AD DS DNS servers.
![00-C - VNet DNS settings using AD DS DNS](./00-C-VNet-DNS-settings-using-AD-DS-DNS.png)
4. Conditional forwarders on AD DS DNS to Azure DNS (`168.63.129.16`).
![00-D - Conditional forwarders on AD DS DNS to Azure DNS - Azure Files](./03-A-Conditional-forwarder-for-file-zone.png)
![00-D - Conditional forwarders on AD DS DNS to Azure DNS - Azure Key Vault](./03-A-Conditional-forwarder-for-key-vault.png)
5. Private DNS zones for Storage and Key Vault.
![00-E - Private DNS zones for Storage and Key Vault](./00-E-Private-DNS-zones-for-Storage-and-Key-Vault.png)

## Architecture diagram

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

## Design decisions summary

- Private Endpoints are centralized in Hub for consistent policy and operations.
- AVD workloads stay in Spoke with segmented subnets.
- DNS remains authoritative in AD DS; private-link names are conditionally forwarded to Azure DNS.
