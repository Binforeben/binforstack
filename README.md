# binforstack.com — Hyper-V Active Directory Lab

A self-hosted Active Directory Domain Services lab built on Windows Server 2025, running under Hyper-V on Windows. This project documents the setup, configuration, and troubleshooting process end-to-end as a demonstration of core Windows Server and infrastructure fundamentals.

## Overview

| Component | Detail |
|---|---|
| Hypervisor | Microsoft Hyper-V |
| Guest OS | Windows Server 2025 (Desktop Experience) |
| Domain Name | binforstack.com |
| NetBIOS Name | BINFORSTACK |
| VM Generation | Generation 2 (UEFI) |
| Virtual Switch | Hyper-V Default Switch (NAT) |
| Roles Installed | AD DS, DNS |

## Architecture

- Single domain controller (`DC01`) hosting a new Active Directory forest at the root domain `binforstack.com`
- DNS Server role installed alongside AD DS, with the DC configured to resolve against itself
- Static IP addressing on the Hyper-V Default Switch NAT network, ensuring a stable address for domain-dependent services

## Build Process

### 1. Hyper-V Setup
- Enabled the Hyper-V role via PowerShell (`Enable-WindowsOptionalFeature`)
- Verified host meets virtualization requirements (VT-x/AMD-V, Pro/Enterprise edition)

![Hyper-V Manager showing DC01 VM](images/01-hyperv-manager.png)
*Hyper-V Manager showing the DC01 virtual machine*

### 2. Virtual Networking
- Initially configured an External Virtual Switch bound to a physical NIC
- Switched to Hyper-V's built-in **Default Switch** after connectivity issues on the external switch — Default Switch provides NAT-based internet access with less host-side configuration overhead, well suited for an isolated lab environment

![Virtual Switch Manager](images/02-vswitch-manager.png)
*Virtual Switch Manager configuration*

### 3. VM Provisioning
- Created a Generation 2 VM (60GB dynamically expanding disk, 4GB RAM)
- Attached the Windows Server 2025 ISO and completed a clean install with Desktop Experience

![VM Settings](images/03-vm-settings.png)
*VM settings — Generation 2, network adapter configuration*

### 4. Networking Configuration
- Identified the correct subnet, gateway, and mask assigned by Default Switch's DHCP before converting to a static IP (avoided hardcoding a mismatched external-network IP scheme)
- Renamed the server to `DC01`
- Locked in a static IP matching the DHCP-leased address/subnet/gateway to prevent IP drift

![Server Manager Local Server view](images/04-local-server.png)
*Server Manager, Local Server — computer name and network config*

![ipconfig /all output](images/05-ipconfig.png)
*Static IP, subnet mask, gateway, and DNS confirmed*

### 5. Active Directory Domain Services
- Installed the AD DS server role
- Promoted the server to a domain controller, creating a new forest and root domain: `binforstack.com`
- DNS Server role was installed automatically as part of promotion

### 6. Verification
- After promotion, updated the DC's preferred DNS server to `127.0.0.1`, pointing it at itself now that the DNS role was active
- Validated the domain, core services, and DNS self-resolution using PowerShell diagnostics

![Get-ADDomain output](images/06-get-addomain.png)
*Confirming the domain is live*

![dcdiag output](images/07-dcdiag.png)
*Domain controller diagnostics passing*

![Get-WindowsFeature and nslookup output](images/08-role-dns-verification.png)
*AD DS/DNS roles confirmed installed, and DNS self-resolution verified for binforstack.com*

## Verification Steps

```powershell
# Confirm the domain is live
Get-ADDomain

# Confirm core services are running
Get-Service -Name NTDS, DNS, Netlogon

# Confirm roles are installed
Get-WindowsFeature -Name AD-Domain-Services, DNS

# Full health check
dcdiag

# Confirm DNS self-resolution
nslookup binforstack.com
```

## Troubleshooting Notes

A real issue encountered and resolved during the build: after switching from an External Switch to the Default Switch, a previously configured static IP (set for the external network's subnet) was still applied — causing complete loss of connectivity, since that address didn't exist on the new NAT subnet. Resolved by reverting to DHCP, identifying the correct Default Switch subnet, and reapplying static configuration that matched.

This reflects a common real-world scenario: network configuration must match the actual network the machine is attached to, not the one it was previously on — especially relevant when switching hypervisor networking modes.

## Next Steps

- [ ] Build OU structure (Departments, Users, Computers, Groups)
- [ ] Create test user accounts and security groups
- [ ] Join a client VM to the domain
- [ ] Configure Group Policy Objects
- [ ] Document DHCP scope configuration
- [ ] Add network diagram and screenshots

## Skills Demonstrated

- Hyper-V installation and virtual networking (External vs. Default Switch, NAT)
- Windows Server 2025 deployment (Generation 2 VM, UEFI)
- Active Directory Domain Services: forest/domain creation, promotion
- DNS Server role configuration and self-resolution
- Static IP configuration and subnet troubleshooting
- PowerShell-based administration and diagnostics (`Get-ADDomain`, `dcdiag`, `Get-Service`)
