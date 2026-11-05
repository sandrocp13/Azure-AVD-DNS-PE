# 05 - Validation and troubleshooting

## Objective

Prove that AVD dependencies (Storage and Key Vault) are reachable only through private networking and DNS behaves as expected.

## Validation matrix

| Test | Source | Expected result |
|---|---|---|
| DNS resolution for storage | AVD session host | Private IP returned |
| DNS resolution for key vault | AVD session host | Private IP returned |
| SMB connectivity | AVD session host to storage endpoint:445 | Reachable |
| Key Vault API reachability | Admin host in allowed network | Reachable |
| Public path check | Non-authorized endpoint | Blocked/fails |

## DNS tests

```powershell
nslookup <storageaccount>.file.core.windows.net
nslookup <keyvault>.vault.azure.net
Resolve-DnsName <storageaccount>.file.core.windows.net
Resolve-DnsName <keyvault>.vault.azure.net
```

[SCREENSHOT: 05-A - DNS resolution success for Storage]  
[SCREENSHOT: 05-B - DNS resolution success for Key Vault]

## Network tests

```powershell
Test-NetConnection <storageaccount>.file.core.windows.net -Port 445
Test-NetConnection <keyvault>.vault.azure.net -Port 443
```

[SCREENSHOT: 05-C - SMB 445 connectivity success]  
[SCREENSHOT: 05-D - HTTPS 443 connectivity success]

## Azure-side checks

1. Private Endpoint connection state = Approved.
2. Private DNS zone has expected `A` records.
3. VNet links exist and are healthy.
4. Storage/Key Vault networking settings reflect private-only posture.

[SCREENSHOT: 05-E - PE connection state approved]  
[SCREENSHOT: 05-F - Private DNS zone record set details]

## Troubleshooting quick table

| Symptom | Likely cause | Fix |
|---|---|---|
| Resolves to public IP | Missing/incorrect conditional forwarder or DNS zone link | Fix forwarders and zone links |
| Name does not resolve | Missing `A` record or broken DNS forwarding path | Recreate PE DNS integration or record |
| DNS resolves private but connection fails | NSG/UDR/firewall block | Validate effective routes and security rules |
| Intermittent resolution | Multiple DNS paths with inconsistent config | Standardize DNS servers and flush caches |
| SMB denied but network is open | Identity/permissions issue | Review RBAC/NTFS/FSLogix permissions |

## Useful cache commands

```powershell
ipconfig /flushdns
Clear-DnsServerCache -Force
```
